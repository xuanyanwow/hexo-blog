---
title: 'SMTP邮件服务器原理解析,php发送邮件,swoole发送邮件'
tags:
  - swoole
id: '88'
categories:
  - swoole
date: 2019-07-18 15:51:47
---

# 写在前面

电子邮件是啥就不用介绍了吧，利用程序发送邮件，可以实现：客户财务报表推送、服务异常预警、自动订阅文章等等功能。 php来发送邮件的类库网上也有。比如：PHPMailer 等 但是由于类库年代久远，或者自己使用过程中出现了一些异常错误，导致一系列苦恼。 所以写下这篇文章，来讲明SMTP邮件服务器的原理，让你在调试对接的过程中，有思路可循。

# 基础知识储备

### TCP：`TCP`是一种面向连接的、可靠的、基于字节流的传输层通信协议。

先去看看TCP协议的基础概念：比如连接握手三次、断开、收发包等等流程。 比如我们访问一个网站，使用的是HTTP协议，`HTTP协议是基于TCP协议`的。 我们本次要讲的`SMTP也是基于TCP协议`的。

### SSL：加密传输

比如我们的http网站和https网站，在传输过程中加密，会比较安全。 大部分的SMTP服务器也会要求加密传输内容。

# SMTP协议的定义

简单邮件传输协议 (Simple Mail Transfer Protocol 简称 SMTP) 是一个`相对简单`的基于`文本`的协议。 在发送方（客户端）和接收方（服务器）间`创建TCP连接`之后 那么接下来就是一个合法的SMTP会话了。（SMTP会话的本质只是一个普通TCP，只是会话的消息按照规范组装发送） 在下面的对话中，所有客户端发送的都以`C:`作为前缀，所有服务器发送的都以`S:`作为前缀。

```
S: 220 smtp.qq.com ESMTP Postfix
C: HELO smtp.other.com
S: 250 Hello smtp.other.com
C: MAIL FROM: <59419979@qq.com>
S: 250 Ok
C: RCPT TO: <123456@qq.com>
S: 250 Ok
C: DATA
S: 354 End data with <CR><LF>.<CR><LF>
C: Subject: 邮件名
C: From: Siam<59419979@qq.com>
C: To: 超级牛逼的QQ号<123456@qq.com>
C:
C: Hello,
C: 这是一个测试邮件.
C: Goodbye~
C: .
S: 250 Ok: queued as 12345
C: quit
S: 221 Bye
```

这就是发送邮件的一个简单的会话过程，其实基本上是一问一答： ① 服务端：连接上了 由服务器推送给客户端 220状态码 连接成功 这里是QQ的邮件服务器 ② 客户端：你好 我是网易的邮件服务器（或者其他...） ③ 服务端：哦好的 网易邮件服务器 ④ 客户端：我是59419979账号，我要发送给123456 ⑤ 服务端：好的、 ⑥ 客户端：我要写内容了。 ⑦ 服务端：好的，最后以 . 换行 结束哦！ ⑧ 客户端：写写写写 ⑨ 客户端：我写完了 ⑩ 服务端：好的 接收到记录了 ...然后就是退出逻辑了 当然，这其中还可以有其他逻辑和报错提示等 比如要授权登陆（QQ号不能随便让别人操作吧？） 比如要求会话是SSL加密传输的，明文传输我不接受，断开连接了哦。（下文演示会出现这个情况）

# 实战本地连接SMTP

下载一个简单的TCP测试软件，打开，创建连接。 QQ的SMTP服务器地址为：`smtp.qq.com` 端口为 465 或者 587 然后点击连接 ![](https://www.siammm.cn/wp-content/uploads/2019/07/smtp01.jpg) ![](https://www.siammm.cn/wp-content/uploads/2019/07/smtp02.jpg) ![](https://www.siammm.cn/wp-content/uploads/2019/07/smtp03.jpg) 因为到这里，本地测试的工具不支持加密传输，所以运行不了了。 接着是使用swoole提供的tcp客户端来链接操作。 以下演示代码仅提供核心部分。

```php
$this->client = new Client( SWOOLE_TCP  SWOOLE_SSL);
$this->client->set([
    'open_eof_check' => true,
    'package_eof' => "\r\n",
    'package_max_length' => 1024 * 1024 * 2,
]);
```

```php
if ($this->client->connect('smtp.qq.com', 465,$this->timeout) === false) {
     throw new Exception("connect fail");
 }
$str = $this->recvCodeCheck('220');
$ehloHost = explode(' ',$str)[1];
$this->client->send("ehlo {$ehloHost}\r\n");
//先看是否得到250应答,并清除多余应答
$this->recvCodeCheck('250');
while (1){
    $peek = $this->client->recv($this->timeout);
    if(empty($peek)){
        throw new Exception('waiting 250 code error');
    }else{
        if(substr($peek,3,1) != '-'){
            break;
        }
    }
}
// 验证身份 登陆 不是所有的邮箱都一样的流程
$this->client->send("auth login\r\n");
$this->recvCodeCheck('334');
$this->client->send(base64_encode('59419979')."\r\n");
$this->recvCodeCheck('334');
$this->client->send(base64_encode('这里是授权码 在QQ邮箱后台拿到')."\r\n");
$this->recvCodeCheck('235');
//start send data
$this->client->send("mail from:<59419979@qq.com>\r\n");
$this->recvCodeCheck('250');
$this->client->send("rcpt to:<123456@qq.com>\r\n");
$this->recvCodeCheck('250');
$this->client->send("data\r\n");
$this->recvCodeCheck('354');
//build body
$mail = "";
$mail.= "From: Siam<59419979@qq.com>\r\n";
$mail.= "To: 不认识<123456@qq.com>\r\n";
$mail.= "Subject: Test\r\n";
//构造body
$mail.= "正文\r\n";
$this->client->send($mail);
$this->client->send(".\r\n");
$this->recvCodeCheck('250');
$this->client->send("quit\r\n");
$this->recvCodeCheck('221');
```

在easyswoole框架中，会提供smtp类库，以上代码就是部分的实现 使用类库可以直接使用