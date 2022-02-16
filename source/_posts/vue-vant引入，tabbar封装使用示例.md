---
title: Vue vant引入，tabbar封装使用示例
tags:
  - Vue
  - 常见问题
id: '236'
categories:
  - - 前端
  - - 常见问题
date: 2020-01-14 16:57:32
---

# Vant

tabbar使用教程：https://youzan.github.io/vant/#/zh-CN/tabbar 关于基本的安装、引入组件就不再详细讲解，请自行按照文档安装章节进行。

## 封装Tabbar

在不同页面显示tabbar，如果tabbar修改，则需要变动多个页面 封装成组件后，统一引用组件，修改tabbar则只需要改动组件文件

### 封装后引用代码

active代表要高亮组件中第几个图标，必须为数值

```markup
<template>
  <!-- 这里显示其他内容 -->
  <TabbarHtml v-bind:active=2 />
</template>

<script>
import TabbarHtml from '@/components/Tabbar.vue'

export default {
  components: {
    TabbarHtml
  }
}

</script>
```

### tabbar组件代码

*   使用tabbarTempValue值来监听，使用active值来接收。这是为了防止props为单向数据绑定，在组件内改变值后会产生报错，父页面无法接收
*   onChange事件监听并路由跳转 这里使用的是vue-router

```markup
<template>
  <div class="tabbar">
    <van-tabbar v-model="tabbarTempValue" @change="onChange">
      <van-tabbar-item icon="home-o" url="/Home">标签</van-tabbar-item>
      <van-tabbar-item icon="search">标签</van-tabbar-item>
      <van-tabbar-item icon="friends-o">标签</van-tabbar-item>
      <van-tabbar-item icon="setting-o">标签</van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script>

import Vue from 'vue';
import { Tabbar, TabbarItem } from 'vant';
import { Icon } from 'vant';
import { Notify } from 'vant';

Vue.use(Notify);
Vue.use(Tabbar).use(TabbarItem);
Vue.use(Icon);

export default {
  props: {
    active: Number
  },
  data() {
    return {
      tabbarTempValue: this.active
    }
  },
  methods: {
    onChange(index) {
      const routerArray = [
        "/",
        "/Home",
        "/Home",
        "/Home"
      ];
      this.$router.push(routerArray[index])
    }
  }
}
</script>
```