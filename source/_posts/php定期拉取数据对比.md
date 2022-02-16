---
title: php定期拉取数据对比
tags:
  - PHP
  - 常见问题
  - 计算机基础
id: '183'
categories:
  - - PHP
date: 2019-09-20 20:59:01
---

# 写在前面

今天在网上看帖子提问的时候，看到有人发表了一个提问

> php下载远程的批量文件，每天一次，对比昨天和今天的文件，将旧文件替换成新文件

我们通过这个问题来分析讲解一下其中的知识点。 首先要解决的问题是：如何让程序每天自动执行一次脚本

# php定时执行任务

关于定时执行，最常见的方法是利用系统级别自带的功能

*   linux ( crontab 定时任务命令) windows计划任务

这需要手动修改系统的任务文件，然后使其生效

## 手动在linux添加定时任务

```shell
# crontab -e
```

运行该命令 打开任务编辑 在其中输入任务内容，然后Esc :wq保存退出 任务示例

```shell
0 0 * * * /www/siam/test.sh
```

前面的是运行周期的配置，后面的是sh脚本的路径，该方式一般需要自己编写sh脚本来执行

## 宝塔面板快速计划任务

如果我们使用宝塔面板当成运维工具，那么我们就可以很方便地添加计划任务了，如下图，宝塔中内置了挺多计划任务的类型，如定时请求URL，运行脚本，备份文件等等。 可视化配置，带给我们极大的便利，维护、添加都节约了很多的时间。 这也是为什么宝塔受到那么多人喜爱的原因之一吧。 ![宝塔面板计划任务的面板添加](https://www.siammm.cn/wp-content/uploads/2019/09/bd639225b7afebd3629be434c1cf7019.png)

## 取巧云监控定时执行

以上两种方式都需要服务器的权限，我们才可以管理定时任务，假设我们刚入门时使用的是虚拟主机，没有权限设置脚本运行，那么该如何实现这种功能呢？ 这里记录了我以前学习时利用的一个小方案，大家可以在其中学习一下。 云监控，是很多云服务商提供的一项服务，它可以用来测试、分析接口或者网站的稳定性和执行效率。 我们可以在服务商的后台类似宝塔面板一样去添加任务，然后服务商就会按我们设置的频率，定期访问网址，获取网址的正确执行、时间等信息，记录到他们后台，提供给用户查看分析改进。 我们可以利用这种特性，由服务商向我们的服务发起请求，我们可以填写一个php脚本的url，在其中判断当前时间，如果当前时间周期已经到了你设置的时间，则执行下面的内容 同时因为云监控是不间断地发起（一般最细颗粒是30s） 如果不能重复运行的任务，我们需要及时地把任务标记为已经执行。 可以在本地写文件，当文件锁。 不同云监控服务商有不同的设置和服务提供，网上有挺多免费的。大家可以找一找，如果找不到好的，也可以联系我QQ交流一下。

## 现代化PHP

PHP发展了这么久，其实已经有了很大的改进，比如PHP5OOP特性的完善、PHP7的性能提高、Swoole生态的出现，让PHP能做的事越来越多，越做越好。 在当今环境中，我们可以使用SWOOLE常驻内存的特性完成很多事， 这里推荐一下`EasySwoole`这款基于Swoole环境的框架。 关于定时任务在EasySwoole框架中的文档地址点这里 [EasySwoole Crontab 定时器](https://www.easyswoole.com/Cn/BaseUsage/crontab.html "EasySwoole Crontab 定时器") 常驻内存的程序，在服务器上后台稳定运行， EasySwoole中提供了丰富的组件，比如传统PHPFPM环境很难解决的Mysql数据库连接池、协程Redis客户端、协程Http客户端、芒果DB客户端等等 还有我们这个主题有的一个定时任务的模块，下面看一小段demo代码 首先在主环境事件代码中开启定时任务

```php
public static function mainServerCreate(EventRegister $register)
{
    // 开始一个定时任务计划
    Crontab::getInstance()->addTask(TaskOne::class);
}
```

定时任务的配置和内容

```php
namespace App\Crontab;

use EasySwoole\EasySwoole\Crontab\AbstractCronTask;

class TaskOne extends AbstractCronTask
{

    public static function getRule(): string
    {
        // TODO: Implement getRule() method.
        // 定时周期 （每小时）
        return '@hourly';
    }

    public static function getTaskName(): string
    {
        // TODO: Implement getTaskName() method.
        // 定时任务名称
        return 'taskOne';
    }

    static function run(\swoole_server $server, int $taskId, int $fromWorkerId,$flags=null)
    {
        // 定时任务处理逻辑

        // 我们在这里执行拉取文件、对比处理、保存文件的逻辑就好了
        var_dump('run once per hour');
    }
}
```

# 其他问题

解决了定时执行的问题，那么下载文件和保存文件，我觉得应该都不会是很大的问题