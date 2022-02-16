---
title: ApiCloudApp开发-$api is undefined
tags:
  - Android
  - 前端
id: '255'
categories:
  - - 前端
date: 2020-03-24 16:44:58
---

# 写在前面

在apicloud文档中 关于数据储存的部分，可以支持我们h5开发常用到的`localStore`模块 使用过程中遇到报错提示`$api is undefined` 记录遇到该问题及其解决方案

# 文档

文档demo如下

```javascript
$api.setStorage('name','Tom');
```

但是使用不行。 这里的`$api`与之前我使用的api.xxx不同。所以猜测有没有可能是文档编写错误，直接调用`api.setStorage()` 也是失败的。 原文博客https://www.siammm.cn 原文地址https://www.siammm.cn/archives/255

# 解决问题

直到后面使用到apicloud的前端框架，才知道这个问题是怎么导致的。

*   api对象是全局基础对象，在ApiCloud启动的时候初始化并注入到js的。
*   $api 是前端框架提供的一个对象，默认是没有引入的

出现这个问题主要是因为我们没有太多的精力和时间先完整的学习文档再进行开发，公司任务比较繁重，经常跳着观看，就弄混淆两个对象了。 所以我们只需要引入前端框架的js代码即可。

> 使用APICloud前端框架需引入api.js和api.css文件。api.js、api.css

开源地址：https://github.com/apicloudcom/apicloud-js-framework