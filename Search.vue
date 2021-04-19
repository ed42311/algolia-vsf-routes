<template>
  <ais-instant-search-ssr>
    <ais-search-box />
    <client-only>
      <ais-powered-by />
    </client-only>

    <ais-hits>
      <div slot="item" slot-scope="{ item }">
        <!-- show name -->
        <h2><ais-highlight attribute="name" :hit="item"/></h2>
        <!-- show content -->
        <div>{{ item.content }}</div>
        <!-- show content with highlight -->
        <div><ais-highlight attribute="content" :hit="item"/></div>
        <!-- show content with snippet: need to setup Snipetting in Indices -->
        <div><ais-snippet attribute="content" :hit="item"/></div>
      </div>
    </ais-hits>

    <ais-state-results>
      <template slot-scope="{ state: { query }, results: { hits, nbPages } }">
        <div v-if="query && hits.length == 0">No results</div>
        <div v-else></div>

        <ais-pagination v-if="nbPages > 0"/>
      </template>
    </ais-state-results>
  </ais-instant-search-ssr>
</template>

<script>
import {
  SfSidebar,
  SfButton,
  SfList,
  SfIcon,
  SfHeading,
  SfMenuItem,
  SfProductCard,
  SfProductCardHorizontal,
  SfPagination,
  SfAccordion,
  SfSelect,
  SfBreadcrumbs,
  SfLoader,
  SfNotification,
} from '@storefront-ui/vue';
import {
  AisInstantSearchSsr,
  AisStateResults,
  AisIndex,
  AisConfigure,
  AisRefinementList,
  AisHits,
  AisHighlight,
  AisSearchBox,
  AisStats,
  AisPagination,
  AisSortBy,
  createServerRootMixin
} from "vue-instantsearch"; // eslint-disable-line import/no-unresolved
import AppLoadingIndicator from "~/components/Search/LoadingIndicator"

// Libraries
import { useUiState } from '~/composables';
import algoliasearch from "algoliasearch/lite";
import "instantsearch.css/themes/algolia-min.css";

const indexName = 'instant_search'

const algoliaClient = algoliasearch(
  "latency",
  "6be0576ff61c053d5f9a3225e2a90f76"
);

// to disable listing on load
const searchClient = {
  search(requests) {
    if (requests.every(({ params }) => !params.query)) {
     return Promise.resolve({
        results: requests.map(() => ({
          hits: [],
          nbHits: 0,
          nbPages: 0,
          processingTimeMS: 0,
        })),
      });
    }

    return algoliaClient.search(requests);
  },
};

// remove indexName
function writeState(routeState) {
  if (indexName in routeState)
    routeState = routeState[indexName]

  return routeState
}

// restore indexName
function readState(routeState) {
  routeState = {
    [indexName]: routeState
  }
  return routeState
}

function nuxtRouter(vueRouter) {
  let writeTimer = null;
  let removeAfterEachListener = null;
  let popStateListener = null;

  return {
    read() {
      return readState(vueRouter.currentRoute.query);;
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
            query: writeState(routeState),
          }),
        400
      );
    },
    createURL(routeState) {
      const url = vueRouter.resolve({
        query: writeState(routeState),
      }).href;
      return url
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
      indexName,
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
  setup() {
    const uiState = useUiState();
    return {
      ...uiState
    }
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
    AisSortBy,
    AisPagination,
    AisStateResults,
    SfButton,
    SfSidebar,
    SfIcon,
    SfList,
    SfProductCard,
    SfProductCardHorizontal,
    SfPagination,
    SfMenuItem,
    SfAccordion,
    SfSelect,
    SfBreadcrumbs,
    SfLoader,
    SfHeading,
    SfNotification,
    AppLoadingIndicator
  }
};
</script>

