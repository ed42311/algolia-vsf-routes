<template>
  <SfList class="list">
    <SfListItem
      v-for="item in items"
      :key="item.value"
      :class="level >= 1 ? 'list__item' : 'list__item--subcat'"
    >
      <a
        class="list__search-link"
        :href="createURL(item.value)"
        @click.prevent="refine(item.value)"
      >
        <SfMenuItem
          :data-cy="`search-link_subcategory_${item.label}`"
          :class="[
            'search-link',
            level >= 1 ? 'search-link__block' : 'search-link__header',
            item.isRefined && '--refined'
          ]"
        >
          <template #label>
            <span :class="['search-link__label', '--level' + level]">{{
              item.label
            }}</span>
          </template>
          <template #mobile-nav-icon>
            <span style="display: none"></span>
          </template>
          <template #count>
            <span v-show="level >= 1" class="search-link__count">{{
              item.count
            }}</span>
          </template>
        </SfMenuItem>
      </a>

      <hierarchical-menu-list
        v-if="item.data"
        :items="item.data"
        :level="level + 1"
        :refine="refine"
        :createURL="createURL"
      />
    </SfListItem>
  </SfList>
</template>

<script>
import { SfMenuItem, SfList } from "@storefront-ui/vue";
export default {
  name: "HierarchicalMenuList",
  components: {
    SfMenuItem,
    SfList
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    level: {
      type: Number,
      required: true
    },
    refine: {
      type: Function,
      required: true
    },
    createURL: {
      type: Function,
      required: true
    }
  }
};
</script>

<style lang="scss" scoped>
// TODO: lift these up to global
$primary-green: #5ece7b;
$primary-black: #1d1f22;
$secondary-black: #43464e;
.search-link {
  justify-content: space-between;
  color: $secondary-black;
  &__block {
    font-style: normal;
    font-weight: normal;
    font-size: 16px;
    line-height: 24px;
    padding-top: 0.4em;
    &.--refined {
      text-decoration-line: underline;
    }
    .search-link__label {
      @for $i from 1 through 4 {
        &.--level#{$i} {
          margin-left: ($i * 0.5em);
        }
      }
    }
  }
  &__header {
    font-size: 18px;
    font-weight: 600;
    padding: 0.4em;
    &.--refined {
      color: $primary-green;
    }
    &::after {
      content: "\276F";
      width: 1em;
      height: 1em;
      font-weight: 300;
      text-align: center;
      transform: rotate(90deg);
      transition: all 0.35s;
    }
  }
  &__label {
    text-align: left;
  }
  &__count {
    align-self: baseline;
    padding-right: 0.5em;
  }
}
</style>
