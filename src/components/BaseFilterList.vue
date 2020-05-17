<template>
  <div class="t-relative t-w-full">
    <div
      class="t-flex t-justify-center t-items-center t-relative t-pb-2 t-pt-3"
    >
      <div
        class="t-flex t-justify-center t-items-center t-relative t-w-1/5 t-min-w-1/5 t-max-w-xs"
      >
        <h3
          class="t-flex t-relative t-p-0 t-overflow-hidden t-flex-wrap t-px-2 t-break-words t-break-all t-justify-center"
        >
          {{ name }}
        </h3>
      </div>
      <div class="t-w-4/5 t-pr-2">
        <div
          v-if="isCharactersWarningVisible"
          class="t-relative t-flex t-w-full
        t-justify-center t-text-center t-py-1
        t-text-white t-text-xs t-bg-red-light
        t-rounded-sm t-opacity-75"
        >
          {{ textMinCharactersToFilter }}
        </div>
        <base-filter
          :list="records"
          :keys="keys"
          :threshold="precision"
          :placeholder="placeholder"
          :min-number-of-characters="minCharactersToFilter"
          event-name="recordsFiltered"
          input-change-event-name="inputChanged"
        />
      </div>
    </div>
    <div class="t-p-2" style="max-height: 65vh; overflow: scroll">
      <base-table
        :columns="columns"
        :rows="recordsFiltered"
        :layout="
          columns.filter(column => column.name == specialColumn).length > 0 &&
          columns.length <= 7
            ? 'auto'
            : 'fixed'
        "
      >
        <template slot-scope="{ row, column }" slot="table-row">
          <div
            v-if="column.name == specialColumn && columns.length <= 7"
            style="width: 450px"
          >
            {{ row.item ? row.item[column.name] : row[column.name] }}
          </div>
          <div v-else style="max-width: 91px; min-width: 91px;">
            {{ row.item ? row.item[column.name] : row[column.name] }}
          </div>
        </template>
      </base-table>
    </div>
  </div>
</template>

<script>
import BaseFilter from "./BaseFilter";
import BaseTable from "./BaseTable";
import sort from "fast-sort/dist/sort.es";
import debounce from "v-debounce/debounce";

export default {
  name: "BaseFilterList",
  components: {
    BaseFilter,
    BaseTable
  },
  props: {
    precision: { type: Number, default: 0.01 },
    textMinCharactersToFilter: {
      type: String,
      default: "Input minimum characters not reached"
    },
    minCharactersToFilter: { type: Number, default: 3 },
    records: {
      type: Array,
      default: () => []
    },
    keys: {
      type: Array,
      default: () => []
    },
    placeholder: { type: String, default: "Filter" },
    specialColumn: { type: String, default: "" },
    name: { type: String, default: "" }
  },
  data() {
    return {
      recordsFiltered: [],
      isCharactersWarningVisible: false,
      isFiltering: false
    };
  },
  mounted() {
    this.$on("recordsFiltered", debounce(this.changeRecords, 500));
    this.$on("inputChanged", this.displayWarning);
  },
  watch: {
    name() {
      this.$nextTick(() => {
        this.recordsFiltered = this.records;
      });
    }
  },
  computed: {
    columns() {
      return this.recordsFiltered.length > 1
        ? this.createColumns(
            this.recordsFiltered[1].item
              ? this.recordsFiltered[1].item
              : this.recordsFiltered[1]
          )
        : this.recordsFiltered.length > 0
        ? this.createColumns(
            this.recordsFiltered[0].item
              ? this.recordsFiltered[0].item
              : this.recordsFiltered[0]
          )
        : [];
    }
  },
  methods: {
    changeRecords(filtered) {
      this.recordsFiltered = filtered;
      this.sortFilteredResultsByOriginalPosition();
    },
    sortFilteredResultsByOriginalPosition() {
      if (this.recordsFiltered.length > 0 && this.recordsFiltered[0].refIndex) {
        sort(this.recordsFiltered).asc(row => row["refIndex"]);
      }
    },
    displayWarning(input) {
      this.isCharactersWarningVisible =
        input.length > 0 && input.length < this.minCharactersToFilter;
    },
    createColumns(record) {
      return Object.keys(record)
        .map((key, index) => ({
          id: index + 1,
          label: key.includes("EMPTY") ? "" : key,
          name: key,
          sortable: false
        }))
        .filter((column, index) => index < 7);
    }
  }
};
</script>
