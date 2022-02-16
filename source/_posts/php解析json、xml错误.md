---
title: PHP解析json、xml错误
tags:
  - PHP
id: '125'
categories:
  - - PHP
date: 2019-08-11 23:19:37
---

# 解析json

php内置函数json\_decode() 可以解析json字符串 但是有的时候看起来正确的json，解析却一直返回null。 你知道吗，json是可能解析失败的，此时PHP不会产生提示。 我们需要手动通过`json_last_error()`函数获取

```php
function json_decode_siam($string, $mark = false){
    $data = json_decode($string, $mark);

    switch (json_last_error()) {
        case JSON_ERROR_NONE:
            return $data;
            break;
        case JSON_ERROR_DEPTH:
            echo ' - 已超出最大堆栈深度';
            break;
        case JSON_ERROR_STATE_MISMATCH:
            echo ' - JSON无效或格式错误  状态不匹配';
            break;
        case JSON_ERROR_CTRL_CHAR:
            echo ' - 发现意外的控制字符 可能编码错误';
            break;
        case JSON_ERROR_SYNTAX:
            echo ' - 错误符号，json格式错误';
            break;
        case JSON_ERROR_UTF8:
            echo ' - 格式错误的UTF-8字符，可能是错误编码的';
            break;
        default:
            echo ' - Unknown error';
            break;
    }
}
```

# 解析xml

php中，解析xml有好几种方式，主要是依赖不同的扩展环境。 这里就说说我自己常使用的这种方式吧

```
simplexml_load_string();
simplexml_load_file();
```

可以通过字符串或者文件，加载然后解析，返回Simplexml对象 在该方式中，如果xml格式错误，则会直接产生报错

```
$str = "不是xml字符串";
$data = simplexml_load_string($str);
var_dump($data);
```

得到

```
bool(false)


PHP Warning:  simplexml_load_string(): Entity: line 1: parser error : Start tag expected, '<' not found in /usercode/file.php on line 4
PHP Warning:  simplexml_load_string(): 不是xml字符串 in /usercode/file.php on line 4
PHP Warning:  simplexml_load_string(): ^ in /usercode/file.php on line 4

```

这是PHP错误，而非异常，所以也不能使用try{}catch(){) 处理 以后可能会完善这部分的知识（主要是前辈们的文章写过好多了）