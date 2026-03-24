---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Version difference/PHP 8.4（2024 年 11 月发布）/","title":"PHP 8.4（2024 年 11 月发布）","tags":["flashcards"],"noteIcon":"","created":"2025-05-12T16:45:19.558+08:00","updated":"2026-03-24T17:51:09.681+08:00"}
---

## 1. 属性钩子（Property Hooks）
### 特性：
- 允许直接在属性定义中设置 `get` 和 `set` 钩子，替代传统冗余的 `getter/setter` 方法。
- 支持虚拟属性（仅定义 `get` 钩子）。
- 提升代码简洁性和 IDE/静态分析工具的兼容性。
### 示例：
```php
class User {
    private string $firstName;
    private string $lastName;

    public string $fullName {
        get => $this->firstName . ' ' . $this->lastName;
    }

    public string $firstName {
        set => ucfirst(strtolower($value));
    }

    public string $lastName {
        set {
            if (strlen($value) < 2) {
                throw new InvalidArgumentException('Last name too short');
            }
            $this->lastName = $value;
        }
    }

    public function __construct(string $first, string $last) {
        $this->firstName = $first;
        $this->lastName = $last;
    }
}

$user = new User('john', 'doe');
echo $user->firstName; // Output: John
echo $user->fullName;  // Output: John doe
$user->lastName = 'smith';
echo $user->fullName;  // Output: John smith
// $user->lastName = 'a'; // Throws InvalidArgumentException
```
### 优势：
- **简化代码结构**：无需编写额外的 `getter` 和 `setter` 方法。
- **增强可维护性**：逻辑集中在属性定义中，减少分散的业务逻辑。
## 2. 不对称可见性（Asymmetric Visibility）
### 特性：
- 独立控制属性的读写权限（如 `public get` + `private set`）。
- 避免暴露不必要的写入权限，减少冗余的 `setter` 方法。
### 示例：
```php
class Product {
    public readonly int $id;
    public private(set) string $name;

    public function __construct(int $id, string $name) {
        $this->id = $id;
        $this->name = $name;
    }

    public function rename(string $newName): void {
        $this->name = $newName; // Allowed within the class
    }
}

$product = new Product(1, 'Book');
echo $product->id;   // Public read access
echo $product->name; // Public read access
// $product->name = 'New Book'; // Fatal error: Cannot access private property Product::$name
$product->rename('Updated Book'); // Allowed
echo $product->name; // Output: Updated Book
```
### 应用场景：
- **封装性增强**：限制外部修改属性，仅允许内部逻辑更新。
- **安全性提升**：防止外部直接篡改敏感数据。
## 3. DOM 扩展增强与 HTML5 支持
### 特性：
- 新增 `DomHTMLDocument` 类，支持 HTML5 标准解析（如 `<template>` 标签）。
- 修复旧版对 HTML5 标签的处理错误，增强 CSS 选择器支持。
### 示例：
```php
$doc = DomHTMLDocument::createFromString($html);
$elements = $doc->querySelectorAll('.className');
```
### 优势：
- **兼容性提升**：支持现代 HTML5 特性，简化 DOM 操作。
- **代码简化**：直接使用 CSS 选择器查询元素，无需手动解析。
## 4. BCMath 面向对象改进
### 特性：
- 提供更直观的数学运算方法（如 `add()`、`subtract()`）。
- 支持高精度计算，适用于金融或科学计算场景。
### 示例：
```php
use BCMath\BigInteger;
$a = new BigInteger('12345678901234567890');
$b = new BigInteger('98765432109876543210');
$sum = $a->add($b);
```
### 优势：
- **类型安全**：避免字符串拼接导致的计算错误。
- **代码可读性**：面向对象方法替代传统函数调用。
## 5. JIT 引擎优化（基于 IR）
### 特性：
- 采用中间表示（Intermediate Representation, IR）优化 JIT 编译流程。
- **性能提升**：更高效的寄存器分配、循环展开和死代码消除。
### 应用场景：
- **高负载应用**：如实时数据处理、API 服务。
- **资源受限环境**：容器化应用或共享主机，减少内存占用。
## 6. 新增数组函数
### 特性：
- `array_find()` 和 `array_find_key()` 简化数组操作。
- 支持回调函数快速查找匹配元素。
### 示例：
```php
$items = ['apple', 'banana', 'cherry', 'date'];

$firstLongerThan5 = array_find($items, fn($item) => strlen($item) > 5);
var_dump($firstLongerThan5); // Output: string(6) "banana"

$keyOfFirstLongerThan5 = array_find_key($items, fn($item) => strlen($item) > 5);
var_dump($keyOfFirstLongerThan5); // Output: int(1)

$anyLongerThan5 = array_any($items, fn($item) => strlen($item) > 5);
var_dump($anyLongerThan5); // Output: bool(true)

$allLongerThan3 = array_all($items, fn($item) => strlen($item) > 3);
var_dump($allLongerThan3); // Output: bool(true)
```
### 优势：
- **代码简洁**：替代 `foreach` 或 `array_filter` 实现。
- **可读性提升**：直接表达“查找”意图。
## 7. 方法链省略括号
### 特性：
- 允许直接调用新实例化对象的方法时省略括号。
- 简化方法链语法。
### 示例：
```php
// 旧写法
$name = (new ReflectionClass($object))->getShortName();

// PHP 8.4 新写法
$name = new ReflectionClass($object)->getShortName();
```
### 优势：
- **代码更紧凑**：减少冗余括号，提升可读性。
## 8. 惰性对象（Lazy Objects）
### 特性：
- 延迟初始化对象，仅在首次访问或修改时触发构造函数。
- 依赖反射实现，适用于依赖注入框架。
### 示例：
```php
class People {
    public function __construct() {
        echo "初始化方法被调用\n";
    }
}
$people = LazyObject::make(People::class);
// 此时未初始化
echo $people->name; // 此时才初始化
```
### 优势：
- **资源优化**：减少不必要的对象初始化，提升性能。
## 9. `#[\Deprecated]` 属性
### 特性：
- 标记弃用的函数、方法或类常量，提供替代方案提示。
- 支持自定义弃用消息和版本号。
### 示例：
```php
class PhpVersion {
    #[\Deprecated(message: "use PhpVersion::getVersion() instead", since: "8.4")]
    public function getPhpVersion(): string {
        return $this->getVersion();
    }
    public function getVersion(): string {
        return '8.4';
    }
}
```
### 优势：
- **迁移友好**：明确告知开发者弃用原因和替代方案。
- **兼容性保障**：避免因废弃功能导致的兼容性问题。
## 10. 多字节字符串处理增强
### 功能：
新增 `mb_trim()`、`mb_ltrim()`、`mb_rtrim()` 等函数，支持多字节字符的修剪。
### 示例：
```php
// ====================== 多字节字符串处理增强 ======================

// 1. mb_ucfirst：首字母大写
echo "1. mb_ucfirst:\n";
echo mb_ucfirst("hello 世界", "UTF-8") . "\n"; // 输出: Hello 世界
echo mb_ucfirst("こんにちは", "UTF-8") . "\n"; // 日文无变化
echo "--------------------------\n";

// 2. mb_lcfirst：首字母小写
echo "2. mb_lcfirst:\n";
echo mb_lcfirst("HELLO WORLD", "UTF-8") . "\n"; // 输出: hELLO WORLD
echo mb_lcfirst("HELLO 世界", "UTF-8") . "\n"; // 中文无变化
echo "--------------------------\n";

// 3. mb_trim：去除首尾空格（包括全角空格）
echo "3. mb_trim:\n";
echo mb_trim("  Hello World  ") . "\n"; // 输出: Hello World
echo mb_trim("　　你好　世界　　") . "\n"; // 输出: 你好　世界
echo "--------------------------\n";

// 4. mb_ltrim：去除左侧空格
echo "4. mb_ltrim:\n";
echo mb_ltrim("  Hello World  ") . "\n"; // 输出: Hello World  
echo mb_ltrim("　　你好　世界　　") . "\n"; // 输出: 你好　世界　　
echo "--------------------------\n";

// 5. mb_rtrim：去除右侧空格
echo "5. mb_rtrim:\n";
echo mb_rtrim("  Hello World  ") . "\n"; // 输出:   Hello World
echo mb_rtrim("　　你好　世界　　") . "\n"; // 输出:   你好　世界
echo "--------------------------\n";

// 6. mb_substr：截取多字节字符串
echo "6. mb_substr:\n";
echo mb_substr("你好，世界！", 0, 2, "UTF-8") . "\n"; // 输出: 你好
echo mb_substr("你好，世界！", -3, 2, "UTF-8") . "\n"; // 输出: 世界
echo "--------------------------\n";

// 7. mb_str_pad：填充字符串
echo "7. mb_str_pad:\n";
echo mb_str_pad("你好", 10, "·", STR_PAD_LEFT, "UTF-8") . "\n"; // 输出: ········你好
echo mb_str_pad("你好", 10, "·", STR_PAD_RIGHT, "UTF-8") . "\n"; // 输出: 你好········
echo mb_str_pad("你好", 10, "·", STR_PAD_BOTH, "UTF-8") . "\n"; // 输出: ···你好···
echo "--------------------------\n";

// 8. mb_count_chars：统计字符频率
echo "8. mb_count_chars:\n";
print_r(mb_count_chars("你好，你好！", 1, "UTF-8")); // 输出字符频率数组
print_r(mb_count_chars("Hello, World!", 1, "UTF-8")); // ASCII 字符频率
echo "--------------------------\n";

// 9. mb_str_split：分割多字节字符串
echo "9. mb_str_split:\n";
print_r(mb_str_split("你好，世界！", 1, "UTF-8")); // 输出: ["你", "好", "，", "世", "界", "！"]
print_r(mb_str_split("你好，世界！", 2, "UTF-8")); // 输出: ["你好", "，世", "界！"]
echo "--------------------------\n";

// 10. mb_strrev：反转字符串
echo "10. mb_strrev:\n";
echo mb_strrev("你好，世界！", "UTF-8") . "\n"; // 输出: ！界世，好你
echo mb_strrev("Hello, World!", "UTF-8") . "\n"; // 输出: !dlroW ,olleH
echo "--------------------------\n";

// 11. mb_strtolower / mb_strtoupper：转换大小写
echo "11. mb_strtolower / mb_strtoupper:\n";
echo mb_strtolower("HELLO 世界", "UTF-8") . "\n"; // 输出: hello 世界
echo mb_strtoupper("hello 世界", "UTF-8") . "\n"; // 输出: HELLO 世界
echo "--------------------------\n";

// 12. mb_convert_encoding：编码转换
echo "12. mb_convert_encoding:\n";
$utf8Str = "你好，世界！";
$gbkStr = mb_convert_encoding($utf8Str, "GBK", "UTF-8");
echo mb_convert_encoding($gbkStr, "UTF-8", "GBK") . "\n"; // 输出: 你好，世界！
echo "--------------------------\n";

// 13. mb_strlen：计算字符串长度
echo "13. mb_strlen:\n";
echo mb_strlen("你好，世界！", "UTF-8") . "\n"; // 输出: 6
echo mb_strlen("Hello, World!", "UTF-8") . "\n"; // 输出: 13
echo "--------------------------\n";

// 14. mb_stripos / mb_strrpos：查找子串位置
echo "14. mb_stripos / mb_strrpos:\n";
echo mb_stripos("你好，世界！", "世", 0, "UTF-8") . "\n"; // 输出: 4
echo mb_strrpos("你好，世界！", "界", 0, "UTF-8") . "\n"; // 输出: 5
echo "--------------------------\n";

// 15. mb_str_replace：替换子串
echo "15. mb_str_replace:\n";
echo mb_str_replace("你好", "Hello", "你好，世界！", "UTF-8") . "\n"; // 输出: Hello，世界！
echo mb_str_replace("World", "PHP", "Hello, World!", "UTF-8") . "\n"; // 输出: Hello, PHP!
echo "--------------------------\n";

// 16. mb_str_ireplace：不区分大小写的替换
echo "16. mb_str_ireplace:\n";
echo mb_str_ireplace("hello", "Hi", "Hello, World!", "UTF-8") . "\n"; // 输出: Hi, World!
echo "--------------------------\n";

// 17. mb_str_word_count：统计单词数
echo "17. mb_str_word_count:\n";
echo mb_str_word_count("Hello, World!", 0, "UTF-8") . "\n"; // 输出: 2
echo mb_str_word_count("你好，世界！", 1, "UTF-8") . "\n"; // 输出: 6
echo "--------------------------\n";

// 18. mb_check_encoding：检查编码
echo "18. mb_check_encoding:\n";
var_dump(mb_check_encoding("你好", "UTF-8")); // 输出: true
var_dump(mb_check_encoding("你好", "GBK")); // 输出: false
echo "--------------------------\n";

// 19. mb_encode_numericentity / mb_decode_numericentity：HTML 实体转换
echo "19. mb_encode_numericentity / mb_decode_numericentity:\n";
echo mb_encode_numericentity("你好", [0x80, 0x9f], "UTF-8") . "\n"; // 输出: &#x4f60;&#x597d;
echo mb_decode_numericentity("&#x4f60;&#x597d;", [0x80, 0x9f], "UTF-8") . "\n"; // 输出: 你好
echo "--------------------------\n";

// 20. mb_convert_case：转换大小写格式
echo "20. mb_convert_case:\n";
echo mb_convert_case("Hello World", MB_CASE_LOWER, "UTF-8") . "\n"; // 输出: hello world
echo mb_convert_case("hello world", MB_CASE_TITLE, "UTF-8") . "\n"; // 输出: Hello World
echo "--------------------------\n";
```