<template>
  <span class="cursor-pointer text-right" :style="props.showLabel ? 'min-width:160px' : 'max-width:30px'">
    <q-badge
      v-if="showThresholdBar() && props.showLabel"
      :multi-line="false"
      class="q-mr-sm"
      color="primary"
      text-color="white"
      :label="thresholdLabel()">
    </q-badge>

    <q-circular-progress
      v-if="showThresholdBar()"
      :show-value="props.showLabel"
      reverse
      :value="openTabsCountRatio2"
      :size="props.inSidePanel ? '16px' : '20px'"
      :thickness="0.5"
      :style="thresholdStyle()"
      track-color="grey-3"
      class="q-ml-xs">
    </q-circular-progress>
    <q-tooltip class="tooltip">Open Tabs: {{ useTabsStore2().browserTabs.length }} - click to manage</q-tooltip>
  </span>
  <q-menu :offset="[0, 15]">
    <q-list style="min-width: 200px">
      <q-item v-if="!props.inSidePanel" clickable v-close-popup @click="showOpenTabs">
        <q-item-section>Show open tabs</q-item-section>
      </q-item>
      <q-separator />
      <q-item disable> Close some tabs: </q-item>
      <q-item
        :disable="useTabsetsStore().tabsets?.size === 0 || trackedTabsCount === 0"
        clickable
        v-close-popup
        @click="TabsetService.closeTrackedTabs()">
        <q-item-section>&bull; Close all tracked tabs ({{ trackedTabsCount }})</q-item-section>
      </q-item>
      <!--      <q-item clickable v-close-popup @click="TabsetService.closeDuplictedOpenTabs()">-->
      <!--        <q-item-section>&bull; Close duplicated open tabs</q-item-section>-->
      <!--      </q-item>-->
      <q-item v-if="useFeaturesStore().hasFeature(FeatureIdent.BACKUP)" clickable v-close-popup @click="backupAndClose">
        <q-item-section>&bull; Backup and close current tabs...</q-item-section>
      </q-item>
      <q-item
        :disable="useTabsStore2().browserTabs.length <= 1"
        clickable
        v-close-popup
        @click="TabsetService.closeAllTabs()">
        <q-item-section>&bull; Close all other tabs ({{ useTabsStore2().browserTabs.length - 1 }})</q-item-section>
      </q-item>
      <q-separator />
      <q-item disable v-if="showSpecialTabsets()"> Use special tabsets: </q-item>
      <!--      <q-item v-if="useFeaturesStore().hasFeature(FeatureIdent.SESSIONS) && existingSession"-->
      <!--              clickable v-close-popup-->
      <!--              @click="replaceSession">-->
      <!--        <q-item-section>&bull; Replace existing Session...</q-item-section>-->
      <!--      </q-item>-->
      <!--      <q-item v-else-if="useFeaturesStore().hasFeature(FeatureIdent.SESSIONS) && !existingSession"-->
      <!--              clickable v-close-popup-->
      <!--              @click="startSession">-->
      <!--        <q-item-section>&bull; Start a new Session...</q-item-section>-->
      <!--      </q-item>-->
      <!--      <q-separator v-if="useFeaturesStore().hasFeature(FeatureIdent.SESSIONS) && !props.inSidePanel"/>-->
      <!--      <q-item v-if="useFeaturesStore().hasFeature(FeatureIdent.SESSIONS) && !props.inSidePanel"-->
      <!--              clickable v-close-popup @click="router.push('/settings')">-->
      <!--        <q-item-section>Change Settings</q-item-section>-->
      <!--      </q-item>-->
    </q-list>
  </q-menu>
</template>

<script lang="ts" setup>
import _ from 'lodash'
import { useQuasar } from 'quasar'
import { FeatureIdent } from 'src/app/models/FeatureIdent'
import { useUtils } from 'src/core/services/Utils'
import { useFeaturesStore } from 'src/features/stores/featuresStore'
import BackupAndCloseDialog from 'src/opentabs/dialogues/BackupAndCloseDialog.vue'
import { useSettingsStore } from 'src/stores/settingsStore'
import { Tabset, TabsetType } from 'src/tabsets/models/Tabset'
import TabsetService from 'src/tabsets/services/TabsetService'
import { useTabsetsStore } from 'src/tabsets/stores/tabsetsStore'
import { useTabsStore2 } from 'src/tabsets/stores/tabsStore2'
import { DrawerTabs, useUiStore } from 'src/ui/stores/uiStore'
import { ref, watch, watchEffect } from 'vue'

const settingsStore = useSettingsStore()
const $q = useQuasar()

const openTabsCountRatio = ref(0)
const openTabsCountRatio2 = ref(0)
const trackedTabsCount = ref(0)
const existingSession = ref(false)

const { inBexMode } = useUtils()

const props = defineProps({
  showLabel: { type: Boolean, default: true },
  inSidePanel: { type: Boolean, default: false },
})

if (!inBexMode()) {
  TabsetService.trackedTabsCount().then((res) => (trackedTabsCount.value = res))
}

watchEffect(() => {
  openTabsCountRatio.value = Math.min(
    useTabsStore2().browserTabs.length / settingsStore.thresholds['max' as keyof object],
    1,
  )
  //console.log("threshold", settingsStore.thresholds['max' as keyof object])
  openTabsCountRatio2.value = Math.round(
    100 * Math.min(useTabsStore2().browserTabs.length / settingsStore.thresholds['max' as keyof object], 1),
  )
})

watch(
  () => useTabsStore2().browserTabs.length,
  (after: number, before: number) => {
    if (inBexMode()) {
      TabsetService.trackedTabsCount().then((res) => (trackedTabsCount.value = res))
    }
  },
)

watch(
  () => useTabsetsStore().getCurrentTabs.length,
  (after: number, before: number) => {
    if (inBexMode()) {
      TabsetService.trackedTabsCount().then((res) => (trackedTabsCount.value = res))
    }
  },
)

// watch( useEventStore().events, (val: any) => {
//   console.log("event received", val)
//   TabsetService.trackedTabsCount().then((res) => trackedTabsCount.value = res)
// })

//watchEffect(() => TabsetService.trackedTabsCount().then((res) => trackedTabsCount.value = res))

const showThresholdBar = () => useTabsStore2().browserTabs.length >= settingsStore.thresholds['min' as keyof object]

const thresholdStyle = () => 'color: hsl(' + (120 - Math.round(120 * openTabsCountRatio.value)) + ' 80% 50%)'

const thresholdLabel = () => useTabsStore2().browserTabs.length + ' open tabs'

const showOpenTabs = () => useUiStore().rightDrawerSetActiveTab(DrawerTabs.UNASSIGNED_TABS)

watchEffect(() => {
  existingSession.value =
    _.filter([...useTabsetsStore().tabsets.values()], (ts: Tabset) => ts.type === TabsetType.SESSION).length > 0
})

// const startSession = () => $q.dialog({component: NewSessionDialog, componentProps: {replaceSession: false}})
// const replaceSession = () => $q.dialog({component: NewSessionDialog, componentProps: {replaceSession: true}})
const backupAndClose = () => $q.dialog({ component: BackupAndCloseDialog })

const showSpecialTabsets = () => useFeaturesStore().hasFeature(FeatureIdent.SESSIONS)
</script>
