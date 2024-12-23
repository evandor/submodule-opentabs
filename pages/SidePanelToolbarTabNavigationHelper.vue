<template>
  <template v-if="useFeaturesStore().hasFeature(FeatureIdent.OPEN_TABS)">
    <SidePanelToolbarButton
      icon="chevron_left"
      tooltip="Back to last tab"
      :disable="useTabsStore2().chromeTabsHistoryPosition === 0"
      :color="useTabsStore2().chromeTabsHistoryPosition === 0 ? 'grey' : 'primary'"
      @click.stop="NavigationService.backOneTab()"
    />

    <SidePanelToolbarButton
      class="q-mr-sm"
      icon="chevron_right"
      tooltip="Return to tab before"
      :disable="
        useTabsStore2().chromeTabsHistoryPosition === useTabsStore2().chromeTabsHistory.length - 1
      "
      :color="
        useTabsStore2().chromeTabsHistoryPosition === useTabsStore2().chromeTabsHistory.length - 1
          ? 'grey'
          : 'primary'
      "
      @click.stop="NavigationService.forwardOneTab()"
    />
  </template>
</template>
<script setup lang="ts">
import { FeatureIdent } from 'src/app/models/FeatureIdent'
import NavigationService from 'src/services/NavigationService'
import SidePanelToolbarButton from 'src/core/components/SidePanelToolbarButton.vue'
import { useFeaturesStore } from 'src/features/stores/featuresStore'
import { useTabsStore2 } from 'src/tabsets/stores/tabsStore2'
</script>
