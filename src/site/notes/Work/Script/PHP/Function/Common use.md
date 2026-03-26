---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Common use/","title":"Common use","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:23:33.000+08:00","updated":"2026-03-24T17:38:11.160+08:00","dg-note-properties":{"title":"Common use","tags":["flashcards"],"reference linking":null}}
---

# get_parent_class
返回对象或类的父类名
```php
class Father {}
class Child extends Father
{
    public function __construct()
    {
        echo get_parent_class($this);
    }
}

new Child(); // Father
// 无父类则返回false
var_export(get_parent_class('Father')); // false
```
# phpinfo
查看PHP版本和扩展信息
```php
echo phpinfo();
```
# extension_loaded
检查一个扩展是否已经加载
```php
if (!extension_loaded('redis')) { // 判断是否有redis扩展
    throw new \BadFunctionCallException('not support: redis');
}
```
# memory_get_usage
返回分配给php的内存量
```php
echo memory_get_usage(), PHP_EOL; // 119184
$var = str_repeat("liuhui", 10000);
echo memory_get_usage(), PHP_EOL; // 179320
unset($var); 
echo memory_get_usage(), PHP_EOL; // 119224
```
# getenv
取得系统的环境变量(不支持IIS的isapi方式运行的PHP)
```php
getenv('REMOTE_ADDR'); // 172.19.0.1
// 同样可以获取到客户端的IP地址, 支持IIS的isapi方式运行的PHP
$_SERVER['REMOTE_ADDR']; // 172.19.0.1
```

