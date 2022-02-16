---
title: HTTP跨域详解和解决方式
tags:
  - 前端
  - 计算机基础
id: '138'
categories:
  - - 前端
date: 2019-08-30 10:13:49
---

## HTTP跨域

> Access to XMLHttpRequest at 'xx' from origin 'xx' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

前端的这个报错相信很多人都有遇到过，也知道这是跨域请求的问题。 那么究竟什么是`跨域`，跨域又是怎么产生的，以及跨域请求的问题需要怎么解决。我们一起来了解一下这些知识。  
  
  

## 什么叫跨域

`域` ： 域既是 Windows 网络操作系统的逻辑组织单元，也是Internet的逻辑组织单元，它是`安全边界`。 只有域的所有者才能访问管理域内部的资源，若其他的域要访问或者管理，则需要该域赋予其他域相关权限。 从小角度来讲，在php中的`变量作用域`，就可以体现出安全边界的概念。在以下例子中，调用test函数并不会输出任何内容。

```php
<?php
$a = 123;

function test(){
    echo $a;
}
test();
```

因为函数内调用的是`局部作用域`的变量，而在局部作用域内并没有声明 $a 变量。 除非我们使用`global $a;`从全局作用域引用该变量。  
在PHP脚本中的变量作用域不算复杂，而将一个网站看做一个域，当它要引用其他域的资源时，就需要目标域对原始域进行授权信任。 这种从其他域获取资源的操作就叫做 `跨域`。  
  

## 浏览器的同源策略

`同源策略`是Web的一种安全约定，浏览器的同源策略只是对其的一种实现。 浏览器同源策略将认为任何站点装载的内容都是不安全的。所以会对`跨域的操作或者请求`进行限制，从而让用户安全的上网。  
`同源` 指的是 `域名、协议、端口` 相同。 若有其中一个不同，浏览器将会认为非同源，也就是跨域。  
浏览器的同源策略主要有两种

*   DOM 同源策略 ： 禁止对不同源页面的 Dom 元素进行操作，主要是在 iframe 标签加载跨域页面出现。
*   XMLHttpRequest 同源策略 ： 禁止使用 XHR 对象对不同源地址发起请求。
*   存储在浏览器中的数据，如localStroage、Cooke和IndexedDB不能通过脚本跨域访问

#### Dom 同源策略

如果没有 DOM 同源策略，也就是说不同域的 iframe 之间可以相互访问操作。 那么将会出现这种攻击操作：我们 iframe 包含某个网站的登录页，并且监听目标网站的登录按钮，当用户触发按钮的时候，我们拿到目标网站 input 的dom元素，并且取值，保存到自己的服务器上。 但是因为有 Dom 同源策略的存在，禁止操作不同源页面的dom元素，甚至我们还可以将自己的网站设置 `禁止在非同源网站上 iframe` ，我们来看看下面的例子

```markup
<html>
    <head>
        <title>Siam - Dom同源策略</title>
    </head>

    <body>
        <iframe src="http://www.alipay.com">
    </body>

</html>
```

