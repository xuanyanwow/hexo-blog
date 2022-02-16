---
title: TP5.0.20 - TP5更改网站目录为/public 后运行出错解决
tags:
  - PHP
  - Thinkphp
  - 计算机基础
id: '149'
categories:
  - - PHP
date: 2019-08-30 10:26:00
---

今天在部署TP5的时候，把网站根目录指向到public目录下，运行后产生以下错误

```
Warning: require(): open_basedir restriction in effect. File(/www/wwwroot/xx/thinkphp/start.php) is not within the allowed path(s): (/www/wwwroot/xx/public/:/tmp/:/proc/) in /www/wwwroot/xx/public/index.php on line 18

Warning: require(/www/wwwroot/xx/thinkphp/start.php): failed to open stream: Operation not permitted in /www/wwwroot/xx/public/index.php on line 18

Fatal error: require(): Failed opening required '/www/wwwroot/xx/public/../thinkphp/start.php' (include_path='.:/www/server/php/70/lib/php') in /www/wwwroot/xx/public/index.php on line 18
```

就是require文件的时候出错了，并且带上了文件的路径，一开始以为是路径出错的，于是在index.php中尝试修改 引入的文件路径，发现index.php并没有问题。 百度发现：open\_basedir 的问题  需要在php.ini中修改open\_basedir你的项目路径，或者在nginx中也可以定义。 但是我的两个配置文件中都没有该配置参数，于是继续找问题。后来想到服务器使用了宝塔面板来管理的，指定子目录也是在宝塔面板中进行。 于是到宝塔面板的页面，发现有一个   防跨站攻击(open\_basedir)    的选项 把该选项关闭即可。 原因如下：open\_basedir 将PHP所能打开的文件限制在指定的目录树中，包括文件本身。当程序要使用例如fopen()或file\_get\_contents()打开一个文件时，这个文件的位置将会被检查。当文件在指定的目录树之外，程序将拒绝打开。 本指令不受安全模式打开或关闭的影响。