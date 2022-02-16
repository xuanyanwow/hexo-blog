---
title: 【源码解读】TP5读取本地图片输出后，设置header头无效，图片乱码
tags:
  - PHP
  - 常见问题
id: '257'
categories:
  - 后端
date: 2020-03-25 17:11:40
---

在Thinkphp程序中读取本地图片，做出加工处理（如合并二维码等水印），然后输出给客户端，一直输出图片内容乱码。 设置了header image/png 不生效。 写下这篇TP源码排查文章，看看问题到底出现在哪个步骤。

## 乱码

![](https://www.siammm.cn/wp-content/uploads/2020/03/3dc71e5f0bd67bfa20220a277fd8ba48.png)

## 设置响应头无效

```php
public function test(){
    // 请求头不生效，还是乱码
    header('Content-type: image/png');
    $file = "xxxxx\public\static\img/test.png";
    echo file_get_contents($file);
}
```

## 排查TP源码

还记得我们这篇文章吗：TP为什么可以return就输出字符串或者模板内容等等，在原生PHP不行呢？ [https://www.siammm.cn/archives/168](https://www.siammm.cn/archives/168 "https://www.siammm.cn/archives/168") 从这篇文章，大概的问题还是出现在这个控制器调度类里面，我继续查看该部分源码 还是这段熟悉的源码，一样的配方，不一样的问题（bug）。

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

因为我们在控制器中`没有return任何数据`，这里的$data==NULL，所以会走最后一步的

```php
$response = Response::create();
```

ok，那么我们看看默认的这个Response类都带了什么东东吧。

```php
// 原始数据
protected $data;

// 当前的contentType
protected $contentType = 'text/html';

// 字符集
protected $charset = 'utf-8';

//状态
protected $code = 200;

// 输出参数
protected $options = [];
// header参数
protected $header = [];

protected $content = null;
```

可以看到，这里的contentType默认是text/html；

```php
/**
 * 页面输出类型
 * @param string $contentType 输出类型
 * @param string $charset     输出编码
 * @return $this
 */
public function contentType($contentType, $charset = 'utf-8')
{
    $this->header['Content-Type'] = $contentType . '; charset=' . $charset;
    return $this;
}
```

在TP框架核心中，最后步骤是调用

```php
$response->send();
```

这个send的代码如下，也是这个问题根源所在。

```php
if (!headers_sent() && !empty($this->header)) {
    // 发送状态码
    http_response_code($this->code);
    // 发送头部信息
    foreach ($this->header as $name => $val) {
        if (is_null($val)) {
            header($name);
        } else {
            header($name . ':' . $val);
        }
    }
}
```

所以我们的header头 图片信息，被这里覆盖了

## 如何解决

*   手动调用实例化response类，并且在其中设置响应头

```php
$response = new Response();
$response->header('Content-type: image/png');
$response->data(file_get_contents($file));
return $response;
```

*   我们在控制器输出图片信息后，直接die掉，不要让它走后面的框架默认事件。（虽然这样子不符合框架设计思想）