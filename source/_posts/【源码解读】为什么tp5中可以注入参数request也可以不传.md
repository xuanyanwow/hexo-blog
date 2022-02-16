---
title: 【源码解读】TP框架参数注入，参数绑定
tags:
  - PHP
  - Thinkphp
id: '107'
categories:
  - - 后端
date: 2019-07-23 21:00:58
---

# 写在前面

用过TP框架的应该都知道有这样一种操作： 我们可以把传参绑定在方法的参数中，还可以绑定一些系统类 比如Request类等等

```php
<?php
namespace app\index\Controller;

class Blog
{
    public function read($id)
    {
        return 'id='.$id;
    }

    public function archive($year, $month='01')
    {
        return 'year='.$year.'&month='.$month;
    }
}
```

当我们的url这样子访问的时候，参数就会自动注入到方法参数中

```
http://serverName/index.php/index/blog/read/id/5
http://serverName/index.php/index/blog/archive/year/2016/month/06
```

依赖注入Request

```php
namespace app\index\controller;

use think\Request;

class Index
{
    public function index(Request $request)
    {
        // 获取name请求变量
        return $request->name;
    }
}
```

# 为什么要这样子用

1.api与类之间的兼容。 比如我们有一个这样子的功能：根据用户id返回用户信息

```php
function getInfo($uId){
    return [];
}
```

可以直接由api访问，得到json，或者由其他类调用，返回信息提供给其他地方的代码调用 如果没有使用参数绑定，则方法名是`getInfo()`，外部无法传参使用。 2.便捷，我们可以直接在代码中使用变量操作传参，而不再需要使用input()等函数来获取再操作。

# 源码解析

在源码中，有这样子一段代码

```php
    /**
     * 调用反射执行类的方法 支持参数绑定
     * @access public
     * @param  mixed   $method 方法
     * @param  array   $vars   参数
     * @return mixed
     */
    public function invokeMethod($method, $vars = [])
    {
        try {
            if (is_array($method)) {
                $class   = is_object($method[0]) ? $method[0] : $this->invokeClass($method[0]);
                $reflect = new ReflectionMethod($class, $method[1]);
            } else {
                // 静态方法
                $reflect = new ReflectionMethod($method);
            }
            // siam标注：重点
            $args = $this->bindParams($reflect, $vars);

            return $reflect->invokeArgs(isset($class) ? $class : null, $args);
        } catch (ReflectionException $e) {
            if (is_array($method) && is_object($method[0])) {
                $method[0] = get_class($method[0]);
            }

            throw new Exception('method not exists: ' . (is_array($method) ? $method[0] . '::' . $method[1] : $method) . '()');
        }
    }
```

bindParams就是此操作的重点，该方法的内容如下

```php
    protected function bindParams($reflect, $vars = [])
    {
        if ($reflect->getNumberOfParameters() == 0) {
            return [];
        }

        // 判断数组类型 数字数组时按顺序绑定参数
        reset($vars);
        $type   = key($vars) === 0 ? 1 : 0;
        $params = $reflect->getParameters();

        foreach ($params as $param) {
            $name      = $param->getName();
            $lowerName = Loader::parseName($name);
            $class     = $param->getClass();

            if ($class) {
                $args[] = $this->getObjectParam($class->getName(), $vars);
            } elseif (1 == $type && !empty($vars)) {
                $args[] = array_shift($vars);
            } elseif (0 == $type && isset($vars[$name])) {
                $args[] = $vars[$name];
            } elseif (0 == $type && isset($vars[$lowerName])) {
                $args[] = $vars[$lowerName];
            } elseif ($param->isDefaultValueAvailable()) {
                $args[] = $param->getDefaultValue();
            } else {
                throw new InvalidArgumentException('method param miss:' . $name);
            }
        }

        return $args;
    }
```

> 其主要作用：根据反射类，拿到该方法需要的传参、顺序、数组类型，然后按照优先级寻找符合的参数，储存再$args中。

然后在外部使用`$reflect->invokeArgs(isset($class) ? $class : null, $args);` 带参数执行，就可以按照正确的顺序传参

# 原理（总结）

核心是：使用反射类，拿到需要执行的类、方法属性，然后分析传参的属性，在post、get、类属性等等参数中，按不同优先级搜寻符合注入条件的参数。 最终使用执行，并且提供组装正确的参数数组。 php的反射类，可以分析目标类的各种属性 方法列表、参数、私有共有属性、方法的类型等等 以下提供一个简单的列表

```
ReflectionMethod
    __construct
    export
    getClosure
    getDeclaringClass
    getModifiers
    getPrototype
    invoke
    invokeArgs
    isAbstract
    isConstructor
    isDestructor
    isFinal
    isPrivate
    isProtected
    isPublic
    isStatic
    setAccessible
    __toString
```