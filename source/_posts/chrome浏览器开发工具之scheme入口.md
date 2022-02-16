---
title: Chrome浏览器开发工具之scheme入口
tags:
  - 前端
id: '128'
categories:
  - 前端
date: 2019-08-14 09:20:05
---

# 写在前面

没啥 ，就是自己玩着玩着发现还有这么多没见过，记录一下

### 实验室

chrome://flags/ 一些实验性的功能，可能在未来保留成正式或者被移除，同时也可能会产生不可预期的影响，开发者鼓捣记得备份数据！

> http://blog.sina.com.cn/s/blog\_1806ff8820102xr22.html

知乎文章

> https://www.zhihu.com/question/27380104

### 网络调试

chrome://net-internals/#events net-internals是一套工具集合，用于帮助诊断网络请求与访问方面的问题，它通过监听和搜集 DNS，Sockets，SPDY，Caches等事件与数据来向开发者反馈各种网络请求的过程、状态以及可能产生影响的因素。

> https://jingyan.baidu.com/article/2c8c281d8d2f8f0008252a27.html

### 书签管理

chrome://bookmarks

### 显示浏览器所使用磁盘空间配额的情况

chrome://quota-internals

### 历史记录

chrome://history 跟ctrl+h一样

### 扩展管理

chrome://extensions/shortcuts

### 设置管理

chrome://settings

### 显示同步状态

chrome://sync-internals 查看所有命令（因为会更新的嘛） chrome://about/