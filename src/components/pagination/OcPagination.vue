<template>
  <nav class="oc-pagination" :aria-label="$gettext('Pagination')">
    <ul class="oc-pagination-list">
      <li v-if="hasEdgeNav && currentPage !== 1" class="oc-pagination-list-item">
        <router-link
          class="oc-pagination-list-item-first"
          :aria-label="$gettext('Go to first page')"
          v-text="$gettext('First page')"
        />
      </li>
      <li class="oc-pagination-list-item">
        <router-link
          class="oc-pagination-list-item-prev"
          :aria-label="$gettext('Go to previous page')"
          v-text="$gettext('Previous')"
        />
      </li>
      <li v-for="page in displayedPages" :key="page" class="oc-pagination-list-item">
        <component
          :is="pageComponent(page)"
          class="oc-pagination-list-item-page"
          :class="isCurrentPage(page) ? 'oc-pagination-list-item-current' : null"
          v-bind="pageProps(page)"
          v-text="page"
        />
      </li>
      <li class="oc-pagination-list-item">
        <router-link
          class="oc-pagination-list-item-next"
          :aria-label="$gettext('Go to next page')"
          v-text="$gettext('Next')"
        />
      </li>
      <li v-if="hasEdgeNav && currentPage !== pages.length" class="oc-pagination-list-item">
        <router-link
          class="oc-pagination-list-item-last"
          :aria-label="$gettext('Go to last page')"
          v-text="$gettext('Last page')"
        />
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: "OcPagination",

  props: {
    /**
     * Asserts whether links to the first and last page are displayed
     */
    hasEdgeNav: {
      type: Boolean,
      required: false,
      default: true,
    },
    /**
     * Array of pages to be displayed
     */
    pages: {
      type: Array,
      required: true,
    },
    /**
     * Currently active page
     */
    currentPage: {
      type: Number,
      required: true,
    },
    /**
     * Number of pages to be displayed. Needs to be an even number.
     * Pages will be equaly split into two parts and remaining pages will be displayed as `...`
     */
    maxDisplayed: {
      type: Number,
      required: false,
      default: null,
      validator: value => {
        if (value % 2 === 0) {
          return true
        }

        // Since Vue doens't support custom validator error message, log the error manually
        console.error("maxDisplayed needs to be an even number")
      },
    },
  },

  computed: {
    displayedPages() {
      if (this.maxDisplayed && this.maxDisplayed < this.pages.length) {
        console.log(this.pages.slice(0, this.maxDisplayed))
        console.log(this.pages.slice(this.maxDisplayed + 1))
        const pages = [
          ...this.pages.slice(0, this.maxDisplayed),
          "...",
          ...this.pages.slice(this.maxDisplayed + 1),
        ]

        return pages
      }

      return this.pages
    },
  },

  methods: {
    pageLabel(page) {
      const translated = this.$gettext("Go to page %{ page }", page.toString())

      return this.$gettextInterpolate(translated, { page: page })
    },

    isCurrentPage(page) {
      return this.currentPage === page
    },

    pageComponent(page) {
      return this.isCurrentPage(page) ? "span" : "router-link"
    },

    pageProps(page) {
      if (this.isCurrentPage(page)) {
        return {
          "aria-current": "true",
        }
      }

      return {
        "aria-label": this.pageLabel(page),
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.oc-pagination {
  &-list {
    align-items: center;
    display: flex;
    gap: $space-small;
    list-style: none;

    &-item {
      &-page {
        background-color: $brand-primary;
        border-radius: 4px;
        color: $inverse-color;
        padding: $space-xsmall $space-small;
        transition: background-color $transition-duration-short ease-in-out;

        &:hover {
          background-color: $action-primary;
          color: $inverse-color;
          text-decoration: none;
        }
      }

      &-current {
        background-color: $action-primary;
        font-weight: bold;
      }
    }
  }
}
</style>

<docs>
```vue
<template>
  <div>
      <oc-pagination :pages="pages" :currentPage="3" />
      <oc-pagination :pages="pages" :currentPage="1" />
      <oc-pagination :pages="pages" :currentPage="5" />
      <oc-pagination :pages="pages" :currentPage="3" :hasEdgeNav="false" />
      <oc-pagination :pages="pages" :currentPage="3" :maxDisplayed="2" />
  </div>
</template>
<script>
export default {
  data: () => ({
    pages: [1, 2, 3, 4, 5]
  })
}
</script>
```
</docs>
