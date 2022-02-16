---
title: JS的字符串插值，变量长文本换行
tags:
  - 前端
id: '185'
categories:
  - 前端
date: 2019-09-27 00:18:09
---

# 苦逼的PHPer要写前端

作为一个PHPer，经常需要在html中写js jq来解析数据，形成列表、选项等等。 （谁让我们PHPer还要兼顾页面呢？？ 又不会Vue，只能这样子讨讨生活。） 那么就经常遇到Html代码拼接，或者字符串拼接，可能是这样子的问题

```javascript
let html = "";

for(...){
    html += "<li> " + data.name + "</li>";
}
```

这种还是简单的，只有一个li，如果是2层、3层的div嵌套，那么这里就会是一团糟糕 有没有优雅一点的写法呢，比如php中的

```php
$text = <<<xml
    ....
    222
    $$$
>>>
```

# 字符串插值特性

一些语言提供了字符串插值，幸运的是，JavaScript 正是其中之一。

```javascript
let name = 'siam';
let html  = `Siam博客是一个干净的博客
   作者: ${name}
   年龄: 21
`;

alert(html);
```

我们将会得到这样子的结果 ![](https://www.siammm.cn/wp-content/uploads/2019/09/1f66b1250f9c1ff3e1efffd17a4b37c5.png) 可以看到，在字符串中，我们使用`${}`来使用变量。 这里也可以使用对象的属性 比如`$(this.job)`等等 非常的方便 优雅 是一个你必须知道的JS特性！！！