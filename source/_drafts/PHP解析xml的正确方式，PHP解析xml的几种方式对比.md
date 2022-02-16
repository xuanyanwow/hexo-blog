---
title: PHP解析xml的正确方式，PHP解析xml的几种方式对比
tags: []
id: '217'
categories:
  - - uncategorized
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

对接我php接口的是安卓客户端，json字符串中在一个元素的时候是对象类型，多个元素的时候是数组类型，安卓客户端解析就失败了。 所以引申出这篇文章，详细测试、记录一下