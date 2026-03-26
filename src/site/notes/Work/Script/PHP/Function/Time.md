---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Time/","title":"Time","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:25:38.000+08:00","updated":"2026-03-24T17:37:48.368+08:00","dg-note-properties":{"title":"Time","tags":["flashcards"],"reference linking":null}}
---

# hrtime
获取系统的高精度时间**纳秒数**
从任意时间点开始统计，返回系统的高精度时间（**h**igh **r**esolution **time**）。 获取的时间戳为单调时间，无法被用户调整。
### 返回值 [¶](https://www.php.net/manual/zh/function.hrtime.php#refsect1-function.hrtime-returnvalues)
参数 `as_number` 为 false 时，返回的整型数组格式为 `[seconds, nanoseconds]`。否则会以 [int](https://www.php.net/manual/zh/language.types.integer.php) （64 位平台）或 [float](https://www.php.net/manual/zh/language.types.float.php) （32 位平台）返回奈秒（nanoseconds）。失败时返回 false。
```php
echo hrtime(true), PHP_EOL; // 获取当前纳秒数
print_r(hrtime()); // 获取当前时间戳和纳秒数数组[seconds, nanoseconds]
/*
265667675352166
Array
(
    [0] => 265667
    [1] => 675360666
)
*/
```
# microtime
返回当前 Unix 时间戳和**微秒数**
```php
echo microtime() . PHP_EOL; // 0.31301300 1531104655
echo microtime(true); // 1531104655.3131
echo time(); // 1531104655
```
# sleep
程序延迟执行指定的 seconds 的秒数。
```php
sleep(10); // 延迟10秒执行
```
# usleep
以指定的微秒数延缓程序的执行。(1微秒（micro second）是百万分之一秒。)
```php
usleep(2000000);// 延迟2秒执行
```
# date_default_timezone_set
设定用于一个脚本中所有日期时间函数的默认时区
```php
/*一些常用的时区标识符说明:
Asia/Shanghai # 上海GMT+8:00
Asia/Chongqing # 重庆
Asia/Urumqi # 乌鲁木齐
Asia/Hong_Kong # 香港
Asia/Macao # 澳门
Asia/Taipei # 台北
Asia/Singapore # 新加坡
PRC # 北京时间
America/New_York # 美国纽约
*/
ini_set('date.timezone', 'Asia/Shanghai'); // 设置配置文件时区
date_default_timezone_set('Asia/Shanghai');
```
# gmstrftime
程序延迟执行指定的 seconds 的秒数。
>注：gmstrftime()函数与strftime()函数用法大致相同，唯一的不同点是gmstrftime()函数返回的格林威治时间（GMT：Greenwich Mean Time）
```php
// 一个应用场景需要用到倒计时的时分秒，比如新浪微博授权有效期剩余: 7天16小时47分钟42秒……
define("BJTIMESTAMP", time()); //服务器当前时间
$expires_in = '1439577160';//到期时间
$expires = $expires_in - BJTIMESTAMP;
// 一个应用场景需要用到倒计时的时分秒，比如新浪微博授权有效期剩余: 7天16小时47分钟42秒……
function time2second($seconds)
{
    $seconds = (int)$seconds;
    if ($seconds < 86400) {//如果不到一天
        $format_time = gmstrftime('%H时%M分%S秒', $seconds);
    } else {
        $time = explode(' ', gmstrftime('%j %H %M %S', $seconds));//Array ( [0] => 04 [1] => 14 [2] => 14 [3] => 35 )
        $format_time = ($time[0] - 1) . '天' . $time[1] . '时' . $time[2] . '分' . $time[3] . '秒';
    }
    return $format_time;
}

echo "新浪微博授权有效期剩余: " . time2second($expires) . PHP_EOL;


// 注：gmstrftime() 返回的天数是一年中的第几天，因此时间超过一年，请使用下述代码。
function time2second($seconds)
{
    $seconds = (int)$seconds;
    if ($seconds > 3600) {
        if ($seconds > 24 * 3600) {
            $days = (int)($seconds / 86400);
            $days_num = $days . "天";
            $seconds = $seconds % 86400;//取余
        }
        $hours = intval($seconds / 3600);
        $minutes = $seconds % 3600;//取余下秒数
        $time = $days_num . $hours . "小时" . gmstrftime('%M分钟%S秒', $minutes);
    } else {
        $time = gmstrftime('%H小时%M分钟%S秒', $seconds);
    }
    return $time;
}

echo "新浪微博授权有效期剩余: " . time2second($expires) . '<hr>';
```