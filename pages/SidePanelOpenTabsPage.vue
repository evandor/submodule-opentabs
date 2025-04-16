<template>
  <!-- SidePanelOpenTabsPage -->
  <q-page padding style="padding-top: 86px">
    <div class="q-ma-none">
      <div class="q-ma-none">
        <div class="row q-ma-none q-pa-none">
          <div class="col-12 q-ma-none q-pa-none">
            <SidePanelOpenTabsListViewer
              ref="sidePanelOpenTabsListViewerRef"
              :filterTerm="filterTerm"
              @filtered-tabs="(v) => (searchHits = v)"
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
        :search-hits="searchHits"
        :tabSelection="tabSelection" />
    </q-page-sticky>
  </q-page>
</template>

<script lang="ts" setup>
import Analytics from 'src/core/utils/google-analytics'
import SidePanelOpenTabsListViewer from 'src/opentabs/pages/SidePanelOpenTabsListViewer.vue'
import OpenTabsFilterToolbarHelper from 'src/pages/sidepanel/helper/OpenTabsFilterToolbarHelper.vue'
import ViewToolbarHelper from 'src/pages/sidepanel/helper/ViewToolbarHelper.vue'
import { onMounted, ref } from 'vue'

const filterTerm = ref<string | undefined>(undefined)
const tabSelection = ref<Set<string>>(new Set<string>())
const sidePanelOpenTabsListViewerRef = ref()
const searchHits = ref(0)

onMounted(() => {
  Analytics.firePageViewEvent('SidePanelOpenTabsPage', document.location.href)
})

const termChanged = (val: { term: string }) => (filterTerm.value = val.term)
const selectionInverted = () => sidePanelOpenTabsListViewerRef.value!.invertSelection()
const tabSelectionChanged = (val: Set<string>) => (tabSelection.value = val)
</script>
