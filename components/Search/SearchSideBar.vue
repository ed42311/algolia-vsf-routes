<template>
  <AisHierarchicalMenu
    :class-names="{ 'ais-HierarchicalMenu': 'murdochs-hierarchical-menu' }"
    :attributes="[
      'categoryHierarchy.lvl0',
      'categoryHierarchy.lvl1',
      'categoryHierarchy.lvl2',
      'categoryHierarchy.lvl3',
    ]"
    :show-more-limit="999"
  >
    <div slot-scope="{ items, refine, createURL }">
      <SfSidebar
        :visible="isFilterSidebarOpen"
        title="Filters"
        @close="toggleFilterSidebar"
      >
        <HierarchicalMenuList
          :items="items"
          :refine="refine"
          :level="0"
          :createURL="createURL"
        />
      </SfSidebar>
    </div>
  </AisHierarchicalMenu>
</template>

<script>
import { SfSidebar } from '@storefront-ui/vue';
import { AisHierarchicalMenu } from 'vue-instantsearch';
import HierarchicalMenuList from './HierarchicalMenuList';

// Hooks
import { useUiState } from '~/composables';

export default {
  components: {
    AisHierarchicalMenu,
    SfSidebar,
    HierarchicalMenuList,
  },
  setup() {
    const uiState = useUiState();
    return {
      ...uiState,
    };
  },
};
</script>

<style lang="scss"></style>
