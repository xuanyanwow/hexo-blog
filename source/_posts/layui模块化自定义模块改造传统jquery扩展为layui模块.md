---
title: 'Layui模块化,改造传统jquery扩展为layui模块'
tags:
  - 前端
id: '73'
categories:
  - 前端
date: 2019-07-04 22:26:21
---

## 此篇文章比较偏向笔记类型

在我使用jquery扩展，拖拽组件的时候，因为使用的布局模板有些冲突，导致无法使用扩展，所以才会解决之后写下这篇文章。 **Layui中内置了jquery** 只有你所使用的模块有依赖到它，它才会加载，并且如果你的页面已经script引入了jquery，它并不会重复加载。内置的jquery模块去除了全局的$和jQuery。 这是layui文档中的描述，它内置了jq，但是去除了全局的$和jQuery对象，也就是在window的全局对外接口被删除了。 **拖拽组件的实现** 假设siam.js是一个扩展，里面提供了一个类似这样的方法

```markup
<div id='test'>原始内容</div>

<script src="https://cdn.bootcss.com/jquery/3.4.0/jquery.min.js"></script>
<script>
$.fn.siam = function(params){
    var dom = this;
    dom.html(params);
};


setTimeout(function(){
    $("#test").siam('这是新内容');
},800);
</script>
// 延迟执行完之后会把div内容变为 > 这是新内容
```

这就是一些传统jq扩展的实现原理，对于你调用的dom，它会继续处理操作，如本文开始说的，我使用的是拖拽组件，扩展会通过这样子的`对外接口` 将dom处理为可以拖拽的，并且带有其他事件的元素。

## 问题冲突

以上两点是问题的基础补充，在layui中，去除了全局的$和Jquery对象，默认扩展中有以下代码

```javascript
;(function($, window, document, undefined){
    .....扩展内容
})(window.jQuery  window.Zepto, window, document);
```

在最后面，它依赖加载`window.Jquery对象,window对象,document对象` 传递到上面的闭包中 对应`$, window, document, undefined`（因为没有传递 所以也一样） 所以导致闭包中使用的$是没有值的，一加载这个扩展就报错

```
$ is not defined
或者
Typeerror Cannot Read Property fn of undefined
```

测试过单独引入jq文件也解决不了问题，（我使用的模板加载顺序的原因，先加载了layui内置的jq）

## layui自定义模块

这是官网的介绍

```javascript
layui.code
/**
  扩展一个test模块
**/

layui.define(function(exports){ //提示：模块也可以依赖其它模块，如：layui.define('layer', callback);
  var obj = {
    hello: function(str){
      alert('Hello '+ (str'mymod'));
    }
  };

  //输出test接口
  exports('mymod', obj);
});
```

我们可以使用layui自定义模块的方法，将layui的其他模块传递进来，使扩展能操作layui中的jq对象

```javascript
layui.define(["jquery"], function (exports) {
    var $ = layui.jquery;
    // 把第一行的 ;(function($, window, document, undefined){ 换成以上
    ...扩展内容

    // 把最后一行的换成以下
    var obj = {
    };
    exports("自定义模块名", obj);
});
```

## 使用

```javascript
 layui.use(['form','jquery','自定义模块名'], function (admin, form) {
     var $ = layui.jquery;
     var obj = layui.自定义模块名;

     // 正常使用 即可  比如我的
     $("#test").desta('open');
});
```

> 注意，此篇文章并不是通用方法，只是简单阐述了我解决这个问题的思路和方案，可以参考学习，如果有其他类型的相似问题欢迎留言一起交流