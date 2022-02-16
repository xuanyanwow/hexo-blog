---
title: 'swoole,swoole_timer_tick() must be callable, array given 报错异常'
tags:
  - easyswoole
  - PHP
  - swoole
  - 常见问题
id: '240'
categories:
  - - easyswoole
  - - PHP
  - - 常见问题
date: 2020-03-04 09:11:55
---

# 报错提示

> Fatal error: Uncaught TypeError: Argument 2 passed to Swoole\\Timer::swoole\_timer\_tick() must be callable, array given

# 触发场景

在easyswoole旧版的Component组件中的Pool抽象方法（用于实现通用连接池），有一行代码是

```php
if ($conf->getIntervalCheckTime() > 0) {
    swoole_timer_tick($conf->getIntervalCheckTime(), [$this, 'intervalCheck']);
}
```

定时触发这个检查方法，来完成`最小连接池保持、掉线检测`等操作。 于是就在这里产生了这个异常

# 解决问题

搜索了php官方对于callable的定义, 是允许数组这种形式传递的 https://www.php.net/manual/zh/language.types.callable.php 咨询swoole开发组的成员twosee，也反馈这个类型判断是调用zendapi完成的，理论不应该出问题 给出的解决方案是使用php推荐新增的`Closure`

```php
if ($conf->getIntervalCheckTime() > 0) {
    swoole_timer_tick($conf->getIntervalCheckTime(), \Closure::fromCallable([$this, 'intervalCheck']));
}
```

easyswoole框架内部交流后也说明这个问题是由于swoole版本变动，很早以前就在新版做了兼容（将intervalCheck改为public方法）