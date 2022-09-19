<template>
  <div v-show="dataVisible">
    <span
      :class="`vjs-tree__brackets ${bracketsClass}`"
      @click.stop="toggleBrackets"
    >
      {{ bracketsFormatter(Array.isArray(data) ? "]" : "}") }}
    </span>
    <span v-if="signComment" class="vjs-comment">
      {{ signComment }}
    </span>
  </div>
</template>

<script>
import bracketsMixin from "src/mixins/brackets-mixin";

export default {
  mixins: [bracketsMixin],
  props: {
    bracketsClass: {
      type: String,
      default: ""
    },
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
    }
  },
  computed: {
    signComment() {
      let signValue = "";
      if (
        this.signKeys &&
        this.signKeys[this.currentKey] !== undefined &&
        this.signKeys[this.currentKey] !== ""
      ) {
        signValue = ` // ${this.signKeys[this.currentKey]}`;
      }
      if (
        this.signKeys &&
        this.signKeys[this.parentKey] !== undefined &&
        this.signKeys[this.parentKey] !== ""
      ) {
        signValue = ` // ${this.signKeys[this.parentKey]}`;
      }
      return this.showSignComment && signValue;
    },
    // 当前数据类型
    dataType() {
      return getDataType(this.data);
    }
  }
};
</script>
