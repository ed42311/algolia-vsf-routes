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
  return {
    read() {
      return vueRouter.currentRoute.query;
    },
    write(routeState) {
      if (this.createURL(routeState) === this.createURL(this.read())) {
        return;
      }
      vueRouter.push({
        query: routeState,
      });
    },
    createURL(routeState) {
      return vueRouter.resolve({
        query: routeState,
      }).href;
    },
    onUpdate(cb) {
      if (typeof window === "undefined") return;

      this._onPopState = (event) => {
        const routeState = event.state;
        if (!routeState) {
          cb(this.read());
        } else {
          cb(routeState);
        }
      };
      window.addEventListener("popstate", this._onPopState);
    },
    dispose() {
      if (typeof window === "undefined") return;

      window.removeEventListener("popstate", this._onPopState);
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
