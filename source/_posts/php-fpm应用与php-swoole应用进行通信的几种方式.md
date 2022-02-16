---
title: php-fpm应用与php-swoole应用进行通信
tags:
  - PHP
  - swoole
id: '229'
categories:
  - - easyswoole
  - - PHP
  - - 常见问题
date: 2020-01-02 14:02:53
---

# 基础要求

*   linux万物皆文件
*   socket通信
*   基础进制转换

原文地址：[https://www.siammm.cn/archives/229](https://www.siammm.cn/archives/229 "https://www.siammm.cn/archives/229")

# 写在前面

这篇文章是自己练习的内容，主要想实现应用间的通信机制。

# Workerman中提供的建议方案

> 与其它mvc框架结合建议以上图的方式(ThinkPHP为例)：  
> 1、ThinkPHP与Workerman是两个独立的系统，独立部署(可部署在不同服务器)，互不干扰。  
> 2、ThinkPHP以HTTP协议提供网页页面在浏览器渲染展示。  
> 3、ThinkPHP提供的页面的js发起websocket连接，连接workerman  
> 4、连接后给Workerman发送一个数据包(包含用户名密码或者某种token串)用于验证websocket连接属于哪个用户。  
> 5、仅在ThinkPHP需要向浏览器推送数据时，才调用workerman的socket接口推送数据。  
> 6、其余请求还是按照原本ThinkPHP的HTTP方式调用处理。  
> 总结：  
> 把Workerman作为一个可以向浏览器推送的通道，仅仅在需要向浏览器推送数据时才调用Workerman接口完成推送。业务逻辑全部在ThinkPHP中完成。

我们使用swoole环境的常驻内存、协程特性来做一些其他事务，如：任务队列及其消费、缓存、异步执行等情况时 可以如建议中第5步所说，FPM环境调用Swoole环境提供的接口（可以用TCP/HTTP等方式）来开始一个任务

# 进程通信

上面的方案可以用在单机中，也可以用在集群部署中。 进程通信一般仅限于单机中使用 进程通信的方式有好几种，这里主要写明我测试的一种。

## unix socket 文件

在linux环境中，万物皆为文件，套接字也可以用文件来表示，然后一个进程（一般是swoole环境）监听它，其他进程（FPM环境）连接它，并且发送数据

> 这里使用的是Easyswoole框架提供的一个基类，如果是纯Swoole环境可以下载框架源码并查看原理

### EasySwoole部分

继承了`AbstractUnixProcess`，封装好了很多内容，直接写明onAccept 接受数据做处理即可

```php
namespace App\UnixSocket;

use EasySwoole\Component\Process\Socket\AbstractUnixProcess;
use Swoole\Coroutine\Socket;

class Siam extends AbstractUnixProcess
{

    function onAccept(Socket $socket)
    {
        // 收取包头4字节计算包长度 收不到4字节包头丢弃该包
        $header = $socket->recvAll(4, 1);

        if (strlen($header) != 4) {
            $socket->sendAll(self::pack(json_encode([
                'res' => 'fail',
                'msg' => '长度有误',
            ], 256)));
            $socket->close();
            return;
        }

        // 收包头声明的包长度 包长一致进入命令处理流程
        // 多处close是为了快速释放连接
        $allLength = self::packDataLength($header);
        $data = $socket->recvAll($allLength, 1);
        if (strlen($data) == $allLength) {
            echo $data;

            // 执行任务逻辑

            $socket->sendAll(self::pack(json_encode([
                'res' => 'ok',
                'msg' => '长度相同',
            ], 256)));
            $socket->close();
        }else{

            $socket->sendAll(self::pack(json_encode([
                'res' => 'fail',
                'msg' => '长度不相等',
            ], 256)));
            $socket->close();
        }
    }

    static function pack($string)
    {
        return pack('N', strlen($string)) . $string;
    }

    static function packDataLength($head)
    {
        return unpack('N', $head)[1];
    }
}
```

写好了任务逻辑，还需要加入启动该进程

```php
EasySwooleEvent.php文件
    public static function mainServerCreate(EventRegister $register)
    {
        $config = new UnixProcessConfig();
        $config->setSocketFile(EASYSWOOLE_ROOT."/Temp/siam_unix.sock");
        $config->setProcessName('siam_unix');

        $siam = new Siam($config);
        ServerManager::getInstance()->getSwooleServer()->addProcess($siam->getProcess());
    }
```

### 普通环境发送数据

```php
<?php
$sock = dirname(__FILE__)."/Temp/siam_unix.sock";

$unixSock = stream_socket_client("unix:///".$sock);

fwrite($unixSock, siam_pack('my name is siam'));

//echo fread($unixSock, 4096)."\n";

fclose($unixSock);




function siam_pack($string)
{
    return pack('N', strlen($string)) . $string;
}

function packDataLength($head)
{
    return unpack('N', $head)[1];
}
```