---
title: 'Mysql误删,恢复数据,binlog闪回,宝塔面板'
tags:
  - mysql
id: '174'
categories:
  - 数据库
date: 2019-09-14 21:51:25
---

# 写在前面

DBA或开发人员，有时会误删或者误更新数据 你是否有`删库`经历？`删库`是否真的无解 如果是线上环境并且影响较大，就需要能快速回滚。 传统恢复方法是利用备份重搭实例，再应用去除错误sql后的binlog来恢复数据。 此法费时费力，甚至需要停机维护，并不适合快速回滚。 也有团队利用LVM快照来缩短恢复时间，但快照的缺点是会影响mysql的性能。 MySQL闪回(flashback)利用binlog直接进行回滚，能快速恢复且不用停机。 本文将简单进行`mysql根据binlog闪回数据`的实战测试

# 基础知识准备

binlog是二进制日志文件，用来记录Mysql内部对数据库的改动（只记录对数据的修改操作），主要用于数据库的主从复制以及增量恢复。 当我们搭建mysql主从复制的时候，两个实例之间也是通过binlog来完成数据的备份同步。 所以有这种根据binlog得到执行sql语句、闪回sql语句，我们只需要利用根据分析binlog，然后就可以找到准确的数据改动sql，并得到闪回sql，检查无误后执行就可以恢复数据了

# 准备工作

我们采用`binlog2sql工具`来分析，由上海美团DBA团队出品 使用的是python语言，所以我们需要提前安装好python语言 我使用的是宝塔面板，宝塔面板已经内置安装了python，所以直接开始安装更三十就好了

## 安装binlog2sql工具

```shell
cd /www/server
```

```shell
git clone https://github.com/danfengcao/binlog2sql.git && cd binlog2sql
```

```shell
pip install -r requirements.txt
```

## 开启mysql server主从配置

在mysql配置文件中填写以下内容

```
[mysqld]
server_id = 1
log_bin = /var/log/mysql/mysql-bin.log
max_binlog_size = 1G
binlog_format = row
binlog_row_image = full
```

> 在宝塔面板中，有几个参数已经是开启的，我们无需修改，看以下内容

在软件管理 mysql 配置修改中 打开配置文件 在`33行`开始，有几个参数已经填写了，我们主要是修改binlog\_format和row\_image binlog文件储存位置默认是 /www/server/data 无需修改也可以

```
log-bin=mysql-bin
binlog_format = row
binlog_row_image = full
server-id = 1
```

# 开始实战

创建测试库、测试表、插入测试数据 然后执行delete 不带where条件 全部删除

```
mysql> select * from siamwp_links;
Empty set (0.01 sec)
```

接下来就是重点了，我们使用工具分析

## 查看当前的binlog文件名

```
mysql> show master status;
```

得到类似

```
+------------------+----------+--------------+------------------+-------------------+
 File              Position  Binlog_Do_DB  Binlog_Ignore_DB  Executed_Gtid_Set 
+------------------+----------+--------------+------------------+-------------------+
 mysql-bin.000006    613171                                                    
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

我们记住`mysql-bin.000006`

## 打开binlog2sql工具分析

进入我们安装后的binlog2sql工具目录

```
cd /www/server/binlog2sql/binlog2sql
```

```
ll
```

可以看到这里面有py脚本

## 得到历史sql语句

```
python binlog2sql.py 
-h127.0.0.1 
-P3306 
-uroot 
-p'密码' 
-d数据库名 
刚刚查找的文件名
--start-file='mysql-bin.000006'   
后面的参数可以不带 筛选时间 
--start-datetime '2019-09-14 22:05:30' 
--stop-datetime '2019-09-14 22:05:45'
```

总的可能是这样子的语句 上面是为了讲解参数意义

```
python binlog2sql.py  -h127.0.0.1 -P3306 -uroot -p'密码' -dwww_siammm_cn --start-file='mysql-bin.000006' --start-datetime '2019-09-14 22:05:30' --stop-datetime '2019-09-14 22:05:45'
```

得到的大概是这样子的记录（这里为了演示方便 用了时间筛选 准确得到只有删除数据的log 正常情况下会有很多 需要耐心查找） ![](https://www.siammm.cn/wp-content/uploads/2019/09/60fbc15b44570acf12e8e0018fb5cf40.png) 有三条语句 然后每一条语句的最后面还有这样子一段注释

```
#start 590075 end 590633 time 2019-09-14 22:05:35
```

这代表的是在log文件中的起始位置和结束位置

## 闪回sql语句

我们有了起始位置和结束位置，就可以利用工具，得到这一部分变化的闪回sql了 前面的大部分参数都一样 后面的筛选日期参数变成了起始位置和结束位置的值 还有一个-B即可

```
python binlog2sql.py  -h127.0.0.1 -P3306 -uroot -p'密码' -dwww_siammm_cn --start-file='mysql-bin.000006' -B --start-pos 590075 --stop-pos 590633
```

就可以得到insert的语句 复制出来，检查无误，就可以执行 恢复数据了