---
title: Mysql 乘法除法精度不一致，除法后四位小数
tags:
  - mysql
  - PHP
id: '242'
categories:
  - 数据库
date: 2020-03-06 16:04:33
---

# 问题

今天在写项目功能的时候，有一个统计金额的情况，然后需要进行单位转换，所以写下了大概如下功能的语句，但得到的数据为小数点后4位精度，正常我们只需要2位就足够。

```sql
select total_fee / 100 from orders
```

继续排查寻找资料，进行精度转换，找了一圈的资料都不太满意，继续进行测试

# 测试

测试bug和未知情况，我们一定要`最小复现，精简测试`，防止其他语句对结果产生干扰。

```sql
select 1 / 100;
// 得到 0.0100
```

```sql
select 1 * 0.01;
// 得到 0.01
```

并且在3/4台设备上运行，不同mysql版本环境都是这样子的结果。 所以初步得知`Mysql中，乘法和除法对小数点后的精度不一致` 在国内的论坛中没有找到合适的资料，于是到国外论坛寻找，提问，交流。

# 答案

首先感谢其他前辈对问题的解答和指点，我们也将尽量详细地记录问题的排查。 文明之所以能延续，是因为它们有记忆。希望文章也能帮到更多的朋友。

*   除法的精度默认是小数点后4位
*   乘法的精度使用`操作数的精度和`的方式来判断，如例子中的`1*0.01` 精度分别是小数点后0位和2位，那么就是`0+2 =2` 结果也将使用2位精度

测试

```sql
select 1.00 * 0.01;
// 结果 0.0100
```

符合上诉结论。感谢国外前辈指教。 原文链接 Siam博客 宣言博客 [https://www.siammm.cn/archives/242](https://www.siammm.cn/archives/242 "https://www.siammm.cn/archives/242")

## 除法使用2位精度

那么我们的问题 如果是要坚持用除法解决，我们可以使用函数来进行转换精度，

```sql
CAST( @x / @y AS DECIMAL(m,n) )
```

DECIMAL的参数可以百度看这个，基本创建过表结构的都能明白。 @x和@y就是除数和被除数。 同时我还提出疑问，是否能在mysql里设置默认除法精度，我们就可以不用每次sql都使用函数计算了。 前辈回复：如果你不想有时候出现出乎意料的情况，那么需要每次都强制使用类型转换。

### mysql相关说明文献

https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html

> In division performed with /, the scale of the result when using two exact-value operands is the scale of the first operand plus the value of the div\_precision\_increment system variable (which is 4 by default). For example, the result of the expression 5.05 / 0.014 has a scale of six decimal places (360.714286).

### 除法的精度规则

由上面引用的文献可知：当使用两个数值进行计算时，结果的精度由`第一个操作数的精度 + 系统变量div_precision_increment的值决定`，如我们例子中的1 精度是0，系统变量精度是4位默认，所以得到的结果是4位精度

```sql
set div_precision_increment = 2;
// 再运行select 1 / 100; 得到 0.01 符合想要的结果
```

所以我们还可以通过修改默认变量，来改变除法的默认精度。 此时我们再测试

```sql
select 1.0 / 100;
// 结果 0.010 符合上文所述结论
```