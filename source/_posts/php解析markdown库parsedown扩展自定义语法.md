---
title: 'PHP解析Markdown库,parsedown扩展自定义语法'
tags:
  - PHP
  - 常见问题
id: '239'
categories:
  - - 常见问题
date: 2020-02-28 11:09:19
---

# 写在前面

在开发系统过程中，有些信息编写储存是使用Markdown通用语法，但由于各个平台的会新增一些独特规范，一般的解析库都是只包含了`标准语法`，对于自定义语法是不支持解析的（如vuepress文档系统中的::: tip 提示语块） 我们从vuepress迁移文档系统到自己实现的文档系统时，特定标签无法解析，需要扩展解析库的功能，来完成自定义语法。 PHP常见的Markdown解析库是parsedown。这个库非常的轻量，只有一个文件，无需依赖其他扩展。

## 如何扩展自定义语法

我们可以在库的wiki中找到 https://github.com/erusev/parsedown/wiki/Tutorial:-Create-Extensions

## 嵌套解析

我们经过上面的教程已经扩展了::: tip的语法 使用如下

```
::: tip
提示语句
:::
```

但是如果中间的内容为其他符合md标准的语法，没办法嵌套解析，所以需要继续修改逻辑 旧代码如下

```
    protected function blockNoticeComplete($Block)
    {
        $text = $Block['element']['text']['text'];
        $Block['element']['text']['text'] = $text;
        return $Block;
    }
```

改为新代码，需要把$text文字再调用一次解析器 解析成html。但是此时会被自动反转义，在页面上显示如下情况 ![](https://www.siammm.cn/wp-content/uploads/2020/02/14fc4fddf29570ae5d6ff453eb08cd6c.png) 所以我们需要追踪在哪里决定转义，并取消该标签的自动转义。 php中转义的函数为`htmlspecialchars` 在这个库里搜索，找到如下方法

```php
protected static function escape($text, $allowQuotes = false)
    {
        return htmlspecialchars($text, $allowQuotes ? ENT_NOQUOTES : ENT_QUOTES, 'UTF-8');
    }
```

再搜索该方法，决定转义的核心代码逻辑如下

```php
    if (isset($Element['text']))
        {
            $text = $Element['text'];
        }
        // very strongly consider an alternative if you're writing an
        // extension
        elseif (isset($Element['rawHtml']))
        {
            $text = $Element['rawHtml'];
            $allowRawHtmlInSafeMode = isset($Element['allowRawHtmlInSafeMode']) && $Element['allowRawHtmlInSafeMode'];
            $permitRawHtml = !$this->safeMode  $allowRawHtmlInSafeMode;
        }
```

所以我们修改标签解析逻辑为返回rawHtml

```php
        unset ($Block['element']['text']['text']);
        $Block['element']['text']['rawHtml'] = html_entity_decode((new static)->text($text));
        $Block['element']['text']['allowRawHtmlInSafeMode'] = true;
```