---
title: php cli web server with rewrite router
tags:
  - PHP
id: '310'
categories:
  - - PHP
  - - 常见问题
date: 2021-07-05 09:35:28
---

## 前言

在此前的文章中，我曾说过php cli开启的web server 没办法像nginx一样实现伪静态等自由的路由规则，这篇文章记录一下，补上这个知识缺口。

## PHP CLI WEB SERVER

```shell
php -S 127.0.0.1:8000
```

以上命令可以开启一个php自带的web server服务，我们可以在后续加上一个文件名，作为入口文件，在其中编写rewrite router规则 如

```shell
php -S 127.0.0.1:8000 router.php
```

## Router代码

```php
<?php
if (is_file($_SERVER["DOCUMENT_ROOT"] . $_SERVER["SCRIPT_NAME"])) {
    return false;
} else {
    // 伪静态.jpg后缀 其实是php
    if (strpos($_SERVER['SCRIPT_NAME'], ".jpg") !== false){
        $_SERVER['SCRIPT_NAME'] = str_replace('.jpg', '.php',$_SERVER['SCRIPT_NAME']);
        require $_SERVER["DOCUMENT_ROOT"].$_SERVER['SCRIPT_NAME'];
    }
}
```