---
title: VMware Workstation客户端 Centos系统 中文乱码 UTF-8字符无法正常显示
tags:
  - VMware
  - 常见问题
  - 虚拟机
  - 计算机基础
id: '182'
categories:
  - - Linux
  - - 常见问题
date: 2019-09-19 12:49:15
---

# 问题由来

发生该问题的时间比较长了，之前没有第一时间整理记录。依稀记得是因为系统重装之后，Vmware软件重新安装，然后导入以前的虚拟机配置文件，就出现了中文乱码的问题。 从百度上查到的各种资料，都是说语言包和配置的问题，需要重新安装、修改配置等等步骤，下面将记录我的尝试步骤和解决问题的方案。

# 尝试步骤

## 系统中文语言包

首先运行以下命令，查看当前系统的语言包中是否有中文语言包

```shell
locale -a grep "zh_CN"
```

![](https://www.siammm.cn/wp-content/uploads/2019/09/0c21d6a79d1311c6e7d0f128f16dc530.png) 如果没有安装那么就先安装语言包，可以执行以下命令（不同系统可能有一些差异 原理一致）

```shell
yum groupinstall "fonts" -y
```

安装好了之后就是要切换系统使用语言的配置

## 切换系统语言配置

先查看一下本机当前使用的配置

```shell
# locale

LANG=zh_CN.utf8
LC_CTYPE="zh_CN.utf8"
LC_NUMERIC="zh_CN.utf8"
LC_TIME="zh_CN.utf8"
LC_COLLATE="zh_CN.utf8"
LC_MONETARY="zh_CN.utf8"
LC_MESSAGES="zh_CN.utf8"
LC_PAPER="zh_CN.utf8"
LC_NAME="zh_CN.utf8"
LC_ADDRESS="zh_CN.utf8"
LC_TELEPHONE="zh_CN.utf8"
LC_MEASUREMENT="zh_CN.utf8"
LC_IDENTIFICATION="zh_CN.utf8"
LC_ALL=
```

可以看到我这里的虚拟机已经是使用了zh\_CN的配置，所以该方法不是我这个问题导致的。

> 如果你这里的配置是en的语言，可以尝试以下步骤进行配置切换尝试

```shell
# vim /etc/locale.conf

LANG="zh_CN"

# source   /etc/locale.conf

```

测试是否切换成功 可以输出日期

```shell
# date
```

## 重装系统

在以上语言包的切换方案不行之后，我还根据还几篇文章 不同的方法安装语言包和切换，都是不行的。 我从网上下载了新的镜像来安装虚拟机，开启之后也是一样的中文乱码。 那么基本可以排查是系统层面导致的问题。 我把目光转到了VM软件上来

## 尝试其他shell工具

我使用了putty这个开源简单的工具，然后就得到了正常的中文结果...

# 结论

应该是VM软件 在重装系统过程中遗留了一些配置文件，然后新安装的软件又版本等问题不一致，导致丢失，中文乱码吧。