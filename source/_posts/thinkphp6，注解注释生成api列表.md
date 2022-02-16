---
title: Thinkphp6，注解注释生成api列表
tags:
  - PHP
  - Thinkphp
id: '300'
categories:
  - - PHP
  - - 常见问题
date: 2021-04-30 15:57:40
---

自己写的一个小功能需要用到，稍微存一下该段代码， 接口代码需要写的格式 ![](https://www.siammm.cn/wp-content/uploads/2021/04/wp_editor_md_9597b279b4f9cf9a7838ef38b4586077.jpg)

```php
// 遍历controller目录下的文件，判断注释中是否包含@Siam-Api
$dir = app_path()."/controller/";
$dir_contain = scandir($dir);
unset($dir_contain[0]);
unset($dir_contain[1]);// . 和 ..

$doc_list = [];

foreach ($dir_contain as $child_path){
    $path = $dir.$child_path . "/";
    // 只遍历控制器目录下的文件 不遍历子目录 可以自行改造
    if (is_dir($dir.$child_path)) continue;

    $file_list = glob($path. '*.php');
    foreach ($file_list as $file_path){
        $className = basename($file_path);
        $className = str_replace(".php", "",$className);

        $namespace = app()->getNamespace()."\\controller\\".$className;

        $class_reflec = new ReflectionClass($namespace);
        $method_list = $class_reflec->getMethods();
        foreach ($method_list as $method_reflec){
            // 包含@Siam-Api才需要解析源码
            $method_doc = $method_reflec->getDocComment();
            if (strpos( $method_doc,"@Siam-Api") == false) continue;
            $method_code = FileHelper::get_file_content_by_line($file_path, $method_reflec->getStartLine(), $method_reflec->getEndLine());

            // 参数解析逻辑，可以通过代码解析，也可以自己在注释里声明  然后解析 逻辑跟@Siam-Api这个字段一致

            // 解析源码里是否有validate  没有则是无参数
            if (strpos($method_code, "validate([") === false) {
                $param = [];
            }else{
                $param = [];
                // 解析参数列表
                $pattern_1 = '/\$this->validate\(([\s\S]*?)\);/';
                preg_match($pattern_1, $method_code, $matches);
                if (!isset($matches[1])) continue;
                // 拿到validate的代码，按行解析
                $param_code = explode("\n", $matches[1]);
                foreach ($param_code as $param_code_one){
                    if (strpos($param_code_one, "=>") === false) continue;
                    $param_code_one = str_replace(" ", "", $param_code_one);
                    // 正则匹配取出 字段名、规则、注释
                    $param_field_pattern = '/\'([\s\S]*?)\'=>\'([\s\S]*?)\'([\s\S]*)/';
                    preg_match($param_field_pattern, $param_code_one, $matches_param_field);
                    $param_field_note = str_replace(",", "", $matches_param_field[3]);
                    $param_field_note = str_replace("//", "", $param_field_note);

                    $param[] = [
                        $matches_param_field[1],// 字段名
                        $matches_param_field[2],// 规则
                        $param_field_note,// 注释
                    ];
                }
            }
            // groupName method_path method_dec逻辑自行扩展
            $doc_list['groupName'][] = [
                'method_name' => $method_reflec->getName(),
                'method_path' => '',
                'method_dec'  => '',
                'param'       => $param
            ];
        }
    }
}
// 应该返回成json等格式，然后前段html接入
var_dump($doc_list);
```