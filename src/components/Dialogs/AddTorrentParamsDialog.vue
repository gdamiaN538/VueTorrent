<script lang="ts" setup>
import HistoryField from '@/components/Core/HistoryField.vue'
import { AppPreferences } from '@/constants/qbit'
import { HistoryKey } from '@/constants/vuetorrent'
import { useMaindataStore, usePreferenceStore } from '@/stores'
import { AddTorrentParams } from '@/types/qbit/models'
import { computed, ref } from 'vue'
import { useI18n } from 'vue-i18n'

const form = defineModel<AddTorrentParams>({ required: true })
const isOpened = defineModel<boolean>('isOpened')

const { t } = useI18n()
const maindataStore = useMaindataStore()
const preferenceStore = usePreferenceStore()

const contentLayoutOptions = [
  { title: t('common.useGlobalSettings'), value: null },
  { title: t('constants.contentLayout.original'), value: AppPreferences.ContentLayout.ORIGINAL },
  { title: t('constants.contentLayout.subfolder'), value: AppPreferences.ContentLayout.SUBFOLDER },
  { title: t('constants.contentLayout.nosubfolder'), value: AppPreferences.ContentLayout.NO_SUBFOLDER }
]
const stopConditionOptions = [
  { title: t('common.useGlobalSettings'), value: null },
  { title: t('constants.stopCondition.none'), value: AppPreferences.StopCondition.NONE },
  { title: t('constants.stopCondition.metadataReceived'), value: AppPreferences.StopCondition.METADATA_RECEIVED },
  { title: t('constants.stopCondition.filesChecked'), value: AppPreferences.StopCondition.FILES_CHECKED }
]

const downloadPathField = ref<typeof HistoryField>()
const savePathField = ref<typeof HistoryField>()

const tagSearch = ref('')

const categorySearch = ref('')
const categoryNames = computed(() => maindataStore.categories.map(c => c.name))
const category = computed<string | undefined>({
  get: () => form.value.category || categorySearch.value || undefined,
  set: value => (form.value.category = value || undefined)
})

const downloadPath = computed<string | undefined>({
  get: () => form.value.download_path || undefined,
  set: value => {
    form.value.use_download_path = !!value || false
    form.value.download_path = value || undefined
  }
})

function getLimit(value?: number) {
  return !value || value === -1 ? '' : (value / 1024).toString()
}

const downloadLimit = computed({
  get: () => getLimit(form.value.download_limit),
  set: value => {
    if (!value) {
      form.value.download_limit = undefined
    } else {
      const parsedValue = parseInt(value)
      if (parsedValue > 0) {
        form.value.download_limit = parsedValue * 1024
      }
    }
  }
})

const uploadLimit = computed({
  get: () => getLimit(form.value.upload_limit),
  set: value => {
    if (!value) {
      form.value.upload_limit = undefined
    } else {
      const parsedValue = parseInt(value)
      if (parsedValue > 0) {
        form.value.upload_limit = parsedValue * 1024
      }
    }
  }
})

const ratioLimit = computed({
  get: () => form.value.ratio_limit,
  set: val => (form.value.ratio_limit = val || undefined)
})

const seedingTimeLimit = computed({
  get: () => form.value.seeding_time_limit,
  set: val => (form.value.seeding_time_limit = val || undefined)
})

const inactiveSeedingTimeLimit = computed({
  get: () => form.value.inactive_seeding_time_limit,
  set: val => (form.value.inactive_seeding_time_limit = val || undefined)
})

function close() {
  downloadPathField.value?.saveValueToHistory()
  savePathField.value?.saveValueToHistory()

  isOpened.value = false
}

const onCategoryChanged = () => {
  form.value.save_path = maindataStore.getCategoryFromName(form.value.category)?.savePath ?? preferenceStore.preferences!.save_path
}
</script>

