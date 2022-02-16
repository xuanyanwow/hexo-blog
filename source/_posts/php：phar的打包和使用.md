---
title: PHP：Phar的打包和使用
tags:
  - PHP
id: '140'
categories:
  - - PHP
date: 2019-08-30 10:17:07
---

## 什么是Phar?

Phar是PHP里类似于`Jar`的一种打包文件，可以将整个应用打包，便于部署等。

### 安装需求

Phar需要 `PHP >= 5.2` ，在 PHP5.3或以上，Phar已经包含在内置的扩展中，在这之前可以通过`PECL`扩展安装。

### 运行时配置

通过`php.ini`的配置项，可以设定一些Phar的函数行为方式。

配置项

默认值

解释

phar.readonly

1

只允许读，只能在php.ini中取消设置

phar.require\_hash

1

强制所有打开的Phar包需要包含某种签名，否则拒绝处理，只能在php.ini中取消设置

phar.extract\_list

从phar 2.0.0开始，此INI设置已被删除，加载路径映射

phar.cache\_list

允许在Web服务器启动时预先解析映射phar存档，从而提供性能改进，使运行文件从phar存档中移出，非常接近从传统的基于磁盘的安装运行这些文件的速度。

### 使用Phar

Phar包在概念上类似于Java的Jar包，但是根据PHP应用程序的需求和灵活性进行了定制，Phar包用于在单个文件中分发完整的PHP应用程序或者库（单一入口）。 使用Phar包和使用其他的PHP库是相同的： 加载文件 --> 调用

```php
<?php
require_once "phar://siam.phar/user.class.php";

$u = new user();
$u->set_name("siam");
```

### 制作Phar包

我们先建立以下层级的文件

```
siam
├── src 目标程序
   ├── test
      └── index.html
   └── A.php
   └── B.php
   └── index.php
└── build.php   打包程序
```

其中src目录下 就是你需要打包的整个程序文件，这里就不展示了、 build.php文件是执行打包的文件

```php
<?php

//产生一个siam.phar文件
$phar = new Phar('siam.phar', 0, 'siam.phar');
// 添加src里面的所有文件到siam.phar归档文件
$phar->buildFromDirectory(dirname(__FILE__) . '/src');
//设置执行时的入口文件，第一个用于命令行，第二个用于浏览器访问，这里都设置为index.php
$phar->setDefaultStub('index.php', 'index.php');
```

设置好包名、打包目标、运行入口文件，我们在浏览器访问build.php即可看到在目录中生成了一个`siam.phar`的文件。

> 第一次访问build.php提示报错:disabled by the php.ini setting phar.readonly，记得看文章上面，在php.ini手动打开，不能通过函数设置的哈~

siam.phar的开头内容大概如下

```php
<?php
$web = 'index.php';

if (in_array('phar', stream_get_wrappers()) && class_exists('Phar', 0)) {
Phar::interceptFileFuncs();
set_include_path('phar://' . __FILE__ . PATH_SEPARATOR . get_include_path());
Phar::webPhar(null, $web);
include 'phar://' . __FILE__ . '/' . Extract_Phar::START;
return;
}

if (@(isset($_SERVER['REQUEST_URI']) && isset($_SERVER['REQUEST_METHOD']) && ($_SERVER['REQUEST_METHOD'] == 'GET'  $_SERVER['REQUEST_METHOD'] == 'POST'))) {
Extract_Phar::go(true);
$mimes = array(
'phps' => 2,
'c' => 'text/plain',
'cc' => 'text/plain',
'cpp' => 'text/plain',
'c++' => 'text/plain',
......
)
```

### 效果预览

```php
<?php
/**
 * 测试siam.phar
 */

# 测试入口文件
require 'phar://siam.phar';

echo "<br>";

# 测试类文件
require 'phar://siam.phar/A.php';

$class = new Siam\A();
echo $class->a();

echo "<br>";

# 测试静态文件
$html = require 'phar://siam.phar/test/index.html';
echo $html;

```

### 命令行模式

