---
title: 手把手教你写一个composer包
tags:
  - PHP
id: '144'
categories:
  - - 后端
date: 2019-08-30 10:20:29
---

## 什么是composer

`Composer` 是一个命令行工具，它的作用就是帮我们的项目`管理所依赖的开发包`，属于`依赖包管理工具`。

## 什么是依赖包管理工具

由于程序届的《开源运动》，我们可以在社区找到很多别人提供的工具，也可以向社区贡献我们的代码。 在github还没有兴起的年代，我们是需要到工具的官网下载代码，比如jquery。然后放到我们自己的项目目录里，再在我们的页面中使用。 但是...当一个网站依赖的包越来越多，我们会发现这是一件非常麻烦的事情：

*   安装新包，代码目录管理等问题
*   升级依赖包，只能手动替换文件
*   安装过程发现某个包需要依赖于另一个包，那我们又要开始安装前面的包...
*   ...

哪怕是每个开源包都有自己的官网，也还是需要用户去下载安装。 即使是有一个类似`github`的大仓库，可以存放所有的项目，实现一站下载。 但不同依赖包之间的关系也还是需要手动维持，包与包之间的关系有可能是相互依赖，也可能是冲突的。

> 比如A包需要的PHP版本是小于7的，而B包需要的PHP版本是大于等于7的，那么你安装下来就是有冲突的，可能你在使用之前都没有注意环境的要求，直到遇到各种坑....

于是就有`依赖包管理工具`的诞生了，如前端使用的`npm`，java使用的`maven`，安卓的`Gradle`等等。在PHP中我们使用的工具叫`Composer`。 当我们需要加载一个新包的时候，我们只需要一条命令，工具就会自动构建安装，并且在安装之前如果检测到环境或者需要依赖其他包，管理工具也会做出相应的处理，比如提示终止、自动安装依赖包。 当我们需要更新包的时候，我们也只需要一条命令，就可以实现工具包的升级，在这过程也依然会检测新版本包需要的环境和依赖等。 所以说，`依赖包管理工具` 为我们在管理依赖包的工作上带来了极大的便利。

## 安装Composer

首先我们需要把composer下载到本地，并且执行安装。在这过程中会检测php的的参数设置，如果某些参数未正确设置则会给出警告。 分别运行以下三条命令

```
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
```

```
php composer-setup.php
```

```
php -r "unlink('composer-setup.php');"
```

#### 局部使用

如果是局部使用，我们到这里就可以了，只需要把下载安装下来的`composer.phar`复制到你的项目根目录，并且执行`php composer.phar`就可以看到提示的内容啦~

#### 全局安装

如果想要在任意的项目目录都可以执行使用的话，我们需要把composer包设置一个系统环境变量。 **Mac或者linux** 把安装包移动到`/usr/local/bin/`目录

```
sudo mv composer.phar /usr/local/bin/composer
```

**windows**

*   首先确保你的php已经在环境变量中（也就是在任意目录打开cmd都可以执行php命令）
*   把composer.phar复制到以上说的php目录中，跟`php.exe`同个级别
*   新建一个`composer.bat`文件，写以下内容并保存，该文件提供composer命令入口。

```
@php "%~dp0composer.phar" %*
```

然后就可以在项目目录里打开cmd 并执行`composer --version`检测看看安装是否成功了！

## 写一个属于自己的composer包

前面我们已经介绍了依赖包管理工具的便利，许许多多的先辈在开源工具的时候往往都会上传到`github`或者制作成`composer包`。

> 如果传到github，可以解决下载问题，但是与其他包的依赖关系却得不到自动处理，一般用于开源整个项目。

.

> composer包则适合在需要与其他依赖包配合的情况下使用。

那么我们下面就来写一个自己的composer包吧， 首先我们创建一个空的目录，并且运行以下命令初始化一个空白的composer包

```
composer init
```

可以在命令窗口看到有返回提示； **需要输入包名**

```
This command will guide you through creating your composer.json config.`
Package name (<vendor>/<name>) :
```

我这里写的是`yancoo.cn/test`，回车 **需要输入描述**

```
Description []:
```

包的描述内容 **需要输入作者**

```
Author [宣言 <59419979@qq.com>, n to skip]:
```

该选项如果有默认值 可以直接回车 **需要输入最低稳定版本**

```
Minimum Stability []:
```

该选项有可选值：stable, RC, beta, alpha, dev 一般填dev **需要输入包类型**

```
Package Type (e.g. library, project, metapackage, composer-plugin) []:
```

可选项在提示中已经有了，我们一般选`library`填`l`即可 **需要输入开源协议**

```
License []:
```

根据自己情况填写，我们填`Apache` **设置包需要依赖的其他环境或者包**

```
Define your dependencies.

