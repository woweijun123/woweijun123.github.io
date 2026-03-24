---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Performance improvement/","title":"Performance improvement","tags":["flashcards"],"noteIcon":"","created":"2026-02-05T17:26:11.847+08:00","updated":"2026-03-24T17:47:41.363+08:00"}
---

## 字符串优化 (Strings)
### 1. 优先使用内置函数
* **效率排序**：`strtr()` > `str_replace()` > `preg_replace()`。
* **避免正则**：只要能用字符串函数解决，绝不用正则表达式。
* **推荐工具**：`strpbrk()`, `strncasecmp()`, `strpos()`, `stripos()`, `strrpos()`, `strripos()`。
### 2. 字符替换策略
* **按需替换**：使用 `strpos` 先查找。如果无需替换，此举可提升约 200% 的效率。
* **strtr 用法**：
```php
strtr($str, 'abc', '123'); // ✅ 效率高
strtr($str, ['a' => '1']); // ❌ 效率相对较低
```
### 3. 输出与引号
* **引号选择**：
	* **单引号**：静态字符串首选，不解析变量。
	* **双引号**：会触发变量解析。虽然现代 PHP 引擎已极大优化，但大量连接时 `.` 拼接仍是更清晰的选择。
* **Echo 技巧**：
	* `echo` 优于 `print`（无返回值）。
	* **PHP 7+ 变化**：在 PHP 7 以后，`echo $a . $b` 的效率已经优于 `echo $a, $b`，因为 `.` 连接符在底层进行了优化。
## 语句与流程控制 (Statements)
### 1. 避免使用错误抑制符 `@`
* `@` 会显著降低运行速度（约 3 倍），因为它涉及到错误处理状态的频繁切换。
### 2. 循环优化
* **计算外提**：禁止在循环条件中使用函数。
```php
// Bad
for ($i = 0; $i < count($arr); $i++)
// Good
$count = count($arr);
for ($i = 0; $i < $count; $i++)
```
* **Foreach**：在处理数组时，`foreach` 通常比 `for` 或 `while` 更快且更易读。
### 3. 分支选择
* **Switch/Match**：
	* `switch` 优于多层 `if-else`。
	* **[补充] PHP 8+ 特性**：优先使用 `match` 表达式，它具有严格比较和返回值特性，且性能略优于 `switch`。
