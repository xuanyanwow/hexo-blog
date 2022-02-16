---
title: PHP图片文字合成居中
tags:
  - PHP
id: '142'
categories:
  - - PHP
date: 2019-08-30 10:19:04
---

### PHP处理图片

PHP使用GD库创建和处理包括GIF，PNG，jpef，wbmp以及xpm在内的多种格式的图像。 以下教程：图片合成文字，实现合成文字水平、垂直居中。

#### 读取图片资源

```
imagecreatefrom 系列函数用于从文件或 URL 载入一幅图像，成功返回图像资源，失败则返回一个空字符串。
```

#### 根据图片格式选用不同函数

```
imagecreatefromgif()：创建一块画布，并从 GIF 文件或 URL 地址载入一副图像
imagecreatefromjpeg()：创建一块画布，并从 JPEG 文件或 URL 地址载入一副图像
imagecreatefrompng()：创建一块画布，并从 PNG 文件或 URL 地址载入一副图像
imagecreatefromwbmp()：创建一块画布，并从 WBMP 文件或 URL 地址载入一副图像
imagecreatefromstring()：创建一块画布，并从字符串中的图像流新建一副图像
```

#### 获取图片尺寸

```
imagesx($image);
imagesy($image);
```

#### 创建颜色

```
imagecolorallocatealpha(resource $image , int $red , int $green , int $blue , int $alpha); // 带透明度
imagecolorallocate(resource $image , int $red , int $green , int $blue);      // 普通
```

#### 获取文字内容所需尺寸

```
imagettfbbox ( float $size, float $angle, string $fontfile, string $text):array
```

取得使用 TrueType 字体的文本的范围。（种类型字体文件的扩展名是.ttf，类型代码是tfil。） 以上是每个步骤使用的关键函数说明。以下是完整代码示例。

```php
<?php
/**
 * Created by PhpStorm.
 * User: Siam
 * Date: 2019/2/4 0004
 * Time: 下午 10:58
 */

$main = imagecreatefromjpeg('./test.jpg');

$fontSize = 38;
$width   = imagesx($main);
$height   = imagesy($main);

//1.设置字体的路径
$font    = "./t.ttf";
//2.填写水印内容
$content = "My name is Siam,中文是宣言";
//3.设置字体颜色和透明度
$color   = imagecolorallocatealpha($main, 255, 255, 255, 0);

$fontBox = imagettfbbox($fontSize, 0, $font, $content);//获取文字所需的尺寸大小 

//4.写入文字 (图片资源，字体大小，旋转角度，坐标x，坐标y，颜色，字体文件，内容)
imagettftext($main, $fontSize, 0, ceil(($width - $fontBox[2]) / 2), ceil(($height - $fontBox[1] - $fontBox[7]) / 2), $color, $font, $content);

// 浏览器输出 也可以换成保存新图片资源
header("Content-type:jpg");
imagejpeg($main);
```

效果： [![Siam博客](http://yancoo.cn/uploads/images/201902/2019190205-123829.png "Siam博客")](# "Siam博客") 最关键的步骤是获取到文字内容所需的尺寸大小

> 原图的大小 - 文字内容的大小 = 剩余空白大小； 剩余空白大小 / 2 的效果就是自动居中。

我们可以在以上基础上封装成一个灵活的函数

```php
<?php
function imageAddText($path, $content, $x = 'auto', $y = 'auto', $fontSize = 38, $font = './t.ttf'){
    $temp = array(1=>'gif', 2=>'jpeg', 3=>'png');
    // 获取图片信息
    $imageInfo = getimagesize($path);
    $imageType = $temp[$imageInfo[2]];

    $getfunc = "imagecreatefrom$imageType";
    $outfunc = "image$imageType";

    $resource = $getfunc($path);

    $width    = imagesx($resource);
    $height   = imagesy($resource);

    $color = imagecolorallocatealpha($resource, 255, 255, 255, 0);

    $fontBox = imagettfbbox($fontSize, 0, $font, $content);//文字水平居中实质

    if ($x === 'auto'){
        $x = ceil(($width - $fontBox[2]) / 2);
    }
    if ($y === 'auto'){
        $y = ceil(($height - $fontBox[1] - $fontBox[7]) / 2);
    }

    imagettftext($resource, $fontSize, 0, $x, $y, $color, $font, $content);

    /*输出图片*/
    //浏览器输出
    header("Content-type:".$imageType);
    $outfunc($resource);
}

// 自动居中
// imageAddText('./test.jpg', 'My name is Siam，中文名是宣言');
// 声明x y值
// imageAddText('./test.jpg', 'My name is Siam，中文名是宣言',200);
// imageAddText('./test.jpg', 'My name is Siam，中文名是宣言','auto', '300');
```