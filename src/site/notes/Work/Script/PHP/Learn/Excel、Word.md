---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Excel、Word/","title":"Excel、Word","tags":["flashcards","#phpspreadsheet","#csv","#excel","#word"],"noteIcon":"","created":"2026-03-10T22:33:54.000+08:00","updated":"2026-03-24T17:47:28.802+08:00","dg-note-properties":{"title":"Excel、Word","tags":["flashcards","#phpspreadsheet","#csv","#excel","#word"],"reference linking":null}}
---

由于PHPExcel已经不再维护，PhpSpreadsheet是PHPExcel的下一个版本。PhpSpreadsheet是一个用纯PHP编写的库，并引入了命名空间，PSR规范等。这里简单介绍下PhpSpreadsheet的导入导出功能。
# excel
## 1、安装
- 使用composer安装：
```bash
composer require phpoffice/phpspreadsheet
```
- [GitHub - PHPOffice/PhpSpreadsheet: A pure PHP library for reading and writing spreadsheet files](https://github.com/PHPOffice/PhpSpreadsheet)
- [PhpSpreadsheet Documentation](https://phpspreadsheet.readthedocs.io/en/latest/)
- [博客文档](https://www.baiyongj.com/news/506.html)
- [spout/tests/Spout/Writer/CSV/WriterTest.php at master · box/spout · GitHub](https://github.com/box/spout/blob/master/tests/Spout/Writer/CSV/WriterTest.php)
## 2、excel文件导出
### xlsx
```php
/**
 * excel文件导出
 */
function export()
{
    require_once __DIR__ . '/vendor/autoload.php';
 
    $data = [
        ['title1' => '111', 'title2' => '222'],
        ['title1' => '111', 'title2' => '222'],
        ['title1' => '111', 'title2' => '222']
    ];
    $title = ['第一行标题', '第二行标题'];
 
    // Create new Spreadsheet object
    $spreadsheet = new \PhpOffice\PhpSpreadsheet\Spreadsheet();
    $sheet = $spreadsheet->getActiveSheet();
 
    // 方法一，使用 setCellValueByColumnAndRow
    //表头
    //设置单元格内容
    foreach ($title as $key => $value) {
        // 单元格内容写入
        $sheet->setCellValueByColumnAndRow($key + 1, 1, $value);
    }
    $row = 2; // 从第二行开始
    foreach ($data as $item) {
        $column = 1;
        foreach ($item as $value) {
            // 单元格内容写入
            $sheet->setCellValueByColumnAndRow($column, $row, $value);
            $column++;
        }
        $row++;
    }
 
    // 方法二，使用 setCellValue
    //表头
    //设置单元格内容
    $titCol = 'A';
    foreach ($title as $key => $value) {
        // 单元格内容写入
        $sheet->setCellValue($titCol . '1', $value);
        $titCol++;
    }
    $row = 2; // 从第二行开始
    foreach ($data as $item) {
        $dataCol = 'A';
        foreach ($item as $value) {
            // 单元格内容写入
            $sheet->setCellValue($dataCol . $row, $value);
            $dataCol++;
        }
        $row++;
    }
 
    // Redirect output to a client’s web browser (Xlsx)
    header('Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
    header('Content-Disposition: attachment;filename="01simple.xlsx"');
    header('Cache-Control: max-age=0');
    // If you're serving to IE 9, then the following may be needed
    header('Cache-Control: max-age=1');
 
    // If you're serving to IE over SSL, then the following may be needed
    header('Expires: Mon, 26 Jul 1997 05:00:00 GMT'); // Date in the past
    header('Last-Modified: ' . gmdate('D, d M Y H:i:s') . ' GMT'); // always modified
    header('Cache-Control: cache, must-revalidate'); // HTTP/1.1
    header('Pragma: public'); // HTTP/1.0
 
    $writer = \PhpOffice\PhpSpreadsheet\IOFactory::createWriter($spreadsheet, 'Xlsx');
    $writer->save('php://output');
    exit;
}
```
结果：
![image](https://weichengjun2.dpdns.org/i/2023/09/21/650be6168068a.png)
### csv
```php
/**
 * 导出 csv
 * @param $fileName [输出Excel文件名]
 * @param $headList [列名]
 * @param $data [导出数据]
 * @return void
 */
function csvExport($fileName, $headList = [], $data = []): void
{
    $fp = fopen('php://memory', 'rw');
    foreach ($headList as $key => $value) {
        $headList[$key] = mb_convert_encoding($value, 'GBK', 'UTF-8');
    }
    // 写入表头
    fputcsv($fp, $headList);

    // 逐行写入
    $count = count($data);
    for ($i = 0; $i < $count; $i++) {
        $row = $data[$i];
        foreach ($row as $key => $value) {
            $row[$key] = mb_convert_encoding($value, 'GBK', 'UTF-8') . "\t";
        }
        fputcsv($fp, $row);
    }

    // 读取写入的内容
    fseek($fp, 0);
    $content = '';
    while ($_content = fread($fp, 1000)) {
        $content .= $_content;
    }
    fclose($fp);

    // 输出文件
    header('Content-Type: application/vnd.ms-excel');
    header('Content-Disposition: attachment;filename=' . urlencode($fileName));
    header('Cache-Control: max-age=0');
    echo $content;
    exit();
}
```
## 3、excel文件保存到本地
```php
/**
 * excel文件保存到本地
 */
function save()
{
    require_once __DIR__ . '/vendor/autoload.php';
 
    $data = [
        ['title1' => '111', 'title2' => '222'],
        ['title1' => '111', 'title2' => '222'],
        ['title1' => '111', 'title2' => '222']
    ];
    $title = ['第一行标题', '第二行标题'];
 
    // Create new Spreadsheet object
    $spreadsheet = new \PhpOffice\PhpSpreadsheet\Spreadsheet();
    $sheet = $spreadsheet->getActiveSheet();
 
    //表头
    //设置单元格内容
    $titCol = 'A';
    foreach ($title as $key => $value) {
        // 单元格内容写入
        $sheet->setCellValue($titCol . '1', $value);
        $titCol++;
    }
    $row = 2; // 从第二行开始
    foreach ($data as $item) {
        $dataCol = 'A';
        foreach ($item as $value) {
            // 单元格内容写入
            $sheet->setCellValue($dataCol . $row, $value);
            $dataCol++;
        }
        $row++;
    }
 
    // Save
    $writer = \PhpOffice\PhpSpreadsheet\IOFactory::createWriter($spreadsheet, 'Xlsx');
    $writer->save('01simple.xlsx');
}
```
## 4、读取excel文件内容
```php
/**
 * 读取excel文件内容
 */
function read()
{
    require_once __DIR__ . '/vendor/autoload.php';
    $inputFileName = dirname(__FILE__) . '/01simple.xlsx';
    $spreadsheet = \PhpOffice\PhpSpreadsheet\IOFactory::load($inputFileName);
    // 方法二
    $sheetData = $spreadsheet->getActiveSheet()->toArray(null, true, true, true);
    return $sheetData;
}
```
结果:

![image](https://weichengjun2.dpdns.org/i/2023/09/21/650be61761346.png)

## 5、获取单元格公式的值
```php
use PhpOffice\PhpSpreadsheet\IOFactory;
$absExcelPath = $_SERVER['DOCUMENT_ROOT'] . '/data.xlsx';
$spreadsheet = IOFactory::load($absExcelPath);
$worksheet = $spreadsheet->getActiveSheet();
$highestRow = $worksheet->getHighestRow(); // 总行数1, 2，3
$rows = [];
for ($rowIndex = 2; $rowIndex <= $highestRow; $rowIndex++) {
    $row['code'] = $worksheet->getCell("B{$rowIndex}")->getValue();
    $row['name'] = $worksheet->getCell("C{$rowIndex}")->getValue();
    $row['unit'] = strval($worksheet->getCell("D{$rowIndex}")->getValue());
    $row['price'] = $worksheet->getCell("E{$rowIndex}")->getValue();
    $row['quantity'] = $worksheet->getCell("F{$rowIndex}")->getCalculatedValue(); // 获得公式计算值
    $row['amount'] = $worksheet->getCell("G{$rowIndex}")->getCalculatedValue(); // 获得公式计算值
    $row['date'] = $worksheet->getCell("G{$rowIndex}")->getFormattedValue(); // 获得日期的格式化数值
    // 格式化为 Y/m/d 日期格式
    $row['date'] = Date::excelToDateTimeObject($worksheet->getCell("E{$rowIndex}")->getValue())->format('Y/m/d');
    if (!$row['code'] || $row['code'] == 'NULL') {
        $row['code'] = $row['name'];
    }
    $rows[] = $row;
}
```
### 常见问题
**1、Fatal error: Uncaught Error: Class 'PhpOffice\PhpSpreadsheet\Spreadsheet' not found**
这是因为没有自动加载。可以手动引入加载文件。
```
require_once __DIR__ . '/vendor/autoload.php';
```
或者：
```
require_once __DIR__ . '/vendor/phpoffice/phpspreadsheet/src/Bootstrap.php';
```
**2、Fatal error: Interface 'Psr\SimpleCache\CacheInterface' not found**
这是因为没有psr文件，缺少simple-cache模块。如果使用composer安装的话会自动生成。没有的话可以手动下载。

GitHub下载地址：[https://github.com/php-fig/simple-cache/releases](https://github.com/php-fig/simple-cache/releases)

**3、科学计数法处理**
```php
// 填充数据且设置单元格为字符串格式
$sheet->setCellValueExplicitByColumnAndRow($beginDataColumn, $beginDataRow, $value, DataType::TYPE_STRING);
```


# word
```php
echo <<<EOT  
<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w="urn:schemas-microsoft-com:office:word" xmlns="http://www.w3.org/TR/REC-html40">  
<head>  
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>  
    <xml><w:WordDocument><w:View>Print</w:View></xml>  
</head>  
<body>  
<h1 style="text-align: center">xxx的简历</h1>  
<h3>编号：000001</h3>  
<table border="1" cellpadding="3" cellspacing="0" >  
    <tr >  
        <td width="93" valign="center" colspan="2" >姓名</td>  
        <td width="160" valign="center" colspan="4" >xxx</td>  
        <td width="89" valign="center" colspan="2" >学历</td>  
        <td width="156" valign="center" colspan="3" >xxx</td>  
        <td width="125" colspan="2" rowspan="4" align="center" valign="middle" ><!--<img src="" width="120" height="120" />--></td>  
    </tr>  
    <tr >        <td width="93" valign="center" colspan="2" >性别</td>  
        <td width="72" valign="center" colspan="2" >xxx</td>  
        <td width="88" valign="center" colspan="2" >出生年月</td>  
        <td width="89" valign="center" colspan="2" >xxx</td>  
        <td width="68" valign="center" >户籍地</td>  
        <td width="87" valign="center" colspan="2" >xxx</td>  
    </tr>  
    <tr >        <td width="93" valign="center" colspan="2" >身高</td>  
        <td width="72" valign="center" colspan="2" >xxxcm</td>  
        <td width="88" valign="center" colspan="2" >体重</td>  
        <td width="89" valign="center" colspan="2" >xxxkg</td>  
        <td width="68" valign="center" >婚姻状况</td>  
        <td width="87" valign="center" colspan="2" >xxx</td>  
    </tr>  
    <tr >        <td width="93" valign="center" colspan="2" >手机</td>  
        <td width="160" valign="center" colspan="4" >xxx</td>  
        <td width="89" valign="center" colspan="2" >Email</td>  
        <td width="156" valign="center" colspan="3" >xxx</td>  
    </tr>  
    <tr >        <td width="93" valign="center" colspan="2"  style="width:93px;">家庭住址</td>  
        <td width="530" valign="center" colspan="11" >xxx</td>  
    </tr>  
    <tr >        <td width="93" valign="center" colspan="2" rowspan="3">求职意向</td>  
        <td width="93" valign="center" colspan="2">希望从事职业</td>  
        <td width="200" valign="center" colspan="2">xxx</td>  
        <td width="93" valign="center" colspan="2">希望薪资</td>  
        <td width="200" valign="center" colspan="5">xxx元/月</td>  
    </tr>  
    <tr>        <td width="93" valign="center" colspan="2" >希望工作地区</td>  
        <td width="200" valign="center" colspan="2" >xxx</td>  
        <td width="93" valign="center" colspan="2" >食宿要求</td>  
        <td width="200" valign="center" colspan="5" >xxx</td>  
    </tr>  
    <tr>        <td width="93" valign="center" colspan="2" >目前状况</td>  
        <td width="200" valign="center" colspan="9" >xxx</td>  
    </tr>  
    <tr>        <td width="93" valign="center" >自我评价</td>  
        <td width="570" valign="center" colspan="12" >xxx</td>  
    </tr>  
    <tr>        <td width="93" valign="center" >工作经历</td>  
        <td width="570" valign="center" colspan="12" >xxx</td>  
    </tr>  
    <tr>        <td width="93" valign="center" >教育经历</td>  
        <td width="570" valign="center" colspan="12" >xxx</td>  
    </tr>  
    <tr>        <td width="93" valign="center" >培训经历</td>  
        <td width="570" valign="center" colspan="12" >xxx  
    </tr>  
</body>  
EOT;  
ob_start(); // 打开缓冲区  
header('Cache-Control: public');  
header('Content-type: application/octet-stream');  
header('Accept-Ranges: bytes');  
header('Content-Disposition: attachment; filename=test.doc');  
header('Pragma:no-cache');  
header('Expires:0');  
ob_end_flush(); // 输出全部内容到浏览器
```

