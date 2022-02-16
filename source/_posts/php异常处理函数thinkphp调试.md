---
title: 'PHP异常处理函数,Thinkphp调试'
tags:
  - PHP
  - Thinkphp
id: '51'
categories:
  - - PHP
date: 2019-06-19 09:47:36
---

在我们开发过程、已上线的应用中，程序经常会因为异常而崩溃。 比如：`数据库执行失败`、`调用了不存在的类`、`调用了不存在的函数/方法`.... 如果是在开发过程中还好，问题肯定是由我们自己发现，可以清楚地看到异常的信息。 那么如果是已经上线的应用，那么出现问题的时候，客户往往是这么说的

> 在xxx的时候 网页没有反应，网页失败了。

So? How to review it? 以上举例是我们本次要解决的问题，我们再来查看一个案例。 用过thinkphp等任何一个框架的都知道，当我们的程序报错时，显示的都是框架美美的报错异常页面。 之所以能显示出框架自定义的页面，都是因为使用了`异常处理函数`来实现的。

### 异常处理函数

在默认的php中，产生异常的时候是这样子的：

* * *

![](https://www.siammm.cn/wp-content/uploads/2019/06/a70ed18f9c5c22caff11cc7cb69e65a9.png)

* * *

php提供了`set_exception_handler`函数，让我们可以自定义异常产生时执行、输出的数据。

```php
<?php
function exception_handler($exception) {
    echo "有异常产生了 傻逼 :\n";
    var_dump($exception);
}

set_exception_handler('exception_handler');

throw new Exception('Uncaught Exception');
```

![](https://www.siammm.cn/wp-content/uploads/2019/06/08563355c62699f0a60e1ee8a179e0c4.png) 此时我们可以看到我们自定义的内容输出了。 在thinkphp中，除了异常类携带的简单file、code、message、trace等 `还会获取`当前服务器的配置、脚本的参数(get/post...)、数据库查询语句等 然后组合成一个有排版、数据充足的页面展示给我们，方便了我们排查问题。

### thinkphp 默认的异常处理器

tp中默认的异常处理器是：`\think\exception\Handle`这个类，同时在配置文件中也预留了我们自定义的配置空间。

### 在thinkphp中实现异常上报模块

我自定义了一个继承了tp默认异常处理器的新类，并在其中记录了`php脚本执行时间`、`tp能获取到的全部数据` 然后上报到数据库(或者其他储存地址) 再自定义了`查看异常记录`的页面 当点击查看记录的时候，在脚本中开启tp的debug模式，然后复现数据 到这里，哪怕线上的脚本挂了，我们也可以通过此模块查看复现报错的地方。 客户说遇到了问题崩溃了，我们查看此模块分析就可以， 甚至可以每天定期查看该模块，检查程序是否有异常产生，提前解决，提高程序的稳健性 ![](https://www.siammm.cn/wp-content/uploads/2019/06/f0d042c3cfbabc635b0e0277ddf0274a.png)