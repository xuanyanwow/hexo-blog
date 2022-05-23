---
title: 宝塔的nginx,在Thinkphp伪静态情况下配置图片裁剪
tags:
  - php
id: 
categories:
  - PHP
date: 2022-5-23 19:37:32
---


在宝塔的`网站--设置--伪静态` 写入以下代码

注意：需要填写在thinkphp的伪静态隐藏index.php规则之前 

```
location ~* (.*)\.(gif|jpg|jpeg|png|bmp|swf)!(\d+)x(\d+)_(\d+)$ {
    expires      30d;
    error_log /dev/null;
    access_log /dev/null;

    set $w $3;
    set $h $4;

    # crop裁剪 resize缩放
    image_filter resize  $w $h;
    image_filter_buffer  100M;
    image_filter_jpeg_quality $5;
    try_files /$1.$2 /404.jpg;
}
```


访问图片路径 就可以扩展为  `536de782104cc8edcdfc7e9bd29a3757.jpg!800x800_80`   自定义尺寸和压缩比例