### 4. 代码健壮性
* **不加结束标记**：纯 PHP 文件不建议加 `?>`，防止末尾意外空格导致 `Headers already sent` 错误。
## 函数与类 (Functions & Classes)
### 1. 路径与包含
* **绝对路径**：使用 `include __DIR__ . '/file.php'`。相对路径会导致 PHP 在 `include_path` 中遍历查找。
* **Once 问题**：`require_once` 和 `include_once` 需检查哈希表以确定文件是否已加载，开销略大。在成熟框架（如 Laravel/Symfony）中通常依靠 Autoload 解决，手动开发时应注意。
### 2. 方法与内存
* **静态方法**：若方法无需访问 `$this`，声明为 `static`。
* **方法拆分**：过度碎片化的 OOP 封装会带来额外的内存开销，需在“可维护性”与“性能”间寻找平衡。
* **Getter/Setter**：直接访问公共属性比通过魔术方法 `__get` 快得多。
### 3. 文件读取
* 优先使用 `file_get_contents()`，它利用了内存映射技术（Memory Mapping）。
## 变量与内存 (Variables)
### 1. 局部变量 vs 全局变量
* **局部变量最快**：存储在栈中，命中缓存几率高。
* **避免全局变量**：`global $var` 查找速度慢，且破坏程序结构。
### 2. 递增运算
* `++$i` 优于 `$i++`。前者只需 3 个 opcode，后者需 4 个（涉及临时变量产生）。
### 3. 及时回收
* 对于大型数组或对象，手动 `unset()` 可以释放内存，尤其是长耗时的脚本（如 CLI 模式）。
* **`$_SERVER['REQUEST_TIME']`**：获取请求开始时间，优于 `time()`。
### 4. 引用参数
* 对于大数据结构，使用 `&` 引用传递可以减少内存复制。但在 PHP 7+ 中，由于 **COW (Copy On Write)** 机制，普通传递也很快，除非你确实需要修改原值。
## 架构级优化 (Architecture)
### 1. 引擎升级
* **PHP 8.x**：引入了 **JIT (Just-In-Time)** 编译器，对于计算密集型任务性能提升巨大。
### 2. 缓存机制
* **OPcache**：必须开启。它将预编译的字节码存入内存，跳过词法和语法分析。
* **数据缓存**：使用 Redis 或 Memcached 存储数据库查询结果或 Session。
### 3. 静态化与压缩
* **Gzip/Brotli**：开启 `zlib.output_compression`。
* **静态化**：对于高频访问且不常变动的页面，直接生成 `.html` 或使用 Nginx 级别的缓存。
## 性能评估工具
* **Xdebug**：代码追踪与 Profile。
* **Spg**：轻量级性能分析。
* **Blackfire.io**：专业的 SaaS 级性能调优工具。
# 实例
```php
$it = 1000000; // 标准迭代次数
header('Content-Type: text/plain; charset=utf-8');

echo "PHP " . PHP_VERSION . " 综合性能基准测试 (迭代: $it)\n";
echo str_repeat('=', 75) . "\n";

/**
 * 1. 长度检查：isset() vs strlen()
 * 佐证：PHP 8+ 中 strlen 是 Specialized Opcode，性能已追平甚至超越 isset
 */
$str = 'hello_world_gemini';
$s   = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $null = strlen($str) > 5;
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $null = isset($str[5]);
}
$t_good = microtime(true) - $s;
printf("1. 长度检查 | Bad: strlen: %.4fs | Good: isset: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 2. 数组索引：引号 vs 常量查找 (PHP 8 模拟)
 * 佐证：不加引号会触发常量查找表，PHP 8+ 找不到常量会抛出 Fatal Error
 */
$row = ['id' => 123];
define('FAKE_ID', 'id');
$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $val = $row[FAKE_ID];
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $val = $row['id'];
}
$t_good = microtime(true) - $s;
printf("2. 数组引号 | Bad: 常量键: %.4fs | Good: 字符键: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 3. 存储位置：数组寻址 vs 局部变量
 * 佐证：局部变量存储在 CV 表，通过偏移量访问；数组需哈希计算 (Hash Lookup)
 */
$testArray = ['id' => 123];
$myId      = 123;
$s         = microtime(true);
for ($i = 0; $i < $it; $i++) {
    if ($testArray['id'] === 123) {
    }
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    if ($myId === 123) {
    }
}
$t_good = microtime(true) - $s;
printf("3. 存储寻址 | Bad: 数组寻址: %.4fs | Good: 局部变量: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 4. 空值判断：is_null() vs === null
 * 佐证：=== 是操作符，is_null 是内置函数。虽然 PHP 8 优化了后者，但操作符仍有微弱优势
 */
$var = null;
$s   = microtime(true);
for ($i = 0; $i < $it; $i++) {
    if (is_null($var)) {
    }
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    if ($var === null) {
    }
}
$t_good = microtime(true) - $s;
printf("4. 空值判断 | Bad: is_null:  %.4fs | Good: === null: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 5. 变量作用域：对象属性 vs 局部变量
 */
class Scope
{
    public $p = 0;
}

$obj   = new Scope();
$local = 0;
$s     = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $obj->p++;
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $local++;
}
$t_good = microtime(true) - $s;
printf("5. 变量存取 | Bad: 对象属性: %.4fs | Good: 局部变量: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 6. 集合处理 (公平竞赛与场景辨析)
 * 佐证：foreach赢在避开了闭包调用(Callback)开销；引用赢在避免内存申请
 */
$data = range(1, 1000);

// 6a. 原位修改：理论上的性能巅峰 (避开内存申请与函数调用)
$s = microtime(true);
for ($i = 0; $i < 1000; $i++) {
    foreach ($data as &$v) {
        $v *= 2;
    }
    unset($v);
}
$t_foreach_ref = microtime(true) - $s;
printf("6a. 原位修改 | foreach&: %.4fs (内存复用 + 无函数压栈)\n", $t_foreach_ref);

// 6b. 生成新数组：公平竞赛 (对比函数调用开销)
$s = microtime(true);
for ($i = 0; $i < 1000; $i++) {
    $res = array_map(fn($v) => $v * 2, $data);
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < 1000; $i++) {
    $res = [];
    foreach ($data as $v) {
        $res[] = $v * 2;
    }
}
$t_good = microtime(true) - $s;
printf("6b. 生成数组 | Bad: array_map:    %.4fs | Good: foreach: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 7. 魔术方法：__get vs 直接访问
 * 佐证：__get 涉及方法查找与动态调用，属于慢速路径 (Slow Path)
 */
class Magic
{
    private $d = ['b' => 1];
    public  $a = 1;

    public function __get($n)
    {
        return $this->d[$n] ?? null;
    }
}

$mg = new Magic();
$s  = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $val = $mg->b;
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $val = $mg->a;
}
$t_good = microtime(true) - $s;
printf("7. 魔术方法 | Bad: __get:    %.4fs | Good: 直接访问: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

/**
 * 8. 错误抑制符：@ 符号代价
 * 佐证：即使不报错，修改错误报告级别也会产生额外的系统调用开销
 */
$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $x = @(1 / 1);
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    $x = 1 / 1;
}
$t_good = microtime(true) - $s;
printf("8. 抑制符号 | Bad: 使用@:   %.4fs | Good: 不使用@: %.4fs | 差距: %.1f倍\n", $t_bad, $t_good, $t_bad / $t_good);

/**
 * 9. 字符替换：盲目替换 vs 先查再换
 * 佐证：避免不必要的写操作 (Write Operation) 和内存申请
 */
$haystack = "Looking for a needle in this haystack";
$needle   = "nonexistent";
$s        = microtime(true);
for ($i = 0; $i < $it; $i++) {
    str_replace($needle, "xyz", $haystack);
}
$t_bad = microtime(true) - $s;

$s = microtime(true);
for ($i = 0; $i < $it; $i++) {
    if (strpos($haystack, $needle) !== false) {
        str_replace($needle, "xyz", $haystack);
    }
}
$t_good = microtime(true) - $s;
printf("9. 字符替换 | Bad: 直接替换: %.4fs | Good: 先查再换: %.4fs | 提升: %.1f%%\n", $t_bad, $t_good, (($t_bad - $t_good) / $t_bad) * 100);

echo str_repeat('=', 75) . "\n测试结束。\n";
```
输出
```
PHP 8.3.21 综合性能基准测试 (迭代: 1000000)
======================================================================
1. 长度检查 | Bad: strlen: 0.0195s | Good: isset: 0.0206s | 提升: -5.8%
2. 数组引号 | Bad: 常量键: 0.0240s | Good: 字符键: 0.0218s | 提升: 9.3%
3. 存储寻址 | Bad: 数组寻址: 0.0200s | Good: 局部变量: 0.0149s | 提升: 25.5%
4. 空值判断 | Bad: is_null:  0.0140s | Good: === null: 0.0140s | 提升: -0.3%
5. 变量存取 | Bad: 对象属性: 0.0168s | Good: 局部变量: 0.0135s | 提升: 19.6%
6. 集合处理 | array_map: 0.1015s | foreach&: 0.0114s | 提升: 88.8%
7. 魔术方法 | Bad: __get:    0.0783s | Good: 直接访问: 0.0178s | 提升: 77.2%
8. 抑制符号 | Bad: 使用@:   0.0237s | Good: 不使用@: 0.0166s | 差距: 1.4倍
9. 字符替换 | Bad: 直接替换: 0.0595s | Good: 先查再换: 0.0551s | 提升: 7.3%
======================================================================
测试结束。
```