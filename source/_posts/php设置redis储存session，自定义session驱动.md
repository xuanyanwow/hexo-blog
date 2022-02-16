---
title: PHP设置Redis储存Session，自定义session驱动
tags:
  - PHP
  - Redis
id: '139'
categories:
  - - PHP
date: 2019-08-30 10:15:07
---

### 思

我们在之前的文章已经讲到了session是将数据储存在本地文件中，并且将session\_id返回给客户端（浏览器会储存在cookies里）。 那么当我们在负载均衡集群环境的时候，`负载调度部分负责把客户端的请求按照不同的策略分配给后端服务节点`。所以会出现我们在A机器设置了session，后面请求在B机器判断session依旧为空的情况。

### 常用的负载均衡环境处理session的策略

PHP也可以配置将session保存在redis或者memcache中，在几种用来存储会话数据的方式。显然是Redis在效率上要更快些，而与memcached相比，因为有持久化，也更安全一些。 所以也是常用的负载均衡环境处理session的策略。 但因为是将信息储存在内存中，可能会出现内存不足、利用率不高等问题。 优点：效率高 缺点：信息储存在内存中，会产生大小不一的内存块，内存无法完全利用，并且可能出现内存不足。

### 设置session处理

php中除了可以通过简单修改配置项来设定使用其他的session处理方式，同时也提供了对应的接口以便于我们自定义session的处理逻辑。 接下来我们来通过自定义redis处理session的逻辑来了解接口。

### session\_set\_save\_handler函数

`session_set_save_handler()`该函数定义用户session逻辑，如写入、取出、关闭等。 该函数的传参如下：

> 该函数有两种用法

##### 在PHP5.4以前的用法

> bool session\_set\_save\_hanler(callback open,callback close,callback read,callback write,callback destory,callback gc)

可见该函数的几个参数接收都是以callback回调函数的形式的。

参数

描述

open

session打开时的回调函数。接收两个参数，第一个参数是保持session的路径，第二个参数是session的名字

close

当session操作完成时调用此函数。不接收参数。

read

以session\_id作为参数。通过以session\_id作为参数从数据存储方中取得数据，并返回此数据。如果数据为空，可以返回一个空字符串。此函数在调用session\_start 前被触发

write

当数据存储时调用。接收两个参数，一个是session\_id，另外一个是session的数据

destory

当调用session\_destroy 函数时触发destroy函数。只有一个参数 session\_id

gc

当php执行session垃圾回收机制时触发

调用方式：

```php
<?php 

// 需要先引入自定义的SiamSession类（该类的实现逻辑于下面PHP5.4以后的实现相同），然后再设置到save_handler中去
// 也可以直接在参数处传递闭包

$siamSession = new SiamSession();
session_set_save_handler(
  [$siamSession,"open"],
  [$siamSession,"close"],
  [$siamSession,"read"],
  [$siamSession,"write"],
  [$siamSession,"destroy"],
  [$siamSession,"gc"]
);

// 开启
session_start();
```

## ※※※

##### 在PHP5.4以后的用法 也是推荐的用法

> session\_set\_save\_handler ( object $sessionhandler \[, bool $register\_shutdown = TRUE \] ) : bool

参数

描述

sessionhandler

实现了 （SessionHandlerInterface， SessionIdInterface）或 SessionUpdateTimestampHandlerInterface 接口的对象， 例如 `SessionHandler`。

register\_shutdown

将函数 `session_write_close()` 注册为 register\_shutdown\_function() 函数。在PHP函数停止执行时可以触发。

> session\_write\_close()函数：结束当前会话并存储会话数据。

调用方式：

```php
<?php 

// 需要先引入自定义的Session处理程序，然后再设置到save_handler中去
// 也可以直接在参数处传递闭包

$siamSession = new SiamSession();
session_set_save_handler($siamSession, true);

// 开启
session_start();
```

我们看到第一个参数的描述，传入的参数应该是一个`实现了 SessionHandlerInterface 接口`的对象 同时还可以附属实现 `SessionIdInterface` 和 `SessionUpdateTimestampHandlerInterface` 接口 那么我们先来看看这几个接口需要实现什么方法

### 从SessionHandler理解几个接口实现

在描述中可以看到举例传入的参数可以为`SessionHandler`，也就是如果我们想要自定义Session处理程序，可以参考该类需要实现的方法。

