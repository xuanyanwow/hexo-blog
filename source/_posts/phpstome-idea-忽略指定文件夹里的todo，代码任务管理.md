---
title: phpstome/idea 忽略指定文件夹里的todo，代码任务管理
tags:
  - PHP
  - 常见问题
id: '267'
categories:
  - - PHP
  - - 常见问题
date: 2020-05-21 10:57:13
---

# 使用TODO管理自己的时间和任务

## 什么是todo

常见的名词是：TODO LIST ，一般出现在“个人规划”中出现，记录一定周期内需要完成的任务、完成任务情况 可能如下图 ![](https://www.siammm.cn/wp-content/uploads/2020/05/wp_editor_md_4766caac363d5574d2c00e24a6c6119f.jpg) 在代码中，我们通过在注释中编写todo，来记录某些`待完善`功能点 如下 ![](https://www.siammm.cn/wp-content/uploads/2020/05/wp_editor_md_a74f8911f31acf930ad028638283678a.jpg)

## phpstorm 中的todo

格式为 两个斜杠加todo名词 // todo 或 // TODO 采用大写小写都正常工作，看个人喜欢。 然后在左下角，有一个`TODO面板`，我们可以在这个面板中查看整个项目中待完成的任务 ![](https://www.siammm.cn/wp-content/uploads/2020/05/wp_editor_md_e3da39e6d5f0dd900055525e34f98ebc.jpg)

## 出现的问题

我们使用composer等包管理，引入他人的包，他们的代码也有包含todo任务注释，我们在这里面板也把他们的任务统计了，不方便我们自己的项目开发管理。 所以我们需要把他们的文件夹忽略（或者说 只监听我们自己的项目目录） siam博客 原文地址： https://www.siammm.cn/archives/267

## 只监听自己设置的目录

我们在TODO面板中，切换到`Scope Based`中，可以看到这里的`Scope`默认是All Places 也就是全部文件，默认预设了好几个选项，大家可以一一测试 我们这里讲一下怎么自定义目录规则 ![](https://www.siammm.cn/wp-content/uploads/2020/05/wp_editor_md_47a5a3757423f72879391ec78b4498a8.jpg)