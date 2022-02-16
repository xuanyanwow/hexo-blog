---
title: 'PHP path_info,Thinkphp controller not exists index\php'
tags:
  - PHP
  - Thinkphp
id: '136'
categories:
  - - PHP
date: 2019-08-24 21:25:32
---

# 写在前面

为什么写下这篇文章，嗯，因为又踩坑了。 问题背景：

*   php7.2
*   nginx
*   thinkphp5

问题体现： url如果`以/为结尾` 比如`index.php/admin/`，不会自动访问默认控制器、方法`index`，而是报错

```
controller not exists:app\admin\controller\index\Php
```

# 求知之路

研究过thinkphp框架源码的，或者深入了解过mvc的，都应该知道thinkphp框架的路由，是根据path\_info值来解析的，甚至传参也可以带在path\_info中

### 排查path\_info的值

一路追踪源码，在`thinkphp\library\think\Request.php` 路径中，找到以下代码

```php
    /**
     * 673行左右
     *
     *
     * 获取当前请求URL的pathinfo信息（含URL后缀）
     * @access public
     * @return string
     */
    public function pathinfo()
    {
        if (is_null($this->pathinfo)) {
            if (isset($_GET[$this->config['var_pathinfo']])) {
                // 判断URL里面是否有兼容模式参数
                $pathinfo = $_GET[$this->config['var_pathinfo']];
                unset($_GET[$this->config['var_pathinfo']]);
                unset($this->get[$this->config['var_pathinfo']]);
            } elseif ($this->isCli()) {
                // CLI模式下 index.php module/controller/action/params/...
                $pathinfo = isset($_SERVER['argv'][1]) ? $_SERVER['argv'][1] : '';
            } elseif ('cli-server' == PHP_SAPI) {
                $pathinfo = strpos($this->server('REQUEST_URI'), '?') ? strstr($this->server('REQUEST_URI'), '?', true) : $this->server('REQUEST_URI');
            } elseif ($this->server('PATH_INFO')) {
                $pathinfo = $this->server('PATH_INFO');
            }

            // 分析PATHINFO信息
            if (!isset($pathinfo)) {
                foreach ($this->config['pathinfo_fetch'] as $type) {
                    if ($this->server($type)) {
                        $pathinfo = (0 === strpos($this->server($type), $this->server('SCRIPT_NAME'))) ?
                        substr($this->server($type), strlen($this->server('SCRIPT_NAME'))) : $this->server($type);
                        break;
                    }
                }
            }

            if (!empty($pathinfo)) {
                unset($this->get[$pathinfo], $this->request[$pathinfo]);
            }

            $this->pathinfo = empty($pathinfo)  '/' == $pathinfo ? '' : ltrim($pathinfo, '/');
        }

        return $this->pathinfo;
    }
```

我尝试在这个方法里（目前来看，这里是分析path\_info的第一门关）打印`$_SERVER['PATH_INFO']` 打印出来的值大概为`admin/index.php` 然后在后续解析中，又会把.替换成/ 也就是`admin/index/php` 对应我们报错的`app\admin\controller\index\Php`类

### 分析path\_info来源

我们知道，`$_SERVER`超全局变量是在php中自动维护的，所以它的来源肯定来自以下两个方面之一

*   php底层
*   web服务器

经过找一些资料，我得知了该变量的值是来自`web服务器`，也就是我使用的nginx 宝塔安装的nginx，会自动维护很多常用配置，比如不同版本的php配置、path\_info配置等等（有些自己编译安装的php没有path\_info 需要自己添加） 在`/www/server/nginx/conf` 下有多个php版本的配置文件，在其中有一个配置项

```
fastcgi_index index.php;
```

fastcgi是什么意思大家可以先自行补充 ^ \_ ^ 也就是该配置项影响了我们的运行 它的定义可以简单理解为：

```
默认值：none
使用字段：http, server, location
如果URI以斜线结尾，文件名将追加到URI后面，这个值将存储在变量$fastcgi_script_name中
```

测试： 把index.php改为index2.php 访问程序，报错变为：`controller not exists:app\admin\controller\index2\Php` 可以证实是该配置影响结果

# 总结处理

Web服务器该配置影响了程序运行，那么我们如何解决该问题

*   ① 修改thinkphp底层，把path\_info最后的index.php替换掉
*   ② 修改web服务器该配置为none 去除
*   ③ 修改程序，遵循规范

基于业务迁移、兼容不同环境考虑，我选择第三种方案。也就是修改程序，不允许跳转或者访问带/结尾。