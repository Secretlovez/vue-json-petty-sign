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

      <span v-if="showLength" :class="signCommentClass">
        {{ lengthGenerator(data) }}
      </span>
      <span v-if="signComment" :class="signCommentClass">
        {{ signComment }}
      </span>
    </span>
    <span
      v-if="deep === 1 && showCopy"
      :class="`vjs-tree__copy ${hasCopied ? 'vjs-tree__copy-success' : ''} `"
      @click="onCopyText"
      >{{ hasCopied ? copyCompleteTitle : copyTitle }}
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
    parentKey: {
      type: [Number, String],
      default: ""
    },
    deep: {
      type: [Number, String],
      default: 1
    },
    // 标记信息
    signKeys: {
      type: Object,
      default: null
    },
    addKeys: {
      type: Object,
      default: null
    },
    // 是否展示复制按钮
    showCopy: {
      type: Boolean,
      default: true
    },
    // 复制按钮文字
    copyTitle: {
      type: String,
      default: "复制"
    },
    // 复制完成按钮文字
    copyCompleteTitle: {
      type: String,
      default: "已复制"
    }
  },
  data() {
    return {
      hasCopied: false
    };
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
    }
  },
  methods: {
    async onCopyText() {
      if (this.hasCopied) return;
      let copyText = "";
      if (typeof this.data === "string") {
        copyText = this.data;
      } else {
        copyText = JSON.stringify(this.data);
      }
      // https或者本地调试可用
      // copyText = await navigator.clipboard.writeText(copyText);
      // http 可用，移动端会出现问题
      const input = document.createElement("input");
      document.body.appendChild(input);
      input.setAttribute("value", copyText);
      input.select();
      document.execCommand("copy");
      document.body.removeChild(input);

      this.hasCopied = true;
      setTimeout(() => {
        this.hasCopied = false;
      }, 2000);
    },
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
