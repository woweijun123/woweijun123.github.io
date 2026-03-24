---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Version difference/PHP 8.3（2023 年发布）/","title":"PHP 8.3（2023 年发布）","tags":["flashcards"],"noteIcon":"","created":"2025-05-12T16:50:04.848+08:00","updated":"2026-03-24T17:51:08.205+08:00"}
---

## 1. 类型化类常量（Typed Class Constants）
### 特性：
- 允许为类常量显式指定类型，增强类型安全性和代码清晰度。
- 防止实现类或子类定义与接口或父类不兼容的常量类型。
### 示例：
```php
// PHP < 8.3（未强制类型）
interface I {
    const PHP = 'PHP 8.2'; // 默认类型为 string
}
class Foo implements I {
    const PHP = []; // 允许定义为数组
}

// PHP 8.3（强制类型）
interface I {
    const string PHP = 'PHP 8.3'; // 显式声明为字符串类型
}
class Foo implements I {
    const string PHP = []; // 报错：无法用数组作为字符串类型的值
}
```
## 2. 新增 `json_validate()` 函数
### 特性：
- **无需解码**即可验证 JSON 字符串的合法性，减少内存占用。
- 支持自定义最大深度和验证标志（如 `JSON_INVALID_UTF8_IGNORE`）。
### 示例：
```php
$json = '{ "example": "title" }';
if (json_validate($json)) {
    echo "Valid JSON"; // 输出: Valid JSON
} else {
    echo "Invalid JSON";
}
```
## 3. Randomizer 类扩展
### 特性：
- 新增生成随机数据的方法，支持更灵活的随机数生成。
- 提供 `getBytesFromString()`、`getFloat()`、`nextFloat()` 等方法。
### 示例：
```php
use Random\Randomizer;
$randomizer = new Randomizer();
$float = $randomizer->getFloat(0.0, 100.0); // 生成 0.0 到 100.0 之间的浮点数
$bytes = $randomizer->getBytesFromString("seed"); // 从种子生成随机字节
```
## 4. 新增 `str_increment()` 和 `str_decrement()` 函数
### 特性：
- 对字母数字字符串执行增减操作（类似 `++` 和 `--`）。
- **限制**：仅支持字母数字字符串，否则抛出 `ValueError`。
### 示例：
```php
$str = "1";
$str = str_increment($str); // 输出: "2"
$str = str_decrement($str); // 输出: "0"

// 报错示例
$str = "A";
$str = str_decrement($str); // 抛出 ValueError: "A" 无法减
```
## 5. `#[\Override]` 属性
### 特性：
- 显式标记覆盖父类或接口的方法，避免因拼写错误导致的“假覆盖”。
- 编译时验证是否存在同名方法，提高代码可维护性。
### 示例：
```php
use PHPUnit\Framework\TestCase;

final class MyTest extends TestCase {
    protected $logFile;

    protected function setUp(): void {
        $this->logFile = fopen('/tmp/logfile', 'w');
    }

    #[\Override]
    protected function taerDown(): void { // 拼写错误（正确应为 tearDown）
        fclose($this->logFile);
        unlink('/tmp/logfile');
    }
}
// 报错：MyTest::taerDown() 有 #[\Override]，但父类中无此方法
```
## 6. 动态获取类常量
### 特性：
- 支持通过变量动态访问类常量，语法更简洁。
- 替代旧版 `constant(Foo::class . "::{$name}")` 的冗余写法。
### 示例：
```php
class Foo {
    const PHP = 'PHP 8.3';
}
$searchableConstant = 'PHP';
echo Foo::{$searchableConstant}; // 输出: PHP 8.3
```
## 7. 只读属性的深拷贝支持
### 特性：
- `readonly` 属性可在 `__clone()` 方法中被修改一次，实现深拷贝。
- 适用于需要克隆只读对象但需修改内部状态的场景。
### 示例：
```php
readonly class Foo {
    public function __construct(
        public PHP $php
    ) {}
}

$original = new Foo(new PHP('8.3'));
$cloned = clone $original;
$cloned->php->version = '8.4'; // 允许在 __clone 中修改 readonly 属性
```
## 8. 其他改进
### (1) 改进 `unserialize()` 错误处理
- 统一反序列化错误行为，避免因输入格式问题导致不可预测的异常。
### (2) 负数索引支持
- 数组支持负数索引（如 `$arr[-1]`），简化末尾元素访问。
### (3) 匿名只读类
- 支持创建匿名只读类，用于临时不可变对象。