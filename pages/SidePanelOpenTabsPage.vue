<template>
  <!-- SidePanelOpenTabsPage -->
  <q-page padding style="padding-top: 84px">
    <div class="q-ma-none">
      <div class="q-ma-none">
        <div class="row q-ma-none q-pa-none">
          <div class="col-12 q-ma-none q-pa-none q-pt-md">
            <SidePanelOpenTabsListViewer
              ref="sidePanelOpenTabsListViewer"
              :filterTerm="filterTerm"
              @tab-selection-changed="(v) => tabSelectionChanged(v)" />
          </div>
        </div>
      </div>
    </div>

    <q-page-sticky expand position="top" class="darkInDarkMode brightInBrightMode">
      <ViewToolbarHelper title="Open Tabs" />
      <OpenTabsFilterToolbarHelper
        @on-term-changed="(val) => termChanged(val)"
        @tab-selection-inverted="selectionInverted()"
        :tabSelection="tabSelection" />
    </q-page-sticky>
  </q-page>
</template>

<script lang="ts" setup>
import OpenTabsFilterToolbarHelper from 'pages/sidepanel/helper/OpenTabsFilterToolbarHelper.vue'
import Analytics from 'src/core/utils/google-analytics'
import SidePanelOpenTabsListViewer from 'src/opentabs/pages/SidePanelOpenTabsListViewer.vue'
import ViewToolbarHelper from 'src/pages/sidepanel/helper/ViewToolbarHelper.vue'
import { onMounted, ref } from 'vue'

const filterTerm = ref<string | undefined>(undefined)
const tabSelection = ref<Set<string>>(new Set<string>())
const sidePanelOpenTabsListViewer = ref()

onMounted(() => {
  Analytics.firePageViewEvent('SidePanelOpenTabsPage', document.location.href)
})

const termChanged = (val: { term: string }) => (filterTerm.value = val.term)
const selectionInverted = () => sidePanelOpenTabsListViewer.value!.invertSelection()
const tabSelectionChanged = (val: Set<string>) => (tabSelection.value = val)
</script>
