---
title: layui数据表格checkbox设置部分不可选
tags:
  - 前端
id: '162'
categories:
  - 前端
date: 2019-08-30 10:46:52
---

### 问题

在layui数据表格中设置了字段为`type:checkbox` 但是想要实现部分不显示，不可选的功能。 [![layui数据表格](http://www.yancoo.cn/uploads/images/201905/30_01_layuitable.png "layui数据表格")](# "layui数据表格") layui内置没有该功能，所以只能自己实现。

## 使用templet实现

```javascript
table.render({
    elem: '#junTable',
    url: '',
    cols: [[
        {
            templet: "#checkbd",
            title: "<input type='checkbox' name='siam_all' title='' lay-skin='primary' lay-filter='siam_all'> ",
            width: 60,
        }
        , {
            field: 'z_id',
            title: 'id'
        }
    ]],
    page: true,
    limit: 10
});
```

```markup
<script type="text/html" id="checkbd">
    {{#  if (d.can_fabu === 1){ }}// 这里是判断要不要显示的条件
    <input type="checkbox" name="siam_one" title="" lay-skin="primary" data-id = "{{ d.z_id }}">
    {{#  } }}
</script>
```

```css
<style>
    .laytable-cell-checkbox .layui-disabled.layui-form-checked i {
        background: #fff !important;
    }
</style>
```

到这里就可以部分数据`不显示复选框`了，但是全选功能和获取id的功能还是不正常

### 全选功能

```javascript
form.on("checkbox(siam_all)", function () {
    var status = $(this).prop("checked");
    $.each($("input[name=siam_one]"), function (i, value) {
        $(this).prop("checked", status);
    });
    form.render();
});
```

### 获取选中数据

```javascript
var ids = [];
$.each($("input[name=siam_one]:checked"), function (i, value) {
    ids[i] = $(this).attr("data-id");  // 如果需要获取其他的值 需要在模板中把值放到属性中 然后这里就可以拿到了
});
```

## 使用done函数禁用

这是网上的做法，但是有瑕疵，全选不可用，并且不可选状态和可选状态的复选框样式很接近，建议重写不可选的样式 （参考上面的）

```markup
<!DOCTYPE html>
<html>

    <head>
        <meta charset="utf-8" />
        <title>layui</title>
        <meta name="renderer" content="webkit" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
        <link rel="stylesheet" href="https://res.layui.com/layui/dist/css/layui.css" media="all" />
        <!-- 注意：如果你直接复制所有代码到本地，上述css路径需要改成你本地的 -->
    </head>

    <body>
        <div style="margin-bottom: 5px;">
            <div id="table-main">
<span class="layui-btn" id="getselect">获取选中行</span>

                <table class="layui-table" id="idTest" lay-filter="demo"></table>
            </div>
            <script type="text/html" id="barDemo">
                < a class = "layui-btn layui-btn-primary layui-btn-mini"
                lay - event = "detail" > 查看 < /a>
            </script>
            <script src="https://res.layui.com/layui/dist/layui.all.js" charset="utf-8"></script>
            <!-- 注意：如果你直接复制所有代码到本地，上述js路径需要改成你本地的 -->
            <script>
                layui.use(['table', 'jquery', 'form', 'layer'], function() {
                    var table = layui.table;
                    var $ = layui.jquery;
                    var form = layui.form;
                    var layer = layui.layer;
                    var tableIns = table.render({ //其它参数在此省略
                        elem: '#idTest',
                        id: 'idTest',
                        url: 'https://www.layui.com/demo/table/user/', // 注意：如果你直接复制所有代码到本地，数据请求需要本地返回数据
                        cols: [
                            [{
                                    checkbox: true,
                                    fixed: true
                                },
                                {
                                    field: 'id',
                                    width: 80,
                                    sort: true,
                                    fixed: true,
                                    title: 'ID'
                                },
                                {
                                    fixed: 'right',
                                    width: 160,
                                    align: 'center',
                                    toolbar: '#barDemo'
                                }
                            ]
                        ],
                        where: {}, //如果无需传递额外参数，可不加该参数
                        limits: [10, 15, 20, 40, 60, 80],
                        limit: 10,
                        page: true, //开启分页
                        done: function(res, curr, count) {
                            var data = res.data;
                            var allck = true;
                            for (var item in data) {
                                if (data[item].score == 57) { //关键点如果data中score包含57那么就不能全选
                                    allck = false;
                                }
                                break;
                            }
                            if (!allck) {
                                $(".layui-table-header").find("input[name = 'layTableCheckbox'][lay-filter='layTableAllChoose']").each(function() {
                                    $(this).attr("disabled", 'disabled').next().removeClass("layui-form-checked");
                                    form.render('checkbox');
                                });
                            }
                            var i = 0;
                            $(".layui-table-body.layui-table-main").find("input[name='layTableCheckbox']").each(function() {
                                if (res.data[i].score == 57) { //关键点如果当前行数据中score包含57那么就不可选
                                    $(this).attr("disabled", 'disabled').removeAttr("checked");
                                    form.render('checkbox');
                                }
                                i++;
                            });
                            i = 0;
                            $(".layui-table-fixed.layui-table-fixed-l").find(".layui-table-body").find("input[name='layTableCheckbox']").each(function() {
                                if (res.data[i].score == 57) { //关键点如果当前行数据中score包含57那么就不可选
                                    $(this).attr("disabled", 'disabled').removeAttr("checked");
                                    form.render('checkbox');
                                }
                                i++;
                            });
                        }
                    });
                    //监听表格复选框选择
                    table.on('checkbox(demo)', function(obj) {
                        console.log(obj)
                    });
                    $("#getselect").click(function() {
                        var checkStatus = table.checkStatus('idTest'); //test即为基础参数id对应的值
                        layer.alert(JSON.stringify(checkStatus.data));
                    });
                });
            </script>
        </div>
    </body>

</html>
```