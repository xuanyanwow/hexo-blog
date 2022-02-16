---
title: PHP生成不重复的订单号
tags:
  - PHP
id: '148'
categories:
  - - PHP
date: 2019-08-30 10:24:32
---

使用场景：商城、微信支付等生成订单号需求

* * *

> 方法一

比较常见的一种简单方法 使用date()函数，获取当前日期的数字，再配合rand()函数，生成几位随机数。便是一个简单的12位订单号了

```
<?php
function getOrderNum(){
    $date = date('Ymd');
    $rand = rand(0,9).rand(0,9).rand(0,9).rand(0,9);
    return $date.$rand;
}
echo getOrderNum();
```

> 方法二

```
<?php
function getOrderNum(){
    $date = date('Ymd');
    $rand = substr(implode(NULL, array_map('ord', str_split(substr(uniqid(), 7, 13), 1))),0,12); 
    return $date.$rand;
}
echo getOrderNum();
```

uniqid()函数基于以微秒计的当前时间，生成一个唯一的 ID。当时前面的7位是不会经常变动的(应该是秒数，一秒一次) 所以我们使用substr()函数，截取字符串，从第8位到13位，接着这里会有一个问题，得到的是数字+字母的随机数，如果你需要的订单号可以包含字母，这里不需要转换也可以。 这里为了纯数字的订单号，所以要继续进行处理。 使用str\_split($string, 1)函数，将字符串，以一个字符的长度分割成变量。也就是一个字符一个变量。 array\_map()函数是将数组遍历执行一次函数，这里使用的是ord函数，返回字符所在的ASCII码，是一个数字。 所有的字符都已经转成了数字，但是长度会波动(因为有写ASCII码可能是1.可能是81) 所以我们还要使用一个字符截取函数，implode()，截取0~12位的字符。合适范围(5~12)，最大12 这里是完全随机的字符。而且是基于时间微秒来生成的，重复的可能性非常非常低，之所以加上时间日期，是为了看起来更加统一。 20180131559751565757 20180131985210250485

* * *

2018-2-1更新

> 方法三

```
function getOrderNum(){
    $yCode = array('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J');
    $orderSn = $yCode[intval(date('Y')) - 2017] . strtoupper(dechex(date('m'))) . date('d') . substr(time(), -5) . substr(microtime(), 2, 6) . sprintf('%02d', rand(0, 99));
    return $orderSn;
}
```