---
title: 'git,程序配置文件管理,忽略本地更改'
tags:
  - GIT
  - 项目管理
id: '116'
categories:
  - 项目管理
date: 2019-08-07 09:28:01
---

# 写在前面

在我们开发过程中，经常会出现数据库配置文件、redis环境配置文件等。 在不同的开发环境（同事与同事之间 开发与测试与生产环境）大概率是不同的。 如果每个人都按普通的流程，Pull然后修改成自己本地的，没有忽略监听更改。 那么当他提交代码时，经常会把配置文件也上传到git仓库中。 会影响其他人的开发。 所以我们应该这样子做：`git仓库提供一份配置文件的基础模板，每个人都拉取到本地修改但是要忽略本地更改监听。`

# 操作步骤

*   1.建立git仓库
*   2.创建基本配置文件模板
*   3.提交并推送到仓库
*   4.本地忽略监听
*   5.服务器部署，拉取仓库
*   6.忽略监听
*   7.更改配置文件

# 协助资料

忽略某个文件或者目录

```
git update-index --assume-unchanged [file_path]
git update-index --assume-unchanged -f [dir_path]
```

查询已经被忽略的文件列表

```
git ls-files -v  grep '^h\ '
```

提取文件路径

```
git ls-files -v  grep '^h\ '  awk '{print $2}'
```

查询已经被忽略的文件列表并取消忽略

```
git ls-files -v  grep '^h'  awk '{print $2}' xargs git update-index --no-assume-unchanged  
```