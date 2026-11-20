---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Object/","title":"Object","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:24:52.000+08:00","updated":"2026-03-24T17:37:59.745+08:00","dg-note-properties":{"title":"Object","tags":["flashcards"],"reference linking":null}}
---

# get_object_vars()
返回由对象属性组成的关联数组
# get_class_vars()
返回由类的默认公有属性组成的关联数组
# get_class_methods()
返回由类的方法名组成的数组
```php
class Foo
{
    private   $a = 1;
    protected $b = 2;
    public    $c = 3;
    public    $cc;

    private static   $d = 4;
    protected static $e = 5;
    public static    $f = 6;

    public function __construct()
    {
        $this->c = 33;
    }

    public function getObjectVars()
    {
        var_export(get_object_vars($this));
    }

    public function getClassVars()
    {
        var_export(get_class_vars(self::class));
    }

    public function getClassMethodsPublic()
    {
        var_export(get_class_methods(self::class));
    }

    private function getPrivate()
    {
        echo __FUNCTION__, PHP_EOL;
    }

    protected function getProtected()
    {
        echo __FUNCTION__, PHP_EOL;
    }
}

$test = new Foo;
// +----------------------------------------------------------------------
// | get_object_vars 返回由对象属性组成的关联数组
// +----------------------------------------------------------------------
// #1 先执行构造方法后, 打印 public级别 的属性「静态属性除外」
var_export(get_object_vars($test));
/*
array (
  'c' => 33,
  'cc' => NULL,
)
*/
// #2 先执行构造方法后，打印 所有级别 的属性「静态属性除外」
$test->getObjectVars();
/*
array (
  'a' => 1,
  'b' => 2,
  'c' => 33,
  'cc' => NULL,
)
*/

// +----------------------------------------------------------------------
// | get_class_vars 返回由类的默认属性组成的数组
// +----------------------------------------------------------------------
// #1 不执行构造方法, 打印 public级别 的属性「包括静态属性」
var_export(get_class_vars('Foo'));
/*
array (
  'c' => 3,
  'cc' => NULL,
  'f' => 6,
)
*/
// #2 不执行构造方法, 打印所有属性
$test->getClassVars();
/*
array (
  'a' => 1,
  'b' => 2,
  'c' => 3,
  'cc' => NULL,
  'd' => 4,
  'e' => 5,
  'f' => 6,
)
*/

// +----------------------------------------------------------------------
// | get_class_methods 返回由类的方法名组成的数组
// +----------------------------------------------------------------------
// #1 打印该类 public级别方法 的数组
var_export(get_class_methods('Foo'));
/*
array (
  0 => '__construct',
  1 => 'getObjectVars',
  2 => 'getClassVars',
  3 => 'getClassMethodsPublic',
)
*/
// #2 打印该类 所有级别方法 的数组
$test->getClassMethodsPublic();
/*
array (
  0 => '__construct',
  1 => 'getObjectVars',
  2 => 'getClassVars',
  3 => 'getClassMethodsPublic',
  4 => 'getPrivate',
  5 => 'getProtected',
)
*/
```
# trait_exists()
检查指定的 trait 是否存在
# interface_exists()
检查接口是否已被存在
# class_exists()
检查类是否已经存在
```php
interface DemoInterface {};
trait DemoTrait {};
class DemoClass {};
var_dump(class_exists('DemoInterface')); // false
var_dump(class_exists('DemoTrait')); // false
var_dump(class_exists('DemoClass')); // true
var_dump(trait_exists('DemoTrait')); // true
var_dump(interface_exists('DemoInterface')); // true
```
# method_exists()
检查类的方法是否存在
```php
$directory = new Directory('.');
var_dump(method_exists($directory,'read')); // true
```
# property_exists()
检查对象或类是否具有该属性
```php
class myClass {
    public $mine;
    private $xpto;
    static protected $test;

    static function test() {
        var_dump(property_exists('myClass', 'xpto')); // true
    }
}

var_dump(property_exists('myClass', 'mine'));   // true
var_dump(property_exists(new myClass, 'mine')); // true
var_dump(property_exists('myClass', 'xpto'));   // true
var_dump(property_exists('myClass', 'bar'));    // false
var_dump(property_exists('myClass', 'test'));   // true
myClass::test();
```