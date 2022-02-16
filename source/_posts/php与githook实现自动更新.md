---
title: php与githook实现自动更新
tags:
  - PHP
  - 常见问题
id: '256'
categories:
  - - PHP
  - - 常见问题
date: 2020-03-25 16:28:40
---

# githook

git系统仓库一般都会支持这个hook配置，在发生事件的时候触发执行，可以是https推送等通知形式。 我们使用gitee+php来达到自动更新项目代码的需求。

## 用户组和权限

*   php 是以 `www` 用户组运行在系统上的，

如果我们使用php的函数 `shell_exec("cd /www/wwwroot/xxxx && sudo git pull origin master");` 来执行的话会返回NULL。执行失败

*   git 属于 `root` 用户组

在php中使用git会因为权限而失败 解决方案： 编辑`/etc/sudoers`文件，如下： (原文博客https://www.siammm.cn https://www.siammm.cn/archives/256)

```vim
root    ALL=(ALL)       找到这一行，在下方加入一行：
www     ALL=NOPASSWD:/usr/bin/git     这一行的意思是让www用户组可以不用密码使用git
```

此时可以使用git客户端。如下可以正常返回，但是执行pull的时候还是返回NULL

```php
var_dump(shell_exec("git version"));
```

涉及文件夹权限，没有权限更改文件

*   可以将文件夹设置777权限 或者归属为www用户组
*   在php shell\_exec 执行中加入sudo

```php
<?php

$json =  file_get_contents("php://input");

$array = json_decode($json , true);

if (isset($array['ref']) && $array['total_commits_count']>0 && isset($array['password']) && $array['password'] == 'xxxxxxx'){
    $res = shell_exec("cd /www/wwwroot/default/testHook/yanpay && sudo git pull origin master");
    var_dump( $res) ;
}
```