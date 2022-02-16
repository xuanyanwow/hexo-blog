---
title: 转 - Android下一次OOM调试过程
tags:
  - Android
id: '249'
categories:
  - - 前端
date: 2020-03-18 17:48:35
---

# 分享原因

*   调试过程记录非常详细，是很好排查思路学习和锻炼的学习资料
*   Android基于Linux内核，Linux对线程、进程、描述符等限制参数的功能
*   技术文章

可以导致OOM的原因有以下几种：

*   文件描述符(fd)数目超限，即proc/pid/fd下文件数目突破/proc/pid/limits中的限制。可能的发生场景有：
*   短时间内大量请求导致socket的fd数激增，大量（重复）打开文件等
*   线程数超限，即proc/pid/status中记录的线程数（threads项）突破/proc/sys/kernel/threads-max中规定的最大线程数。可能的发生场景有：
*   app内多线程使用不合理，如多个不共享线程池的OKhttpclient等等
*   传统的java堆内存超限，即申请堆内存大小超过了 Runtime.getRuntime().maxMemory()
*   （低概率）32为系统进程逻辑空间被占满导致OOM.

# 原文地址

https://blog.csdn.net/zhizhuodewo6/article/details/81486384