---
title: php 微信支付分 开发过程 H5商城先享后付
tags: []
id: '283'
categories:
  - - PHP
  - - 常见问题
date: 2020-07-02 11:28:55
---

# 前言

公司项目需要，在H5商城、小程序商城、APP商城、线下促销场景，推出最新的微信支付分功能。 （类似花呗） 先签约，后续付款 遇到一些问题，写下此文章。 有不明确的地方，欢迎添加我QQ 59419979 一起交流补充。

# 问题

## the permission value is offline verifying

在H5情况下，按照微信支付分的唤起代码执行后，提示该情况。 原因：引入JSSDK后，需要进行获取js\_ticket进行config。详见以下文档地址：

```
https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html#1
```

其中重点文字：**所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用**

## PHP HMAC-SHA256

微信支付分 唤起部分的签名类型，仅支持`HMAC-SHA256` 以下为签名过程代码

```php
function sign_hmac_sha256($data, $key)
{
    ksort($data);
    $string = $this->array2url($data);

    $string .=  "&key=" .$key;
    $tem = hash_hmac("sha256", $string, $key, true);

    return strtoupper(bin2hex($tem));
}

$data = [
    'mch_id'         => $this->mchId,
    'service_id'     => $this->service_id,
    'out_request_no' => "SIAM_59419979".time().rand(1000,9999),
    'timestamp'      => time(),
    'nonce_str'      => md5(time()),
    'sign_type'      => 'HMAC-SHA256',
];

$data['sign'] = sign_hmac_sha256($data, 'xxxx 微信支付 商户后台的key 值');

// array to url 返回前端 即可
```

## 当前服务未上线

微信支付分的是新功能业务，也由于部门的流程升级，需要`先开发，验收后上线`，所以需要开发完成后联系微信官方进行验收。

# 完结语

以上为我在开发微信支付分过程遇到的小问题记录，希望能帮助到有需要的人，后续有遇到新问题将会持续更新本文章。

*   原文博客地址：http://blog.siammm.cn
*   QQ 59419979
*   Siam 宣言

by the way. 吐槽一下微信官方~ 跟支付宝的态度完全没得比 ![](https://www.siammm.cn/wp-content/uploads/2020/07/wp_editor_md_9725e019ba69f656147516478d4edbcf.jpg)