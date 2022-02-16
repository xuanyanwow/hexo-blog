---
title: 'Navicat数据同步,主键重复无法插入'
tags:
  - mysql
id: '190'
categories:
  - 数据库
date: 2019-10-16 09:29:27
---

# 基础知识

`Navicat`是一个非常好用的可视化mysql管理软件（其他数据库也有对应版本的支持） 它拥有非常丰富的功能，结构同步、数据同步、数据传输、进程监控、数据导出导入等等 但这是一个付费软件，新用户可以免费试用，这个问题是笔者在以前试用处理数据的时候遇到的。

# 问题

在A和B机器上分别有`结构相同`，`数据不完全相同`的两个数据库 比如 A机器上的表

id

name

age

1

宣言

21

2

Siam

21

B机器上的表

id

name

age

1

宣言B

22

2

SiamB

22

现在要实现的点是：将两个表的数据合并为一个，以后统一使用一个数据库即可。 在使用数据同步的时候，能筛选出不同数据，但是却不能运行，因为筛选出的数据主键在第二个数据库中已经被占用。 使用软件，选择A同步到B，那么会筛选出id 1 2两条数据 生成的语句却是以下这样子的

```sql
insert into 表名 (id, name, age) values (1, '宣言', 21) ...
```

在B中运行这样的语句。主键id重复，自然就会产生失败了

# 问题怎么解决

因为我这里需要处理的数据量比较小 我这里采用的是比较直接的方法，如果有更好的方式，请大家在评论中留言，一起探讨

*   在A中筛选出差异数据（可以根据软件或者其他筛选条件等）
*   数据压缩成json字符串，大概如图所示 ![](https://www.siammm.cn/wp-content/uploads/2019/10/e2aca9df4caaed6b9aca85e491140a1b.png)
*   json文件上传到B机器中，写一个脚本，读取json 并且删除id主键，重新生成insert语句

```php
$data = file_get_contents("./data.txt");
$data = json_decode($data, true);

foreach ($data as $key => $value){
    unset($value['id']);
    $sql = "INSERT xxx " . array_to_sql($value);
    mysql_query($sql,$db);
}
echo "ok";
```

因为是一个临时的脚本，这里就没有引入其他类库，用了面向过程的php代码来完成，大概思路就是这样子。

> 如果是使用mysqladmin，还可以在导出的时候直接选成导出php array 也许会更方便哦！