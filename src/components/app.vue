<template>
  <div
    :class="{
      'vjs-tree': true,
      'has-selectable-control': isMultiple || showSelectController,
      'is-root': currentDeep === 1,
      'is-selectable': selectable,
      'is-selected': isSelected,
      'is-highlight-selected': isSelected && highlightSelectedNode,
      'is-mouseover': isMouseover
    }"
    @click="handleClick"
    @mouseover.stop="handleMouseover"
    @mouseout.stop="handleMouseout"
  >
    <template v-if="showSelectController && selectable">
      <vue-checkbox
        v-if="isMultiple"
        v-model="currentCheckboxVal"
        @change="handleValueChange('checkbox')"
      />
      <vue-radio
        v-else-if="isSingle"
        v-model="model"
        :path="path"
        @change="handleValueChange('radio')"
      />
    </template>

    <template v-if="Array.isArray(currentData) || isObject(currentData)">
      <!-- 左闭合 -->
      <brackets-left
        :visible.sync="visible"
        :data="currentData"
        :brackets-class="`${signClass}`"
        :show-length="showLength"
        :collapsed-on-click-brackets="collapsedOnClickBrackets"
        :show-comma="notLastKey"
        :show-sign-comment="showSignComment"
        :current-key="currentKey"
        :parent-key="prevKey"
        :sign-keys="signKeys"
        :add-keys="addKeys"
      >
        <span
          v-if="currentDeep > 1 && !Array.isArray(parentData)"
          :class="`vjs-key ${signClass}`"
        >
          {{ prettyKey }}:
        </span>
      </brackets-left>

      <!-- 数据内容, data 为对象时, key 表示键名, 为数组时表示索引 -->
      <div
        v-for="(item, key) in currentData"
        v-show="visible"
        :key="key"
        :class="{
          'vjs-tree__content': true,
          'has-line': showLine
        }"
      >
        <vue-json-pretty-sign
          v-model="model"
          :parent-data="currentData"
          :data="isUndefined(item)"
          :deep="deep"
          :show-length="showLength"
          :show-double-quotes="showDoubleQuotes"
          :show-line="showLine"
          :show-sign-comment="showSignComment"
          :highlight-mouseover-node="highlightMouseoverNode"
          :highlight-selected-node="highlightSelectedNode"
          :path="getChildPath(key)"
          :path-selectable="pathSelectable"
          :selectable-type="selectableType"
          :show-select-controller="showSelectController"
          :select-on-click-node="selectOnClickNode"
          :collapsed-on-click-brackets="collapsedOnClickBrackets"
          :current-key="key"
          :parent-key="prevKey"
          :parent-type="prevType"
          :sign-keys="signKeys"
          :add-keys="addKeys"
          :current-deep="currentDeep + 1"
          :custom-value-formatter="customValueFormatter"
          @click="handleItemClick"
          @change="handleItemChange"
        />
      </div>

      <!-- 右闭合 -->
      <brackets-right
        :visible.sync="visible"
        :data="currentData"
        :brackets-class="`${signClass}`"
        :collapsed-on-click-brackets="collapsedOnClickBrackets"
        :show-comma="notLastKey"
        :show-sign-comment="showSignComment"
        :current-key="currentKey"
        :parent-key="prevKey"
        :sign-keys="signKeys"
        :add-keys="addKeys"
      />
    </template>

    <simple-text
      v-else
      :custom-value-formatter="customValueFormatter"
      :show-double-quotes="showDoubleQuotes"
      :show-comma="notLastKey"
      :parent-data="parentData"
      :data="currentData"
      :current-key="currentKey"
      :parent-key="prevKey"
      :show-sign-comment="showSignComment"
      :sign-keys="signKeys"
      :add-keys="addKeys"
    >
      <span
        v-if="parentData && currentKey && !Array.isArray(parentData)"
        :class="`vjs-key ${signClass}`"
      >
        {{ prettyKey }}:
      </span>
    </simple-text>
  </div>
</template>

<script>
import { set } from "lodash";
import SimpleText from "./simple-text";
import VueCheckbox from "./checkbox";
import VueRadio from "./radio";
import BracketsLeft from "./brackets-left";
import BracketsRight from "./brackets-right";
import { getDataType } from "src/utils";

