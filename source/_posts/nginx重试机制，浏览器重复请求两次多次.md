---
title: Nginx重试机制，浏览器重复请求两次多次
tags:
  - nginx
id: '177'
categories:
  - 后端
date: 2019-09-17 15:05:49
---

# 前言

在研究nginx的时候，偶然看到网上前辈一篇解决问题的实战记录文章，稍微整理一下，学习补充一下知识点。

# 场景还原

问题 用户再浏览器里执行了一次http请求，结果后端服务器执行了两遍，如果这次请求是Insert操作，可想而知，数据库会多出一条一模一样的记录来。

*   网关用Nginx做了反向代理和负载均衡，Nginx下挂着两台阿里云ECS服务器，每台机器上都装着Tomcat，用户打开浏览器，点击页面，访问后端接口，查看Nginx的access.log,结果这一条请求打在了两台服务器上。

# 问题剖析

nginx的重试机制就是容错的一种，在nginx的配置文件中，proxy\_next\_upstream项定义了什么情况下进行重试，官网文档中给出的说明如下：

```
Syntax: proxy_next_upstream error  timeout  invalid_header  http_500  http_502  http_503  http_504  http_403  http_404  off 
Default:    proxy_next_upstream error timeout;
Context:    http, server, location
```

*   默认情况下，当请求服务器发生错误或超时时，会尝试到下一台服务器。
*   问题找到了，原因是Nginx配置文件中，超时时间太短了：proxy\_connect\_timeout 20;；在Nginx的默认配置是：在客户端请求服务器超时的情况下，Nginx会自动转发该请求到另外一台服务器上，这是Nginx的一种容错机制，所以Nginx的访问日志中会出现同一条请求而两台服务器都执行了一遍的情况，这样以来，程序如果没有做幂等性操作的话数据库会出现两条记录。
*   还有一个参数影响了重试的次数：proxy\_next\_upstream\_tries，官方文档中给出的说明如下：

```
    Syntax: proxy_next_upstream_tries number;
    Default:    proxy_next_upstream_tries 0;
    Context:    http, server, location
    This directive appeared in version 1.7.5.
```

# 调整

本来就是Nginx的一种容错机制，这种机制在查询操作还是挺好的，如果是插入操作，那就有点问题了，如果这条插入的请求特别耗时，并且时间超过Nginx的proxy\_connect\_timeout时间设置，Nginx会自动将该请求转发集群中的另外一台服务器的。但是我们不能将这种机制关闭，关闭以后会影响Nginx效率的，那怎么办哪？于是想出了一个临时解决方案，专门针对耗时时间长的几个接口做一下过滤，也就是说，在Nginx的server配置标签中，专门对几个特定的url过过滤，关闭Nginx的重试机制，配置如下

```
server {


       location ~ /api/insertData {
              proxy_connect_timeout 60;
              proxy_send_timeout 60;
              proxy_read_timeout 60;
              proxy_next_upstream off;

        }
 }
```

也可以直接关闭重试机制

```
proxy_next_upstream off;
```