---
title: docker端口映射失败排查
tags:
  - 容器
id: '128'
categories:
  - 开发层
date: 2022-2-22 14:19:45
---

# 前言

win10下，docker启动了apisix dashboard容器，浏览器和CURL命令获取容器服务都`失败`, `curl http://127.0.0.1:9000`

## 排查思路

- 检查容器是否启动正常
- 检查容器端口配置映射是否正常
- 检查宿主机端口是否开启正常
- 检查宿主机端口占用进程

## 检查容器

在docker面板中可以清晰看到容器启动正常、配置内容、启动log success

## 检查宿主机端口是否开启正常

``` sh
telnet 127.0.0.1 9000
```

正常端口

## 检查宿主机端口占用进程

``` sh
# netstat -aon|findstr "9000"

  TCP    0.0.0.0:9000           0.0.0.0:0              LISTENING       5428
  TCP    127.0.0.1:9000         0.0.0.0:0              LISTENING       4676
  TCP    [::]:9000              [::]:0                 LISTENING       5428
  TCP    [::1]:9000             [::]:0                 LISTENING       11388
```

逐个分析进程ID
``` sh
# tasklist|findstr "5428"

com.docker.backend.exe        5428 Console                    1     35,456 K
```
查到是进程id 4676 别的tcp程序也监听了该端口，所以http拒绝服务，`端口冲突还能启动的问题 在后面文章做测试 结论`


杀死进程
``` sh
# taskkill -PID 4676 -F
```

