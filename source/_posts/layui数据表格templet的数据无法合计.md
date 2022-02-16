---
title: Layui数据表格templet的数据无法合计
tags:
  - 前端
id: '119'
categories:
  - 前端
date: 2019-08-09 23:00:02
---

# 写在前面

在使用layui数据表格的时候，有一个列是使用templet，依据其他两个列数据计算得到。 在该列开启合计行，一直显示是0 。其他两列数据合计正常。 在社区和百度上寻找过答案，并没有相关介绍。 在解决了该问题后，写下这篇小记录。

# parseData

配置中提供了parseData方法，可以在请求了接口之后，进一步处理数据格式。 以下是官网的示例

```javascript
table.render({
  elem: '#demp'
  ,url: ''
  ,parseData: function(res){ //res 即为原始返回的数据
    return {
      "code": res.status, //解析接口状态
      "msg": res.message, //解析提示文本
      "count": res.total, //解析数据长度
      "data": res.data.item //解析数据列表
    };
  }
  //,…… //其他参数
});
```

以上的场景，应该在parseData里计算出新的列，然后再渲染到表格里

```javascript
let data = [];
$.each(obj.data, function (index, item) {
    let tem = {
        game_diffcoins: item.game_hardcoins - item.game_coin,
        game_diffjifen: item.game_hardjifen - item.game_jifen,
    };
    data.push($.extend(tem, item))
});
obj.data = data;
```

# 个人理解

templet 应该用来实现样式的调整，比如根据值的不同显示不同颜色 而数据的计算 得出，应该在parseData 或者直接就在接口里计算好返回。