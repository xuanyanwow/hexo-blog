---
title: 'layui2.0数据表格导出复杂表头EXCEL解决方案,table2excel'
tags:
  - 前端
id: '167'
categories:
  - 前端
date: 2019-08-30 10:50:03
---

### layui 数据表格组件

layui是一套面向所有层次的前后端开发者，零门槛开箱即用的前端UI解决方案。很多的后端开发在开发后台系统时候都会选择它。 数据表格组件也是使用非常频繁的，它可以快速从api得到数据并进行处理渲染成表格，并且还有排序、总计、导出表格等等功能。 ![layui数据表格](http://cdn.yancoo.cn/img/20190128/layuitable.png "layui数据表格") 在一次的需求中，需要使用复杂表头并且导出EXCEL表格，发现layui并不支持复杂表头的处理，社区之中也还未找到相关的方案。于是使用了table2excel插件协助完成需求。（如果你有更好更方便的方法，希望你能联系我或者留言交流一下，谢谢） 以下简单记一下笔记和步骤，方便自己和他人。

### talbe2excel

> https://github.com/rusty1s/table2excel

在github上有挺多个叫table2excel的仓库，我选择了以上这个仓库。 在页面引入jquery和table2excel.js 一个快速的demo

```javascript
<script src="table2excel.js"></script>

<script>
  var table2excel = new Table2Excel();
  table2excel.export(document.querySelectorAll("table"));
</script>
```

> 但是此方式在layui生成的数据表格中并不适用。具体原因和解决方案有空待研究~ 其他小伙伴也可以补充哦！

原生写的table标签可以正常导出，并且可以使用复杂表头。 于是绕了一下弯路，在layui数据表格加载完数据后，在页面操作原生tableDom（并且隐藏起来 (_╹▽╹_) ），再使用table2excel导出表格。

```markup
<table id="report-table" cellpadding=1 cellspacing=1 border =1>
    <tr>
        <th rowspan="2">id</th>
        <th colspan="2">信息</th>
    </tr>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
    </tr>
    <tbody id="report-table-tbody">
        <tr>
            <th>1</th>
            <th>Siam</th>
            <th>19</th>
        </tr>
    </tbody>
</table>
```

> javascript代码

```javascript
table.render({ //其它参数在此省略
  done: function(res, curr, count){
    $.each(res.data,function(index,value){
        let html = '';

        html += '<tr>';
        html += '<td>';
        html += '<td>'+value.name+'</td>';
        html += '<td>'+value.other+'</td>';
        html += '</td>';
        html += '</tr>';
        $('#report-table-tbody').append(html);
    });

  }
});
```

```javascript
$('#report-table-downexcel').click(function(){
  var table2excel = new Table2Excel();
  table2excel.export($('#report-table'));
})
```

这样子就可以完成导出复杂表头的表格了。