---
title: Chrome缓存锁，php非堵塞文件锁无效的问题详解追查
tags:
  - PHP
  - 前端
id: '123'
categories:
  - - 前端
date: 2019-08-10 00:03:43
---

# 写在前面

为什么写下这篇文章 在编写PHP文件锁的时候，非堵塞模式 `LOCK_NB` 加了却没有效果 预期反应：当前面有请求在执行了，后续的请求马上返回客户端：拿不到锁，执行失败。 现实状况：假设业务逻辑需要执行5s，第一个请求发送后，马上有第二个请求发送，第二个请求会等待第一个请求结束，然后再执行自己本次请求，总共耗时10s

# 简单说说php写下的锁

为了后面更好地讲解和理解，简单带上php文件锁的几行代码

```php
<?php
$p_file = "index.lock";
$o_file = fopen($p_file, 'w+');

// 非堵塞模型 拿不到锁马上返回false
$lock = flock($o_file, LOCK_EX  LOCK_NB);
if (!$lock) {
  var_dump('被锁了');
}else {
  sleep(5);
  echo "完成";
}
```

# 追寻之路

## 最直观地搜索

百度`PHP LOCK_NB 无效` 找文章 找到一篇文章如下：

* * *

![](https://www.siammm.cn/wp-content/uploads/2019/08/d222dccd2db7bba01db692e354dfb6a9.png)

* * *

引用

> 如果你是在本机测试，你要分两个不同的浏览器测试。 例如一个chrome，一个firefox， chrome先执行，firefox再执行，就可以看到效果了，如果你用同一个浏览器会等待的。

确实，我们可以`使用两个浏览器`来执行，就可以达到预期想要的非堵塞效果。但是原因呢，几乎没有文章讲解。 `宣言不会死心`，想要坚持找到合适的答案。

## 猜测有什么会影响

根据不够成熟的经验，简单猜测

*   服务端导致：可能由于系统等等特殊场景，会导致非堵塞失效。
*   前端导致：可能由于浏览器处理问题，导致请求堵塞。

这两个排查顺序，首先我检查了PHP手册和其他站点的文章，`确保函数和参数的使用正确`，也没有在文档中发现比较特别的注意事项。 在Windows和Linux系统上都出现过此问题。所以也排除windows系统不支持的可能。 接下来就只能进行前端的排查，通过开发者面板——>网络请求——>尝试分析`请求头/响应头`等信息 一开始的猜想是：要么是客户端请求头，要么是服务端响应头，会有一个标识在等待或者转发等等 分析看到大概如下的情况 （请求里的请求头信息很少，很正常，所以看到外面的状态） ![](https://www.siammm.cn/wp-content/uploads/2019/08/267b075db82deb9cc048dcc76b071def.png) 此时显示的是Pending 在等待状态。

## 浏览器Pending状态

想要知道该状态会在什么情况下产生，搜索这方面的文章。。 看到的可能如下：

*   建议你把浏览器先卸载,用360安全卫士清理系统后重装一下。
*   #现象Chrome打开任何网页都显示“喔唷崩溃啦”。#尝试关于页无法进入，没法升级浏览器；设置页无法进入，没法清除浏览数据或重置浏览器；

等等垃圾答复（机器人或者刷积分的）

#### 但是，也有精品的文章

该文章属于FEX百度团队 《关于请求被挂起页面加载缓慢问题的追查》 在大概2014年的时候，百度团队追查前端`偶发`的缓慢请求(可能一两分钟)的一篇技术文章

> 涉及面非常专业，排查思路非常值得学习！赞！！

也是从中得到本篇文章想要的知识点 下面会陆续引用文章中的一小段内容

## 神秘的CACHE LOCK

稳重提到，在Stackoverflow上找到一个[问题](https://stackoverflow.com/questions/27513994/chrome-stalls-when-making-multiple-requests-to-same-resource "问题")，跟FEX面临的问题有些类似点：

*   偶发，并不是必然出现的。这里我们的问题也是偶发，很难复现，需要反复刷。
*   也是请求被Pending了很久，从请求的时间线来看，体现在Stalled上。

在问题中，有提供以下报错 ERR\_CACHE\_LOCK\_TIMEOUT 20秒

```
t=33627 [st= 5] HTTP_CACHE_ADD_TO_ENTRY [dt=20001] –> net_error = -409 (ERR_CACHE_LOCK_TIMEOUT)
```

同时还有一段这样子的描述

> The error message refers to a patch added to Chrome six months ago (https://codereview.chromium.org/345643003), which implements a 20-second timeout when the same resource is requested multiple times. In fact, one of the bugs the patch tries to fix (bug number 46104) refers to a similar situation, and the patch is meant to reduce the time spent waiting.

核心句子： 这是Chormeium发布的一个补丁：它在多次请求同一资源时实现20秒的超时。 也从中得知了`浏览器有缓存锁`的知识。

> 浏览器对一个资源发起请求前，会先检查本地缓存，此时这个请求对该资源对应的缓存的读写是独占的。此时后续的请求，在请求这个资源的时候，就需要等待拿锁。（在上面这个补丁发布之前，会无限等待，补丁是让等待最多20秒）

那么如何让浏览器不使用缓存锁呢

*   对请求加个时间戳或者参数等让请求变得唯一
*   或者服务器响应头设置为无缓存

嗯，先简单尝试一下，在Chrome中，打开文章一开始的php脚本，但是此时有一个请求带上参数 `index.php?t=1` `index.php` 顺利实现预期效果：一个执行5秒，一个马上返回拿锁失败的消息。

## 测试CACHE LOCK的超时时间

再简单修改sleep的时间为30，目的是测试上文补丁中说的，最长等待时间是20s。 第二个请求出现以下状况，完全符合。20秒后，拿不到该资源的本地缓存锁，则发送请求执行。 ![](https://www.siammm.cn/wp-content/uploads/2019/08/62cd6392c6874d3bd5672f0112f1de8f.png)

# 总结

*   Chrome浏览器有缓存锁概念（其他浏览器还未测试）
*   PHP LOCK\_NB没有失效，表面现象是因为前端特性导致的
*   Chrome的缓存锁超时时间是20s
*   可以通过不同参数或者由服务端响应头来控制浏览器不缓存文件,也就不会去检查拿本地缓存锁。