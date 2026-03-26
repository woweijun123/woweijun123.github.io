---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Path/","title":"Path","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T11:34:34.000+08:00","updated":"2026-03-24T17:37:57.008+08:00","dg-note-properties":{"title":"Path","tags":["flashcards"],"reference linking":null}}
---

# debug_backtrace()
产生一条回溯跟踪(backtrace)
使用场景:
1. 多个方法复用一个方法时，判断上级方法名称进行不同逻辑的切换
```php
class A
{
    public static function functionA()
    {
        self::functionB();
    }
    public static function functionB()
    {
        var_export(debug_backtrace());
    }
}
A::functionA();
/*
array(
    0 => array(
        'file' => '/opt/project/php/test/t1.php',
        'line' => 7,
        'function' => 'functionB',
        'class' => 'A',
        'type' => '::',
        'args' =>
            array(),
    ),
    1 => array(
        'file' => '/opt/project/php/test/t1.php',
        'line' => 16,
        'function' => 'functionA',
        'class' => 'A',
        'type' => '::',
        'args' =>
            array(),
    )
);
*/
```
查询上一个调用方法
```php
/**  
 * 方法来源  
 * @Author Wcj  
 * @email 1054487195@qq.com * @DateTime 2020/8/27 15:19 * @return mixed|string  
 */  
public static function from()  
{  
    return debug_backtrace()['2']['function'] ?? '';  
}
```
# parse_url()
解析 URL，返回其组成部分
```php
$url = "http://www.sina.com.cn/abc/de/fg.php?id=1";
var_export(parse_url($url));
/*
array (
  'scheme' => 'http',
  'host' => 'www.sina.com.cn',
  'path' => '/abc/de/fg.php',
  'query' => 'id=1',
)
*/
```
# pathinfo() 
返回文件路径的信息
```php
var_export(pathinfo('/abc/de/fg.php'));
/*
array (
  'dirname' => '/abc/de',
  'basename' => 'fg.php',
  'extension' => 'php',
  'filename' => 'fg',
)
 */
```
# dirname()
返回路径中的目录名称部分
```php
echo dirname("c:/testweb/home.php"); // c:/testweb
```
# basename()
返回路径中的文件名部分
```php
echo basename("c:/testweb/home.php"); // home.php
```
# getcwd()
获取当前工作的目录名称部分(绝对路径)
```php
echo getcwd(); // C:\Users\Administrator\Desktop\test
```
# realpath()
函数返回包含目录部分和文件名的绝对路径。该函数删除所有符号连接（比如 '/./', '/../' 以及多余的 '/'），并返回绝对路径名。 如果失败，该函数返回 FALSE
```php
echo realpath('test.php');
// C:\Users\1\Desktop\test\test.php
echo realpath('C:/Users/1/Desktop/../Desktop/test/test.php');
// C:\Users\1\Desktop\test\test.php
```
# glob()
寻找与模式匹配的文件路径
```php
foreach (glob("*.txt") as $filename) {
    echo "{$filename} size " . filesize($filename) . PHP_EOL;
}
/*
funclist.txt size 44686
funcsummary.txt size 267625
quickref.txt size 137820
*/
```