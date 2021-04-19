<template>
  <div v-if="state && state.searchMetadata.isSearchStalled">
    <p>Loading…</p>
  </div>
</template>

<script>
import { createWidgetMixin } from 'vue-instantsearch';

const connectSearchMetaData = (renderFn, unmountFn) => (widgetParams = {}) => ({
  init() {
    renderFn({ searchMetadata: {} }, true);
  },

  render({ searchMetadata }) {
    renderFn({ searchMetadata }, false);
  },

  dispose() {
    unmountFn();
  },
});

export default {
  name: 'AisStateResults',
  mixins: [createWidgetMixin({ connector: connectSearchMetaData })],
};
</script>