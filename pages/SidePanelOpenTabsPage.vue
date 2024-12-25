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
      <FirstToolbarHelper2 title="Open Tabs">
        <template v-slot:iconsRight>
          <SidePanelToolbarTabNavigationHelper />

          <SidePanelToolbarButton
            icon="close"
            tooltip="Close this view"
            @click="useUiStore().sidePanelSetActiveView(SidePanelViews.MAIN)"
          />
        </template>
      </FirstToolbarHelper2>
    </q-page-sticky>
  </q-page>
</template>

<script lang="ts" setup>
import FirstToolbarHelper2 from 'pages/sidepanel/helper/FirstToolbarHelper2.vue'
import { SidePanelViews } from 'src/app/models/SidePanelViews'
import SidePanelToolbarButton from 'src/core/components/SidePanelToolbarButton.vue'
import Analytics from 'src/core/utils/google-analytics'
import SidePanelOpenTabsListViewer from 'src/opentabs/pages/SidePanelOpenTabsListViewer.vue'
import SidePanelToolbarTabNavigationHelper from 'src/opentabs/pages/SidePanelToolbarTabNavigationHelper.vue'
import { useUiStore } from 'src/ui/stores/uiStore'
import { onMounted } from 'vue'

onMounted(() => {
  Analytics.firePageViewEvent('SidePanelOpenTabsPage', document.location.href)
})
</script>
