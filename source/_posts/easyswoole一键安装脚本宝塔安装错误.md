---
title: 'easyswoole一键安装脚本,宝塔安装错误'
tags:
  - PHP
id: '186'
categories:
  - 后端
date: 2019-10-09 12:09:06
---

# 常见问题

在新接触easyswoole的phper中，经常遇到以下几个问题

*   安装步骤多 麻烦
*   宝塔等集成环境下容易出错
*   自己会安装，但是懒 有没有一键的？

# 开始创造

本人作为easyswoole开发组组员之一。为生态的完善和偷懒着想，在某一天讨论中就开始有了这个想法。 并且写下了这个小脚本

> 需要注意的是，这只是几句很简单的命令，并且在文档中都有出现。只是文档有比较多的场景描述，可能导致有些新人没有细心观看到。

在宝塔面板中，如果按照easyswoole文档第一步骤进行安装的话，是会产生错误的，在文档后续步骤会有解决方案，但是很多新人到了报错这里就不看了，或者就走了弯路。 使用这个脚本，可以直接安装成功，比较方便 最大的作用还是偷懒吧~

# 正文

```sh
#!/bin/bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/;
composer require easyswoole/easyswoole=3.x;
php vendor/easyswoole/easyswoole/bin/easyswoole install;
php easyswoole start;
```

后续会将脚本放在服务器中，提供下载，真正达到一行命令安装。

> 2019-10-10更新

已经将脚本放到es官方资源中 只要在linux中运行以下一行命令即可完成安装！Hello World!

```sh
wget https://www.easyswoole.com/install.sh && sed -i 's/\r$//' install.sh && chmod +x ./install.sh && ./install.sh
```

# 注意点

该脚本会把全局的composer镜像切换为阿里云。 安装好了会默认自动启动