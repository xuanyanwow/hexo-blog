---
title: 宝塔备份大数据导入本地失败 max_allowed_packet
tags:
  - mysql
id: '305'
categories:
  - 运维
date: 2021-06-21 15:43:57
---

# 问题场景

从线上备份拉了一个备份sql压缩包，想导入到本地开发脚本测试 使用`Navicat Premium`(试用) 导入时，报错max\_allowed\_packet参数相关的错误， 原因是mysql.ini配置中，设置了相关的执行脚本的最大包，默认是2M，我导出的文件有190+M，故 执行失败

## 解决问题

最大执行包 设置为200M（具体根据你的脚本文件，可以设置大一点。）

```
set global max_allowed_packet = 200*1024*1024
```

在本地执行完就无所谓啦，如果在线上生产中使用，记得导入完成之后要把参数改为默认的哦~