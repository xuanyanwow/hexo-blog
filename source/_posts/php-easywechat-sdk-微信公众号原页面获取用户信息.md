---
title: PHP EasyWechat SDK 微信公众号原页面获取用户信息
tags:
  - PHP
  - 常见问题
  - 微信
  - 源码分享
id: '288'
categories:
  - - PHP
  - - 常见问题
date: 2020-11-16 12:39:30
---

# 前因

由于EasyWechat文档编写的不够完善，在这里补充一下。 官方提供的文档如下

```php
$app = Factory::officialAccount($config);
$oauth = $app->oauth;

// 未登录
if (empty($_SESSION['wechat_user'])) {

  $_SESSION['target_url'] = 'user/profile';

  return $oauth->redirect();
  // 这里不一定是return，如果你的框架action不是返回内容的话你就得使用
  // $oauth->redirect()->send();
}

// 已经登录过
$user = $_SESSION['wechat_user'];
```

但是这不适用于`在当前页面获取授权`的需求，而是跳转到一个新的页面 `user/profile`

# 实现方法

在这里提供一下我的写法参考

```php
public static function login()
{
    $app      = Factory::officialAccount($config);
    $customer = Session::get('customer_info');
    if (!$customer){
        if (!input('?code')){
            $app->oauth->redirect(Url::full())->send();
            return '';
        }else{
            $customer_info = $app->oauth->user();

            Session::set('customer_info', $customer_info);
            return $customer;
        }
    }

    return $customer;
}
```

接着在`任意地方`调用上述封装方法即可

```php
$customer = Auth::login();
```

## 注意事项

*   代码中的Session为TP框架，其他框架或者原生 请替换为对应的处理。