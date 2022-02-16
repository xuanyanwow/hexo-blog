---
title: 宝塔面板多PHP版本中编译安装升级Swoole
tags:
  - PHP
  - swoole
id: '154'
categories:
  - 运维
date: 2019-08-30 10:40:23
---

今天在使用最新版easyswole框架的过程中，需要依赖的swoole版本必须`>= 4.2.13`，到2019-2-25，宝塔面板能支持安装的swoole版本只有4.2.10，所以就看一下如何自己安装编译swoole扩展吧~

#### swoole 下载地址

```
https://github.com/swoole/swoole-src/releases
http://pecl.php.net/package/swoole
http://git.oschina.net/swoole/swoole
```

首先我们下载4.2.13版本的包，进入目录。

```
cd swoole
```

侦测php

```
sudo phpize （原文档）
```

因为我们安装多PHP版本，所以我们指定一下php的路径

```
sudo /www/server/php/72/bin/phpize
```

> phpize是用来扩展php扩展模块的，通过phpize可以建立php的外挂模块。当php编译完成后，php的bin目录下会有phpize这个脚本文件。在编译你要添加的扩展模块之前，执行phpize就可以了；

到了这里会生成`configure`文件

```
sudo ./configure （原文档）
```

我们需要指定php的配置文件路径

```
sudo ./configure --with-php-config=/www/server/php/72/bin/php-config
```

接着就是最后一步了

```
make && make install
```

等待编译完成后，查看一下swoole的版本即可

```
/www/server/php/72/bin/php --ri swoole  grep Version
```