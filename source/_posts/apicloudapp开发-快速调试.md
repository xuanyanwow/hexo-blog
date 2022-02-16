---
title: ApiCloudApp开发 - 快速调试
tags:
  - Android
  - 前端
id: '251'
categories:
  - - 前端
date: 2020-03-21 21:43:21
---

# 写在前面

*   快速监听、预览、复发 HTTP网络请求
*   在USB连线开发时如何电脑查看console.log日志
*   编译后无法运行的错误

原文博客：http://blog.siammm.cn

## 解决过程

@@@ 在需要调试的页面引入vconsole 但每个页面都需要引入 换一种思路：封装一个console方法，储存到数据库中 再加上一个页面可以查询 ￥￥￥

## 封装通用工具

@@@ 封装日志上报方法

```javascript
log(str){
    // 储存到数据库
    let consoleData = api.getGlobalData({
        key :"siamConsoleData"
    })
    if (consoleData == ''){
        consoleData = []
    }else{
        consoleData = JSON.parse(consoleData)
    }
    consoleData.unshift(str);
    api.setGlobalData({
        key :"siamConsoleData",
        value: JSON.stringify(consoleData)
    })
}
```

在首页增加一个按钮，进入日志详情页

```javascript
openLog(){
    console.log("测试吧")
    if (this.siamconsole ){
        api.closeFrame({
            name: 'siamConsole'
        });
        this.siamconsole = false;
        return true;
    }
    this.siamconsole = true;
    api.openFrame({
        name: 'siamConsole',
        url: 'widget://html/siam/console.html',
        rect: $.rect(),
        pageParam: {
            name: 'test'
        }
    });
},
```

```markup
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="maximum-scale=1.0,minimum-scale=1.0,user-scalable=0,initial-scale=1.0,width=device-width" />
    <meta name="format-detection" content="telephone=no,email=no,date=no,address=no">
    <title>Siam Console </title>
    <link rel="stylesheet" href="../../../css/swiper.min.css">
    <link rel="stylesheet" href="../../../css/common.css">
    <script src="../../script/vue.min.js" type="text/javascript"></script>
    <script src="../../script/frame.js" type="text/javascript"></script>
    <script src="../../script/baseConfig.js" type="text/javascript"></script>
    <script src="../../script/common.js" type="text/javascript"></script>
    <script src="../../script/swiper.min.js" type="text/javascript"></script>
    <style>
    </style>
</head>
<body>
    <div id="app" v-cloak>
        <ul>
          <li v-for="(item,index) in consoleList" style="font-size:14px">
            {{item}}
          </li>
        </ul>
    </div>
    <script type="text/javascript">
        apiready = function() {
            var app = new Vue({
                el:'#app',
                data:{
                  consoleList:[
                    '测试'
                  ]
                },
                methods:{
                  init(){

                              api.refreshHeaderLoadDone();//管他有不有下拉刷新，都给他关了。<Z></Z>
                      let consoleData = YY.getData("siamConsoleData")
                      if (consoleData == ''){
                        consoleData = [];
                      }else{
                        consoleData = JSON.parse(consoleData)
                      }

                      this.consoleList = consoleData;
                  }
                },
                mounted:function(){
                  this.init();
                }
            });


            //下拉刷新
            $.pullDown({
                bgColor:"transparent",
                success:function(){
                  YY.log("??")
                  app.init();
                }
            });
        }

    </script>
</body>
</html>
```

在ajax封装里上报日志

```javascript
// 注入网络请求记录
YY.log(JSON.stringify({
    send :send,
    ret: ret,
    err: err
}))
```

￥￥￥