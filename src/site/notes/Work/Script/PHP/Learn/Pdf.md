---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Pdf/","title":"Pdf","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:26:56.000+08:00","updated":"2026-03-24T17:47:37.944+08:00"}
---

官方文档：[Image() – mPDF functions – mPDF Manual](http://mpdf.github.io/reference/mpdf-functions/image.html)
# 1. 基本使用

## 安装
1.在项目下composer文件中 添加 "mpdf/mpdf":"~7.1.9" 测试环境为Yii项目 其他框架composer安装大同小异
```shell
composer require "mpdf/mpdf" "~7.1.9"
```

然后在bash命令行执行
```shell
composer update mpdf/mpdf
```

安装完毕之后打开mpdf文件夹下这些目录的写权限
```shell
src/
tmp/
ttfonts/
```

在controller中使用
```php
use Mpdf\Mpdf;

// yii中获取 html 的方法 PDFHtml为需要转换的html文件 $data为动态展示的数据
$html = "抓取html页面作为字符串赋值到$html中使用 例如:<h1>我是html中的一个标题</h1> 可以直接使用file_get_content()获取";


$mpdf = new Mpdf();
// 设置中文字体
$mpdf->autoScriptToLang = true;
$mpdf->autoLangToFont = true;
$mpdf->WriteHTML($html);

// 直接展示 
$mpdf->Output();
// 保存文件
$mpdf->Output('路径和文件名称.pdf');
```
# 2.基本定制化使用
## 安装
官方手册中安装方法为composer安装 Packagist php扩展包库地址 https://packagist.org/packages/mpdf/mpdf
截止到2019年5月 mpdf版本更新到8.0.1>，主要大版本为6~. 7~. 8~
6~版本 php版本要求为5.4-7.0，版本不符会报错，不需要php扩展依赖
7~版本 php版本要求为5.6- 7.2，需要php扩展支持 gd库与mbstring库
8~版本 php版本要求为5.6-7.3，需要php扩展支持 gd库与mbstring库，php版本高可以试下。目前官方文档支持到7.1.

更改目录权限
7~版本需要打开mpdf以下目录的写权限。
```php
src/
tmp/
ttfonts/
```

 6~版本打开以下目录的写权限。
```php
ttfontdata/
tmp/
graph_cache/
```
## 使用
引用方法大版本有明显差异
```php
//6~版本
use mpdf;
$mpdf = new mPDF();

//7~版本
use Mpdf\Mpdf;
$mpdf = new mPDF();
$mpdf = new Mpdf();
```
## 属性设置及方法参数
### 输出方法 Output()
```php
$mpdf = new \Mpdf\Mpdf();
$mpdf->WriteHTML('Hello World');
$name = "路径和文件名称.pdf";

// 1 直接展示在浏览器页面
$mpdf->Output();

// 2 以文件形式保存到服务器
$mpdf->Output($name);

// 3 带第二个参数
$mpdf->Output($name, "F"); //服务器保存以$name为名称的文件
$mpdf->Output($name, "D"); //浏览器下载以$name为名称的文件
$mpdf->Output($name, "S"); //以字符串形式返回 $name忽略
$mpdf->Output($name, "I"); //浏览器展示,但当用户另存为时以$name为默认文件名
```
### 写入方法 WriteHTML()
```php
$mpdf->WriteHTML($html, 0); // 默认 以html为标准分析写入内容
$mpdf->WriteHTML($css, 1); // 会以style样式录入写入内容 等同于<style></style>标签中的内容
$mpdf->WriteHTML($body, 2); // 会以html body体形式录入写入内容 等同于<body></body>标签内的内容
$mpdf->WriteHTML($thml, 0); // 默认 以html为标准
```
自动分析录入内容字体
```php
$mpdf->autoScriptToLang = true;
$mpdf->autoLangToFont = true;
```
### 加水印
```php
// 参数一是图片的位置(图片相对目录 为处理脚本的相对目录)，参数二是透明度0.1-1
$mpdf->SetWatermarkImage('../web/static/img/water-min.png', 1);
$mpdf->showWatermarkImage = true;
```
### 设置页眉页脚
```php
// 设置PDF页眉内容
$header = '<table width="95%" style="margin:0 auto;border-bottom: 1px solid #4F81BD; vertical-align: middle; font-family:serif; font-size: 9pt; color: #000088;"><tr>
<td width="10%"></td><td width="80%" align="center" style="font-size:16px;color:#A0A0A0">页眉</td>
<td width="10%" style="text-align: right;"></td></tr></table>';

// 设置PDF页脚内容
// 在页脚html中添加 {PAGENO}/{nb} (当前页/总页数) 可添加页码
$footer = '<table width="100%" style=" vertical-align: bottom; font-family:serif; font-size: 9pt; color: #000088;"><tr style="height:30px"></tr><tr>
<td width="10%"></td><td width="80%" align="center" style="font-size:14px;color:#A0A0A0">页脚</td>
<td width="10%" style="text-align: left;">页码：{PAGENO}/{nb}</td></tr></table>';

$mpdf->SetHTMLHeader($header);
$mpdf->SetHTMLFooter($footer);
```
### 合并多张PDF
```php
$mpdf = new Mpdf();
$mpdf->SetImportUse(); 

// 单页pdf导入方式
$pagecount = $mpdf->SetSourceFile('单页PDF地址');
$tplId = $mpdf->ImportPage(1);
$mpdf->UseTemplate($tplId);
$mpdf->WriteHTML(''); 

// 多页pdf导入方式
$pagecount = $mpdf->SetSourceFile('多页PDF地址');
for ($i = 1; $i <= $pagecount; $i++) {
    $mpdf->AddPage();
    $tplId = $mpdf->ImportPage($i);
    $mpdf->UseTemplate($tplId);
    $mpdf->WriteHTML('');
}
$mpdf->Output($outPut);
```
### 关于字体设置
```php
// 获取mpdf扩展字体配置文件目录(此处路径需要根据项目空间规则修改 例如我的是设置了命名空间 Mpdf为composer use引用所以使用绝对路径)
$defaultConfig = (new \Mpdf\Config\ConfigVariables())->getDefaults();
$fontDirs = $defaultConfig['fontDir'];

// 获取mpdf扩展字体配置文件配置信息
$defaultFontConfig = (new \Mpdf\Config\FontVariables())->getDefaults();
$fontData = $defaultFontConfig['fontdata'];

$mpdf = new \Mpdf\Mpdf([
    'fontDir' => array_merge($fontDirs, [
        __DIR__, // 此处的路径为你ttf文件存放目录
    ]),
    'fontdata' => $fontData + [
            'msyh' => [
                'R' => 'msyh.ttf', // 此处为你从字体网站下载的想要显示的字体
                'I' => 'msyh.ttf',
            ]
        ],
    'default_font' => 'msyh'
]);

$mpdf->SetFont('msyh'); //设置你要使用的字体配置名称

//注意如此设置需要注释掉
//$mpdf->autoScriptToLang = true;
//$mpdf->autoLangToFont = true;
```
### 关于中文符号在数字后面乱码的问题
```html
<!-- 个别字体会出现数字后面的中文字符出现乱码情况,目前没有很好的解决方式,只列举一个简单方式:使用gb字体显示中文字符，例如要显示:【213456789】 -->
<span style:"font-family:自定义字体;">【123456798<span style="font-family: gb;">】</span></span>
```

