---
title: dtm分布式事务——解决导入RM数据库失败
tags:
  - 分布式事务
id: '206'
categories:
  - 开发层
date: 2022-3-1 22:44:29
---

# 问题背景

dtm中的子事务屏障，需要与数据库交互，xa事务模式，也需要与数据库交互。

根据部署文档RM数据库的导入教程，出现以下报错

```
Specified key was too long; max key length is 1000 bytes
```

# 解决

创建表的mysql脚本没有指定`存储引擎`,当自己的环境默认引擎是`MyIsam`的时候将会出现这个问题,索引长度过长

在创建脚本后指定引擎即可解决,或者可以配置服务器的默认引擎为`Innodb`

```
create table if not exists dtm_busi.user_account(
  id int(11) PRIMARY KEY AUTO_INCREMENT,
  user_id int(11) UNIQUE,
  balance DECIMAL(10, 2) not null default '0',
  trading_balance DECIMAL(10, 2) not null default '0',
  create_time datetime DEFAULT now(),
  update_time datetime DEFAULT now(),
  key(create_time),
  key(update_time)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
```
