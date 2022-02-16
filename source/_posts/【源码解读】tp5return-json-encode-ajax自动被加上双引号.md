---
title: 【源码解读】TP5return json_encode ajax自动被加上双引号
tags:
  - PHP
  - Thinkphp
id: '168'
categories:
  - 后端
date: 2019-09-02 15:20:01
---

# 事情起因

在thinkphp5中，return数据才是控制器正确的时候方式，而不是直接echo 然后die 或者exti 因为框架有后置数据的落地处理等等，直接让程序退出并不友好，既然我们选择了框架，就应该遵循框架的设计理念 这样子能让我们避免一些坑爹事件的发生。 此次我在控制器中，并没有使用tp的`Json Response`对象，而是想通过`return json_encode($arr);`返回字符串的形式 正常应该输出如下

```json
{"name":"siam", "age":21}
```

结果却输出为

```
"{"name":"siam", "age":21}"
```

这样子就明显乱套了，前端解析直接崩溃。

# 控制器原理

追寻response后框架的处理，框架会根据控制器`return`的数据类型做不同的处理

*   返回Reponse子类，比如Json、Jsonp、Xml、View、重定向等等，则会执行子类的run()
*   返回不是Reponse子类，则会自动识别响应输出类型

# 源码追踪

以下是tp5中该问题核心的代码

```php
// 输出数据到客户端
if ($data instanceof Response) {
    $response = $data;
} elseif (!is_null($data)) {
    // 默认自动识别响应输出类型
    $type = $request->isAjax() ?
    Config::get('default_ajax_return') :
    Config::get('default_return_type');

    $response = Response::create($data, $type);
} else {
    $response = Response::create();
}
```

$data是我们在控制器中返回的数据 可能是

```php
return '文本';
return json($arr);
return jsonp($arr);
return xml($arr);
```

等等 当我们返回json\_encode的时候，不是Reponse类，会走第二个判断 它会判断是否为ajax请求，如果是ajax请求，则走默认的ajax\_return type 在tp5中 默认是json 所以会把字符串`再度json序列化` 就会出现我们问题的那种情况了

# 解决方案

解决该问题有多种方式 其实也只是小问题，这里做一个汇总吧

*   返回Reponse子类，是文本类型

```php
Response::create($this->encrypt(json_encode($array, 256)), "html");
```

*   返回Reponse子类，是json类型

```php
// tp5内置json助手函数
return json($array);
```

*   输出后退出（不符合tp设计理念 非常非常不推荐）
*   修改配置文件，默认ajax\_return为html