运行以上代码，我们会看到支付宝的网站是禁止在了非同源网站上ifarme。 ![dom运行结果](http://yancoo.cn/uploads/images/201904/20_01.png) 我们可以看到报错

> v.asp:1 Refused to display 'https://www.alipay.com/' in a frame because it set 'X-Frame-Options' to 'sameorigin'.

`X-Frame-Options` 是一个HTTP标头（header），用来告诉浏览器这个网页是否可以放在iFrame内。 用法

```
X-Frame-Options: DENY     // 不允许iframe
X-Frame-Options: SAMEORIGIN   // 只允许同源的网站iframe
X-Frame-Options: ALLOW-FROM http://yancoo.cn/  // 只允许指定网站iframe
```

  
  

#### XMLHttpRequest 同源策略

如果没有 XHR 同源策略，以及不允许跨域获取cookies等的限制，那么攻击者将可以发起 `CSRF (跨站请求伪造)` 攻击 场景可以如下：

1.  你登录了某个银行网站，www.siambank.com，银行网站返回你的登录状态并且保存在cookies中。
2.  你没有安全退出清空cookies，又刚好不小心浏览到了恶意网站 www.ggg.com
3.  一进入 www.ggg.com ，它将会向 `银行网站` 发起XHR请求。（发送请求将会带上目标网站设置的cookies）
4.  银行拿到cookies，验证通过，返回数据。  
      
    

## 跨域的解决方法

前面我们已经说了，如果想要跨域请求访问或者管理资源，需要目标域赋予权限，到目前为止我们只说了浏览器同源策略的限制，下面我们就再说说赋予权限进行跨域访问相关的知识。  
  

#### CORS 跨域资源共享

`CORS` 是一个 `W3C标准`,该标准定义了在访问跨域资源时，服务端和客户端需要如何沟通，如何授权信任。 CORS的原理是：使用 `http自定义头部` ，请求头附带客户端信息，服务端验证，并且返回响应头告诉客户端是否允许访问。 所以该标准需要客户端和服务端同时配合支持，当前所有的浏览器都支持该标准。 CORS 对于用户来说是无感知的，由`浏览器自动完成` 。 因为当前所有浏览器都支持该标准，并且由浏览器自动完成检测，所以当我们需要使用CORS的时候，只需要由`服务端改动，前端不需要改动`。 CORS将http请求分为`简单请求`和`非简单请求`。 浏览器对于两种类型的请求的处理步骤有一些不同：  
**简单请求** `简单请求`：从名字来理解，就是发送请求的类型或者数据不复杂。 必须`同时满足`以下两个条件的请求，才是简单请求  

*   请求方法只能是在以下三种之中。
    *   GET
    *   POST
    *   HEAD
*   HTTP头部信息不自定义，也就是只能设置默认字段的信息
    *   Accept
    *   Accept-Language
    *   Content-Language
    *   Last-Event-ID
    *   Content-Type 只限于三个值 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

  
处理步骤：

1.  浏览器在Http头部带上原始域的标识 `Origin`
2.  服务端根据该标识来判断是否需要信任授权，如果信任就在响应头部返回相同的标识。
3.  浏览器判断响应头是否匹配，做相应结果处理 默认情况下 请求和响应都不带cookies

如果需要附带cookies信息 ajax的 `withCredentials` 设置为 true 服务端 响应头需要增加 `Access-Control-Allow-Credentials: true` **非简单请求** 处理步骤：

1.  在发送真正请求之前，会先发送一次`预检`请求，来判断服务端是否支持非简单请求的类方法。`预检` 请求包含`跟简单请求一样的Origin`、`Access-Control-Request-Method 真实请求的方法 如PUT`、`Access-Control-Request-Headers自定义复杂头部（可选）`
2.  预检通过之后，浏览器会再次使用`真实请求方法`发起请求

**实践** 我们先配置两个网站`www.siam.com` `www.siam2.com`

> 因为域名不同，所以是非同源请求，会产生跨域。

在siam网站写下index.html文件，让它使用ajax去请求`siam2`网站的内容。

```markup
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>首页1</title>
</head>
<body>
        <h1>这是原始页面的内容</h1>
    <script src="https://cdn.bootcss.com/jquery/3.4.0/jquery.min.js"></script>
    <script>
    $(function(){
        $.ajax({
            url : "http://www.siam2.com/index2.php",
            success:function(res){
                $('body').html(res);
            }
        })
    })
    </script>
</body>
</html>
```

  
  
siam2的index.php

```php
<?php
echo "来自index2.php的内容";
```

访问index.html。会看到浏览器已经发送了请求，但是产生了报错

> (index):1 Access to XMLHttpRequest at 'http://www.siam2.com/index2.php' from origin 'http://siam.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

![dom运行结果](http://yancoo.cn/uploads/images/201904/20_2.png) 因为我们还没有在服务端中信任`www.siam.com`，所以浏览器拿不到信任站点信息，跨域请求失败。 但我们可以看到 http的请求码是200，代表请求成功，在preview中也可以看到php脚本的正常返回，所以 `跨域请求失败，php脚本也会正常运行结束`。 接下来我们在服务端添加信任siam网站，是需要在响应头中增加字段，来添加信任站点的域名。

```php
<?php
header('Access-Control-Allow-Origin:http://www.siam.com');
echo "来自index2.php的内容";
```

  
![ajax运行结果](http://yancoo.cn/uploads/images/201904/20_3.png)  
到这里就跨域请求成功了。但这仅仅是简单请求的场景下，我们还要来测试一下非简单请求的情况。  
因为简单请求必须是HEAD，GET，POST其一，所以我们这里直接使用`PUT`方法来测试就可以出现非简单请求的场景了。当然你也可以自定义HTTP头部来实现非简单请求。 我们把index.html的ajax方法改为put 然后请求

```
$.ajax({
    url : "http://www.siam2.com/index2.php",
    type: "PUT",
    success:function(res){
        $('body').html(res);
    }
})
```

  
![ajax运行结果](http://yancoo.cn/uploads/images/201904/20_4.png)  
可以看到在请求中，我们填的是`PUT`，但是这里产生的却是`OPTIONS`，前面我们也说了，非简单请求会先产生一次`预检`请求，带上origin和真实的方法 `在这里是PUT` ，服务端验证通过了origin和方法之后，浏览器才会使用真实的方法`PUT`发送一次请求。 我们还没有在服务端返回头部告诉浏览器说我们支持PUT方法，所以浏览器这里拿不到权限，报错了。 我们在服务端的代码添加头部

```php
<?php
header('Access-Control-Allow-Origin:http://www.siam.com');
header('Access-Control-Allow-Methods:PUT,DELETE'); // 需要同意两种类型，就用逗号隔开

echo "来自index2.php的内容";
```

到这里就可以正常的请求了，但是可以在浏览器中看到，产生了两次请求，也就是说php脚本执行了两次。 我们例子中只是简单输出一个字符，如果是查询数据库等操作呢？ 是不是就多出了一次无用的请求。 所以我们可以在服务端拦截预检请求，直接返回同意访问的头部，后面的脚本就不需要执行了。 还有前面的简单请求，哪怕是还没有添加信任，跨域请求失败，脚本也一样会运行。

> 这是因为`http协议`并没有跨域的概念，请求发送了就会执行，而到达了浏览器的时候，才由浏览器解析响应头，查看是否有相应的字段来决定要不要继续执行。

我们可以将脚本优化一下

```php
<?php
// 如果不是同意的来源 就不用运行了
if (strpos($_SERVER['HTTP_ORIGIN'], 'http://www.siam.com') === false){
    die;
}
header('Access-Control-Allow-Origin:http://www.siam.com');
// 如果是预检请求，则通知信任即可，不需要执行脚本。
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS'){
    header('Access-Control-Allow-Methods:PUT,DELETE');
    die;
}

echo "来自index2.php的内容";
```

同时我们可以看一下，是否每一个`非简单请求`都需要先发送预检请求。我们在一个页面连续请求两次

```
$.ajax({
    url : "http://www.siam2.com/index2.php",
    type: "PUT",
    success:function(res){
        $('body').html(res);
        $.ajax({
            url : "http://www.siam2.com/index2.php",
            type: "PUT",
            success:function(res){
                $('body').html(res);
            }
        })
    }
})
```

发现浏览器只有请求了3次：1次OPTIONS，2次PUT。 ![ajax运行结果](http://yancoo.cn/uploads/images/201904/20_5.png)

> 在一个页面中，预检操作只需要进行一次。

到这里CORS的基本就弄懂了。 优点 \* CORS 通信与同源的 AJAX 通信没有差别，代码完全一样，容易维护。 \* 支持所有类型的 HTTP 请求。 缺点 \* 第一次发送非简单请求时会多一次请求，增加服务器压力。

#### JSONP 跨域解决

在浏览器中，我们可以使用`script`标签来加载js脚本，如果使用过cdn的童鞋应该知道，我们可以直接填写不同源的地址，因为浏览器允许`script`加载跨域资源。我们可以通过该标签来加载动态脚本，但是`需要服务端调整数据结构`。 相当于让服务端输出`调用js函数`的语句 首先我们在html中写下以下代码，创建一个script，调用动态脚本

```markup
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Siam - script 同源解决</title>
</head>
<body>
    <h1>这是原始页面的内容</h1>
    <script src="https://cdn.bootcss.com/jquery/3.4.0/jquery.min.js"></script>
    <script>
    // 这里需要先写好相应的回调处理函数，然后服务端的脚本调用 传参
    function test(text){
        $('body').append(text);
    }

    $(function(){
        $("body").append("<script src='http://www.siam2.com/script.php'><\/script>");
    })
    </script>
</body>
</html>
```

服务端脚本

```php
<?php
echo "test('这是返回内容')";
```

这样子也可以正常的运行返回 优点 \* 兼容性好，现在主流的跨域解决方案之一 缺点 \* 只支持get \* 要确定 JSONP 请求是否失败并不容易。虽然 HTML5 给 script 标签新增了一个 onerror 事件处理程序，但是存在兼容性问题

#### 服务器代理

除了使用以上的两种方案，我们还可以在nginx配置反向代理，在www.siam.com下某个路径代理到www.siam2.com即可 我们打开nginx.conf

```
server {
    listen       80;
    server_name  www.siam.com;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }

    location ^~ /apis {
        proxy_pass http://www.siam2.com;
    }
}
```

通过反向代理，我们就可以通过 www.siam.com/apis/index2.php 这个路径来访问原来部署在www.siam2.com下的内容。 这样子就是同源请求了。