Would you like to define your dependencies (require) interactively [yes]?
```

如果需要设置依赖环境或者其他包 则输入yes回车，会让你搜索，我们这里给包设置php版本必须大于5.6，所以搜索php。

```
Enter the version constraint to require (or leave blank to use the latest version):
```

输入最低要求版本号 >=5.6.0 [![composer初始化](http://blog.siammm.cn/wp-content/uploads/images/201904/composer_01.png "composer初始化")](# "composer初始化") 如果需要设置多个环境要求，则重复搜索填写即可，如果不需要了，则在`Search for a package:`不填写内容，直接`回车`即可 接下来设置依赖包

```
Would you like to define your dev dependencies (require-dev) interactively [yes]?
```

我们不需要 直接回车回车。

```
{
    "name": "yancoo.cn/test",
    "description": "This is a test package,form yancoo.cn,author is siam.",
    "type": "l",
    "require": {
        "php": ">=5.6.0"
    },
    "license": "Apache",
    "authors": [
        {
            "name": "宣言",
            "email": "59419979@qq.com"
        }
    ],
    "minimum-stability": "dev"
}

Do you confirm generation [yes]?
```

确认信息，yes 回车 初始化完成 Do you confirm generation \[yes\]? yes Would you like to install dependencies now \[yes\]? yes 安装完成，可以看到我们的目录生成了如下结构。 [![composer目录](http://blog.siammm.cn/wp-content/uploads/images/201904/composer_02.png "composer目录")](# "composer目录")

> `vendor`是composer存放包的地址，所有的包都存放在该路径中统一管理，并且composer提供了命名空间自动加载的功能，我们在使用过程中只需要引入composer的统一入口文件即可

```php
require "vendor/autoload.php";
```

但是我们也需要在包的配置文件中写上需要自动加载哪个目录才有效哈！ 编辑composer.json文件 [![composer的dump-autoload](http://blog.siammm.cn/wp-content/uploads/images/201904/composer_03.png "composer的dump-autoload")](# "composer的dump-autoload") 这里的路径需要根据你自己的来定，也可以参考以上写法即可。 写完之后需要运行一下命令行`composer dump-autoload`更新composer的命名空间与文件夹映射关系。

> 上一步骤非常重要 漏了就不正常执行了。

然后我们创建`src`文件夹，再创建`siam`文件夹，在里面创建`Test.php`文件 写上命名空间 Siam; 根据psr-4规范，类名要与文件名相同。

```php
<?php
//Test.php文件
namespace Siam;

class Test
{
    function test()
    {
        echo "Form Test -> test()";
    }
}
```

再在最外层写下index.php测试文件，正确做法是创建demo文件夹 然后再写测试文件。

```
<?php
require "vendor/autoload.php";

$Test = new Siam\Test();
$Test->test();
```

于是我们现在的文件目录结构如下，运行index.php正确得到内容`Form Test -> test()` [![demo目录结构](http://blog.siammm.cn/wp-content/uploads/images/201904/composer_04.png "demo目录结构")](# "demo目录结构") 可以根据你自己的想法来写类，只需要注意命名空间的层级与文件夹层级相同，类名与文件名相同即可自动加载。

## 上传composer包

我们需要把包上传到仓库中，别人才可以通过composer命令安装 我们需要先把包上传到github中，再把github仓库地址复制到composer官方仓库提交，composer就会自动拉取你的包并且提供给别人下载安装了！  

### 首先在github上创建仓库

[![创建仓库](http://blog.siammm.cn/wp-content/uploads/images/201904/composer_05.png "创建仓库")](# "创建仓库")  
[![git仓库](http://blog.siammm.cn/wp-content/uploads/images/201904/composer06.png "git仓库")](# "git仓库")  

### 在你本地电脑上运行命令，将我们刚刚创建的示例包文件与github仓库关联起来，并推送上去。

### 推送成功之后，复制仓库的地址到composer官方提交。

[![提交comopser](http://blog.siammm.cn/wp-content/uploads/images/201904/composer07.png "提交comopser")](# "提交comopser")  

### composer的官方仓库是`https://packagist.org/`

我们打开，并且注册一个账号。然后点击右上角的submit。把地址填写进去即可。 [![提交comopser](http://blog.siammm.cn/wp-content/uploads/images/201904/composer08.png "提交comopser")](# "提交comopser") 到这里，我们的github仓库与packagist已经建立了关联，但是这个使用还是用不了的，因为我们还没有发布正式版本！ 需要在git上打标签，才认为我们发布了一个新的版本，packagist才会去拉取并且提供给别人安装。 git打标签需要运行以下命令

```
git tag -a v1.0.1 -m "第一个版本"
git push origin v1.0.1
```

然后就在线上仓库打标签成功了，此时运行composer安装也正常了 在新的一个空白目录中运行

```
composer require yancoo.cn/test    是我们初始化填写的包名！
```

因为我们在开发composer包的目录，跟在实际项目中安装后的有一些不一样。 所以记得自己新建测试文件，载入composer的自动加载文件然后再测试哦

> 我们开发的包，只是项目所有包的其中之一，还有很多其他包，所以目录结构不一样