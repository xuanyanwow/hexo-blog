---
title: WordPress二开-文章内容部分隐藏
tags:
  - PHP
  - 前端
  - 常见问题
id: '254'
categories:
  - - PHP
  - - 前端
  - - 常见问题
date: 2020-03-22 17:59:52
---

# 原理

在php从数据库读取文件出来之后，不要马上输出，先执行正则替换、删除的步骤即可

# 修改文件地址

WordPress是设计了模板主题的概念的，模板主题所在目录为：`wordpress/wp-content/themes` 在该目录下，每一套主题又有一个新的目录，假设我们使用的模板主题名字为siam 那么完整路径应该为`wordpress/wp-content/themes/siam` 在该目录下搜索文件内容`the_content` 有调用该函数的就是对应的文章内容（可能有多个，对应多种布局，比如图片列表文章、纯文字文章等等 根据自己主题判断） 修改逻辑 这里贴上我的处理逻辑参考 原文博客：http://blog.siammm.cn 原文地址：https://www.siammm.cn/archives/254

```php
ob_start();
the_content();
$content = ob_get_contents();
ob_end_clean();

if(!current_user_can('manage_options')){
    // 循环遍历
    $replace = true;
    while($replace){
        $b= (strpos($content,"……"));
        $c= (strpos($content,"***"));
        if ($b && $c){
            // 处理了一次，那么看看是否需要继续处理
            $content = substr_replace($content,'<h5 style="border:1px solid #000;">SIAM 暂时隐藏该部分内容~ 很抱歉</h5>', $b,$c-$b+strlen("&&&"));
        }else{
            $replace = false;
        }
    }
}


echo $content;
```

# 效果

@@@ 隐藏内容 ￥￥￥