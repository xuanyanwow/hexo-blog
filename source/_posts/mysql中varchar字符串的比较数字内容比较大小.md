---
title: 'Mysql中varchar字符串的比较,swoole预处理参数绑定'
tags:
  - mysql
  - PHP
  - swoole
id: '95'
categories:
  - 数据库
date: 2019-07-18 23:17:26
---

# 写在前面

事情起因： 使用了一个varchar类型的字段储存数字值。 在根据该字段进行大小筛选的时候，出现该问题。 类似`select * from sheets where s_status > 3`

# 分析

php调用时条件传的是数字类型 组件生成的SQL语句直接执行正常

# 排查

打开了mysql的运行日志，分析到最终运行的sql语句大概如下

```
where s_status > '3'
```

使用的是php swoole，预处理。 解决有两条路

*   mysql的字段类型改为数字
*   研究swoole的参数预处理问题，可以测试普通PHP的预处理是否也有问题

能学习的点

*   字符串类型字段的比较规则

### mysql中字符串类型字段的比较规则

找了一圈资料，相关文章比较少，终于在比较不起眼的角落里找到资料。

> 字符串比较 是根据ascii码比较 只有当第一个字符相同才对比第二个字符。以此类推。 在线转换ascii码工具 https://www.iamwawa.cn/ascii.html

假设我们现在表中有2条字段

id

s\_status

s\_name

1

4

测试1

2

258710588

测试2

如果按正常的sql执行 我筛选>3应该是2条结果都有，但是程序运行只能得到1条结果： id = 1的数据 那么我们上面说到 字符串的比较规则，从第一个字符开始比较，只有第一个字符相等 才会比较第二个字符... '4' > '3' 通过

```
字符 4 对应的ASCII码为 52
字符 3 对应的ASCII码为 51
```

'258710588' > '3' 不通过

```
字符 2 对应的ASCII码为 50
字符 3 对应的ASCII码为 51
此时已经有结果 不需要对比第二个字符
```

如果是'31' > '3' 也会通过

```
第一个字符相同，则对比第二个字符，而3没有第二个字符了 所以是小于。
```

### 研究：php预处理时，参数绑定

```php
// 省去连接等等
// 预处理及绑定
$stmt = $conn->prepare("SELECT * FROM `siam_test_bug` WHERE `s_wechat_cross_status` > ? ");

$condition = 3;
$stmt->bind_param("i", $condition); // 生成语句 > 3
$stmt->bind_param("s", $condition); // 生成语句 > '3'  就变成了字符串比较 不正常 

$res    = $stmt->execute();
$result = $stmt->get_result();

while ($myrow = $result->fetch_assoc()) {
    var_dump($myrow);
    echo "<br/>";
}
```

### 确定swoole

经过开发组内各位大哥的协助确定，是swoole的参数绑定，不支持决定类型，所以会出现这个坑。 已经提交swoole rfc 待解决