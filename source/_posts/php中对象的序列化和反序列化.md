---
title: PHP中对象的序列化和反序列化
tags:
  - PHP
id: '126'
categories:
  - - PHP
date: 2019-08-12 09:15:17
---

# php的serialize函数和unserialize函数

serialize() 返回字符串，可以存储于任何地方。 serialize() 可处理除了 resource 之外的任何类型。甚至可以 serialize() 那些包含了指向其自身引用的数组。 这有利于存储或传递 PHP 的值，同时`不丢失其类型和结构`。 在需要恢复的地方使用unserialize()函数即可

# php类魔术方法中的\_\_sleep和\_\_wakeup

在众多的php类魔术方法中(另一篇文章有简单介绍 [PHP类，魔术方法](https://www.siammm.cn/archives/112 "PHP类，魔术方法"))，有两个是跟序列化有关的。 `__sleep()` 在对象被调用serialize时隐式唤起，可以返回需要参与序列化的属性数组 `__wakeup()` 当调用unserialize恢复对象的时候，会被隐式唤起，可以做一些初始化工作

# 简单实战

假设，我们在cli模式的php程序，会根据调用命令解析到不同的类执行。 该类拥有以下3个属性，其中isDev,isCli应该根据运行入口、配置文件等状态而决定。 所以当我们在序列化该类的对象时，不应该包含这两个属性，而应该在wakeup的时候，动态取配置文件的值然后设置进去。

```php
class Command{
    public $name;  // 命令名
    public $isDev; // 是否为开发环境
    public $isCli; // 是否为命令行运行

    public function run()
    {
        if ($this->isDev){
            echo "debug\n";
        }
        if (!$this->cli){
            echo "only cli\n";
        }
    }

    // 设置规定参与序列化的属性
    public function __sleep()
    {
        return ['name'];    
    }

    public function __wakeup()
    {
        // 从配置文件读取
        $config = file_get_content("siam.conf");
        $this->isDev = $config['dev'] ?? true;
        // 运行环境判断
        $this->isCli = true;
    }
}
```

实例化对象 并序列化

```php
$class = new Command();
$class->isDev = true;
$class->isCli = true;
$str =  serialize($class);

var_dump(unserialize($str));
// 得到以下对象，isDev不会序列化原始的对象属性，而是通过wakeup重新定义
// object(Command)#3 (3) { ["name"]=> NULL ["isDev"]=> bool(false) ["isCli"]=> bool(true) }
```