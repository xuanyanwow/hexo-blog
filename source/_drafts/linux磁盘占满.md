---
title: linux磁盘占满
tags: []
id: '217'
categories:
  - - 运维
---


df -h


lsof | grep deleted


比如一个inode需要占用2kb    那么疯狂创建1kb的图片文件， 几亿个   