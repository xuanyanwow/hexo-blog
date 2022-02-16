---
title: TP5.0安装testing 单元测试 报错
tags:
  - PHP
  - Thinkphp
  - 常见问题
id: '292'
categories:
  - - PHP
  - - 常见问题
date: 2020-12-01 11:24:41
---

# 常见问题记录

The each() function is deprecated. This message will be suppressed on further calls

*   原因：使用了比较高版本的php，topthink/tesing v1.x仅限php7.1使用 太高太低都会出现报错

\[think\\exception\\ErrorException\] Class 'TestCase' not found

*   原因：默认生成的单元测试示例没有加入命名空间。

在ExampleTest.php和TestCase.php中加入

```php
namespace tests;
```