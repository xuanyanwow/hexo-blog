---
title: session在浏览器关闭时进行何处理?以及回收机制
tags:
  - PHP
  - 计算机基础
id: '146'
categories:
  - - PHP
date: 2019-08-30 10:22:44
---

Session会话机制被广泛应用在JSP、ASP、PHP等语言中。一般用来储存登陆状态或者其他的一些需要验证权限的状态。 以下类似代码在每个系统里应该都会存在

```
<?php
$userAccount = $_POST['user_account'];
$passWord    = $_POST['password'];

# 这里一般查询数据库验证用户是否存在、密码是否正常等
$vif = true;
if ( $vif ) {
    $_SESSION = $userInfo;
    echo '登陆成功';
} else{
    echo '登陆失败';
}
```

接着就可以在浏览器中浏览需要登陆状态的页面了。 那么，当我们关闭浏览器的时候，服务器上的session都进行了什么处理？

# Session的储存机制

我们先来看一下session的创建储存。 SESSION的实现中采用COOKIE技术。 SESSION会在客户端保存一个包含session\_id(SESSION编号)的COOKIE； 在服务器端保存其他session变量，比如session\_name等等。 当用户请求服务器时也把session\_id一起发送到服务器，通过 session\_id提取所保存在服务器端的变量，就能识别用户是谁了。 所以当我们创建一个session会话时候进行了如下的处理：

*   向服务器端写入session内容(一般默认是文件格式,文件储存位置可以通过配置文件修改) 比如我们上面储存的 `$userInfo` 变量信息，并且产生了一个 `SessionId` 编号。
*   将 `SessionId` 编号通过响应内容顺带返回给客户端
*   客户端将 `SessionId` 编号储存在 `Cookies` 中。
*   接下来客户端向该服务器发送的请求将带上 `SessionId` 编号，服务端便可以通过编号得到用户登录状态和信息。

# 浏览器关闭

当浏览器关闭的时候，会 `清空Cookies` ，这是浏览器对自己软件的操作，但是并不能对服务端的储存文件进行操作，所以这个时候服务端的session文件将继续生存。 当我们关闭浏览器，甚至电脑重启，短时间内服务端的session仍保存着，直到它被回收，这个时候我们通过一些手段模拟sessionid，仍可以继续保持会话进行。（当然你必须在你关闭浏览器之前把sessionid记下来了） 让session失效的原因只有两个：

> 超时，服务器自动回收。可以在配置文件中决定它的生存时间等。 程序主动销毁。比如 `$_SESSION = NULL;`。

# gc回收机制

PHP采用Garbage Collection process(gc)对过期session进行回收。 上面已经讲到可以通过配置文件修改session的生存周期（创建后不进行活动开始计时） 比如我们登陆了一个页面，然后再也没有进行过操作，一直在挂机着，一段时间后将会自动过期退出登陆 所以说每个服务端的session文件都会记录 `最后的活动时间`，等当前时间已经大于`最后活动时间+生存周期`，GC机制将会把该session文件清空回收。 那该gc机制是不是一直在监听检测每一个session文件？当然不是了~当访问量过大时，session文件将会很多，不停处理会让服务器造成不小的开销。 gc是按照`一定概率`启动的， 三个与PHP session过期相关的参数(php.ini中)：

> session.gc\_probability = 1 session.gc\_divisor = 1000 session.gc\_maxlifetime = 1440

gc启动概率 = gc\_probability / gc\_divisor = 0.1% 意思是每次session文件更新时，便会有 0.1% 的几率进行检测回收过期的session。但是如果访问量很小，可能会造成很多session文件过期了，但是仍然没有进行检测回收，这个时候我们就要通过修改上面的三个参数，来让GC启动的几率变大，让session过期的检测会更准确。