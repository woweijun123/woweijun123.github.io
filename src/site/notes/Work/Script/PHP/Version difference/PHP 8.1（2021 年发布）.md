---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Version difference/PHP 8.1（2021 年发布）/","title":"PHP 8.1（2021 年发布）","tags":["flashcards"],"noteIcon":"","created":"2025-05-12T16:50:03.200+08:00","updated":"2026-03-24T17:51:04.857+08:00","dg-note-properties":{"title":"PHP 8.1（2021 年发布）","tags":["flashcards"],"reference linking":null}}
---

# 1. 语法改进
- **构造函数属性提升**  
简化类属性的初始化，直接在构造函数中声明属性，减少冗余代码。  
```php
class User {
  public function __construct(
	  public string $name,
	  public int $age
  ) {}
}
```
- **命名参数**  
调用函数时通过参数名指定值，避免参数顺序错误，提高可读性。  
```php
function createProduct(string $name, float $price) {}
createProduct(price: 9.99, name: "Book");
```
- **字符串键数组解包**  
支持使用 `...` 合并包含字符串键的数组，语义与 `array_merge` 一致。  
```php
$array = ["a" => 1, ...["b" => 2]]; // ["a" => 1, "b" => 2]
```
- **只读属性**  
使用 `readonly` 声明属性，确保对象创建后不可变，适用于 DTO（数据传输对象）。  
```php
class Data {
  public function __construct(
	  readonly public string $value
  ) {}
}
```
# 2. 类型系统增强
- **联合类型**  
允许变量接受多种类型，提升类型灵活性。  
```php
function formatInput(string|int $input): string {}
```
- **元组类型**  
定义固定数量和类型的值集合，用于函数参数和返回值。  
```php
function getCoordinates(): [float, float] {
  return [40.71, -74.01];
}
```
- **枚举类型（Enums）**  
支持定义命名常量集合，增强代码可读性和类型安全。  
```php
enum Status: string {
  case Active = 'active';
  case Inactive = 'inactive';
}
```
- **Never 返回类型**  
标记函数永远不会正常返回（如抛出异常），帮助管理代码流程。  
```php
function throwError(): never {
  throw new Exception("Invalid input");
}
```
# 3. 性能优化
- **JIT（即时编译）**  
将 PHP 代码编译为本地机器码，显著提升执行效率，尤其适合计算密集型任务。
- **内存管理优化**  
改进垃圾回收算法，减少内存分配开销，降低服务器负载。
# 4. 新增内置函数
- **`str_contains()`**  
判断字符串是否包含子字符串，替代 `strpos()`，更直观。  
```php
if (str_contains($text, "hello")) { ... }
```
- **`array_is_list()`**  
检查数组是否为连续索引的列表（如 `[0 => "a", 1 => "b"]`）。
- **`fsync()` / `fdatasync()`**  
强制将文件数据同步到磁盘，确保数据持久化。
# 5. 并发与异步
- **Fiber（纤程）**  
轻量级协程，支持在单线程内实现非阻塞任务调度，优化高并发场景。  
```php
$fiber = new Fiber(function () {
  echo Fiber::suspend(); // 暂停并返回值
  echo "Resumed";
});
$fiber->start(); // 启动
$fiber->resume("Hello"); // 恢复执行
```
# 6. 其他改进
- **异常链（Exception Chaining）**  
异常抛出时可携带原始异常上下文，便于调试分布式系统错误。  
```php
throw new Exception("Error", 0, $previousException);
```
- **强制属性初始化**  
要求类属性在构造函数中初始化，避免未定义状态。
- **八进制整数表示法**  
支持 `0o` 或 `0O` 前缀定义八进制数（如 `0o16` 等同于 `14`）。
- **`final` 类常量**  
常量标记为 `final` 后，子类无法覆盖。