上面我们演示了的是其他php程序加载调用phar包的情况。 我们也可以用命令行来运行phar包。 首先我们先改造一下入口文件

```php
<?php
foreach ($argv as $key => $value) {
    if ($key == 0){
        continue;
    }

    switch ($value) {
        case '-v':
            echo "当前版本 v1.0";
            break;

        case '-m':
            echo "siam";
            break;

        default:
            echo "未知命令";die;
            break;
    }
}
```

然后再次构建phar包，在命令行模式下分别输入以下命令试试吧

```
php ./siam.phar 
php ./siam.phar -v
php ./siam.phar -v -m 
php ./siam.phar -v -t
```

### Phar中目录路径相关

我们都知道在PHP中是可以通过函数和常量来获取运行脚本所在目录路径的，那么在Phar打包的程序中，展示的目录路径又会是怎么样的？ 我们将`src/index.php`中的文件再次改为以下内容来进行测试

```php
<?php
// getcwd()返回当前工作目录
echo "getcwd -->" . getcwd();
echo "\n";

// 获取当前文件的绝对路径
echo "__FILE__ -->" .__FILE__;
echo "\n";

// 获取当前脚本的目录
echo "__DIR__ -->" .__DIR__;
echo "\n";

// 当前执行脚本的绝对路径。记住，在CLI方式运行php是获取不到的
echo "SCRIPT_FILENAME -->" .$_SERVER["SCRIPT_FILENAME"];
echo "\n";

// 当前运行脚本所在的文档根目录。在服务器配置文件中定义
echo "DOCUMENT_ROOT -->" .$_SERVER["DOCUMENT_ROOT"];
echo "\n";
```

接着我们分别运行`src/index.php`和`siam.phar` [![phar运行结果对比](http://yancoo.cn/uploads/images/201902/20190330-1.png "phar运行结果对比")](# "phar运行结果对比") 在结果中我们可以看到类似如图的结果

phar

正常PHP脚本

getcwd

得到phar包所在目录

得到php脚本所在目录

\_\_FILE\_\_

phar:// 数据流包装器，指向入口脚本所在绝对路径（注意：phar包名作为一个目录层级）

得到php脚本文件所在绝对路径

\_\_DIR\_\_

phar:// 数据流包装器，指向入口脚本所在目录绝对路径

得到php脚本所在目录绝对路径

$\_SERVER\["SCRIPT\_FILENAME"\]

phar包名

php脚本文件名

$\_SERVER\["DOCUMENT\_ROOT"\]

应该是本地测试原因为空，后面补充

#### Phar包中的临时文件存放

假设我们的程序打包成了phar包，那么在运行中产生的日志记录，我们应该怎么来存放。 根据上面的测试，我们知道了 `__FILE__` `__DIR__` 两个常量得到的是`phar:// 数据流包装器`，如果我们使用这两个常量来设置Log文件存放路径，是否能正常储存?

```php
<?php
$logPath = __DIR__ .   "/test.log";
echo $logPath."\n";
file_put_contents($logPath, "test\n");

// 写完再读出来
echo file_get_contents($logPath);
```

> 打包，运行，会得到以下结果

```
phar://F:/WWW/learn/phar/siam.phar/test.log
test
```

但是我们的日志需要储存一般都是用`FILE_APPEND`追加内容储存。 然而phar包中的运行你将会得到以下结果

```
Warning: file_put_contents(phar://F:/WWW/learn/phar/siam.phar/test.log): failed to open stream: phar error: open mode append not supported in phar://F:/WWW/learn/phar/siam.phar/index.php on line 4
```

关键报错：open model append not supported in phar 可见phar内的文件写入不支持追加模式打开。 并且在后续的日志查看中 也极其不方便，因为phar包内的文件我们并不能直接查看，所以我们储存临时文件应该存放在外部。

```php
<?php
$logPath = getcwd() .   "/test.log";
echo $logPath."\n";
file_put_contents($logPath, "test\n", FILE_APPEND);
```

getcwd()函数将会得到phar包所在目录，然后在同级将创建test.log文件存放日志内容。