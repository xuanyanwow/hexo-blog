---
title: 入门 - Docker容器数据储存和转移
tags:
  - Docker
id: '158'
categories:
  - Docker
date: 2019-08-30 10:43:03
---

# 容器和层

容器和镜像之间的主要区别是顶部的可写层。所有对容器添加新的或修改现有数据的内容都存储在该可写层中。当容器被删除时，可写层也被删除。底层镜像保持不变。 同一个镜像可以被创建多个同时运行的容器，相当于最上层的可写层不同而已，Docker版“披上羊皮的狼”。

# 容器数据储存

默认情况下，在容器内创建的所有文件都存储在可写层中。这意味着：

*   当该容器不再运行时，数据不会持久存在，如果另一个进程需要，则可能很难从容器中获取数据。
    
*   容器的可写层紧密耦合到运行容器的主机。无法轻松地将数据移动到其他位置。
    

Docker有两个容器选项可以在主机中存储文件，因此即使在容器停止之后文件仍然存在：`数据卷`和`挂载目录`。如果你在Linux上运行Docker，你也可以使用_tmpfs mount_。 下面是用法：

> `-v`或者`--volume`标志在单独容器中使用，`--mount`标志用于群集服务容器

\-v或--volume：由三个字段组成，用冒号字符（:）分隔。字段必须按正确的顺序排列，并且每个字段的含义不是很明显。

*   对于命名卷，第一个字段是卷的名称，并且在给定主机上是唯一的。对于匿名卷，省略第一个字段。
    
*   第二个字段是文件或目录在容器中安装的路径。
    
*   第三个字段是可选的，是逗号分隔的选项列表，例如ro。这些选项将在下面讨论。
    

\--mount：由多个键值对组成，以逗号分隔，每个键\=组由一个元组组成。该--mount语法比更详细的-v或--volume，但按键的顺序并不显著，并且标志的价值更容易理解。

*   该type安装件，其可以是bind，volume，或tmpfs。本主题讨论卷，因此类型始终是volume。
    
*   该source安装的。对于命名卷，这是卷的名称。对于匿名卷，省略此字段。可以指定为source或src。
    
*   将destination文件或目录安装在容器中的路径作为其值。可以指定为destination，dst或target。
    
*   该readonly选项（如果存在）导致绑定装入以只读方式装入容器中。
    
*   该volume-opt选项可以多次指定，它采用由选项名称及其值组成的键值对。
    

# 使用数据卷

数据卷是保存Docker容器生成和使用的数据的首选机制。数据卷完全由Docker管理。有几个优点： 与挂载目录相比，卷更易于备份或迁移。 可以使用Docker CLI命令或Docker API管理数据卷 卷适用于Linux和Windows容器。可以在多个容器之间更安全地共享卷。 卷驱动程序允许在远程主机或云提供程序上存储卷，加密卷的内容或添加其他功能。 新卷可以通过容器预先填充其内容。 卷不会增加使用它的容器的大小，并且卷的内容存在于给定容器的生命周期之外 docker run-d-P--name nginx-v【重点my-vol重点】:/webapp nginx docker run-d-P--name nginx--mount source=my-vol,target=/webapp nginx ①如果本地数据卷或者本地目录尚未创建，-v命令则会自动创建（此时创建的是匿名数据卷）,--mount则会报错 匿名数据卷：没有指定名称标识的数据卷，docker随机生成不重复的标识，依赖于一个容器，如果该容器消亡，则因为数据卷标识无法知道，所以无法复用。 ②如果是容器里的目录不存在，两者都会自动创建 创建数据卷 docker volume create my-vol 数据卷列表 docker volume ls 查看数据卷具体信息 docker volume inspect my-vol 删除数据卷 docker volume rm my-vol 清除无主的数据卷 docker volume prune 备份数据卷 当您需要备份，还原或将数据从一个Docker主机迁移到另一个Docker主机时，卷是更好的选择。您可以使用卷停止容器，然后备份卷的目录（例如/var/lib/docker/volumes/）。

# 使用挂载目录

将主机上的目录或者文件（绝对路径）挂载到容器指定的路径中（绝对路径），也是比较快捷高效的做法，但是数据卷拥有更好的优点，如果你在开发新的应用，请尝试使用数据卷。 docker run-d-p 8082:8080--name tomcat-mount-v/usr/local/kun/aa:/usr/local/tomcat/webapps/aa tomcat docker run-d-p 8082:8080--name tomcat-mount--mount type=bind,src=/usr/local/kun/aa,target=/usr/local/tomcat/webapps/aa tomcat