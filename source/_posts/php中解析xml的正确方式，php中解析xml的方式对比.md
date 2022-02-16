---
title: PHP中simplexml_load_string解析xml的正确方式
tags:
  - PHP
  - 常见问题
id: '221'
categories:
  - - PHP
  - - 常见问题
date: 2019-12-18 14:57:28
---

# 前言

为什么写这篇文章，因为他娘的宣言又踩坑了。 在对接别人接口时，他们使用的是xml格式交互的。 其中的格式可能如下

```markup
    <RSP2003>
      <TotalNum>39</TotalNum>
      <CityList>
        <CityCode>N1127</CityCode>
        <CityName>三水四会</CityName>
        <CityTypeCode>3</CityTypeCode>
        <CityTypeName>内陆城市</CityTypeName>
        <PinyinJX>SSSH</PinyinJX>
        <PinyinQP>Sanshuisihui</PinyinQP>
        <CityOrder>207</CityOrder>
        <CityOperationType>2</CityOperationType>
      </CityList>
      // 这是一个list  如果还有更多元素 就在下面继续
      <CityList>
        <CityCode>N1127</CityCode>
        <CityName>三水四会</CityName>
        <CityTypeCode>3</CityTypeCode>
        <CityTypeName>内陆城市</CityTypeName>
        <PinyinJX>SSSH</PinyinJX>
        <PinyinQP>Sanshuisihui</PinyinQP>
        <CityOrder>207</CityOrder>
        <CityOperationType>2</CityOperationType>
      </CityList>
    </RSP2003>
```

有的情况下，`CityList里只有一个元素`，一般情况下是`多个` 一开始写的php程序如下

```php
<?php
$xml = '伪代码 xml字符串如上';

echo json_encode(simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOCDATA));
```

## 只有一个元素的时候

```json
"RSP2003": {
    "TotalNum": "39",
    "CityList": {
        "CityCode": "N1127",
        "CityName": "\u4e09\u6c34\u56db\u4f1a",
        "CityTypeCode": "3",
        "CityTypeName": "\u5185\u9646\u57ce\u5e02",
        "PinyinJX": "SSSH",
        "PinyinQP": "Sanshuisihui",
        "CityOrder": "207",
        "CityOperationType": "2"
    }
}
```

## 多个元素的时候

```json
"RSP2003": {
    "TotalNum": "39",
    "CityList": [{
        "CityCode": "N1127",
        "CityName": "\u4e09\u6c34\u56db\u4f1a",
        "CityTypeCode": "3",
        "CityTypeName": "\u5185\u9646\u57ce\u5e02",
        "PinyinJX": "SSSH",
        "PinyinQP": "Sanshuisihui",
        "CityOrder": "207",
        "CityOperationType": "2"
    }, {
        "CityCode": "N1128",
        "CityName": "\u5927\u6ca5\u76d0\u6b65",
        "CityTypeCode": "3",
        "CityTypeName": "\u5185\u9646\u57ce\u5e02",
        "PinyinJX": "DLYB",
        "PinyinQP": "daliyanbu",
        "CityOrder": "208",
        "CityOperationType": "2"
    }]
```

## 问题所在

对接我php接口的是安卓客户端，json字符串中在一个元素的时候是对象类型，多个元素的时候是数组类型，安卓客户端解析就失败了。 所以引申出这篇文章，详细测试、记录一下php中解析xml方式和细节

# simplexml\_load\_string

`simplexml_load_string`函数将会把`每一个节点`都解析成一个`SimpleXMLElement`对象 php官方文档地址：https://www.php.net/manual/zh/class.simplexmlelement.php 注意这里我描述的是：每一个节点。 首先我们先来解析一个最简单的例子

```php
$xml = <<<xml
<?xml version="1.0" encoding="UTF-8"?>
<RSP2003>
    <TotalNum>39</TotalNum>
</RSP2003>
xml;
$object = simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOCDATA);
var_dump($object);
```

输出内容

```
object(SimpleXMLElement)#1 (1) {
  ["TotalNum"]=>
  string(2) "39"
}
```

可以看到，这里是一个对象，我们需要怎么获取里面的TotalNum节点呢，TotalNum这个值又是什么类型的？ `在这一步打印出来它是一个string类型` 我们接着看吧

```php
var_dump($object->TotalNum);
```

输出 TotalNum又是一个SimpleXMLElement对象，它的值储存在\[0\]中 `我们写数组的下标`

```
object(SimpleXMLElement)#2 (1) {
  [0]=>
  string(2) "39"
}
```

继续取出

```
var_dump($object->TotalNum[0]);
```

输出内容 `注意哈。这里是真实的运行结果，不是我复制重复了(对象的编号已经增加了)，自己可以去测试一下`

```
object(SimpleXMLElement)#4 (1) {
  [0]=>
  string(2) "39"
}
```

那么我们这个值到底怎么取出呢！！

## 取出SimpleXMLElement对象的值

```
var_dump($object->TotalNum->__toString());
```

回到我们最开始的问题，怎么解析xml列表

## 解析列表，（只有一个元素也为数组）

```php
<?php

$xml = <<<xml
<?xml version="1.0" encoding="UTF-8"?>
<RSP2003>
    <TotalNum>39</TotalNum>
    <CityList>
        <CityName>第一个城市</CityName>
    </CityList>
    <CityList>
        <CityName>第二个城市</CityName>
    </CityList>
</RSP2003>
xml;

$object = simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOCDATA);

var_dump(count($object->TotalNum)); // 1
var_dump(count($object->CityList)); // 2
var_dump($object->CityList->count()); // 2

var_dump($object->CityList[0]);
var_dump($object->CityList[1]);
```

## 用法探讨

尝试了挺多种逻辑，都无法用函数封装成自动解析（因为每一个节点都是平等的，怎么知道它要解析成数组还是对象呢？） `如果你有好想法，希望能留言一起讨论` 我觉得只能面向过程式地手动组装成数组，然后输出api结果