<template>
  <input
    v-debounce="delay"
    v-model="value"
    :placeholder="placeholder"
    type="text"
    tabindex="1"
    autocomplete="off"
    class="t-input t-appearance-none t-rounded-sm
      t-py-2 t-px-3 t-outline-none t-h-full
      t-w-9/10 t-border-grey t-border t-w-full
      t-w-full t-text-center t-h-8 t-border t-rounded-lg t-center
      t-text-base"
  />
</template>

<script>
import Fuse from "fuse.js";
import debounce from "v-debounce";

export default {
  name: "BaseFilter",
  directives: {
    debounce
  },
  props: {
    placeholder: {
      type: String,
      default: ""
    },
    search: {
      type: String,
      default: ""
    },
    eventName: {
      type: String,
      default: "fuseResultsUpdated"
    },
    inputChangeEventName: {
      type: String,
      default: "fuseInputChanged"
    },
    defaultAll: {
      type: Boolean,
      default: true
    },
    list: {
      type: Array,
      default: () => []
    },
    caseSensitive: {
      type: Boolean,
      default: false
    },
    includeScore: {
      type: Boolean,
      default: false
    },
    includeMatches: {
      type: Boolean,
      default: false
    },
    tokenize: {
      type: Boolean,
      default: false
    },
    matchAllTokens: {
      type: Boolean,
      default: false
    },
    findAllMatches: {
      type: Boolean,
      default: false
    },
    id: {
      type: String,
      default: ""
    },
    shouldSort: {
      type: Boolean,
      default: true
    },
    threshold: {
      type: Number,
      default: 0.6
    },
    location: {
      type: Number,
      default: 0
    },
    distance: {
      type: Number,
      default: 100
    },
    maxPatternLength: {
      type: Number,
      default: 32
    },
    minMatchCharLength: {
      type: Number,
      default: 1
    },
    minNumberOfCharacters: {
      type: Number,
      default: 1
    },
    keys: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      fuse: null,
      value: "",
      result: [],
      delay: 40
    };
  },
  computed: {
    options() {
      var options = {
        caseSensitive: this.caseSensitive,
        includeScore: this.includeScore,
        includeMatches: this.includeMatches,
        tokenize: this.tokenize,
        matchAllTokens: this.matchAllTokens,
        findAllMatches: this.findAllMatches,
        shouldSort: this.shouldSort,
        threshold: this.threshold,
        location: this.location,
        distance: this.distance,
        maxPatternLength: this.maxPatternLength,
        minMatchCharLength: this.minMatchCharLength,
        keys: this.keys
      };
      if (this.id !== "") {
        options.id = this.id;
      }

      return options;
    }
  },
  watch: {
    list() {
      this.fuse.list = this.list;
      this.fuseSearch();
    },
    search() {
      this.value = this.search;
    },
    value() {
      this.$parent.$emit(this.inputChangeEventName, this.value);
      this.$emit(this.inputChangeEventName, this.value);
      this.fuseSearch();
    },
    result() {
      this.$emit(this.eventName, this.result);
      this.$parent.$emit(this.eventName, this.result);
    },
    threshold() {
      this.initFuse();
    }
  },
  mounted() {
    this.initFuse();
  },
  methods: {
    initFuse() {
      this.fuse = new Fuse(this.list, this.options);
      if (this.defaultAll) {
        this.result = this.list;
      }
      if (this.search) {
        this.value = this.search;
      }
    },
    fuseSearch() {
      this.$emit("filtering:start");
      if (
        this.value.trim() === "" ||
        this.value.trim().length < this.minNumberOfCharacters
      ) {
        if (this.defaultAll) {
          this.result = this.list;
        } else {
          this.result = [];
        }
      } else {
        this.result = this.fuse.search(this.value.trim());
      }
      this.$emit("filtering:stop");
    }
  }
};
</script>
