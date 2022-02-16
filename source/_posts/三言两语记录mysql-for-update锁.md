---
title: 三言两语记录mysql for update锁
tags:
  - mysql
  - PHP
id: '291'
categories:
  - - 数据库
date: 2020-11-25 22:48:53
---

# FOR UPDATE

中文直译的意思是：用于更新。 理解：这次查询的数据我要用于更新操作，所以麻烦Mysql帮我加锁，其他进程在我更新完成之前不能发起for update请求（可以发起普通select请求， 用于前端展示） 用途：防止高并发情况下，比如用户连续快速点击两次购买，导致商品数量超卖 为负数等情况

# 必要条件

*   mysql innodb引擎
*   在事务中启用for update（直到commit 或者rollback 此次更新操作结束 释放锁）
*   mysql暂无for update nowait 需要封装，增加控制超时时间的逻辑，这样子伪nowait
*   select命中索引或者主键，则为行锁，没有命中则为表锁（需要注意 避免影响业务）

# 测试步骤

1.一个连接A 发起事务，执行select for update

```sql
START TRANSACTION;
select * from test where  id = "Auth" for update;
```

2.另一个连接B 发起普通select请求，正常返回结果 3.连接B 发起select for update请求，由于第一个步骤的事务还没有结束，所以不能获取，会一直堵塞，直到超时 或者锁被释放后返回 4.没有nowait 可以封装如下逻辑

```php
function testNowait(){
    // 执行sql  超时时间更改为0.5s
    // 执行for update
    // 0.5s后则返回失败（默认可能长达1分钟）
    // 恢复为默认的超时时间，避免影响其他sql执行
}
```