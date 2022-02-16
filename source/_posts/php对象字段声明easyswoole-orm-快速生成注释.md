---
title: 'php对象字段声明,easyswoole ORM 快速生成注释'
tags:
  - easyswoole
  - PHP
  - swoole
  - 常见问题
  - 计算机基础
id: '210'
categories:
  - - easyswoole
  - - PHP
  - - 常见问题
date: 2019-11-12 11:40:03
---

# ide提示

在PHPSTORM IDE中，我们可以通过注释给类写明可调用字段名，这样子才有语法提示。 比如在Thinkphp中，虽然允许我们可以通过对象属性方式去调用，但是并没有语法提示。 添加注释后 就舒服很多了。 格式如下

```php
/**
 * Class RefundDetail
 * @property test_field 测试字段名
 */
class RefundDetail extends Mode
{

}
```

使用

```php
$class = new RefundDetail();
$class->test
```

当我们输入一部分的时候，IDE就会提示我们语法啦~直接选中就可以了

# easyswoole

在easyswoole中也是一样的，我们可以快速给类生成注释来达到语法提示 我写了一个小工具，可以通过SQL create table 语句，分析生成注释

```javascript
$("#value").on("change", function () {
    let string = $("#value").val();

    if (string.slice(0, 6) !== "CREATE" && string.slice(0, 6) !== "create") {
        alert("sql非法 请传入create table sql");
        return false;
    }


    let array = string.split(/[\n]/);

    let firstLine = array[0];
    var regExp = /`(.*?)`/gi;
    let dbName = regExp.exec(firstLine)[1];
    let tableName = regExp.exec(firstLine)[1];
    let returnString = `
/**
 * ${tableName}`;
    $.each(array, function (index, item) {
        if (index == 0) {
            return true;
        }
        // 判断是否为索引
        if (item.indexOf("PRIMARY KEY") != -1) {
            return false;
        }
        let regExpField = /`(.*?)`/gi;
        let field = regExpField.exec(item);

        if (field == null) {
            return true;
        }
        let comment = '';
        let commentExp = /'(.*?)'/gi;
        let commentReg = commentExp.exec(item);
        if (commentReg !== null) {
            comment = commentReg[1];
        }
      returnString += `
 * @property $${field[1]} ${comment}`;
    });
  returnString += `
 */`;
  console.log(returnString);
})
```

效果如图 ![](https://www.siammm.cn/wp-content/uploads/2019/11/de0746d4eb7762f74c72dc934fa7d01f.png) 我的博客即将同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite\_code=8vto1mh8z7c8