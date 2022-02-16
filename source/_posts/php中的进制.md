---
title: php中的进制和编码
tags:
  - PHP
  - 计算机基础
id: '115'
categories:
  - - PHP
  - - 杂乱知识
date: 2019-08-06 23:12:18
---

# 进制和编码的关系

> 进制是数字上的关系

*   我们日常使用的是10进制，因为我们有10个手指，这是习惯和发展使然。
*   计算机的基础是2进制，因为电路只有通电、不通电两种状态，用0、1表示。一个数字成为一个`位`

随着计算机的发展，需要表示的符号越来越多，从一开始的2位代表一字节，到后面的8位代表一字节至今。

*   其他的还有8进制、16进制等等。 进制之间的转换 [工具](https://tool.lu/hexconvert?_blank "工具")

> 编码是符号的映射表示关系

字符串在线转2进制 [工具](http://www.5ixuexiwang.com/str/binary.php?_blank "工具") 由于计算机是MG发明的，一开始的映射表是ASSIC码，用一个字节（8位）表示一个符号或者字母 比如小写字母a对应的是97 相应的2进制为`01100001` 8个位的2进制最大值是`11111111` 所以当它不够用之后，就出现了`双字节字符集`，比如GBK,Unicode等 再之后为了优化传输 出现了UTF-8,UTF-16等规范标准。 见这张我自己画的小图吧~ ![siam制作 进制和编码的关系](https://note.youdao.com/yws/api/personal/file/314F4408CF434BA78503F93FFBF0FC4E?method=getImage&version=6793&cstk=fE-i6TSJ "siam制作 进制和编码的关系")

# php中的进制转换

在php中 内置了挺多的进制转换函数

*   bindec() — 二进制转换为十进制
*   decbin() — 十进制转换为二进制
*   dechex() — 十进制转换为十六进制
*   decoct() — 十进制转换为八进制
*   hexdec() — 十六进制转换为十进制
*   octdec() — 八进制转换为十进制
*   base\_convert()– 在任意进制之间转换数字

# php中的2进制输出

在我们日常写程序的时候，我们面向的是`编码`，而不是进制。 代码会经过编译器或者解释器变成机器指令，再转换为2进制。 常见的文件编码格式现在有：`GBK`、`UTF-8` 在机器传输过程中只能2进制，不管是GBK编码还是UTF-8编码，都可能是这样子的数据`01010001111010101001111`，至于怎么解析，就看机器通信之间的规定了 从关系图中可以得知：`UTF-8是Unicdoe的实现，Unicode又兼容了assic码的定义。` 所以当我们在UTF-8文件的php程序输出小写字母`a`的时候，经过解析会转换得到97这个10进制的数。 如果要输出16进制或者2进制的数据，其实我们可以先转换为`10进制的数字`，然后使用chr()函数，转换得到assic码，输出。

> assic码在传输过程会变成2进制，与我们一开始设定的16进制或者2进制数据其实是一样的，进制是可以互相转换的。

# 简单代码

连接tcp服务器 并且发送不同进制的数据，从服务器测观察拿到的结果

```
<?php
//使用 stream_socket_client 打开 tcp 连接
$fp = stream_socket_client("tcp://127.0.0.1:6000");

//向句柄中写入数据 延迟一下 本地tcp服务器 可能监听慢
sleep(1);

// 发送16进制数据  16进制转10进制str  然后chr 转assic码 传输
// $hexStr = "A3 B5 C1";
// $hexStr = str_replace(' ', '', $hexStr);

// $send = '';
// for ($i=0; $i < strlen($hexStr); $i = $i+2) { 
//     $decStr = base_convert($hexStr[$i].$hexStr[$i+1], 16, 10);
//     $send  .= chr($decStr);
// }

// 第二种方式  chr可以直接传数字（10进制）  0x （16进制） 还有八进制
// fwrite($fp, chr(0xA3).chr(0xb5).chr(0xC1));
// sleep(5);

// 发送2进制数据  2进制转10进制str  然后chr 转assic码 传输
// $binStr = '00011111';
// $decStr = base_convert($binStr, 2,10);
// $send   = chr($decStr);

// fwrite($fp, $send);
// sleep(5);

$ret = "";
// //循环遍历获取句柄中的数据，其中 feof() 判断文件指针是否指到文件末尾
// while (!feof($fp)){
//     stream_set_timeout($fp, 2);
//     $ret .= fgets($fp, 128);
// }
//关闭句柄
fclose($fp);
echo $ret;
```