<template>
  <v-dialog v-model="isOpened" :class="$vuetify.display.mobile ? '' : 'w-75'" :fullscreen="$vuetify.display.mobile" scrollable>
    <v-card>
      <v-card-title class="ios-margin">
        <v-toolbar color="transparent">
          <v-toolbar-title>{{ t('dialogs.add.params.title') }}</v-toolbar-title>
          <v-btn icon="mdi-close" @click="close" />
        </v-toolbar>
      </v-card-title>

      <v-card-text>
        <v-row>
          <v-col cols="12" md="6">
            <v-combobox
              v-model="form.tags"
              v-model:search="tagSearch"
              :hide-no-data="false"
              :items="maindataStore.tags"
              :label="t('dialogs.add.params.tags')"
              chips
              clearable
              hide-details
              multiple
              autocomplete="tags">
              <template v-slot:prepend>
                <v-icon color="accent">mdi-tag</v-icon>
              </template>
              <template v-slot:no-data>
                <v-list-item>
                  <v-list-item-title v-if="tagSearch?.length > 0">
                    {{ t('dialogs.add.params.noTagMatch', { query: tagSearch }) }}
                  </v-list-item-title>
                  <v-list-item-title v-else>
                    {{ t('dialogs.add.params.noTags') }}
                  </v-list-item-title>
                </v-list-item>
              </template>
            </v-combobox>
          </v-col>

          <v-col cols="12" md="6">
            <v-combobox
              v-model="category"
              v-model:search="categorySearch"
              :hide-no-data="false"
              :items="categoryNames"
              :label="$t('dialogs.add.params.category')"
              clearable
              hide-details
              autocomplete="categories"
              @update:modelValue="onCategoryChanged">
              <template v-slot:prepend>
                <v-icon color="accent">mdi-label</v-icon>
              </template>
              <template v-slot:no-data>
                <v-list-item>
                  <v-list-item-title v-if="categorySearch?.length > 0">
                    {{ t('dialogs.add.params.noCategoryMatch', { query: categorySearch }) }}
                  </v-list-item-title>
                  <v-list-item-title v-else>
                    {{ t('dialogs.add.params.noCategories') }}
                  </v-list-item-title>
                </v-list-item>
              </template>
            </v-combobox>
          </v-col>

          <v-col cols="12">
            <HistoryField
              v-model="downloadPath"
              :history-key="HistoryKey.TORRENT_PATH"
              ref="downloadPathField"
              :disabled="form.use_auto_tmm"
              :label="t('dialogs.add.params.downloadPath')"
              hide-details>
              <template v-slot:prepend>
                <v-icon color="accent">mdi-tray-arrow-down</v-icon>
              </template>
            </HistoryField>
          </v-col>

          <v-col cols="12">
            <HistoryField
              v-model="form.save_path"
              :history-key="HistoryKey.TORRENT_PATH"
              ref="savePathField"
              :disabled="form.use_auto_tmm"
              :label="t('dialogs.add.params.savePath')"
              hide-details>
              <template v-slot:prepend>
                <v-icon color="accent">mdi-content-save</v-icon>
              </template>
            </HistoryField>
          </v-col>

          <v-col cols="12" md="6">
            <v-select
              v-model="form.content_layout"
              :items="contentLayoutOptions"
              :label="t('constants.contentLayout.title')"
              color="accent"
              hide-details
              rounded="xl"
              variant="solo-filled" />
          </v-col>

          <v-col cols="12" md="6">
            <v-select
              v-model="form.stop_condition"
              :items="stopConditionOptions"
              :label="t('constants.stopCondition.title')"
              color="accent"
              hide-details
              rounded="xl"
              variant="solo-filled" />
          </v-col>
        </v-row>

        <v-row class="mx-3">
          <v-col cols="12" md="6">
            <v-checkbox v-model="form.stopped" :label="t('dialogs.add.params.stopped')" color="accent" density="compact" hide-details />
          </v-col>
          <v-col cols="12" md="6">
            <v-checkbox v-model="form.add_to_top_of_queue" :label="t('dialogs.add.params.add_to_top_of_queue')" color="accent" density="compact" hide-details />
          </v-col>
          <v-col cols="12" md="6">
            <v-checkbox v-model="form.skip_checking" :label="t('dialogs.add.params.skip_checking')" color="accent" density="compact" hide-details />
          </v-col>
          <v-col cols="12" md="6">
            <v-checkbox v-model="form.use_auto_tmm" :label="t('dialogs.add.params.use_auto_tmm')" color="accent" density="compact" hide-details />
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12">
            <v-expansion-panels>
              <v-expansion-panel color="primary" :title="$t('dialogs.add.params.limit_collapse')">
                <v-expansion-panel-text>
                  <v-row>
                    <v-col cols="12" md="6">
                      <v-text-field v-model="downloadLimit" :label="$t('dialogs.add.params.download_limit')" hide-details suffix="KiB/s">
                        <template v-slot:prepend>
                          <v-icon color="accent">mdi-download</v-icon>
                        </template>
                      </v-text-field>
                    </v-col>
                    <v-col cols="12" md="6">
                      <v-text-field v-model="uploadLimit" :label="$t('dialogs.add.params.upload_limit')" hide-details suffix="KiB/s">
                        <template v-slot:prepend>
                          <v-icon color="accent">mdi-upload</v-icon>
                        </template>
                      </v-text-field>
                    </v-col>

                    <v-col cols="12" md="4">
                      <v-text-field v-model="ratioLimit" :hint="$t('dialogs.add.params.limit_hint')" :label="$t('dialogs.add.params.ratio_limit')" type="number" />
                    </v-col>
                    <v-col cols="12" md="4">
                      <v-text-field
                        v-model="seedingTimeLimit"
                        :label="$t('dialogs.add.params.seeding_time_limit')"
                        :hint="$t('dialogs.add.params.limit_hint')"
                        :suffix="$t('units.minutes')"
                        type="number" />
                    </v-col>
                    <v-col cols="12" md="4">
                      <v-text-field
                        v-model="inactiveSeedingTimeLimit"
                        :label="$t('dialogs.add.params.inactive_seeding_time_limit')"
                        :hint="$t('dialogs.add.params.limit_hint')"
                        :suffix="$t('units.minutes')"
                        type="number" />
                    </v-col>
                  </v-row>
                </v-expansion-panel-text>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
        </v-row>
      </v-card-text>

      <v-card-actions class="mb-2">
        <v-spacer />
        <v-btn :text="$t('common.close')" color="" variant="flat" @click="close" />
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
