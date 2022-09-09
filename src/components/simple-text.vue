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
    <span v-if="signComment" class="vjs-comment"> {{ signComment }}</span>
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
      return this.showSignComment && signValue;
    },
    // 当前数据类型
    dataType() {
      return getDataType(this.data);
    }
  },
  methods: {
    defaultFormatter(data) {
      let text = data;
      if (this.dataType === "string") text = `"${text}"`;
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
