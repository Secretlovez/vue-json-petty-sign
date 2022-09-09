<template>
  <div>
    <slot />

    <!-- Expand -->
    <span
      v-show="dataVisible"
      :class="`vjs-tree__brackets ${bracketsClass}`"
      @click.stop="toggleBrackets"
    >
      {{ Array.isArray(data) ? "[" : "{" }}
    </span>

    <!-- Collapse -->
    <span v-show="!dataVisible">
      <span
        :class="`vjs-tree__brackets ${bracketsClass}`"
        @click.stop="toggleBrackets"
      >
        {{ closedBracketsGenerator(data) }}
      </span>

      <span v-if="showLength" class="vjs-comment">
        {{ lengthGenerator(data) }}
      </span>
      <span v-if="signComment" class="vjs-comment">
        {{ signComment }}
      </span>
    </span>
  </div>
</template>

<script>
import bracketsMixin from "src/mixins/brackets-mixin";

export default {
  mixins: [bracketsMixin],
  props: {
    showLength: Boolean,
    bracketsClass: {
      type: String,
      default: ""
    },
    showSignComment: {
      type: Boolean,
      default: true
    },
    currentKey: {
      type: [Number, String],
      default: ""
    },
    // 错误信息
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
      return this.showSignComment && signValue;
    }
  },
  methods: {
    // 关闭括号生成器
    closedBracketsGenerator(data) {
      const brackets = Array.isArray(data) ? "[...]" : "{...}";
      return this.bracketsFormatter(brackets);
    },

    // 长度标记生成器
    lengthGenerator(data) {
      // 若展示长度, 形如 [...] // 3 items
      const text = Array.isArray(data)
        ? `${data.length} items`
        : `${Object.keys(data).length} keys`;
      return ` // ${text}`;
    }
  }
};
</script>
