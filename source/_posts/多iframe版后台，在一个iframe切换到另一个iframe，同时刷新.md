---
title: 多Iframe版后台，在一个iframe切换到另一个iframe，同时刷新
tags:
  - 前端
id: '165'
categories:
  - 前端
date: 2019-08-30 10:49:20
---

当我们使用多标签iframe的后台管理模板时，需要在一个iframe中跳转到另一个iframe，并且对新iframe进行操作，这篇文章记录一下我在开发过程中编写的代码。 有一个标签 #tab 用于储存已经打开的标签页 [![tab标签说明](http://yancoo.cn/uploads/images/20190103/2019190103-045938.png "tab标签说明")](# "tab标签说明") 存放iframe的标签.tabsbody-item 结构如下

```
.tabsbody-item 订单列表
——iframe
.tabsbody-item 收款点
——iframe
```

```
# 在iframe中，对父级窗口进行操作，搜索#tab中已打开标签的列表
var iframe = $(window.parent.document).find("#tab > li");
if (iframe.length > 0) {
    # 遍历已经打开的标签，并且判断是否为自己想要操作的目标。
    iframe.each(function (index, element) {
        if ($(this).attr('id') === "test.html") {
            # 模拟点下该标签
            $(this).trigger('click');
            # 刷新
            var iframet = $(window.parent.document).find('.tabsbody-item').eq(index).find('.iframe-class');
            iframet[0].contentWindow.location.reload(true);
        }
    });
}
```

这样子就从A标签内操作父窗口，打开B标签页，并且刷新B标签页的内容。