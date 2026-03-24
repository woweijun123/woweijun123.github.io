---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Math/","title":"Math","tags":["flashcards"],"noteIcon":"","created":"2023-12-03T01:11:44.000+08:00","updated":"2026-03-24T17:38:02.625+08:00"}
---

# 计算相关
## 高精度计算问题
```php
var_dump(intval(0.58 * 100)); // int(57)
var_dump((0.1 + 0.7) == 0.8); // false
```
说明：`如果用php的+-*/计算浮点数的时候，可能会遇到一些计算结果错误的问题，比如上面 的 echo intval( 0.58*100 );会打印57，而不是58，这个其实是计算机底层二进制无法精确表示浮点数的一个bug，是跨语言的，我用python也遇到这个问题。所以基本上大部分语言都提供了精准计算的类库或函数库，比如php有BC高精确度函数库。`
```php
// 配置默认小数点位数，相当于就是Linux bc中的”scale=”
bcscale(2);

// 高精度加减乘除
var_dump(bcadd(0.58, 0.01, 2)); // string(4) "0.59"
var_dump(bcsub(0.58, 0.01, 2)); // string(4) "0.57"
var_dump(bcmul(0.58, 100, 2)); // string(5) "58.00"
var_dump(bcdiv(58, 100, 2)); // string(4) "0.58"

// 比较两个高精度数字，返回-1, 0, 1
var_dump(bccomp('1', '2')); // int(-1)
// 求高精度数字余数
var_dump(bcmod('2', '4')); // string(1) "2"
// 求高精度数字乘方
var_dump(bcpow('4.2', '3', 2)); // string(5) "74.08"
// 求高精度数字乘方求模，数论里非常常用
var_dump(bcpowmod('2', '3', 16)); // string(1) "8"
// 求高精度数字平方根
var_dump(bcsqrt('4', 2)); // string(4) "2.00"
```
## pwo
返回参数的几次方(方便记忆：怕我)
```php
echo pow(2, 3); // 8
```
## sqrt
返回参数的平方根(方便记忆：是抢人头)
```php
echo sqrt(9); // 3
```
## intdiv
用来进行整数的除法运算。(PHP7)
```php
var_dump(intdiv(10, 3)); // 3
```

# 进制转换
## decbin
十进制转二进制、bindev // 二进制转十进制
```php
echo decbin(10); // 1010
echo bindec(1010); // 10
```
## 十进制转62进制以内的任意进制
自增主键,看起来像是随机字母+数字
>慎用：15位数字以上会有问题
```php
// 十进制转62进制以内的任意进制「最高支持62位」  
function systemConvert($int, $format = 58): string  
{  
    $dic = [  
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',  
        'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j',  
        'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't',  
        'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D',  
        'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N',  
        'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X',  
        'Y', 'Z'  
    ];  
    $arr = [];  
    $loop = true;  
    while ($loop) {  
        $int = scToNum($int);  
        $arr[] = $dic[bcmod($int, $format)];  
        $int   = floor(bcdiv($int, $format));  
        if ($int == 0) {  
            $loop = false;  
        }  
    }  
    return implode('', array_reverse($arr));  
}  
  
function scToNum($num, $double = 0){  
    if(false !== stripos($num, "e")){  
        $a = explode("e",strtolower($num));  
        return bcmul($a[0], bcpow(10, $a[1], $double), $double);  
    }else{  
        return $num;  
    }  
}  
echo systemConvert('999999999999999999');
```