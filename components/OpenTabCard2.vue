<template>
  <div class="row q-ma-none q-pa-none" @mouseenter="showIcons = true" @mouseleave="showIcons = false">
    <div class="col-2 q-mt-xs" style="text-align: center">
      <q-checkbox
        v-if="showSelectIcon(chromeTab)"
        v-model="chromeTab.selected"
        size="30px"
        checked-icon="task_alt"
        @update:model-value="(val) => selectionChanged(val)"
        unchecked-icon="check_box_outline_blank" />
      <TabFaviconWidget :preventDragAndDrop="true" :tab="toTab(chromeTab)" width="20px" height="20px" />
    </div>

    <div
      class="col-7 q-mt-sm q-ml-xs text-body2 ellipsis cursor-pointer"
      :class="TabService.isCurrentTab(toTab(chromeTab)) ? 'text-bold' : ''"
      @click="NavigationService.openChromeTab(chromeTab)">
      {{ chromeTab?.title }}
      <q-tooltip class="tooltip" v-if="useFeaturesStore().hasFeature(FeatureIdent.DEV_MODE)">
        {{ chromeTab.id }} / {{ chromeTab.url }} / {{ chromeTab.index }}
      </q-tooltip>
      <q-tooltip class="tooltip" v-else>
        {{ chromeTab.url }}
      </q-tooltip>
    </div>
    <div class="col-1 q-mt-xs text-right">
      <q-icon v-if="existsInTabset" name="link" :color="existsInCurrentTabset ? 'green' : 'warning'">
        <q-tooltip class="tooltip-small" v-if="existsInCurrentTabset"
          >Already contained in the current tabset</q-tooltip
        >
        <q-tooltip class="tooltip-small" v-else>Already contained in a tabset</q-tooltip>
      </q-icon>
    </div>
    <div class="col q-mt-xs text-right">
      <template v-if="!props.useSelection && showIcons">
        <q-icon
          name="o_add_circle"
          :color="alreadyInCurrentTabset ? 'grey' : 'warning'"
          class="q-mr-xs"
          :class="alreadyInCurrentTabset ? '' : 'cursor-pointer'"
          size="xs"
          @click="addToCurrentTabset">
          <q-tooltip class="tooltip" v-if="alreadyInCurrentTabset"
            >This tab has already been added to
            {{ useTabsetsStore().currentTabsetName }}
          </q-tooltip>
          <q-tooltip class="tooltip" v-else
            >Click here to add the tab to your current tabset
            {{ useTabsetsStore().currentTabsetName }}
          </q-tooltip>
        </q-icon>
        <q-icon name="close" class="cursor-pointer" @click="closeTab(chromeTab)" size="xs">
          <q-tooltip class="tooltip">Close this tab in the browser</q-tooltip>
        </q-icon>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { uid } from 'quasar'
import { FeatureIdent } from 'src/app/models/FeatureIdent'
import { useCommandExecutor } from 'src/core/services/CommandExecutor'
import { useFeaturesStore } from 'src/features/stores/featuresStore'
import NavigationService from 'src/services/NavigationService'
import TabService from 'src/services/TabService'
import { CreateTabFromOpenTabsCommand } from 'src/tabsets/commands/CreateTabFromOpenTabs'
import { Tab } from 'src/tabsets/models/Tab'
import { useTabsetService } from 'src/tabsets/services/TabsetService2'
import TabFaviconWidget from 'src/tabsets/widgets/TabFaviconWidget.vue'
import { onMounted, PropType, ref } from 'vue'
import { useTabsetsStore } from '../../tabsets/stores/tabsetsStore'

const props = defineProps({
  chromeTab: { type: Object as PropType<chrome.tabs.Tab>, required: true },
  windowId: { type: Number, required: true },
  useSelection: { type: Boolean, default: false },
})

const emits = defineEmits(['selectionChanged', 'addedToTabset', 'hasSelectable'])

const showIcons = ref(false)
const alreadyInCurrentTabset = ref(false)
const existsInTabset = ref(false)
const existsInCurrentTabset = ref(false)

onMounted(() => {
  if (props.chromeTab?.url) {
    existsInTabset.value = useTabsetService().urlExistsInATabset(props.chromeTab.url)
    existsInCurrentTabset.value = useTabsetService().urlExistsInCurrentTabset(props.chromeTab.url)
  }
})

const closeTab = (tab: chrome.tabs.Tab) => {
  NavigationService.closeChromeTab(tab)
}

const addToCurrentTabset = () => {
  useCommandExecutor()
    .executeFromUi(new CreateTabFromOpenTabsCommand(props.chromeTab as chrome.tabs.Tab, 0))
    .then(() => (alreadyInCurrentTabset.value = true))
    .then(() => emits('addedToTabset', { tabId: props.chromeTab.id, tabUrl: props.chromeTab.url }))
}

const selectionChanged = (val: any) => {
  emits('selectionChanged', { tabId: props.chromeTab.id, selected: val })
}

alreadyInCurrentTabset.value = useTabsetService().urlExistsInCurrentTabset(props.chromeTab.url || '')

const showSelectIcon = (tab: chrome.tabs.Tab) =>
  props.useSelection && !useTabsetService().urlExistsInCurrentTabset(tab.url || '')

const toTab = (chromeTab: chrome.tabs.Tab) => new Tab(uid(), chromeTab)
</script>

<style lang="sass" scoped>

.tabBorder
  border-radius: 5px 5px 0 0
  border: 1px solid $lightgrey
  border-bottom: 0
</style>
