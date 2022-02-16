---
title: PhpSpreadsheet导出Excel表格，长数字自动转科学计数法
tags:
  - PHP
id: '143'
categories:
  - - PHP
date: 2019-08-30 10:19:35
---

## 原代码

```
public function down($data)
{
    $spreadsheet = new Spreadsheet();
    $sheet       = $spreadsheet->getActiveSheet();

    $lieCount = count($data['data'][0]);
    # 全部设为自动列宽
    for($i=65;$i< (65 +$lieCount);$i++){
        $sheet->getColumnDimension(strtoupper(chr($i)))->setAutoSize(true);
    }
    # 最快捷设置数据
    $sheet->fromArray($data['data']);
    # 导出
    $writer = new Xlsx($spreadsheet);
    $writer->save('php://output');
}
```

这样子就可以实现传入一个数组data，然后快速导出成Excel表格了。 但是遇到长数字的时候，就会被转成科学计数法的数字，并且会丢失最后的精度 全部转成了 `0` 原因：

> 凡数字超过11位数，Excel 表格就会用科学记数法显示。如果要输入超过11位的数，得把单元格设为文本形式或在输入数字前先输入一个英文单引号（'）。(单引号在英文输入法下输入）

也就是在传入data之前先遍历 添加符号 但是这样子在我们程序自动导出是不能生效的，需要我们再 `双击单元格` 它才会转成文本形式。 导出后的效果为 `'11111111111111`

## 网上的方案 （ PHPExcel 旧版的 ）

*   1.  在数据前后加上 `\t` 跟 `'` 差不多
*   2.  $objActSheet->setCellValueExplicit('A1', '330602198804224688', PHPExcel\_Cell\_DataType::TYPE\_STRING);
*   3.  $objActSheet->setCellValue('A1', ' '.'330602198804224688');

## PhpSpreadsheet 解决

> 当然是除了拼接字符串的方案了！ 以下划重点 要考！

PhpSpreadsheet也有它的前驱者PHPExcel一样的方式，可以通过setCellValueExplicit指定方案。 所以将原来的程序改造成以下

```
    private $mustStringArray = [];

    /**
     * 将列强制设置成文本，避免长文本出现转科学计数法
     * @param array $array
     */
    function setMustString(Array $array)
    {
        $this->mustStringArray = $array;
    }

    /**
     * 导出表格
     * @todo 弹窗导出表格
     * @param $data array 数组 可选:filename文件名,data数据(二维数组),
     * @return bool
     * @throws \PhpOffice\PhpSpreadsheet\Exception
     * @throws \PhpOffice\PhpSpreadsheet\Writer\Exception
     */
    public function down($data)
    {
        if (empty($data)) return false;

        $filename = !empty($data['filename']) ? $data['filename'] . ".xlsx" : date('Y-m-d H:i:s') . ".xlsx";

        header('Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
        header('Content-Disposition: attachment;filename="' . $filename . '"');
        header('Cache-Control: max-age=0');

        $spreadsheet = new Spreadsheet();
        $sheet       = $spreadsheet->getActiveSheet();

        $lieCount = count($data['data'][0]);
        for($i=65;$i< (65 +$lieCount);$i++){
            $sheet->getColumnDimension(strtoupper(chr($i)))->setAutoSize(true);
        }

        // 以下代码基于fromArray改造
        // 开始列和开始行数，默认全部
        $startColumn = 'A';
        $startRow = '1';

        foreach ($data['data'] as $rowData) {
            $currentColumn = $startColumn;
            foreach ($rowData as $cellValue) {
                    if ($cellValue !== null) {
                        if (in_array($currentColumn, $this->mustStringArray)){
                            $sheet->getCell($currentColumn . $startRow)->setValueExplicit($cellValue,'s');
                        }else{
                            $sheet->getCell($currentColumn . $startRow)->setValue($cellValue);
                        }
                    }
                ++$currentColumn;
            }
            ++$startRow;
        }

        $writer = new Xlsx($spreadsheet);
        $writer->save('php://output');
    }
```

## 关键代码

```
$sheet->getCell($currentColumn . $startRow)->setValueExplicit($cellValue,'s');
```

第二个参数其实也是要传入一个类的静态变量，然后我追踪了它的代码，直接将值给写进去了~ 有兴趣的同学可以查看这个类文件 `PhpOffice\PhpSpreadsheet\Cell\DataType` 里面还有其他几个类型的常量列表

```
// Data types
const TYPE_STRING2 = 'str';
const TYPE_STRING = 's';
const TYPE_FORMULA = 'f';
const TYPE_NUMERIC = 'n';
const TYPE_BOOL = 'b';
const TYPE_NULL = 'null';
const TYPE_INLINE = 'inlineStr';
const TYPE_ERROR = 'e';
```