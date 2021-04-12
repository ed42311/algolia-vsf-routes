<template>
  <ais-instant-search-ssr>
    <ais-search-box />
    <ais-stats />
    <ais-refinement-list attribute="brand" />
    <ais-hits>
      <template slot="item" slot-scope="{ item }">
        <p>
          <ais-highlight attribute="name" :hit="item" />
        </p>
        <p>
          <ais-highlight attribute="brand" :hit="item" />
        </p>
      </template>
    </ais-hits>
    <ais-pagination />
  </ais-instant-search-ssr>
</template>

<script>
import {
  AisInstantSearchSsr,
  AisIndex,
  AisConfigure,
  AisRefinementList,
  AisHits,
  AisHighlight,
  AisSearchBox,
  AisStats,
  AisPagination,
  createServerRootMixin
} from "vue-instantsearch"; // eslint-disable-line import/no-unresolved
import algoliasearch from "algoliasearch/lite";
import "instantsearch.css/themes/algolia-min.css";

const searchClient = algoliasearch(
  "latency",
  "6be0576ff61c053d5f9a3225e2a90f76"
);

function nuxtRouter(vueRouter) {
  let writeTimer = null;
  let removeAfterEachListener = null;
  let popStateListener = null;

  return {
    read() {
      return vueRouter.currentRoute.query;
    },
    write(routeState) {
      // Only push a new entry if the URL changed (avoid duplicated entries in the history)
      if (this.createURL(routeState) === this.createURL(this.read())) {
        return;
      }

      if (writeTimer) clearTimeout(writeTimer);

      writeTimer = setTimeout(
        () =>
          vueRouter.push({
            query: routeState,
          }),
        400
      );
    },
    createURL(routeState) {
      return vueRouter.resolve({
        query: routeState,
      }).href;
    },
    onUpdate(cb) {
      removeAfterEachListener = vueRouter.afterEach((to, from) => {
        cb(to.query);
      });

      if (typeof window === "undefined") return;

      popStateListener = () => {
        cb(this.read());
      };
      window.addEventListener("popstate", popStateListener);
    },
    dispose() {
      if (writeTimer) clearTimeout(writeTimer);

      if (removeAfterEachListener) removeAfterEachListener();

      if (typeof window === "undefined") return;

      window.removeEventListener("popstate", popStateListener);
    },
  };
}

export default {
  provide() {
    return {
      $_ais_ssrInstantSearchInstance: this.instantsearch,
    };
  },
  data() {
    const mixin = createServerRootMixin({
      searchClient,
      indexName: "instant_search",
      routing: {
        router: nuxtRouter(this.$router),
      },
    });
    return {
      ...mixin.data(),
    };
  },
  serverPrefetch() {
    return this.instantsearch.findResultsState(this).then(algoliaState => {
      this.$ssrContext.nuxt.algoliaState = algoliaState;
    });
  },
  beforeMount() {
    const results =
      (this.$nuxt.context && this.$nuxt.context.nuxtState.algoliaState) ||
      window.__NUXT__.algoliaState;

    this.instantsearch.hydrate(results);

    delete this.$nuxt.context.nuxtState.algoliaState;
    delete window.__NUXT__.algoliaState;
  },
  components: {
    AisInstantSearchSsr,
    AisIndex,
    AisConfigure,
    AisRefinementList,
    AisHits,
    AisHighlight,
    AisSearchBox,
    AisStats,
    AisPagination
  }
};
</script>

<style>
.ais-Hits-list {
  text-align: left;
}
.ais-Hits-list:empty {
  margin: 0;
}
.ais-InstantSearch {
  margin: 1em;
}
</style>
