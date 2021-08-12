<template>
  <AisInstantSearchSsr>
    <ais-configure :hits-per-page.camel="100" />
    <AisSearchBox />
    <div class="navbar section">
      <div class="navbar__aside desktop-only">
        <SfHeading :level="3" title="Search" class="navbar__title" />
      </div>
      <div class="navbar__main">
        <SfButton
          data-cy="category-btn_filters"
          class="sf-button--text navbar__filters-button"
          aria-label="Filters"
          @click="toggleFilterSidebar"
        >
          <SfIcon
            size="18px"
            color="dark-secondary"
            icon="filter"
            class="navbar__filters-icon"
            data-cy="category-icon_"
          />
          Filters
        </SfButton>
        <div class="navbar__sort desktop-only">
          <span class="navbar__label">Sort by:</span>
          <AisSortBy
            :items="[
              { value: 'instant_search', label: 'Featured' },
              { value: 'instant_search_price_asc', label: 'Price asc.' },
              { value: 'instant_search_price_desc', label: 'Price desc.' }
            ]"
          />
        </div>
        <div data-cy="algolia_hits-counter" class="navbar__counter">
          <AisStats>
            <template slot-scope="{ nbHits }">
              <span class="navbar__label desktop-only">Products found: </span>
              <span
                data-cy="algolia_hits-counter-text--desktop"
                class="desktop-only"
                >{{ nbHits }}</span
              >
              <span
                data-cy="algolia_hits-counter-text--mobile"
                class="navbar__label smartphone-only"
                >{{ nbHits }} Items</span
              >
            </template>
          </AisStats>
        </div>
        <div class="navbar__view">
          <span class="navbar__view-label desktop-only">{{ $t("View") }}</span>
          <SfIcon
            data-cy="category-icon_grid-view"
            class="navbar__view-icon"
            :color="isCategoryGridView ? 'black' : 'dark-secondary'"
            icon="tiles"
            size="12px"
            role="button"
            aria-label="Change to grid view"
            :aria-pressed="isCategoryGridView"
            @click="toggleCategoryGridView"
          />
          <SfIcon
            data-cy="category-icon_list-view"
            class="navbar__view-icon"
            :color="!isCategoryGridView ? 'black' : 'dark-secondary'"
            icon="list"
            size="12px"
            role="button"
            aria-label="Change to list view"
            :aria-pressed="!isCategoryGridView"
            @click="toggleCategoryGridView"
          />
        </div>
      </div>
    </div>
    <div class="main section">
      <div class="sidebar desktop-only">hello</div>
      <div class="products">
        <AisStats>
          <template slot-scope="{ nbHits, query }">
            <h1 data-cy="algolia_search-title">
              <span v-if="nbHits === 0"><small>No Products Found</small></span>
              <span v-else><small>Search results for:</small> {{ query }}</span>
            </h1>
          </template>
        </AisStats>
        <AisHits :class-names="{ 'ais-Hits-item': 'override_Hits-item' }">
          <div slot="item" slot-scope="{ item }">
            <LazyHydrate when-idle>
              <SfProductCard
                :key="item.objectID"
                data-cy="category-product-card"
                :title="item.name"
                :image="item.image"
                :max-rating="5"
                :is-on-wishlist="false"
                class="products__product-card"
              />
            </LazyHydrate>
          </div>
        </AisHits>
        <AisPagination />
      </div>
    </div>
    <SfSidebar
      :visible="isFilterSidebarOpen"
      title="Filters"
      @close="toggleFilterSidebar"
    >
      <template #default>
        <!-- <ul>
          <li>
            test
          </li>
        </ul> -->
        <AisRefinementList attributes="brand" />
      </template>
    </SfSidebar>
  </AisInstantSearchSsr>
</template>

<script>
import {
  SfSidebar,
  SfButton,
  SfIcon,
  SfHeading,
  SfProductCard
} from "@storefront-ui/vue";
import {
  AisInstantSearchSsr,
  AisHits,
  AisSortBy,
  AisSearchBox,
  AisPagination,
  AisStats,
  AisRefinementList,
  createServerRootMixin
} from "vue-instantsearch";

// Libraries
import LazyHydrate from "vue-lazy-hydration";
import { useUiState } from "~/composables";
import algoliasearch from "algoliasearch/lite";
import "instantsearch.css/themes/algolia-min.css";

const indexName = "instant_search";
const searchClient = algoliasearch(
  "latency",
  "6be0576ff61c053d5f9a3225e2a90f76"
);

// remove indexName
// Props to: https://code.luasoftware.com/tutorials/algolia/setup-algolia-instantsearch-on-nuxt-with-query-url/
function writeState(routeState) {
  if (indexName in routeState) routeState = routeState[indexName];

  return routeState;
}

// restore indexName
// Props to: https://code.luasoftware.com/tutorials/algolia/setup-algolia-instantsearch-on-nuxt-with-query-url/
function readState(routeState) {
  routeState = {
    [indexName]: routeState
  };
  return routeState;
}

function nuxtRouter(vueRouter) {
  return {
    read() {
      return readState(vueRouter.currentRoute.query);
    },
    write(routeState) {
      if (this.createURL(routeState) === this.createURL(this.read())) {
        return;
      }
      vueRouter.push({
        query: writeState(routeState)
      });
    },
    createURL(routeState) {
      return vueRouter.resolve({
        query: writeState(routeState)
      }).href;
    },
    onUpdate(cb) {
      if (typeof window === "undefined") return;

      this._onPopState = event => {
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
    }
  };
}

export default {
  serverPrefetch() {
    return this.instantsearch.findResultsState(this).then(algoliaState => {
      this.$ssrContext.nuxt.algoliaState = algoliaState;
    });
  },
  components: {
    AisInstantSearchSsr,
    AisHits,
    AisSearchBox,
    AisPagination,
    AisSortBy,
    AisStats,
    AisRefinementList,
    SfSidebar,
    SfButton,
    SfIcon,
    SfHeading,
    SfProductCard,
    LazyHydrate
  },
  provide() {
    return {
      $_ais_ssrInstantSearchInstance: this.instantsearch
    };
  },
  setup() {
    const uiState = useUiState();

    return {
      AisSortBy,
      AisRefinementList,
      ...uiState
    };
  },
  data() {
    const mixin = createServerRootMixin({
      searchClient,
      indexName,
      routing: {
        router: nuxtRouter(this.$router)
      }
    });
    return {
      ...mixin.data()
    };
  },
  beforeMount() {
    const results =
      this.$nuxt.context.nuxtState.algoliaState || window.__NUXT__.algoliaState;
    this.instantsearch.hydrate(results);
  }
};
</script>

<style lang="scss">
@import "~@storefront-ui/vue/styles";
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

.ais-Hits-list:empty {
  margin: 0;
}

.ais-Hits-item {
  &.override_Hits-item {
    border: none;
    box-shadow: none;
  }
}

.ais-InstantSearch {
  margin: 1em;
}
</style>
