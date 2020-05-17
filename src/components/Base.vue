<template>
  <div
    class="t-fullscreen t-overflow-auto t-w-full t-h-full t-bg-hero-happy-intersection-grey"
  >
    <div class="t-w-9/10 t-m-auto">
      <div
        class="t-w-full t-flex t-flex-row t-bg-white t-pb-0 t-shadow-sm t-rounded t-m-auto"
      >
        <div
          class="t-flex t-justify-center t-items-center t-relative t-w-1/5 t-min-w-1/5"
        >
          <h1 class="t-p-0">Ods filter</h1>
        </div>
        <div class="t-flex t-justify-center t-items-center t-relative ">
          <div class="t-w-32">
            <button
              class="t-border-none t-bg-green hover:t-bg-green-dark t-text-white t-font-bold t-py-2 t-px-4 t-w-full t-text-center t-inline-flex t-items-center t-rounded"
            >
              <svg
                fill="#FFF"
                height="18"
                viewBox="0 0 24 24"
                width="18"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path d="M0 0h24v24H0z" fill="none" />
                <path d="M9 16h6v-6h4l-7-7-7 7h4zm-4 2h14v2H5z" />
              </svg>
              <span class="t-ml-2">Choose file</span>
            </button>
            <input
              class="t-mt-6 t-cursor-pointer t-absolute t-block t-opacity-0 t-right-0 t-top-0 t-w-32 t-h-10"
              type="file"
              name="file"
              id="file-selector"
              accept=".ods"
            />
          </div>
        </div>
        <div class="t-flex t-relative t-flex-wrap t-m-2 t-mx-4 t-w-16 t-pl-3">
          <input
            v-model="precision"
            :placeholder="0.01"
            type="text"
            tabindex="1"
            autocomplete="off"
            class="t-input t-appearance-none t-rounded-sm
              t-py-2 t-px-3 t-outline-none t-h-full
              t-w-9/10 t-border-white t-border t-w-full
              t-w-full t-text-center t-h-8 t-border t-rounded-lg t-center
              t-text-base"
          />
        </div>
        <div class="t-flex t-relative t-flex-wrap t-m-2 t-mx-4">
          <button
            class="t-border-none t-bg-green-lighter hover:t-bg-green-dark t-text-black t-font-bold t-py-2 t-px-4 t-rounded t-m-1"
            v-for="(sheet, index) in sheets"
            :key="`sheet-${index}`"
            @click="selectSheet(index)"
          >
            {{ sheet.name }}
          </button>
        </div>
      </div>

      <div
        class="t-flex t-relative t-w-full t-m-auto t-items-center t-justify-center t-bg-white t-shadow-sm t-rounded t-flex-wrap t-mt-6 t-mb-8 t-overflow-scroll-y"
      >
        <div
          v-for="(sheet, index) in selected"
          :key="`sheet-selected-${index}`"
          class="t-relative t-w-full"
        >
          <base-filter-list
            :name="sheet.name"
            v-if="sheet.json.length > 2"
            :records="sheet.json"
            :keys="Object.keys(sheet.json[1])"
            special-column="UWAGI"
            :precision="Number(precision) ? Number(precision) : 0.01"
            v-slot:default="slotProps"
          >
            <div>
              <strong>{{ slotProps.recordKey }}:</strong>
              {{ slotProps.record[slotProps.recordKey] }}
            </div>
          </base-filter-list>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import XLSX from "xlsx";
import BaseFilterList from "./BaseFilterList";

export default {
  name: "Base",
  components: {
    BaseFilterList
  },
  data() {
    return {
      precision: 0.01,
      sheets: [],
      selected: []
    };
  },
  mounted() {
    const fileSelector = document.getElementById("file-selector");
    fileSelector.addEventListener("change", this.previewFiles);
  },
  watch: {
    selected() {
      if (this.selected.length > 0) {
        this.selected[0].json.forEach(record => {
          if (record.IMIE && record.NAZWISKO) {
            record.NAZWISKO_IMIE = `${record.NAZWISKO} ${record.IMIE}`;
          }
        });
      }
    }
  },
  methods: {
    selectSheet(index) {
      this.selected = [this.sheets[index]];
    },
    previewFiles(event) {
      const files = event.target.files;
      const file = files[0];
      const reader = new FileReader();
      reader.onload = this.handleReaderOnLoad;
      reader.readAsArrayBuffer(file);
    },
    handleReaderOnLoad(event) {
      const data = new Uint8Array(event.target.result);
      const workbook = XLSX.read(data, { type: "array" });

      workbook.SheetNames.forEach(sheetName => {
        const worksheet = workbook.Sheets[sheetName];
        const worksheetAsJson = this.temporarySwap(
          XLSX.utils.sheet_to_json(worksheet, {
            raw: false
          })
        );

        this.sheets.push({
          name: sheetName,
          sheet: worksheet,
          json: worksheetAsJson
        });
      });
    },
    temporarySwap(array) {
      var left = null;
      var right = null;
      var length = array.length;
      for (left = 0, right = length - 1; left < right; left += 1, right -= 1) {
        var temporary = array[left];
        array[left] = array[right];
        array[right] = temporary;
      }
      return array;
    }
  }
};
</script>
