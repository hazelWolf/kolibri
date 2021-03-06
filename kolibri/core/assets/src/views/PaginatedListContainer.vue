<template>

  <div>
    <KGrid>
      <KGridItem :layout12="{ span: 7 }">
        <slot name="otherFilter"></slot>
      </KGridItem>
      <KGridItem
        :layout12="{ span: 5, alignment: 'right' }"
        class="text-filter"
      >
        <FilterTextbox
          v-model="filterInput"
          :placeholder="filterPlaceholder"
        />
      </KGridItem>
    </KGrid>

    <div>
      <slot v-bind="{ items: visibleFilteredItems, filterInput }"></slot>
    </div>

    <nav class="pagination-nav">
      <span dir="auto" class="pagination-label">
        {{ $tr('pagination', { visibleStartRange, visibleEndRange, numFilteredItems }) }}
      </span>
      <KButtonGroup>
        <KIconButton
          :ariaLabel="$tr('previousResults')"
          :disabled="previousButtonDisabled"
          size="small"

          icon="back"
          @click="changePage(-1)"
        />
        <KIconButton
          :ariaLabel="$tr('nextResults')"
          :disabled="nextButtonDisabled"
          size="small"

          icon="forward"
          @click="changePage(+1)"
        />
      </KButtonGroup>
    </nav>
  </div>

</template>


<script>

  import clamp from 'lodash/clamp';
  import FilterTextbox from 'kolibri.coreVue.components.FilterTextbox';

  export default {
    name: 'PaginatedListContainer',
    components: {
      FilterTextbox,
    },
    props: {
      // The entire list of items
      items: {
        type: Array,
        required: true,
      },
      filterFunction: {
        type: Function,
      },
      filterPlaceholder: {
        type: String,
        required: true,
      },
      itemsPerPage: {
        type: Number,
        required: false,
        default: 30,
      },
    },
    data() {
      return {
        filterInput: '',
        currentPage: 1,
      };
    },
    computed: {
      filteredItems() {
        if (this.filterFunction) {
          return this.filterFunction(this.items, this.filterInput);
        }
        return this.items;
      },
      numFilteredItems() {
        return this.filteredItems.length;
      },
      totalPages() {
        return Math.ceil(this.numFilteredItems / this.itemsPerPage);
      },
      startRange() {
        return (this.currentPage - 1) * this.itemsPerPage;
      },
      visibleStartRange() {
        return Math.min(this.startRange + 1, this.numFilteredItems);
      },
      endRange() {
        return this.currentPage * this.itemsPerPage;
      },
      visibleEndRange() {
        return Math.min(this.endRange, this.numFilteredItems);
      },
      visibleFilteredItems() {
        return this.filteredItems.slice(this.startRange, this.endRange);
      },
      previousButtonDisabled() {
        return this.currentPage === 1 || this.numFilteredItems === 0;
      },
      nextButtonDisabled() {
        return (
          this.totalPages === 1 ||
          this.currentPage === this.totalPages ||
          this.numFilteredItems === 0
        );
      },
    },
    watch: {
      numFilteredItems: {
        handler() {
          this.currentPage = 1;
          this.$emit('pageChanged', 1);
        },
      },
    },
    methods: {
      changePage(change) {
        // Clamp the newPage number between the bounds if browser doesn't correctly
        // disable buttons (see #6454 issue with old versions of MS Edge)
        this.currentPage = clamp(this.currentPage + change, 1, this.totalPages);
        this.$emit('pageChanged', this.currentPage);
      },
    },
    $trs: {
      previousResults: 'Previous results',
      nextResults: 'Next results',
      pagination:
        '{ visibleStartRange, number } - { visibleEndRange, number } of { numFilteredItems, number }',
    },
  };

</script>


<style lang="scss" scoped>

  .pagination-nav {
    margin-bottom: 8px;
    text-align: right;
  }

  .text-filter {
    margin-top: 14px;
  }

  .pagination-label {
    position: relative;
    top: -2px;
    display: inline;
  }

</style>
