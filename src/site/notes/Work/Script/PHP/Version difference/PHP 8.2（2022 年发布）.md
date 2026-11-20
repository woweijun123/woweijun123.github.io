---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Version difference/PHP 8.2（2022 年发布）/","title":"PHP 8.2（2022 年发布）","tags":["flashcards"],"noteIcon":"","created":"2025-05-12T16:50:06.337+08:00","updated":"2026-03-24T17:51:06.233+08:00","dg-note-properties":{"title":"PHP 8.2（2022 年发布）","tags":["flashcards"],"reference linking":null}}
---

# 只读类（Readonly Class）  
类的所有属性默认为只读。
```php
readonly class Point {
   public function __construct(
	   public int $x,
	   public int $y
   ) {}
}
$point = new Point(1, 2);
// $point->x = 3; // 报错：Cannot modify readonly property
```
# 独立类型支持  
`true`、`false`、`null` 作为独立类型。
```php
function isTrue(mixed $value): true {
   return $value === true;
}
```
# 析取范式类型（DNF Types）  
支持复杂类型组合（如 `(A|B) & (C|D)`）。
```php
function process((A|B) & (C|D) $value) {
   // 处理满足 A 或 B，同时满足 C 或 D 的值
}
```
# MySQLi 新 API  
新的 `mysqli_execute_query` 函数和 `mysqli::execute_query` 方法:
```php
$mysqli = new mysqli("localhost", "user", "password", "database");

if ($mysqli->connect_errno) {
    die("Connect failed: " . $mysqli->connect_error);
}

$query = "SELECT * FROM users WHERE id = ?";
$result = $mysqli->execute_query($query, [1]);

if ($result) {
    while ($row = $result->fetch_assoc()) {
        var_dump($row);
    }
    $result->free();
} else {
    echo "Error executing query: " . $mysqli->error;
}

$mysqli->close();
```
# 在常量表达式中获取枚举属性
```php
enum Status: string {
    case Active = 'active';
    case Inactive = 'inactive';

    public const ACTIVE_VALUE = self::Active->value;
    public const INACTIVE_NAME = self::Inactive->name;
}

echo Status::ACTIVE_VALUE . "\n"; // Output: active
echo Status::INACTIVE_NAME . "\n"; // Output: Inactive
```
# Traits 中的常量
```php
trait MyTrait
{
    public const VERSION = '1.0';

    public function getVersion(): string
    {
        return self::VERSION;
    }
}

class MyClass
{
    use MyTrait;
}

$obj = new MyClass();
echo $obj->getVersion() . "\n"; // Output: 1.0
echo MyClass::VERSION . "\n";   // Output: 1.0
```