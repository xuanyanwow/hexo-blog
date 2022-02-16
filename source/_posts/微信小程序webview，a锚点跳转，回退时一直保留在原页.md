---
title: 微信小程序webview，a锚点跳转，回退时一直保留在原页面
tags:
  - 前端
id: '285'
categories:
  - - 前端
date: 2020-07-30 15:47:42
---

# 写在前面

公司业务，需要写一个页面导航，大概功能如下（APP排版，webview嵌套在小程序中） ![](https://www.siammm.cn/wp-content/uploads/2020/07/wp_editor_md_9e40612d7c54bb960332699721f5db9c.jpg) 当点击导航的时候，也可以自动跳转到相应的地方。使用的是`a标签的锚点跳转` 功能和效果都是在浏览器上测试正常的，但在小程序上遇到以下问题

## 点击回退按钮无法退出页面

当我们有点击过导航的时候，假设从One点击到Two 相当于url变动：index.html#One -> index.html#Two 当点击小程序右上角的回退按钮的时候，不会退出当前webview页面 返回到`小程序的夫级页面` 而是从index.html#Two -> index.html#One，页面也滑动到One区域。 如果在此页面上点击过10个导航，那么将需要点击11次回退才能退出当前页面。 不符合业务逻辑。所以需要更改实现方案。

## 解决方案

使用jq滑动跳转页面区域。 实现代码如下 .nav-one是一开始的a标签，现在改为div，但是class不改变

```javascript
$(".nav-one").on("click", function(){
    // 高亮状态改变
    $(this).siblings().removeClass('nav-now');
    $(this).addClass('nav-now');
    // 区域名
    let clickStr = $(this).data("str");
    // 筛选到合适的html里的dom元素，获取高度，然后将浏览器向下滑动
    $.each($("h1"), function(index,item){
        if (item.innerText === clickStr){
            let htmlDom = $('html');
            htmlDom.scrollTop($(item).offset().top - 50);
            return false;
        }
    });
});
```