<style lang="scss">
@import '~@storefront-ui/vue/styles';
#search {
  box-sizing: border-box;
  @include for-desktop {
    max-width: 1240px;
    margin: 0 auto;
  }
}
.main {
  &.section {
    padding: var(--spacer-xs);
    @include for-desktop {
      padding: 0;
    }
  }
}
.breadcrumbs {
  padding: var(--spacer-base) 0;
}
.navbar {
  position: relative;
  display: flex;
  border: 1px solid var(--c-light);
  border-width: 0 0 1px 0;
  @include for-desktop {
    border-width: 1px 0 1px 0;
  }
  &.section {
    padding: var(--spacer-sm);
    @include for-desktop {
      padding: 0;
    }
  }
  &__aside,
  &__main {
    display: flex;
    align-items: center;
    padding: var(--spacer-sm) 0;
    flex: 1;
    padding: 0;
    @include for-desktop {
      padding: var(--spacer-xs) var(--spacer-xl);
    }
  }
  &__aside {
    flex: 0 0 20%;
    padding: var(--spacer-sm) var(--spacer-sm);
    border: 1px solid var(--c-light);
    border-width: 0 1px 0 0;
  }
  &__title {
    --heading-title-font-weight: var(--font-weight--light);
    --heading-title-font-size: var(--font-size--xl);
  }
  &__filters-icon {
    margin: 0 var(--spacer-sm) 0 0;
  }
  &__filters-button {
    --button-color: var(--c-link);
    --button-font-size: var(--font-size--base);
    --button-font-weight: var(--font-weight--normal);
    --button-text-decoration: none;
    display: flex;
    align-items: center;
    @include for-mobile {
      order: 1;
    }
    svg {
      fill: var(--c-text-muted);
      transition: fill 150ms ease;
    }
    &:hover {
      svg {
        fill: var(--c-primary);
      }
    }
  }
  &__label {
    font-family: var(--font-family--secondary);
    font-weight: var(--font-weight--normal);
    color: var(--c-link);
    margin: 0 var(--spacer-xs) 0 0;
  }
  &__select {
    --select-option-font-size: var(--font-size--base);
    --select-padding: var(--spacer-sm) 0;
    ::v-deep .sf-select__dropdown {
      font-size: var(--font-size--base);
      margin: var(--spacer-2xs) 0 0 0;
    }
  }
  &__sort {
    display: flex;
    align-items: center;
    margin: 0 auto 0 var(--spacer-2xl);
  }
  &__counter {
    font-family: var(--font-family--secondary);
    margin: auto;
    @include for-desktop {
      margin: auto 0 auto auto;
    }
  }
  &__view {
    display: flex;
    align-items: center;
    @include for-desktop {
      margin: 0 0 0 var(--spacer-2xl);
    }
    @include for-mobile {
      order: -1;
    }
    &-icon {
      cursor: pointer;
      margin: 0 var(--spacer-base) 0 0;
      &:last-child {
        margin: 0;
      }
    }
    &-label {
      margin: 0 var(--spacer-sm) 0 0;
      font: var(--font-weight--normal) var(--font-size--base) / 1.6
        var(--font-family--secondary);
      color: var(--c-link);
    }
  }
}
.sort-by {
  --select-dropdown-z-index: 1;
  flex: unset;
  width: 11.875rem;
}
.main {
  display: flex;
}
.sidebar {
  flex: 0 0 20%;
  padding: var(--spacer-sm);
  border: 1px solid var(--c-light);
  border-width: 0 1px 0 0;
}
.sidebar-filters {
  --sidebar-title-display: none;
  --sidebar-top-padding: 0;
  @include for-desktop {
    --sidebar-content-padding: 0 var(--spacer-xl);
    --sidebar-bottom-padding: 0 var(--spacer-xl);
  }
}
.list {
  --menu-item-font-size: var(--font-sm);
  &__item {
    &:not(:last-of-type) {
      --list-item-margin: 0 0 var(--spacer-sm) 0;
    }
  }
}
.products {
  box-sizing: border-box;
  flex: 1;
  margin: 0;
  &__grid,
  &__list {
    display: flex;
    flex-wrap: wrap;
  }
  &__grid {
    justify-content: space-between;
  }
  &__grid,
  &__list {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    @include for-desktop {
      justify-content: flex-start;
    }
  }
  &__product-card {
    --product-card-add-button-transform: translate3d(0, 30%, 0);
    flex: 1 1 50%;
    @include for-desktop {
      --product-card-padding: var(--spacer-sm);
      flex: 1 1 25%;
    }
  }
  &__product-card-horizontal {
    flex: 0 0 100%;
  }
  &__slide-enter {
    opacity: 0;
    transform: scale(0.5);
  }
  &__slide-enter-active {
    transition: all 0.2s ease;
    transition-delay: calc(0.1s * var(--index));
  }
  &__pagination {
    display: flex;
    justify-content: center;
    margin: var(--spacer-xl) 0 0 0;
  }
  @include for-desktop {
    margin: var(--spacer-sm) 0 0 var(--spacer-sm);
    &__pagination {
      &__options {
        display: flex;
        align-items: baseline;
        justify-content: flex-end;
      }
      &__label {
        font-family: var(--font-family--secondary);
        font-size: var(--font-size--sm);
        font-weight: var(--font-weight--normal);
        color: var(--c-link);
        margin-right: var(--spacer-2xs);
      }
    }
    &__product-card-horizontal {
      margin: var(--spacer-lg) 0;
    }
    &__product-card {
      flex: 1 1 25%;
    }
    &__list {
      margin: 0 0 0 var(--spacer-sm);
    }
  }
}
.filters {
  &__accordion-item {
    --accordion-item-content-padding: 0;
    position: relative;
    left: 50%;
    right: 50%;
    margin-left: -50vw;
    margin-right: -50vw;
    width: 100vw;
  }
}

.ais-Hit-img {
  width: 100%;
}

.ais-Hits-list:empty {
  margin: 0;
}

.ais-InstantSearch {
  margin: 1em;
}
</style>