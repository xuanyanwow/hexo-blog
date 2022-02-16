---
title: vue vant cdn引入方式，组件使用样式错乱
tags:
  - Vant
  - Vue
  - 前端
  - 常见问题
id: '265'
categories:
  - - 前端
  - - 常见问题
date: 2020-05-13 16:24:29
---

# 问题复现

使用的是vant cdn方式引入框架，而非vue-cli 在使用一些组件，如宫格，复制文档的示例代码， 但是并不能正常运行，效果如下图。

```markup
<div id="app">
    <van-grid>
      <van-grid-item icon="photo-o" text="文字" />
      <van-grid-item icon="photo-o" text="文字" />
      <van-grid-item icon="photo-o" text="文字" />
      <van-grid-item icon="photo-o" text="文字" />
    </van-grid>
</div>
```

效果图： ![](https://www.siammm.cn/wp-content/uploads/2020/05/wp_editor_md_e85abb5e9fdb8112bb42e0dad032c468.jpg)

# 出现问题的原因

在经过一番寻找资料后，最终在github的issue中找到关于该问题的答复：

> Vue 不支持在 HTML 里直接使用自闭合标签，Vue 官方文档里有说明的，请使用完整的标签

在vue的文档中找到以下相关描述

> 自闭合组件表示它们不仅没有内容，而且刻意没有内容。其不同之处就好像书上的一页白纸对比贴有“本页有意留白”标签的白纸。而且没有了额外的闭合标签，你的代码也更简洁。  
> 不幸的是，HTML 并不支持自闭合的自定义元素——只有官方的“空”元素。所以上述策略仅适用于进入 DOM 之前 Vue 的模板编译器能够触达的地方，然后再产出符合 DOM 规范的 HTML。

html规范中对于自闭和标签有强制规范，用户不可自定义新增， 所以我们在示例代码中的·van-grid-item·标签不能正常工作。 在vue-cli中能正常工作的原因是，我们的xxx.vue文件会经过vue编译器，编译成规范的html代码，然后再运行。 原文博客链接：https://www.siammm.cn/archives/265

# 问题解决

```markup
<div id="app">
    <van-grid>
      <van-grid-item icon="photo-o" text="文字"></van-grid-item>
      <van-grid-item icon="photo-o" text="文字"></van-grid-item>
      <van-grid-item icon="photo-o" text="文字"></van-grid-item>
      <van-grid-item icon="photo-o" text="文字"></van-grid-item>
    </van-grid>
</div>
```