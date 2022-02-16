---
title: HTTP协议中的GET、POST请求方法的区别
tags:
  - 后端
id: '54'
categories:
  - 后端
date: 2019-06-23 14:20:31
---

在我们日常打开网页、对接接口时，使用到的一般都是HTTP协议。 HTTP 的工作方式是客户端与服务器之间的请求-响应。 HTTP 请求方法有：HEAD、PUT、DELETE、OPTIONS、CONNECT 两种最常被用到的HTTP方法是：`GET` 和 `POST`。 本篇文章讲讲`GET`和`POST`两种请求方法的区别。

### 在浏览器上表现的区别

**GET**

*   GET 请求可被缓存
*   GET 请求保留在浏览器历史记录中
*   GET 请求可被收藏为书签
*   GET 请求参数在URL中的是可见的
*   GET 请求有长度限制

**POST**

*   POST 请求不会被缓存
*   POST 请求不会保留在浏览器历史记录中
*   POST 不能被收藏为书签
*   POST 请求参数在URL中的是不可见的
*   POST 请求对数据长度没有要求

在浏览器上的表现是最表面的，所以大部分的人都已经知道。 简单的就不再说了，这里再说说`请求参数的可见性`和容易让人产生误区的`数据长度限制`

##### 请求参数可见性

在GET请求中，查询字符串是在 GET 请求的 URL 中发送的

```
index.php?content=这是get方式里面的一个字段的值
```

get方式请求头和请求体 ![get方式请求头和请求体](https://www.siammm.cn/wp-content/uploads/2019/06/9488ac79d8b647b56a1315ba90f5b468.png) 在POST请求中，查询字符串是在 POST 请求的 HTTP 消息主体中发送的

```
POST index.php HTTP/1.1
Host: www.siammm.cn

content=这是post方式里面的一个字段的值
```

post方式请求头和请求体 ![post方式请求头和请求体](https://www.siammm.cn/wp-content/uploads/2019/06/6a48dd20072a9e92c2751174d59fa58d.png)

> 因为post请求是将参数放在HTTP主体中，所以在常规浏览器地址栏上是看不到参数的，这就是请求参数在URL中的可见性的不同。

两种请求方法请求头和请求体的对比 可以看到参数存放位置不一样 ![两种请求方法请求头和请求体的对比](https://www.siammm.cn/wp-content/uploads/2019/06/4d8aea9ad35575c5d2fbc4f5bec25b3c.png)

##### 数据长度限制

从上面的请求参数可见性我们已经知道 `GET`请求的所有参数都是在URL中发送的 我们常说的GET请求有数据长度限制，其实那只是`浏览器对URL长度的限制` 嗯，这里要看清一个点：是浏览器 而不是HTTP协议的规定，同时在web服务器上也有对于长度的限制（这些下面的文章会讲）

> 因为post请求是将参数放在HTTP主体中，所以不会受到此限制

不同的浏览器对于URL长度的限制是不同的，这个可以自行测试得出 首先：我们找到一篇长文章。（文章可以从短到长进行测试，会从正常搜索然后到达url长度限制） 然后打开`https://www.baidu.com/s?wd=文章内容` 这个网址，进行百度搜索。 ![siam博客,get/post的区别](https://www.siammm.cn/wp-content/uploads/2019/06/1d6821d226b21467b6478a492b153ac0.png) 看图片上的文字说明 把搜索内容替换成超级长的文章，再怎么按回车或者跳转按钮都没效果，页面还是保留一开始的。 也就是说`url的长度已经到达了浏览器的限制`，所以`浏览器不处理该请求`了。 这张截图是在win10自带的Edge测试的，同时在搜狗浏览器也是一样的情况。 但是在谷歌Chrome浏览器就是另一种场景了。浏览器会正常跳转，但是却不能正常响应。（该情况涉及到的知识，在下面会讲） **先附带一下百度上提供的资料** 各个浏览器对于url长度的限制

1.  IE浏览器对URL的长度现限制为2048字节(自己测试最多为2047字节)。
2.  360极速浏览器对URL的长度限制为2118字节。
3.  Firefox(Browser)对URL的长度限制为65536字节。
4.  Safari(Browser)对URL的长度限制为80000字节。
5.  Opera(Browser)对URL的长度限制为190000字节。
6.  Google(chrome)对URL的长度限制为8182字节。

### 在http协议上的规定

HTTP 协议没有规定URL的最大长度，也没有规定HTTP请求体的最大长度。 所以在HTTP协议上，对于GET请求和POST请求的数据长度，是`没有限制`的。 但规定服务器如果不能处理太长的URL，就得返回414状态码（Request-URI Too Long）。 这也是我们上面说到的，在谷歌Chrome浏览器中，会正常跳转，但却无法正常响应的结果。 ![414状态码](https://www.siammm.cn/wp-content/uploads/2019/06/5a6874a59f4f67fec448a2f0b52b0d7b.png)

> 请注意，该结果不是由http协议直接返回，而是规定服务器可以这样子处理（不是强制性 看你web服务器想要处理多长的url），所以该情况是属于web服务器上的限制，在下面知识会继续讲解

### 在web服务器配置限制url长度

如果请求正常通过了浏览器的限制，则会发送到web服务器上了（如apache nginx） 在进入web服务器时，也需要进行一次限制的检测。 如果我们的服务器不想服务那么长的url，可以在这里通过修改配置参数，来决定最大接收的长度。 如果超过该长度，则遵循HTTP协议，返回414状态码，返回响应并终止此次请求。 **以nginx为例** 在nginx的配置参数中，有两个配置项可以决定要服务的url长度。 因为url长度是属于http请求头的一部分，所以配置项上的体现是以控制请求头最大长度的。 这两个配置项是

```
client_header_buffer_size
large_client_header_buffers
```

首先看第一个参数：`client_header_buffer_size` 当请求进来的时候，web服务器会根据这个参数分配一块内存，用来容纳请求的请求头。 接着是第二个参数：`large_client_header_buffers` 如果分配的内存无法容纳请求头，则会根据该参数的，再次分配大一点的内存。 如果还是不够容纳，则已经超出了web服务器设置的服务长度，就会返回给客户端414状态码。

> 如果只把`client_header_buffer_size`设置小了，`large_client_header_buffers`还是很大的话是没有用的，还会浪费多一次分配内存的操作。

我这里将两个参数都设置成了1k

```
client_header_buffer_size 1k;
large_client_header_buffers 4 1k;
```

（改完配置记得重启服务器） 然后进行一个简单的get请求，带上1024个字节的参数(或者更长)，服务器返回`414 Request-URI Too Large` 到这里，在服务器上限制get传递的数据长度的操作就完成了。

> 如果是apache或者其他的web服务器，也是一样的原理来进行设置。

## 总结

*   GET 请求会被浏览器缓存，POST 请求不会
*   GET 请求会被浏览器保留在历史记录中，POST 请求不会
*   GET 请求可以被浏览器收藏为书签，POST 请求不能
*   GET 请求参数在URL中可见，POST 请求参数不能
*   GET 请求对数据长度有要求，POST 请求没有（这里指的是浏览器对url长度的要求）
*   在HTTP协议中，对于GET、POST的数据长度是没有限制的
*   在WEB服务器中，可以通过配置参数来决定要服务的URL长度限制（通过是控制最大请求头的长度）POST请求是将参数放在请求体中，所以不受该长度限制
*   如果WEB服务器不能处理过长的URL，根据HTTP协议需要返回414状态码。