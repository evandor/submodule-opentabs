<template>
  <!-- SidePanelOpenTabsPage -->
  <q-page padding style="padding-top: 34px">
    <div class="q-ma-none">
      <div class="q-ma-none">
        <div class="row q-ma-none q-pa-none">
          <div class="col-12 q-ma-none q-pa-none q-pt-md">
            <SidePanelOpenTabsListViewer />
          </div>
        </div>
      </div>
    </div>

    <!-- place QPageSticky at end of page -->
    <q-page-sticky expand position="top" class="darkInDarkMode brightInBrightMode">
      <ViewToolbarHelper title="Open Tabs" />
      <SearchToolbarHelper placeholder="Search in all tabsets..." />
    </q-page-sticky>
  </q-page>
</template>

<script lang="ts" setup>
import SearchToolbarHelper from 'pages/sidepanel/helper/SearchToolbarHelper.vue'
import Analytics from 'src/core/utils/google-analytics'
import SidePanelOpenTabsListViewer from 'src/opentabs/pages/SidePanelOpenTabsListViewer.vue'
import ViewToolbarHelper from 'src/pages/sidepanel/helper/ViewToolbarHelper.vue'
import { onMounted } from 'vue'

onMounted(() => {
  Analytics.firePageViewEvent('SidePanelOpenTabsPage', document.location.href)
})
</script>
