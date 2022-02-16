---
title: PHP类，魔术方法
tags:
  - PHP
id: '112'
categories:
  - - PHP
date: 2019-08-04 21:09:24
---

以下方法在 PHP 中被称为魔术方法（Magic methods）

```php
__construct()
__destruct()
__call()
__callStatic()
__get()
__set()
__isset()
__unset()
__sleep()
__wakeup()
__toString()
__invoke()
__set_state()
__clone()
__debugInfo()
```

在命名自己的类方法时不能使用这些方法名，除非是想使用其魔术功能。 下面介绍每个方法的作用

# \_\_construct

构造函数，在实例化类的时候会隐式调用该方法，`可以接收传参`。如：

```php
class A{
    public function __construct($size) {
        $this->maxSize = $size; // 做一些初始化设置等等
        echo $this->maxSize;
    }
}

new A(3);
```

如果有一个类B继承了上面的类A ，如

```php
class B extends A{
    public function __construct($size) {
        echo "hello";
    }
}

new B(3);
```

在此例子中，不会设置和输出maxSize属性，只会输出hello。

> 因为在子类重写构造方法时，需要`显式`调用父类构造函数 `parent::__construct()`

注意

> 如果在A类的构造函数，不是写为`public`，而是private，则无法被子类继承使用。

# \_\_destruct

析构函数，当类被手动销毁，或者脚本结束时，gc回收触发。可以执行一些后置操作，比如删除临时目录下的文件。

> 注意

*   哪怕脚本调用exit(),die() 类的析构函数也会被执行
*   如果在析构函数中调用exit() 则该函数内部的逻辑后续不再执行

```php
public __destruct()
{
    echo 1;
    exit();
    echo 2;// 不会输出
}
```

*   与构造函数相同，子类继承后需要显式调用父类的析构函数
*   试图在析构函数（在脚本终止时被调用）中抛出一个异常会导致致命错误。

# \_\_call

当调用一个对象中的不能用的方法的时候就会执行这个函数。有两个参数：

```php
function __call($function_name, $args)
```

测试

```php
class A{
    public function __call($funcname, $args){
        var_dump($funcname);
        var_dump($args);
    }
}

$a = new A();
$a->one();
$a->tow('一个参数');

// 以下是输出

/*
string(3) "one"
array(0) {
}
string(3) "tow"
array(1) {
  [0]=>
  string(12) "一个参数"
}
string(5) "three"
array(2) {
  [0]=>
  string(12) "一个参数"
  [1]=>
  string(12) "两个参数"
}
*/
```

# \_\_callStatic

跟\_\_call一样，但是该函数触发的是调用的静态方法。

```
A::test();
```

# \_\_get

读取不可访问属性的值时，\_\_get() 会被调用。 猜想：在thinkphp框架的ORM中，关联模型 先在Orders模型中设置大概如下的方法

```
// 本模型的user ，代表要关联Users模型的一个数据，本模型的u_id = Users模型的id
public function user()
{
    return $this->belongTo('Users', 'u_id', 'id');
}
```

当在程序中调用，因为本身的Orders模型没有该属性，所以会尝试是否有设置该关系的方法，有则调用，然后返回Users的信息。

```
$orders = Orders::get(1);
var_dump(orders->user);
```

# \_\_set

在给不可访问属性赋值时，\_\_set() 会被调用。

# \_\_isset

当对不可访问属性调用 isset() 或 empty() 时，\_\_isset() 会被调用。

# \_\_unset

当对不可访问属性调用 unset() 时，\_\_unset() 会被调用。

# \_\_sleep

# \_\_wakeup

这两个魔术方法是 `类的序列化` 使用的，后续会有一篇专门的文章讲解。

# \_\_toString

`__toString()` 方法用于一个类被当成字符串时应怎样回应。 比如，在我们接入微信支付的时候，经常需要把参数排序、拼接成url格式 我们完全可以定义一个类，然后在toString魔术方法中，写明排序、转换为url格式的操作。

```php
// 伪代码

$params = new SiamWechatParams();
$params->appid = '1';
$params->total_fee = 200;

// http请求
Curl::send(self::url, $params->__toString());

// 其他地方直接输出，不手动显式调用
echo $params;
```

# \_\_invoke

当尝试以调用函数的方式调用一个对象时，\_\_invoke() 方法会被自动调用。

```php
class A
{
    function __invoke($params) {
        var_dump($params);
    }
}
$obj = new A();
$obj(5);
var_dump(is_callable($obj));
```

# \_\_set\_state

自 PHP 5.1.0 起当调用 var\_export() 导出类时，此静态 方法会被调用。

# \_\_clone

当对象复制完成时调用

# \_\_debugInfo

当调用var\_dump函数时候，定义需要显示的属性列表 如果没有在对象上定义该方法，那么将显示所有公共、受保护和私有属性。