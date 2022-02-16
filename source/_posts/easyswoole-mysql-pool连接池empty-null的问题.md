---
title: easyswoole mysql-pool连接池empty null的问题
tags:
  - PHP
  - swoole
id: '188'
categories:
  - - 后端
date: 2019-10-11 15:27:06
---

# 问题由来

在easyswoole的群里，每天都需要回答各种各样的问题，其中不乏一些问题反复被小白们问起，比如今天的这个主题：`连接池取出empty 为null`导致的问题 本文会简单引申出`什么是连接池`、`连接池数量如何设置`、`连接池的优点`等问题。

# 什么是连接池

> 连接池是创建和管理一个连接的缓冲池的技术，这些连接准备好被任何需要它们的线程使用。

简单来说，就是创建一个容器，并且把资源提前准备好放在里面，比如我们常用的redis连接、mysql连接。

# 连接池的优点

计算机是由许多零件组装而成，比如CPU、内存、硬盘等等。 当我们进行网络连接、请求的时候，就需要在不同组件中传递和返回各种信号、数据 比如在CPU、内存、网卡中，数据的传递，请求，获取。 如果在短时间内进行一万次mysql的连接，就需要在这个往返过程循环，在路上浪费了很多时间、性能消耗。 如果我们先把连接连接好，并且放在连接池中，程序中需要使用就从池中获取，执行操作。 就省去了反复创建连接、断开连接的操作。 可以减少I/O操作，提高资源利用率。

# 连接池数量如何设置

那么一个池需要设置多少数量比较合适呢？是不是越多越好？ 在此之前，我们需要先了解计算机的进程原理，一个CPU伪造出多进程并行的假象。（我们电脑能一边听歌一边聊天等等） 我们把一个池中的连接看成一个进程（在实际中也可能是线程级别），如果设置过多，就会在系统中创建太多进程，切换进程上下文就会比较慢了。 一般我们把连接池数量设置为CPU的1~2倍即可（非固定）

# easyswoole中为什么会pool empty

这个问题有好几个可能性。

*   连接信息错误，导致一个资源都没有
*   程序有问题，把资源拿出去，没有归还到池内，后续就拿到空了
*   并发高，池的数量少，需要检查资源占用率，如果占用率没问题，则提高池内的数量

## 连接信息错误

如果我们的mysql配置信息错误，在easyswoole框架启动之后，就会去初始化连接池。 此时一直连接失败，也就没有产生资源，也没有将资源放在池内 当你在后续程序获取池内资源的时候。自然就报了空池的错误提示。

## 程序问题

先来一个连接池的伪代码

```php
<?php

class Pool{
    public static function getIn(){
        // 单例模式
    }
    /**
     * 初始化
     */
    public function init()
    {
        // pool准备好就填充指定的资源 比如10个连接
        $this->pool = $array;
    }

    public function get(){
        return array_pop($this->pool);
    }   
    public function push($obj)
    {
        $this->pool[] = $obj;
    }
}
```

如果我们的程序有这样子的使用场景

```php
<?php

    $db = Pool::getIn()->get();
    $res = $db->query('sql语句');
```

然后没有进行push 归还操作，那么池内资源一旦拿完，就没有资源可用了。 在easyswoole框架中，有提供以下方法获取资源（以mysql-pool为例）

```php
$db = MysqlPool::defer();
$db->rawQuery('select version()');
```

```php
$data = MysqlPool::invoker(function (MysqlConnection $db){
    return $db->rawQuery('select version()');
});
```

```php
$db = PoolManager::getInstance()->getPool(MysqlPool::class)->getObj();
$data = $db->get('test');
//使用完毕需要回收
PoolManager::getInstance()->getPool(MysqlPool::class)->recycleObj($db);
```

defer方法将会在本次请求协程退出的时候自动回收 invoker是闭包函数方式 一次运行完马上自动回收 get方式 就是我们伪代码的方式 需要自己回收 使用这种方式就需要特别注意啦~！！！

> 两种自动回收方式怎么选择 请接着往下看！

## 并发高 资源占用率

上面说到两种自动回收资源的方式，defer和invoker 首先我们来看一个点，defer是在协程退出时自动回收，正常来说，在一个请求到达的时候，swoole会自动创建一个协程给他，比如我们一个http api的请求，就需要整个api跑完，这个协程才会退出 （相当于我们传统fpm php中 一个脚本全部执行完） 这个时候问题来了，如果我们的业务是这样子的

```php
<?php

    $db = MysqlPool::defer();
    $db->rawQuery('select version()');

    // 执行好mysql了  做其他任务

    // 耗时1.5s 完成其他


```

实际上使用到mysql资源的可能只有0.1s不到，但是其他运算占用了脚本大量执行时间，要等全部执行完，协程退出了，资源才会回收，这个时候就比较浪费资源的利用率了。占用率比较低。 如果整个程序都是这样子的场景。那么一个池内有十几二十个连接是完全不够用的。这也是大部分新人为什么在pool里设置100个连接的理由。。。。。我真的佩服哦！！！ 如果可以的话 ，我们推荐使用invoker 执行一条 马上回收资源 此时要注意一个点，如果程序有比较多执行语句，要么在一个invoker里执行，要么合理使用invoker 不然就会把性能消耗转移到不断get recycle上了 如果以上排查都没问题，并且确认你的用户量比较多，并发高，就可以适当提高pool的number