---
title: 'JSON序列化接口,JsonSerializable'
tags:
  - PHP
id: '131'
categories:
  - 后端
date: 2019-08-21 22:05:35
---

# 写在前面

php中操作json的函数有`json_encode()`,`json_decode()` 在该文档中，encode的传入值可以是`除了resource 类型之外的任何数据类型。`

# 简单序列化一个类

```php
class Siam{
    public $name = 'siam';
    protected $age = 21;
    private $sex = "男";
    public static $lover = "undefined";

    public function test()
    {
        return "??";
    }
}

echo json_encode(new Siam());

// 得到  {"name":"siam"}
```

> 默认的json\_encode，只能序列化类中的public属性。

# 自定义类的序列化接口

php还提供了一个自定义类序列化的接口，JsonSerializable 实现 JsonSerializable 的类可以 在 json\_encode() 时定制他们的 JSON 表示法。

```php
JsonSerializable {
    /* 方法 */
    abstract public jsonSerialize ( void ) : mixed
}
```

需要实现的方法jsonSerialize()，它的返回值： 返回能被 json\_encode() 序列化的数据， 这个值可以是除了 resource 外的任意类型。

# 简单测试

```php
class Siam implements  JsonSerializable
{
    public $name = 'siam';
    protected $age = 21;
    private $sex = "男";
    public static $lover = "undefined";

    public function test()
    {
        return "??";
    }

    public function jsonSerialize()
    {
        return [
            'name' => $this->name,
            'age'  => $this->age,
            'lover'=> self::$lover
        ];
    }
}

echo json_encode(new Siam());

// 得到 {"name":"siam","age":21,"lover":"undefined"}
```

当我们定义一些类的时候，它们经常参与json序列化和传输，同时默认的public属性序列化不能满足，我们就可以自定义序列化接口，提供我们想要的数据。

# 总结

*   json不能序列化资源
*   json序列化类的时候默认只序列化public属性
*   php提供了JsonSerializable自定义序列化接口