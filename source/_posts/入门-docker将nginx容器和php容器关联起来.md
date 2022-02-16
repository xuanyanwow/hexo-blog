---
title: 入门 - Docker将nginx容器和php容器关联起来
tags:
  - Docker
id: '156'
categories:
  - Docker
date: 2019-08-30 10:42:31
---

概念教程和介绍有一大堆，我就不多说了。主要记录一下操作，方便你我查阅。 首先是在菜鸟教程里看的教程，里面把各种镜像、容器的概念和基本操作都说了。但是每一步都直到怎么测试运行起来。 如：nginx，运行成功了，然后呢？没了。其他环境要怎么弄？ 在百度上找教程，看到有人先是开了一个centos镜像的容器，然后在上面跟一个基本服务器一样去yum各种环境，如php、nginx、mysql。 然后再把镜像更新commit，说是环境就搭建好了。方法① 但这样子的话，主机上pull下来的php和nginx又有什么用？（按着菜鸟教程走下来的时候pull的） 于是请教前辈，是按方法①去操作还是php,nginx各开一个容器再去连接方法②，得到了方法②的回复，于是开始了漫长的道路。

*   单容器易于分发、维护。因为它们是独立的，所有的东西都运行在同一个容器中，这点就像是一个虚拟机。但这也意味着，当你要升级其中的某样东西（比如PHP新版本）的时候，需要重新构建整个容器。
*   多容器可以在添加组件时提供更好的模块化。因为每个容器包含了堆栈的一部分：Web、PHP、MySQL等，这样可以单独扩展每个服务或者添加服务，并且不需要重建所有的东西。

需要先把php镜像和nginx镜像pull下来。查看已有镜像 docker images  先新建一个php容器

```
docker run--name php1 -v/home/wwwroot/service_config/php_config:/usr/local/php/etc -v/home/wwwroot/:/home/wwwroot/ -d php:7.0-fpm
```

\-v/home/wwwroot/service\_config/php\_config:/usr/local/php/etc这一句搭建可以省略 这是将主机的目录挂载到容器里，也就是让容器可以共享这个目录里的文件。这样子可以在主机灵活地去修改php配置，nginx同理。 坑：如果没有把配置文件挂载出来，会出现配置文件出错，然后容器就无法start了，也无法进入修改，只能删除重新建立一个容器。 接着开启nginx容器

```
docker run--name nginx   -v/home/wwwroot/:/home/wwwroot/   -v/home/wwwroot/service_config/nginx_config:/etc/nginx/conf.d   --link php1:php1   -p 80:80   -d nginx
```

  同样的两个配置挂载目录，第一个是放项目文件的，第二个是放配置文件的   然后再link刚刚开启的php容器，名称是php1，端口映射都用的80   在开启两个容器之前，需要先新建好主机目录，也就是/home/wwwroot/service\_config/nginx\_config等一列目录   然后/home/wwwroot/service\_config/nginx\_config文件夹中有两个文件（这两文件docker官方下载下来的nginx镜像是没有的）： 

```
fastcgi_params文件

fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

fastcgi_param QUERY_STRING $query_string;

fastcgi_param REQUEST_METHOD $request_method;

fastcgi_param CONTENT_TYPE $content_type;

fastcgi_param CONTENT_LENGTH $content_length;

fastcgi_param SCRIPT_NAME $fastcgi_script_name;

fastcgi_param REQUEST_URI $request_uri;

fastcgi_param DOCUMENT_URI $document_uri;

fastcgi_param DOCUMENT_ROOT $document_root;

fastcgi_param SERVER_PROTOCOL $server_protocol;

fastcgi_param HTTPS $https if_not_empty;

fastcgi_param GATEWAY_INTERFACE CGI / 1.1;

fastcgi_param SERVER_SOFTWARE nginx / $nginx_version;

fastcgi_param REMOTE_ADDR $remote_addr;

fastcgi_param REMOTE_PORT $remote_port;

fastcgi_param SERVER_ADDR $server_addr;

fastcgi_param SERVER_PORT $server_port;

fastcgi_param SERVER_NAME $server_name;

#PHP only, required

if PHP was built with--enable - force - cgi - redirectfastcgi_param REDIRECT_STATUS 200;
```

nginx.conf文件（根据你多少个网站，配置多少个。下面配置若是不懂，请查看相关文档）

```
server {

    listen 80;

    server_name www.test.com test.com;

    index index.html index.htm index.php;

    root / home / wwwroot /

        default;#

    error_page 404 / 404.html;

    location~[ ^ /].php(/  $) {

        try_files $uri = 404;

        fastcgi_pass php1: 9000;#

        极其重要fastcgi_index index.php;

        include / etc / nginx / conf.d / fastcgi_params

    }

    location / nginx_status {

        stub_status on;

        access_log off

    }

    location~.*.(gif  jpg  jpeg  png  bmp  swf) $ {

        expires 30d

    }

    location~.*.(js  css) ? $ {

        expires 12h

    }

    location~/.{deny all}}
```

新建完文件后就可以开启容器了，开启后应该就正常了，访问你的服务器ip（默认就是80端口，应该就可以正常访问nginx） 然后在刚刚的主机目录/home/wwwroot/下新建一个目录default （因为在nginx里设置的默认目录，可以自己修改） 然后新建test.php 写入php代码测试运行。