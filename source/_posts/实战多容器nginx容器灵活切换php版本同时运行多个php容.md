---
title: '实战 - 多容器,Nginx容器灵活切换PHP版本!同时运行多个PHP容器'
tags:
  - Docker
id: '159'
categories:
  - - Docker
date: 2019-08-30 10:44:06
---

容器link原理 =========== 在前面一章中，我们使用 `--link`来将php容器和nginx容器关联在一起，并在nginx容器中的配置写下了如下代码，写下的php001就是我们在--link中设置的别名，其实这是通过本地host实现的。

```
{
    try_files $uri =404;
    fastcgi_pass php001:9000;   #极其重要
    fastcgi_index index.php;
    include /etc/nginx/conf.d/fastcgi_params;  #这里也是需要注意的，之前就是在这里还惨我了，需要绝对路径。不然路径默认从nginx的配置目录开始。
}
```

我们进入容器，并运行

```
$ cd /etc/
$ vim hosts
```

便可以看到设置的本地host。

# 实现灵活切换php版本

我们先拉取两个不同版本的php镜像

```
$ docker pull php:7.0-fpm
$ docker pull php:7.2-fpm
```

然后开启两个容器

```
$ docker run --name php70 -v /home/wwwroot/service_config/php_config:/usr/local/php/etc -v /home/wwwroot/:/home/wwwroot/ -d php:7.0-fpm

$ docker run --name php72 -v /home/wwwroot/service_config/php_config:/usr/local/php/etc -v /home/wwwroot/:/home/wwwroot/ -d php:7.2-fpm
```

注意挂载目录不需要同我的命令一致 自己修改 因为默认都是9000端口 所以不能同时运行 ，创建完一个先stop 创建第二个。需要同时运行的看下面的步骤↓↓↓ 运行需要的php版本容器 运行容器后查看容器的ip $ docker inspect php70 查找以下行 "IPAddress": "172.17.0.2", 如果要在nginx使用别名去访问  则需要把hosts文件挂载出来  因为修改了nginx配置需要重启机器，手动修改是没用的。！ 接着开启nginx容器，然后进入nginx容器，在nginx的配置文件里修改，（我已经挂载在主机本地目录，详细看前一章节）

```
{
    try_files $uri =404;
    fastcgi_pass 172.17.0.2:9000这里修改了;
    fastcgi_index index.php;
    include /etc/nginx/conf.d/fastcgi_params;
}
```

Esc 然后:wq 保存退出重启即可   $ docker restart nginx001 切换成7.2的步骤：

```
$ docker stop php70
$ docker start php72
$ docker inspect php72
```

得到容器运行ip，进入nginx 修改配置 （因为两个容器不是同时运行，当70版本的容器结束，再开启72版本的容器 还是同一个ip 所以不需要修改配置） 假设ip更换了 则需要修改配置然后重启机器

# 同时运行多个PHP容器

在开启容器的时候需要使用不同的外网ip，因为php-fpm默认监听的是9000端口 所以运行的命令就成了这样子

```
$ docker run -p 9001:9000 --name php70 -v /home/wwwroot/service_config/php_config:/usr/local/php/etc -v /home/wwwroot/:/home/wwwroot/ -d php:7.0-fpm

$ docker run -p 9002:9000 --name php72 -v /home/wwwroot/service_config/php_config:/usr/local/php/etc -v /home/wwwroot/:/home/wwwroot/ -d php:7.2-fpm
```

这里的9001和9002是你的宿主机没有被占用的端口即可 可以看到两个php容器已经同时可以运行了  $ docker ps  在nginx.conf配置中使用对应容器的ip:9000即可使用对应的PHP版本去编译。 记得修改完IP需要重启nginx！ 容器端口号和主机端口号关系的理解 =================== Docker的所有容器都相当于在同一个`内网`的很多机器 所以每一个容器都有一个ip   每个机器都有自己的端口使用情况    
所以不同容器可以使用一样的端口 ，所以我们两个容器都使用php-fpm默认的9000端口并没有冲突。 但是每一个容器都需要映射一个端口到主机上，这个端口是在主机上的，所以不能重复， 我们使用9001和9002。