export default {
  name: "VueJsonPrettySign",
  components: {
    SimpleText,
    VueCheckbox,
    VueRadio,
    BracketsLeft,
    BracketsRight
  },
  props: {
    /* outer props */
    // 当前树的数据
    data: {
      type: [String, Number, Boolean, Array, Object],
      default: null
    },
    // 定义树的深度, 大于该深度的子树将不被展开
    deep: {
      type: Number,
      default: Infinity
    },
    // 是否显示数组|对象的长度
    showLength: {
      type: Boolean,
      default: false
    },
    // key名是否显示双引号
    showDoubleQuotes: {
      type: Boolean,
      default: true
    },
    // 数据层级顶级路径
    path: {
      type: String,
      default: ""
    },
    // 定义数据层级支持的选中方式, 默认无该功能
    selectableType: {
      type: String,
      default: "" // ''|multiple|single    radio, checkbox
    },
    // 是否展示左侧选择控件
    showSelectController: {
      type: Boolean,
      default: false
    },
    showLine: {
      type: Boolean,
      default: true
    },
    // 是否在点击树的时候选中节点
    selectOnClickNode: {
      type: Boolean,
      default: true
    },
    // 存在选择功能时, 定义已选中的数据层级
    //    多选时为数组['root.a', 'root.b'], 单选时为字符串'root.a'
    value: {
      type: [Array, String],
      default: () => ""
    },
    // 定义某个数据层级是否支持选中操作
    pathSelectable: {
      type: Function,
      default: () => true
    },
    // highlight current node when mouseover
    highlightMouseoverNode: {
      type: Boolean,
      default: false
    },
    // highlight current node when selected
    highlightSelectedNode: {
      type: Boolean,
      default: true
    },
    // collapsed control
    collapsedOnClickBrackets: {
      type: Boolean,
      default: true
    },
    // custom formatter for values
    customValueFormatter: {
      type: Function,
      default: null
    },
    // 是否展示标记信息
    showSignComment: {
      type: Boolean,
      default: true
    },
    // 标记键值对
    signKeys: {
      type: Object,
      default: null
    },
    // 标记键值对
    addKeys: {
      type: Object,
      default: null
    },

    /* outer props */

    /* inner props */
    // 当前树的父级数据
    parentData: {
      type: [String, Number, Boolean, Array, Object],
      default: null
    },
    // 当前树的深度, 以根节点作为0开始, 所以第一层树的深度为1, 递归逐次递增
    currentDeep: {
      type: Number,
      default: 1
    },
    // 当前树的数据 data 为数组时 currentKey 表示索引, 为对象时表示键名
    currentKey: {
      type: [Number, String],
      default: ""
    },
    // 父组件键值名
    parentKey: {
      type: [Number, String],
      default: ""
    },
    // 父组件类型
    parentType: {
      type: String,
      default: "null"
    }
    /* outer props */
  },
  data() {
    return {
      visible: this.currentDeep <= this.deep,
      isMouseover: false,
      currentCheckboxVal: Array.isArray(this.value)
        ? this.value.includes(this.path)
        : false
    };
  },
  computed: {
    model: {
      get() {
        const defaultVal =
          this.selectableType === "multiple"
            ? []
            : this.selectableType === "single"
            ? ""
            : null;
        return this.value || defaultVal;
      },
      set(val) {
        this.$emit("input", val);
      }
    },

    currentData() {
      let currentData = this.data;
      if (this.currentDeep === 1 && this.addKeys) {
        for (let key in this.addKeys) {
          set(currentData, key, this.addKeys[key]);
        }
      }
      if (this.data === "undefined") return "undefined";
      else return currentData;
    },
    // 获取当前 data 中最后一项的 key 或 索引, 便于界面判断是否添加 ","
    lastKey() {
      if (Array.isArray(this.parentData)) {
        return this.parentData.length - 1;
      } else if (this.isObject(this.parentData)) {
        let arr = Object.keys(this.parentData);
        return arr[arr.length - 1];
      }
      return "";
    },

    // 是否不是最后一项
    notLastKey() {
      return this.currentKey !== this.lastKey;
    },

    // 当前的树是否支持选中功能
    selectable() {
      return (
        this.pathSelectable(this.path, this.currentData) &&
        (this.isMultiple || this.isSingle)
      );
    },

    // 多选模式
    isMultiple() {
      return this.selectableType === "multiple";
    },

    // 单选模式
    isSingle() {
      return this.selectableType === "single";
    },

    isSelected() {
      if (this.isMultiple) {
        return this.model.includes(this.path);
      } else if (this.isSingle) {
        return this.model === this.path;
      } else {
        return false;
      }
    },

    prettyKey() {
      return this.showDoubleQuotes ? `"${this.currentKey}"` : this.currentKey;
    },
    // 上一级如果为标记键值对则向下传递，否则传递当前键值名称
    prevKey() {
      if (this.parentKey && this.signKeys) {
        return this.signKeys[this.currentKey] !== undefined
          ? this.currentKey
          : this.signKeys[this.parentKey] !== undefined
          ? this.parentKey
          : this.path;
      } else {
        return this.currentKey;
      }
    },
    // 上一级数据类型
    prevType() {
      return getDataType(this.currentData);
    },
    // 当前键值对为标记键值
    signClass() {
      let signClass = "";
      if (
        this.signKeys &&
        (this.signKeys[this.currentKey] !== undefined ||
          this.signKeys[this.parentKey] !== undefined ||
          this.signKeys[`${this.path}`] !== undefined)
      ) {
        signClass = "vjs-key__sign";
      }
      if (
        this.addKeys &&
        (this.addKeys[this.currentKey] !== undefined ||
          this.addKeys[this.parentKey] !== undefined ||
          this.addKeys[`${this.path}`] !== undefined)
      ) {
        signClass = "vjs-key__notexists";
      }
      return `${signClass}`;
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
    propsError() {
      const error =
        this.selectableType &&
        !this.selectOnClickNode &&
        !this.showSelectController;
      return error
        ? "When selectableType is not null, selectOnClickNode and showSelectController cannot be false at the same time, because this will cause the selection to fail."
        : "";
    }
  },
  watch: {
    deep(newVal) {
      this.visible = this.currentDeep <= newVal;
    },
    propsError: {
      handler(message) {
        if (message) {
          throw new Error(`[vue-json-pretty-sign] ${message}`);
        }
      },
      immediate: true
    }
  },
  methods: {
    isUndefined(item) {
      // 键值名为undefined时候标记
      if (getDataType(item) === "undefined")
        return "undefined-{03C69C01-9EA8-4E19-98B5-539AFED96E94}";
      else return item;
    },
    handleValueChange(emitType) {
      if (this.isMultiple && (emitType === "checkbox" || emitType === "tree")) {
        // handle multiple
        const index = this.model.findIndex(item => item === this.path);
        const oldVal = [...this.model];
        if (index !== -1) {
          this.model.splice(index, 1);
        } else {
          this.model.push(this.path);
        }

        if (emitType !== "checkbox") {
          this.currentCheckboxVal = !this.currentCheckboxVal;
        }
        this.$emit("change", this.model, oldVal);
      } else if (
        this.isSingle &&
        (emitType === "radio" || emitType === "tree")
      ) {
        // handle single
        if (this.model !== this.path) {
          const oldVal = this.model;
          const newVal = this.path;
          this.model = newVal;
          this.$emit("change", newVal, oldVal);
        }
      }
    },

    /**
     * emit click event
     * @param  {string} emitType tree/checkbox/radio
     */
    handleClick(e) {
      // Event can not be stopPropagation, because user may be listening the click event.
      // So use _uid to simulated.
      if (e._uid && e._uid !== this._uid) return;
      e._uid = this._uid;

      this.$emit("click", this.path, this.currentData);
      if (this.selectable && this.selectOnClickNode) {
        this.handleValueChange("tree");
      }
    },

    // handle children's click, and propagation
    handleItemClick(path, data) {
      this.$emit("click", path, data);
    },

    // handle children's change, and propagation
    handleItemChange(newVal, oldVal) {
      // 可选的时候change事件才有意义
      if (this.selectable) {
        this.$emit("change", newVal, oldVal);
      }
    },

    handleMouseover() {
      // 可选择的树|普通展示树, 都支持mouseover
      this.highlightMouseoverNode &&
        (this.selectable || this.selectableType === "") &&
        (this.isMouseover = true);
    },

    handleMouseout() {
      this.highlightMouseoverNode &&
        (this.selectable || this.selectableType === "") &&
        (this.isMouseover = false);
    },

    // 是否对象
    isObject(value) {
      return getDataType(value) === "object";
    },

    getChildPath(key) {
      return this.path !== ""
        ? this.path +
            (Array.isArray(this.currentData)
              ? `[${key}]`
              : key.includes(".")
              ? `["${key}"]`
              : `.${key}`)
        : Array.isArray(this.currentData)
        ? `${key}`
        : key.includes(".")
        ? `"${key}"`
        : `${key}`;
    }
  },
  // 捕获一个来自子组件的错误
  //    因为是递归组件，因此错误只对外暴露一次，子组件的错误不再对外传递
  errorCaptured() {
    return false;
  }
};
</script>