> *   这个类是设计用于公开当前内部PHP Session处理程序，如果想要自己实现PHP Session处理程序，请实现 `SessionHandlerInterface`接口
> *   从SessionHandler继承的类，可以通过调用父类方法来重写覆盖具体操作，例如将数据加密储存。并且将新类通过session\_set\_save\_handler()设置为PHP Session处理程序

```php
<?php
// SessionHandler 实现了SessionHandlerInterface和SessionIdInterface两个接口
// 其中 SessionIdInterface 提供了 create_sid 接口，可以自定义session_id的生成规则
// 其他的方法则由 SessionHandlerInterface 提供，主要是session的回调处理，如打开、关闭、gc、写入、读取
SessionHandler implements SessionHandlerInterface , SessionIdInterface {
    /**
     * close方法，当session关闭的时候触发
     */
    public close ( void ) : bool
    /**
     * create_sid方法，返回一个新创建的session_id
     */
    public create_sid ( void ) : string
    /**
     * destroy方法，当调用session_destroy的时候触发
     */
    public destroy ( string $session_id ) : bool
    /**
     * gc方法，当php程序gc清理的时候触发，主要用于清除已经过期的session
     */
    public gc ( int $maxlifetime ) : int
    /**
     * open方法，当session打开的时候触发
     */
    public open ( string $save_path , string $session_name ) : bool
    /**
     * read方法，读取session的处理逻辑，可以在这里解密储存数据
     * 在session_start后会触发
     */
    public read ( string $session_id ) : string

    /**
     * write方法，将session数据写入到储存中，可以在这里加密数据
     */
    public write ( string $session_id , string $session_data ) : bool
}
```

还有另一个接口是`SessionUpdateTimestampHandlerInterface` 我们看看它又提供了什么方法的接口

```php
SessionUpdateTimestampHandlerInterface {
    /**
     * 更新时间戳，即更新session过期时间的
     */
    abstract public updateTimestamp ( string $key , string $val ) : bool
    /**
     * 验证session_id 是否还在线
     */ 
    abstract public validateId ( string $key ) : bool
}
```

> SessionHandlerInterface 接口是PHP >= 5.4.0 提供的 SessionIdInterface 接口是PHP >= 5.5.1 提供的 SessionUpdateTimestampHandlerInterface 接口是PHP >= 7.0 提供的

* * *

接下来我们通过代码来实践一下，通过实现SessionHandlerInterface接口，来写一个redis的PHP Session处理程序

```php
<?php

class SiamSession  implements \SessionHandlerInterface
{
    private $redis;
    private $expTime = 30; // 默认超时时间 根据业务场景设置

    function __construct(){
        // 连接redis
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1',6379);

        // 设置session处理回调 并且将session_write_close注册为register_shutdown_function函数
        session_set_save_handler($this, true);

        // 开启
        session_start();

    }

    function open($path, $name)
    {
        return true;
    }

    function close(){
        return true;
    }

    function read($session_id)
    {
        $value = $this->redis->get("siam_".$session_id);
        if ($value){
            return $value;
        }
        return '';
    }

    function write($session_id, $data)
    {
        if( $this->redis->set("siam_".$session_id, $data) ){
            $this->redis->expire("siam_".$session_id, $this->expTime);
            return true;
        }
        return false;
    }

    function destroy($session_id)
    {
        if ( $this->redis->delete("siam_".$session_id) )
        {
            return true;
        }
        return false;
    }

    function gc($maxlifetime)
    {
        return true; // 因为redis设置了过期时间，不需要再gc回收
    }

    function __destruct()
    {
        session_write_close();
    }
}

new SiamSession();
```

接着我们在另一个文件中写下测试代码

```php
<?php
require_once "SiamSession.php";

$_SESSION['name'] = "siam";
echo $_SESSION['name'];
```

可以看到浏览器正常出现了`siam` 那么我们进入phpredisadmin查看一下数据 可以看到类似图片的情况 [![phpredisadmin查看演示](http://yancoo.cn/uploads/images/201904/13_1.png "phpredisadmin查看演示")](# "phpredisadmin查看演示")

> 其他的储存可以参考上面的处理，对数据进行处理，就可以实现自己的session处理器了