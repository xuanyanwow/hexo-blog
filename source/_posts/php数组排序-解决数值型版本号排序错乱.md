---
title: PHP数组排序 解决数值型版本号排序错乱
tags:
  - PHP
  - 常见问题
id: '294'
categories:
  - - PHP
  - - 常见问题
date: 2020-12-02 11:31:19
---

# 问题

本人在写EasySwoole新的组件时，使用到了插件思维，所以需要做包的解析等逻辑。在解析下列版本解析时，发现一些小问题。做个记录。

```php
v1.0.php
v2.0.php
v10.0.php
```

普通调用

```php
asort($list);
```

返回的结果是

```php
v1.0.php
v10.0.php
v2.0.php
```

# 解决

```php
ksort($list, SORT_STRING  SORT_FLAG_CASE  SORT_NATURAL); // 对键排序
asort($list, SORT_STRING  SORT_FLAG_CASE  SORT_NATURAL); // 对值排序
```