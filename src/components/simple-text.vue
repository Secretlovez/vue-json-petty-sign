<template>
  <div>
    <slot />
    <span
      v-if="customValueFormatter"
      :class="valueClass"
      v-html="customFormatter(data)"
    />
    <span v-else :class="valueClass">{{ defaultFormatter(data) }}</span>
    <span v-if="showComma" :class="valueClass">,</span>
    <span v-if="signComment" :class="signCommentClass"> {{ signComment }}</span>
    <!-- <div>{{ signComment }}</div> -->
  </div>
</template>

<script>
import { getDataType } from "src/utils";

export default {
  props: {
    showDoubleQuotes: Boolean,
    parentData: {
      type: [String, Number, Boolean, Array, Object],
      default: null
    },
    data: {
      type: [String, Number, Boolean],
      default: ""
    },
    showComma: Boolean,
    currentKey: {
      type: [Number, String],
      default: ""
    },
    parentKey: {
      type: [Number, String],
      default: ""
    },
    showSignComment: {
      type: Boolean,
      default: true
    },
    // 标记键值对
    signKeys: {
      type: Object,
      default: null
    },
    addKeys: {
      type: Object,
      default: null
    },
    customValueFormatter: {
      type: Function,
      default: null
    }
  },
  computed: {
    valueClass() {
      let signClass = "";
      // 如果当前键值为标记键值对，则添加标记类
      if (this.signKeys && this.signKeys[this.currentKey] !== undefined) {
        signClass = "vjs-value__sign";
      }
      // 如果父级键值为标记键值对，则添加标记类
      if (
        this.signKeys &&
        this.parentKey &&
        this.signKeys[this.parentKey] !== undefined
      ) {
        signClass = "vjs-value__sign";
      }
      if (
        this.addKeys &&
        ((this.currentKey &&
          this.addKeys[this.currentKey] !== undefined &&
          this.addKeys[this.currentKey] !== "") ||
          (this.parentKey &&
            this.addKeys[this.parentKey] !== undefined &&
            this.addKeys[this.parentKey] !== ""))
      ) {
        signClass = `vjs-value__notexists`;
      }
      return `vjs-value vjs-value__${this.dataType} ${signClass}`;
    },
    signComment() {
      // 如果当前键值为标记键值对，且展示标记注释，则返回标记注释
      let signValue = "";
      if (
        this.signKeys &&
        this.signKeys[this.currentKey] !== undefined &&
        this.signKeys[this.currentKey] !== ""
      ) {
        signValue = ` // ${this.signKeys[this.currentKey]}`;
      }
      const index = this.parentKey.lastIndexOf(".");
      if (
        index !== -1 &&
        this.signKeys &&
        this.signKeys[this.parentKey] !== undefined &&
        this.signKeys[this.parentKey] !== "" &&
        this.parentKey.slice(index + 1, this.parentKey.length) ===
          this.currentKey
      ) {
        signValue = ` // ${this.signKeys[this.parentKey]}`;
      }
      return this.showSignComment && signValue;
    },
    signCommentClass() {
      let signCommentClass = "vjs-comment";
      if (
        this.addKeys &&
        ((this.currentKey &&
          this.addKeys[this.currentKey] !== undefined &&
          this.addKeys[this.currentKey] !== "") ||
          (this.parentKey &&
            this.addKeys[this.parentKey] !== undefined &&
            this.addKeys[this.parentKey] !== ""))
      ) {
        signCommentClass = `vjs-comment__notexists`;
      }

      return signCommentClass;
    },
    // 当前数据类型
    dataType() {
      if (this.data === "undefined-{03C69C01-9EA8-4E19-98B5-539AFED96E94}")
        return `undefined`;
      return getDataType(this.data);
    }
  },
  methods: {
    defaultFormatter(data) {
      let text = data;
      if (this.dataType === "string") text = `"${text}"`;
      if (this.dataType === "null" && data === null) text = `${text}`;
      if (data === "undefined-{03C69C01-9EA8-4E19-98B5-539AFED96E94}")
        text = `undefined`;
      return text;
    },

    customFormatter(data) {
      return this.customValueFormatter
        ? this.customValueFormatter(
            data,
            this.currentKey,
            this.parentData,
            this.defaultFormatter(data)
          )
        : this.defaultFormatter(data);
    }
  }
};
</script>
