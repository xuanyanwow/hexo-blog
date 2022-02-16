---
title: mysql从5.7迁移表结构到5.5报错 near '(0) NULL DEFAULT NULL'
tags:
  - mysql
id: '234'
categories:
  - 数据库
date: 2020-01-09 15:31:17
---

# 问题由来

问题如标题所示，在开发过程的时候，需要创建一张表，从另一个环境导出的表结构sql文件，在我电脑上导入，遇到该报错

```
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '(0) NULL DEFAULT NULL'
```

报错的那一行内容为

```
`refund_success_time` datetime(0) NULL DEFAULT NULL COMMENT '退款成功时间',
```

宣言博客 Siam 原文链接：[https://www.siammm.cn/archives/234](https://www.siammm.cn/archives/234 "https://www.siammm.cn/archives/234")

# 排查思路

将导出的sql导入回原来的环境中（另开一个数据库），测试结果：`正常`。 那么sql语句一般是正常没问题的， 一般是环境差异导致的，如（版本不同） 原来的表创建过程是使用软件可视化的，datetime长度这里没有填写，默认是为0，所以首先是对这个的不理解 从这里去找了资料，发现对datetime长度的说明资料很少，但还是有一个百度回答说到了（虽然不够准确） 原文为：

> 在navicat里面datetime的长度好像指的是秒后面的小数点位数，可以设置为0-6位

不准确的地方有以下

*   并不是在navicat这个软件里，而是mysql数据库中
*   在mysql数据库中也会有不同的版本差异（导致这篇文章遇到问题的原因）
*   所用词“好像指的是”，代表回答该问题的前辈并没有找过官方文献、测试

# 官方文献

宣言为了测试该问题，并准确定位和分析，找到了mysql官方的文献，原文为：

> 11.2.7 Fractional Seconds in Time Values MySQL 5.6 has fractional seconds support for TIME, DATETIME, and TIMESTAMP values, with up to microseconds (6 digits) precision: To define a column that includes a fractional seconds part, use the syntax type\_name(fsp), where type\_name is TIME, DATETIME, or TIMESTAMP, and fsp is the fractional seconds precision. For example:

重点为第一句，mysql在5.6后支持了小数秒，精度高达微秒（6位）

# 解决该问题

解决该问题（或者说从根源上避免遇到此类问题），应该保证开发环境的一致，同一项目的所有开发人员都应该保持所有环境的版本号一致（最好精确到小版本） 如果只是为了临时在mysql5.5完成测试，并且确认业务程序不需要使用到时间的小数秒，可以将sql文件中的长度设置删除，然后导入

```
datetime(0) NULL DEFAULT NULL
改为 datetime NULL DEFAULT NULL
```