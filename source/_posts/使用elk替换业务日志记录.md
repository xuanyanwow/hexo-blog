---
title: 使用ELK替换业务日志记录
tags: 
  - 架构
id: '320'
categories:
  - - 运维
date: 2022-02-15 12:10:20
---

公司在快速开发阶段，将日志存入了Mysql，不利于做各做条件的检索查询。速度较慢。使用ELK方案，优化该部分逻辑。并开发toB日志自查系统，提供给客户快速对接。

# ELK

Elasticsearch、Logstash 和 Kibana 三个软件的简称

## 简单架构图

![](http://blog.siammm.cn/wp-content/uploads/2022/02/wp_editor_md_763628059c5ca787cf55fd1e6e5a9663.jpg) 原文地址-> https://blog.siammm.cn/archives/320 来自：siam博客。简单理解和笔记

## 资源记录

*   Elasticsearch： https://www.elastic.co/cn/downloads/elasticsearch
*   Logstash：https://www.elastic.co/cn/downloads/logstash
*   Kibana：https://www.elastic.co/cn/downloads/kibana
*   elasticsearch-head：https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm/
*   Elasticsearch: 权威指南 ：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html
*   Elasticsearch-php：https://www.elastic.co/guide/cn/elasticsearch/php/current/index.html

### Elasticsearch

这个不用说，必须安装

### Logstash

用于收集日志，在程序和ES之间，可以让程序产生的日志不用直接提交到ES，减少传输时间。 程序可以插入到Redis，Kafka Logstash相当于`队列消费者`，将日志写入ES

### Kibana和elasticsearch-head

可视化客户端，不是必须安装，如果是公司内部自己用的日志，那么可以直接安装，简化开发流程。 如果是对B业务，需要将日志`在业务开发平台，提供给客户查询`，则可以不安装Kibana，需要自行开发业务系统，做`权限管理、查询Elasticsearch和展示`