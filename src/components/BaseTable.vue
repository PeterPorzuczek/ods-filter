<template>
  <table
    class="b-responsive b-table bordered"
    :style="`table-layout: ${layout};`"
  >
    <caption />
    <div v-if="isLoading">
      <slot name="loadingContent" />
    </div>
    <thead
      v-if="enabled.header"
      :class="`${!isLoading ? 't-opacity-100' : 't-opacity-25'}`"
    >
      <tr>
        <th
          v-for="(column, index) in columns"
          :key="`column_${index}`"
          :class="{ sortable: column.sortable }"
          scope="col"
          class="b-left-align b-auto-size"
          @click.prevent="changeSortingParameters(column)"
        >
          <slot :name="`header__${column.name}`" :column="column">
            <span v-if="(column.sortable || false) && sortedRows.length > 0">
              {{ column.label }}
            </span>
            <template v-else>
              {{ column.label }}
            </template>
          </slot>
        </th>
      </tr>
    </thead>
    <tbody :class="`${!isLoading ? 't-opacity-100' : 't-opacity-25'}`">
      <tr v-for="(row, index) in sortedRows" :key="`row_${index}`">
        <td
          v-for="(column, index) in columns"
          :key="`${row[column.name]}_${index}`"
        >
          <slot :row="row" :column="column" name="table-row">
            {{ row[column.name] }}
          </slot>
        </td>
      </tr>
      <tr v-if="sortedRows.length === 0">
        <slot :columns="columns" name="noResult">
          <td :colspan="columns.length" class="text-center pt-4 p-3">
            <slot name="empty">No results found</slot>
          </td>
        </slot>
      </tr>
    </tbody>
  </table>
</template>

<script>
import sort from "fast-sort/dist/sort.es";

export default {
  name: "BaseTable",
  props: {
    enabled: {
      type: Object,
      default: () => ({
        header: true
      })
    },
    layout: { type: String, default: "fixed" },
    columns: { type: Array, default: () => [] },
    rows: { type: Array, default: () => [] }
  },
  data() {
    return {
      sortBy: "",
      sortDirection: "desc",
      isLoading: false
    };
  },
  computed: {
    sortedRows() {
      if (this.sortBy) {
        const sorted =
          this.sortDirection === "asc"
            ? sort([...this.rows]).asc(row => row[this.sortBy])
            : sort([...this.rows]).desc(row => row[this.sortBy]);

        return sorted;
      }

      return this.rows;
    }
  },
  methods: {
    fieldExistsInRow(row, name) {
      return Object.keys(row).includes(name);
    },
    getSortKey(column) {
      return typeof column.sortable === "string"
        ? column.sortable
        : column.name;
    },
    isSorted(column) {
      return this.getSortKey(column) === this.sortBy;
    },
    changeSortingParameters(column) {
      if (column.sortable) {
        this.sortBy = this.getSortKey(column);
        this.sortDirection = this.sortDirection === "asc" ? "desc" : "asc";
        this.$emit("column:sort", {
          sortBy: this.sortBy,
          sortDirection: this.sortDirection
        });
      }
    },
    stopLoading() {
      this.isLoading = false;
    },
    startLoading() {
      this.isLoading = true;
    }
  }
};
</script>

<style scoped>
table.b-table {
  font-size: 16px;
  border-collapse: collapse;
  background-color: #fff;
  width: 100%;
  max-width: 100%;
  border: 1px solid #dcdfe6;
  font-size: inherit;
}
.b-table.bordered td,
.b-table.bordered th {
  border: 1px solid #dcdfe6;
}
.b-table th {
  padding: 0.75em 1.5em 0.75em 0.75em;
  vertical-align: middle;
  position: relative;
}
.b-table th.sortable {
  cursor: pointer;
}
.b-table thead th {
  color: #606266;
  vertical-align: bottom;
  border-bottom: 1px solid #dcdfe6;
  padding-right: 1.5em;
  background: linear-gradient(#f4f5f8, #f1f3f6);
}
.b-table.bordered td,
.b-table.bordered th {
  border: 1px solid #dcdfe6;
}
table.b-table td {
  padding: 0.75em;
  vertical-align: top;
  border-bottom: 1px solid #dcdfe6;
  color: #606266;
  word-break: break-word;
}
.b-right-align {
  text-align: right;
}
.b-left-align {
  text-align: left;
}
.b-auto-size {
  min-width: auto;
  width: auto;
}
.b-responsive {
  width: 100%;
  overflow-x: auto;
  position: relative;
}
</style>
