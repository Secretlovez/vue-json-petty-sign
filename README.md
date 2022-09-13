# vue-json-pretty-sign

本项目基于 vue-json-pretty-sign@1.7.1 版本开发，主要实现标记键值与展示标记键值注释的功能。
注意如果父级为标记键值,则其子级将全部被标记.

一个将 JSON 字符串渲染成树形结构的 Vue2.x 组件

- 基础功能: JSON 的美化
- 高级功能: JSON 数据对应层级数据的获取

## 添加参数

| 属性            | 级别 | 说明               | 类型    | 默认值 |
| --------------- | ---- | ------------------ | ------- | ------ |
| showSignComment | 基础 | 是否标记的键值注释 | boolean | true   |
| signKeys        | 高级 | 标记的键值对       | Object  | null   |

## 链接

- 当前项目 demo： [Demo](https://secretlovez.github.io/vue-json-petty-sign/)
- vue-json-pretty-sign： [Github](https://github.com/leezng/vue-json-pretty-sign)

## 安装

请下载后打包编译

## 快速开始

```html
<template>
  <div>
    ...
    <vue-json-pretty-sign
      :path="'res'"
      :data="{ key: 'value' }"
      @click="handleClick"
    >
    </vue-json-pretty-sign>
  </div>
</template>
```

```js
// 更多用法请见demo页面
import VueJsonPretty from "vue-json-pretty-sign";

export default {
  components: {
    VueJsonPretty
  }
};
```

## Props

- 若仅使用基础功能(JSON 美化)，只需关注功能级别为 `基础` 的属性。
- 若使用高级功能(选择数据)，你可以同时使用 `基础` 与 `高级` 的属性。

| 属性                   | 级别 | 说明                                      | 类型                                          | 默认值   |
| ---------------------- | ---- | ----------------------------------------- | --------------------------------------------- | -------- |
| data                   | 基础 | 待美化的源数据，注意不是 `JSON` 字符串    | `JSON` 对象                                   | -        |
| deep                   | 基础 | 数据深度, 大于该深度的数据将不被展开      | number                                        | Infinity |
| showLength             | 基础 | 是否在数据线闭合的时候展示长度            | boolean                                       | false    |
| showLine               | 基础 | 是否显示缩紧标识线                        | boolean                                       | true     |
| showDoubleQuotes       | 基础 | 是否展示 key 名的双引号                   | boolean                                       | true     |
| showDoubleQuotes       | 基础 | 是否展示 key 名的双引号                   | boolean                                       | true     |
| showSignComment        | 基础 | 是否标记的键值注释                        | boolean                                       | true     |
| signKeys               | 高级 | 标记的键值对                              | Object                                        | null     |
| highlightMouseoverNode | 基础 | 是否在 mouseover 的时候高亮               | boolean                                       | false    |
| v-model                | 高级 | 双向绑定选中的数据层级                    | string, array                                 | -, []    |
| path                   | 高级 | 定义最顶层数据层级                        | string                                        | root     |
| pathChecked            | 高级 | 定义哪些数据层级是已被选中的              | array                                         | []       |
| pathSelectable         | 高级 | 定义哪些数据层级是可以被选中的            | Function(itemPath, itemData)                  | -        |
| selectableType         | 高级 | 定义组件支持的选中方式，默认无选中功能    | enum: -, multiple, single                     | -        |
| showSelectController   | 高级 | 是否展示选择控制器                        | boolean                                       | false    |
| selectOnClickNode      | 高级 | 是否在点击节点的时候触发 v-model 双向绑定 | boolean                                       | true     |
| highlightSelectedNode  | 高级 | 是否高亮已选项                            | boolean                                       | true     |
| customValueFormatter   | 高级 | 可以进行值的自定义渲染                    | Function(data, key, parent, defaultFormatted) | -        |

## Events

| 事件名 | 说明                                   | 回调参数         |
| ------ | -------------------------------------- | ---------------- |
| click  | 点击某一个数据层级时触发的事件         | (path, data)     |
| change | v-model 改变的事件(仅在选择模式下可用) | (newVal, oldVal) |
