---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/String/","title":"String","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T11:34:34.000+08:00","updated":"2026-03-24T17:37:51.343+08:00","dg-note-properties":{"title":"String","tags":["flashcards"],"reference linking":null}}
---

# substr_count
统计**子字符串**在字符串中出现**次数**
```php
$text = 'This is a test';
echo strlen($text); // 14

echo substr_count($text, 'is'); // 2

// 字符串被简化为 's is a test'，因此输出 1
echo substr_count($text, 'is', 3);

// 字符串被简化为 's i'，所以输出 0
echo substr_count($text, 'is', 3, 3);

// 因为 5+10 > 14，所以生成警告
echo substr_count($text, 'is', 5, 10);


// 输出 1，因为该函数不计算重叠字符串
$text2 = 'gcdgcdgcd';
echo substr_count($text2, 'gcdgcd');
```
# str_split
如果指定了可选的 split_length 参数(第二个参数)，返回数组中的每个元素均为一个长度为 split_length 的字符块，否则每个字符块为单个字符。
如果 split_length 小于 1，返回 FALSE。如果 split_length 参数超过了 string 超过了字符串 string 的长度，整个字符串将作为数组仅有的一个元素返回。
```php
$str = 'riven';
var_export(str_split($str));
var_export(str_split($str, 2));
/*
array (
    0 => 'r',
    1 => 'i',
    2 => 'v',
    3 => 'e',
    4 => 'n',
)
array (
    0 => 'ri',
    1 => 've',
    2 => 'n',
) 
*/
```
# substr_replace
替换字符串的子字符串
```php
$var = 'ABCDEFGH:/MNRPQR/';
/* 这两个例子使用 "bob" 替换整个 $var。*/
echo substr_replace($var, 'bob', 0) . PHP_EOL;
echo substr_replace($var, 'bob', 0, strlen($var)) . PHP_EOL;

/* 将 "bob" 插入到 $var 的开头处。*/
echo substr_replace($var, 'bob', 0, 0) . PHP_EOL;

/* 下面两个例子使用 "bob" 替换 $var 中的 "MNRPQR"。*/
echo substr_replace($var, 'bob', 10, -1) . PHP_EOL;
echo substr_replace($var, 'bob', -7, -1) . PHP_EOL;

/* 从 $var 中删除 "MNRPQR"。*/
echo substr_replace($var, '', 10, -1) . PHP_EOL;

$input = ['A: XXX', 'B: XXX', 'C: XXX'];

// A simple case: replace XXX in each string with YYY.
echo implode('; ', substr_replace($input, 'YYY', 3, 3)) . PHP_EOL;

// A more complicated case where each replacement is different.
$replace = array('AAA', 'BBB', 'CCC');
echo implode('; ', substr_replace($input, $replace, 3, 3)) . PHP_EOL;

// Replace a different number of characters each time.
$length = array(1, 2, 3);
echo implode('; ', substr_replace($input, $replace, 3, $length)) . PHP_EOL;

/*
bob
bob
bobABCDEFGH:/MNRPQR/
ABCDEFGH:/bob/
ABCDEFGH:/bob/
ABCDEFGH://
A: YYY; B: YYY; C: YYY
A: AAA; B: BBB; C: CCC
A: AAAXX; B: BBBX; C: CCC
*/

// 实例: 隐藏电话号码，敏感数据加密
echo substr_replace('18700000001', '****', 3, 4); // 187****0001

// 隐藏字符
function desensitize($string, $start = 0, $length = 0, $re = '*')
{
    if (empty($string) || empty($length) || empty($re)) {
        return $string;
    }
    $end = $start + $length;
    $strLen = mb_strlen($string);
    $strArr = [];
    for ($i = 0; $i < $strLen; $i++) {
        if ($i >= $start && $i < $end) {
            $strArr[] = $re;
        } else {
            $strArr[] = mb_substr($string, $i, 1);
        }

    }
    return implode('', $strArr);
}

$name = '德玛西亚';
echo desensitize($name, 1, mb_strlen($name)); // 德***
```
# parse_str
将字符串解析成多个变量
```php
$str = "first=value&arr[]=foo+bar&arr[]=baz";
parse_str($str, $output);
print_r($output);
/*
Array
(
    [first] => value
    [arr] => Array
        (
            [0] => foo bar
            [1] => baz
        )

)
*/
```
# uniqid
生成唯一id(此函数努力创建唯一识别符，但它不保证返回值得唯一性)
```php
echo uniqid(); // 5d4bd22d4271c
```
# get_magic_quotes_gpc
获取当前 magic_quotes_gpc 的配置选项设置
```php
// 如果启用了魔术引号
$str = "O\'reilly";
echo addslashes($str) . PHP_EOL; // O\\\'reilly
// 适用各个 PHP 版本的用法
if (get_magic_quotes_gpc()) {
    $lastName = stripslashes($str);
} else {
    $lastName = $str;
}
echo $lastName . PHP_EOL; // O\'reilly
// 如果使用 MySQL
$lastName = mysql_real_escape_string($lastName);
echo $lastName . PHP_EOL; // O\\\'reilly
$sql = "INSERT INTO lastnames (lastname) VALUES ('$lastName')";
echo $sql . PHP_EOL; //
```
# addslashes
使用反斜线引用字符串(在 '、"、\、NUL(NULL 字符) 字符前加上反斜线)
```php
echo addslashes('Shanghai is the "biggest" city in China.');
// Shanghai is the \"biggest\" city in China.
```
# stripslashes
反引用一个引用字符串(去除反斜杠)
```php
// #1 单个字符串
$str = "Is your name O\\'reilly?";
echo stripslashes($str); // Is your name O'reilly?

// #2 对数组递归使用
function stripslashes_deep($value)
{
    $value = is_array($value) ? array_map('stripslashes_deep', $value) : stripslashes($value);
    return $value;
}
// 范例
$array = ["f\\'oo", "b\\'ar", ["fo\\'o", "b\\'ar"]];
$array = stripslashes_deep($array);
// 输出
print_r($array);
/*
Array
(
    [0] => f'oo
    [1] => b'ar
    [2] => Array
        (
            [0] => fo'o
            [1] => b'ar
        )

)
*/
```
# gzcompress
压缩一个字符串(ZLIB压缩格式)
# gzuncompress
解压压缩字符串
```php
$level = 9; // 压缩等级0~9, 缺省值6
$compressed = gzcompress('Compress me', $level); // 压缩字符串
echo gzuncompress($compressed); // 解压字符串
```
# gzdeflate
压缩一个字符串(DEFLATE压缩格式), 压缩效率优于gzcompress()

# gzinflate
解压压缩字符串

```php
$level = 9; // 压缩等级0~9, 缺省值6
$compressed = gzdeflate('Compress me', $level); // 压缩字符串
echo gzinflate($compressed); // 解压字符串
```
# strpbrk
查找字符的首次出现，并返回从该位置到字符串结尾的所有字符
```php
$text = 'This is a Simple text.';
echo strpbrk($text, 'mi'); // is is a Simple text.('i' 先被匹配)
echo strpbrk($text, 'S'); // Simple text.(字符区分大小写)
```
# strtr
strtr()函数的效率优于str_replace()
```php
echo strtr("baab", "ab", "01"), PHP_EOL; // 1001
echo strtr("baab", ["ab" => "01"]); // ba01
```
# strstr
查找字符串的首次出现，并返回从该位置到字符串结尾的所有字符
# stristr
用法与strstr()一致, 不区分大小写
```php
// 返回 'c' 第一次出现之后的字符串部分
echo strstr('efficiency', 'c'), PHP_EOL; // ciency
// 返回 'c'「ASCII值=99」 第一次出现之后的字符串部分
echo strstr('efficiency', 99), PHP_EOL; // ciency「需要web环境」
// 返回 'ency' 第一次出现之前的字符串部分
echo strstr('efficiency', 'ency', true), PHP_EOL; // effici
```
# mbstring
## mb_substr
截取字符串
## mb_strrpos
从后查找字符串
## mb_check_encoding
检查字符串在指定的编码里是否有效
```php
// 返回 '.' 最后一次出现之前的字符串部分
$str = 'a.b.c.d';
echo mb_substr($str, 0, mb_strrpos($str, '.')); // a.b.c


/**  
 * 尝试使用 mb_check_encoding 来检查数据是否可以被解释为 UTF-8 编码。
 * 如果不能，这意味着数据可能包含无法用 UTF-8 表示的二进制数据。  
 * @param $data  
 * @return bool  
 */  
function is_binary($data) {  
    return !mb_check_encoding($data, 'UTF-8');  
}
```
# strrchr
查找字符串的末次出现，并返回从该位置到字符串结尾的所有字符，区分大小写
(方便记忆：如此好人)
```php
echo strrchr('Hello world!', 'o'); // orld!
// 通过 'o' 的ASCII值搜索字符串，并返回字符串的其余部分(7.3.4版本后会抛出异常)
echo strrchr('Hello world!', 111); // orld!

// 获取扩展名
$path = '/www/public_html/index.html';
$filename = substr(strrchr($path, "/"), 1);
echo $filename; // index.html
```
# str_repeat
函数把字符串重复指定的次数
```php
echo str_repeat('.', 5);// .....(重复5次)
```
# ucfirst
将字符串的第一个字符改成大写,返回首字符大写的字符串
```php
$foo = 'hello world!';
echo ucfirst($foo);// Hello world!
```
# lcfirst
将字符串的第一个字符改成小写,返回首字符小写的字符串

```php
$foo = 'Hello World!';
echo lcfirst($foo);// hello World!
```
# ucwords
将字符串的每个单词的首字符变成大写
```php
$foo = 'hello world!';   
echo ucwords($foo);// 输出结果: Hello World!    
```
# preg_split
通过一个正则表达式分隔字符串,返回数组
```php
// 使用逗号或空格(包含" ", \r, \t, \n, \f)分隔短语
$keywords = preg_split(
    '/[\s,]+/',
    'hypertext language, programming'
);
print_r($keywords);
/*
Array
(
    [0] => hypertext
    [1] => language
    [2] => programming
)
*/
```
# number_format
函数通过千位分组来格式化数字（切记不用来计算）
```php
// 注释: 该函数支持一个、两个或四个参数(三个不支持)
echo number_format('1000000'); // 1,000,000
echo number_format('1000000', 2); // 1,000,000.00
echo number_format('1000000', 2, ',', '.');// 1.000.000,00
```