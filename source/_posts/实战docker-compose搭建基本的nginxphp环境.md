---
title: 实战 - docker-compose搭建基本的nginx+php环境
tags:
  - Docker
id: '160'
categories:
  - - Docker
date: 2019-08-30 10:44:40
---

# 安装docker-compose

简单说几句，具体可以参照官网的详细教程。

*   确保已经安装docker
    
*   从github拉取docker-compose 
    

```
# curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

*   添加可执行权限

```
# chmod +x /usr/local/bin/docker-compose
```

*   运行docker-compose

```
# docker-compose --version
docker-compose version 1.22.0, build 1719ceb
```

# docker-compose基本使用

docker-compose使用后缀为yml的文件定义你的服务容器关系 下面我们用一个nginx+php的简单例子来演示 创建项目总目录

```
$ mkdir work && cd work
```

创建代码存放目录

```
$ mkdir app
```

创建配置存放目录

```
$ mkdir config && cd config
```

创建nginx配置文件

```
$ vim site.conf
```

写入你需要的nginx服务器配置，我这里写的是

```
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /home/wwwroot/;


    location ~ .php {
        # try_files $uri =404;
        fastcgi_split_path_info ^(.+.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        include fastcgi_params;
    }
}
```

开始编写docker-compose的yml文件

```
$ vim docker-compose.yml
```

我已经写了简单的注释，其他的可选项可以在官网或者其他教程学习，这里只是演示最基本的搭建。

```
version: '2'

services:
    web:
        # 使用镜像
        image: nginx:latest
        # 端口映射
        ports:
            - "80:80"
        # 目录挂载
        volumes:
            - ./app:/home/wwwroot/
            - ./config/nginx/site.conf:/etc/nginx/conf.d/default.conf
        # 网络
        networks:
            - code-network
    php:
        image: php:7.0-fpm
        volumes:
            - ./app:/home/wwwroot/
        networks:
            - code-network

networks:
    code-network:
        driver: bridge
```

开始构建

```
$ docker-compose up -d
Starting work_web_1 ... done
Starting work_php_1 ... done
```

打开你网址 查看是否nginx是否运行成功 （这里应该会提示nginx 403，没有则可能不正常） 接着进入代码存放目录，编写第一个php文件

```
$ cd app 
$ vim index.php
```

```
<?php
phpinfo();
```

刷新网址，![](http://img.baidu.com/hi/jx2/j_0002.gif)我已经运行成功了，那你呢？ 最终的文件目录结构如下

```
work 总目录
├── app  代码存放目录
│   └── index.php
├── config 配置存放目录
│   └── nginx
│       └── site.conf
└── docker-compose.yml
```

docker-compose的其他几个常用指令 ========================== 进入你的项目目录 则运行以下其他命令 查看容器运行状态

```
$docker-compose ps
```

停止该项目运行

```
$docker-compose stop
```

关于为什么要使用docker和docker-compose将在下一章进行讨论!