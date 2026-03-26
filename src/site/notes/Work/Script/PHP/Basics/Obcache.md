---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Basics/Obcache/","title":"Obcache","noteIcon":"","created":"2025-12-25T09:39:50.000+08:00","updated":"2026-03-24T17:21:30.896+08:00","dg-note-properties":{"title":"Obcache","tags":null,"reference linking":null}}
---

# 缓冲层

## 1. PHP内部缓冲层（2层）
### 1. PHP 内部缓冲层 (Output Buffering)
PHP 内部其实只有 **OB 缓存** 和 **SAPI 缓存** 两大类。你提到的“用户级”和“系统级”在 PHP 内部是**共享同一个栈结构**的。
#### 1.1 OB 缓冲栈 (Output Buffering Stack)
这是由 PHP 引擎管理的内存区域。
* **系统默认层 (Top Level)**：由 `php.ini` 中的 `output_buffering` 开启。如果设置为 `4096`，PHP 会自动启动一个隐式的 `ob_start()`。
```ini
# php.ini配置
output_buffering = 4096  # 开启4KB缓冲
output_buffering = Off   # 关闭系统级缓冲
```
* **用户嵌套层**：由开发者通过 `ob_start()` 显式开启，支持嵌套、自定义回调函数。
```php
ob_start('uppercase'); // 创建第一层输出缓冲区，设置回调函数
echo "Hello"; // "Hello" 被写入第一层缓冲区
ob_start('uppercase'); // 创建第二层输出缓冲区，设置回调函数
echo "World"; // "World" 被写入第二层缓冲区
$content = ob_get_clean(); // 获取并清空第二层缓冲区内容「不触发回调函数」
file_put_contents('log.txt', $content); // 将第二层内容写入到文件
ob_end_flush(); // 将ob缓存内容刷新到程序缓存，并且关闭ob「会触发回调函数」
// 自定义回调函数示例（将输出转为大写）
function uppercase($content)
{
    return strtoupper($content);
}
```
* **特性**：
	* **栈式管理**：后开启的层级在最底部，优先接收 `echo`。
	* **回调机制**：`ob_end_flush()` 会触发回调；`ob_get_clean()` **不会**触发回调。
	* **报错拦截**：OB 层开启时，`header()` 可以在 `echo` 后执行（只要 OB 没溢出）。
#### 1.2 SAPI 缓冲层 (Server API)
这是 PHP 与外部 Web 服务器（Nginx/Apache）通信的最后一公里。
* **作用**：将 PHP 渲染后的最终字符串打包，交给 Web 服务器。
* **关键函数**：`flush()`。它的作用是强制将 SAPI 缓冲区的内容“推”出去。
* **配置**：`implicit_flush = On` 会导致每次 `echo`（在经过 OB 层后）都自动执行一次 `flush()`。
```ini
# CLI模式默认配置（php.ini）
implicit_flush = On # 自动刷新缓冲
```
### 2. 外部缓冲层 (Web Server)
虽然不在 PHP 内部，但它是导致 `flush()` “失效”的头号元凶。
* **Nginx 层**：`fastcgi_buffers`。Nginx 会再次缓冲 PHP 发来的数据。
* **解决办法**：
	* Nginx 配置：`fastcgi_buffering off;`
	* PHP 代码：发送 `header('X-Accel-Buffering: no');` 告知 Nginx 不要缓冲。
### 3. 修正后的代码示例与注释
```php
// 1. 系统级缓冲：如果 php.ini 开启了 output_buffering，这里已经是 Level 1 了
// 2. 手动开启用户层（Level 2）
ob_start('strtoupper'); 
echo "hello"; // 进入 Level 2 缓冲
// 3. 强制流转路径：
ob_flush(); // [OB层 -> SAPI层] 将 Level 2 内容刷入 SAPI (变为 HELLO)
flush();    // [SAPI层 -> 服务器] 将 SAPI 内容刷给 Nginx/Apache
// 4. 关闭并清理
ob_end_clean(); // 关闭当前 OB 层
```
### 核心区分点表
| 层级名称          | 控制方式                         | 核心目标                    |
| ------------- | ---------------------------- | ----------------------- |
| **OB 缓存 (栈)** | `ob_start` / `ini`           | 逻辑控制（拦截、修改输出、处理 Header） |
| **SAPI 缓存**   | `flush()` / `implicit_flush` | 传输优化（减少系统调用次数）          |
| **Web服务器缓存**  | Nginx 配置 / Header            | 网络优化（提高高并发下的响应吞吐）       |
## 2. 外部环境缓冲层（4层）
### 2.1 Web服务器层缓冲（如Nginx/Apache）
- **作用**：Web服务器自身的缓冲机制（如Nginx的FastCGI缓冲）。
- **配置/示例**：
```nginx
# Nginx禁用FastCGI缓冲
location ~ \.php$ {
  include fastcgi_params;
  fastcgi_pass php-fpm;
  fastcgi_buffering off; # 关闭缓冲
}
```
### 2.2 浏览器缓冲（Browser Buffering）
- **作用**：浏览器本地缓冲，延迟显示内容。
- **示例**：
```php
echo "Loading..."; 
flush(); // 刷新底层缓冲，强制浏览器立即显示
```
### 2.3 中间层缓存（如CDN、代理服务器）
- **作用**：网络中间设备（如CDN）缓存内容以加速访问。
- **示例**：
```nginx
# Cloudflare缓存策略配置（示例）
Cache-Control: max-age=3600
```
### 2.4 网络传输层缓冲（TCP/IP Buffering）
- **作用**：操作系统内核的TCP/IP协议栈缓冲。
- **示例**：
```php
// 强制刷新内核缓冲（需配合其他层的刷新）
flush();
ob_flush();
```
## 3. 缓冲层执行顺序
输出内容会按以下顺序经过各层：
**用户级缓冲** → **程序级缓冲** → **SAPI层缓冲** → **Web服务器缓冲** → **网络传输层缓冲** → **浏览器缓冲**。
## 关键注意事项
### 1. HTTP头限制
所有缓冲层不会缓存HTTP头（如 `header()` 的输出），需在任何输出前调用 `header()`。
### 2. 嵌套缓冲处理
用户级缓冲可嵌套多层，需按“**先进后出**”顺序关闭：
```php
ob_end_flush(); // 先关闭最近的缓冲层
ob_end_flush(); // 再关闭外层缓冲层
```
### 3. 性能优化建议
- **静态页面**：启用系统级缓冲（如 `output_buffering=4096`）。
- **实时应用**：禁用所有缓冲层，并在关键点调用 `flush()`。
## 示例
### 模板代码生成
#### 模板代码
```php
declare (strict_types=1);

class <?= $modelName ?>Struct extends <?= $stBaseName ?>

{
    // 模型字段
<?php foreach ($numberField as $attr) { ?>
    public <?= $attr['COLUMN_TYPE'] ?> $<?= $attr['COLUMN_NAME'] ?> = <?= $attr['COLUMN_DEFAULT'] ?>; // <?= $attr['COLUMN_COMMENT'].PHP_EOL ?>
<?php } ?>
}
```
#### obcache解析并生成代码
```php
$context = [
    "modelName"   => "My",
    "stBaseName"  => "MyStruct",
    "numberField" => [
        "id"   => [
            'COLUMN_TYPE'    => 'int',
            'COLUMN_NAME'    => 'id',
            'COLUMN_DEFAULT' => '0',
            'COLUMN_COMMENT' => 'ID',
        ],
        "name" => [
            'COLUMN_TYPE'    => 'string',
            'COLUMN_NAME'    => 'name',
            'COLUMN_DEFAULT' => '0',
            'COLUMN_COMMENT' => '姓名',
        ],
    ],
];
extract($context);
ob_start();
include_once "struct.php"; // 将文件作为 PHP 代码执行，变量经过 PHP 解析器处理
$res = ob_get_contents();
ob_end_clean();
echo $res;
```
### 过滤敏感词
```php
// 1. 开启外层缓冲 (Level 1)
ob_start();
// 2. 写入外层：此内容不经过回调，不会被过滤
echo "Hello";
// 3. 开启内层缓冲 (Level 2)，并绑定过滤回调
ob_start('sensitize');
// 4. 写入内层：此内容待处理
echo "狗日的";
// 5. 刷新内层：内容先经 sensitize 函数过滤，再合并到外层
ob_end_flush();
// 6. 获取外层全部内容 ("Hello" + "敏感词") 并保存
file_put_contents('log.txt', ob_get_contents());
function sensitize($content)
{
    return str_replace("狗日的", "敏感词", $content);
}
```
### 实时输出（如聊天应用）
```php
// 禁用系统级缓冲（php.ini中设置output_buffering=Off）
ob_implicit_flush(1); // 自动刷新缓冲

echo "Message 1<br>";
flush(); // 刷新所有缓冲层
sleep(1);
echo "Message 2<br>";
flush();
```
### 压缩输出（GZIP压缩）
```php
// 开启压缩缓冲  
if (!ob_start("ob_gzhandler")) {  
    ob_start();  
}  
// 模拟生成大量文本  
echo "<h2>GZIP 压缩效率测试</h2>";  
$data = str_repeat("PHP Output Buffering GZIP Test. ", 500);  
echo "<p>$data</p>";  
// 获取压缩前的字节数  
$originalSize = ob_get_length();  
echo "<p>原始字节数：$originalSize</p>";  
// 刷新缓冲区，发送压缩后的数据  
ob_end_flush();  
// 注意：ob_end_flush 之后无法再获取压缩后的精确体积，  
// 压缩体积通常由浏览器在 Network 面板的 "Size" 栏显示（会显示 16.3 KB -> 400 B 这种对比）。
```
### 替换输出内容
```php
// 自定义回调函数：替换所有"PHP"为"Framework"
function replace_php($content) {
    return str_replace("PHP", "Framework", $content);
}

ob_start("replace_php");
echo "PHP is great!";
ob_end_flush(); // 输出 "Framework is great!"
```
## 总结
PHP的输出缓冲机制涉及**7层**，分为**PHP内部3层**（用户级、系统级、SAPI层）和**外部环境4层**（Web服务器、浏览器、中间层、网络传输层）。开发者需根据场景需求配置各层，平衡性能、实时性和兼容性。例如：
- **实时应用**：禁用所有缓冲层，或在关键点调用 `flush()`。
- **静态页面**：启用系统级和用户级缓冲以减少传输次数。
# Obcache
ob就是output_buffer(输出缓存)的简写。
数据并不是直接从 PHP 代码到达浏览器的，而是经过以下路径： **PHP OB 缓存** -> **程序缓存 (Server/SAPI Cache)** -> **浏览器缓存**
## 函数
- [ob_start](https://www.php.net/manual/zh/function.ob-start.php) — 打开输出控制缓冲
- [flush](https://www.php.net/manual/zh/function.flush.php) — 刷新缓冲区的内容
- [ob_clean](https://www.php.net/manual/zh/function.ob-clean.php) — 清空（擦掉）活动输出缓冲区的内容「不关闭ob」
- [ob_end_clean](https://www.php.net/manual/zh/function.ob-end-clean.php) — 清空（擦除）活动缓冲区的内容并关闭ob「不会触发回调函数」
- [ob_end_flush](https://www.php.net/manual/zh/function.ob-end-flush.php) — 将ob缓存内容刷新到程序缓存，并且关闭ob「会触发回调函数」
- [ob_flush](https://www.php.net/manual/zh/function.ob-flush.php) — ob缓存内容刷到程序缓存「不关闭ob」
- [ob_get_clean](https://www.php.net/manual/zh/function.ob-get-clean.php) — 获取活动缓冲区的内容并将其关闭
- [ob_get_contents](https://www.php.net/manual/zh/function.ob-get-contents.php) — 返回输出缓冲区的内容
- [ob_get_flush](https://www.php.net/manual/zh/function.ob-get-flush.php) — 将ob缓存内容刷新到程序缓存，以字符串形式返回内容，并关闭ob
- [ob_get_length](https://www.php.net/manual/zh/function.ob-get-length.php) — 返回输出缓冲区内容的长度
- [ob_get_level](https://www.php.net/manual/zh/function.ob-get-level.php) — 返回输出缓冲机制的嵌套级别
- [ob_get_status](https://www.php.net/manual/zh/function.ob-get-status.php) — 得到所有输出缓冲区的状态
- [ob_implicit_flush](https://www.php.net/manual/zh/function.ob-implicit-flush.php) — 打开/关闭绝对刷送
- [ob_list_handlers](https://www.php.net/manual/zh/function.ob-list-handlers.php) — 列出所有使用的输出处理程序
- [output_add_rewrite_var](https://www.php.net/manual/zh/function.output-add-rewrite-var.php) — 添加 URL 重写器的值
- [output_reset_rewrite_vars](https://www.php.net/manual/zh/function.output-reset-rewrite-vars.php) — 重设 URL 重写器的值
- [ob_gzhandler](https://www.php.net/manual/zh/function.ob-gzhandler.php) — ob_start 回调函数压缩输出缓冲区

| 函数                | 是否触发回调 | 说明                                            |
| ----------------- | ------ | --------------------------------------------- |
| ob_end_flush()    | ✅ 触发   | 输出缓冲区内容，触发回调函数处理内容，然后关闭缓冲区。                   |
| ob_get_flush()    | ✅ 触发   | 获取缓冲区内容不触发回调，然后刷新输出并关闭缓冲区并触发回调。               |
| ob_start()        | ❌ 不触发  | 开启缓冲区并绑定回调函数，但不立即触发。回调在缓冲区关闭时触发。              |
| ob_flush()        | ❌ 不触发  | 刷新当前缓冲区内容到下一层缓冲区或浏览器，但不关闭缓冲区，不触发回调。           |
| ob_get_clean()    | ❌ 不触发  | 等价于 ob_get_contents() + ob_end_clean()，不触发回调。 |
| ob_clean()        | ❌ 不触发  | 清空当前缓冲区内容，但不关闭缓冲区，不触发回调。                      |
| ob_end_clean()    | ❌ 不触发  | 关闭并清空缓冲区内容，不触发回调（内容直接丢弃）。                     |
| ob_get_contents() | ❌ 不触发  | 仅获取缓冲区内容，不关闭或触发回调。                            |
## 实例
### 一次性输出了全部的数字
```php
for ($i = 1; $i <= 5; $i++) {
    echo $i;
    sleep(1);
}
```
期望: 每秒输出一个数字
结果: 一次性输出了全部的数字
### 每秒输出一个数字
```php
for ($i = 0; $i < 3; $i++) {
    // 填充空字符保证浏览器正常输出, 默认为php.ini「output_buffering = 4096」
    echo str_repeat(' ', 4096);
    echo $i;
    flush(); // 刷新输出缓冲
    // ob_implicit_flush(true); // 确保每次输出都立即刷新缓冲区
    sleep(1);
}
```
期望: 每秒输出一个数字
结果: 每秒输出一个数字
#### 原因
- **缓冲阈值限制**：默认情况下，只有当数据达到 `output_buffering` 配置的大小（通常是 4096 字节）或脚本执行结束时，缓存才会自动发送。
- **手动干预**：
    - `str_repeat(' ', 4096)`：手动填满缓冲区，迫使 PHP 认为“容器已满”从而溢出数据。
    - `flush()`：强制将程序缓存中的内容发送到客户端。
- **协作关系**：在某些环境下，需要同时调用 `ob_flush()`（将 OB 刷入程序缓存）和 `flush()`（将程序缓存刷给用户）才能生效。
### 在header 前输出
```php
echo 111;
header("content-type:text/html; charset=utf-8");
echo 222;
```
运行后报了个警告：
```php
Warning: Cannot modify header information - headers already sent by (output started at /Users/weichengjun/dnmp/www/test/php/test/t1.php:2) in /Users/weichengjun/dnmp/www/test/php/test/t1.php on line 3
```
**报错根源**：HTTP 协议规定，**响应头 (Headers)** 必须在 **响应体 (Body)** 之前发送。一旦有任何内容（哪怕是一个空格）输出，PHP 就会自动发送 Header 并进入 Body 发送阶段，此时再执行 `header()` 就会报错。
#### ob_start 打开输出缓存
```php
ob_start(); // 打开输出缓存
echo 111;
header("content-type:text/html; charset=utf-8");
echo 222;
```
成功运行
#### 原因
**`ob_start()` 的拦截逻辑：**
1. **拦截输出**：开启 OB 后，所有的 `echo` 内容被截留在 **OB 缓存** 中，不再直接进入发送阶段。
2. **保留通道**：此时**程序缓存是空的**，`header()` 函数可以自由地将头部信息写入程序缓存。
3. **重新排序**：脚本结束时，PHP 自动执行“先发程序缓存里的 Header，再发 OB 缓存里的 Body”。
4. **结果**：在物理顺序上，**Header 永远保证了在 Body 数据之前**被发送给浏览器。
## 实例2
>注意：以下的实例测试均在`php.ini`配置 `output_buffering = Off`的条件下运行
### `ob_get_contents` 在不同环境下的行为差异分析
#### 1. 函数行为说明
- **`ob_end_flush()`**：将缓冲区内容输出到浏览器，并关闭缓冲区
- **`ob_get_contents()`**：获取当前缓冲区的内容（不清理）
#### 2. 浏览器环境行为
```php
// 执行流程：
ob_start();           // 启用缓冲
echo 111;             // 写入缓冲区
ob_end_flush();       // 输出111到浏览器，关闭缓冲区
```
- **关键点**：`ob_end_flush()` 只在**输出到浏览器时触发**，但不会立即清除PHP内部的缓冲状态
- **`ob_get_contents()`** 仍能获取到缓冲区内容，因为PHP内部状态未完全清理
#### 3. 脚本执行环境行为
- **脚本执行**：直接在命令行或非HTTP环境下运行
- **缓冲机制**：没有HTTP输出目标，`ob_end_flush()` 的行为更严格
- **结果**：`ob_get_contents()` 返回空值，因为缓冲区已被彻底清理
#### 4. 核心原因
- **HTTP环境**：存在输出目标（浏览器），缓冲区内容被发送后仍保留状态
- **脚本环境**：无输出目标，缓冲区被完全清理
- **PHP内部机制**：缓冲区管理在不同环境下有不同的清理策略
### 实例1
```php
ob_start(); // 开启ob缓存，后面的输出都会被放入ob缓存
echo 1; // 1 放入ob缓存
header('content-type:text/html; charset=utf-8'); // header放入程序缓存
echo 2; // 2 放入ob缓存
// 获取当前ob中的所有内容
$ob = ob_get_contents();
// 把ob中的内容写入到log.txt文件
file_put_contents('log.txt', $ob); // 此时ob缓存中有 12，被获取到$ob中，所以文件里是 12
echo 3; // 3 放入ob缓存
```
命令行环境
- 输出：123
- 文件：12
浏览器环境
- 输出：123
- 文件：12
### 实例2
```php
ob_start(); // 开启ob缓存，后面的输出都会被放入ob缓存
echo 1;
ob_clean(); // 清空输出缓冲区
header('content-type:text/html; charset=utf-8');
echo 2;
$ob = ob_get_contents(); // 获取当前ob中的所有内容
file_put_contents('log.txt', $ob);
echo 3;
```
命令行环境
- 输出：23
- 文件：2
浏览器环境
- 输出：23
- 文件：2
### 实例3
```php
ob_start(); // 开启ob缓存，后面的输出都会被放入ob缓存
echo 1;
ob_end_clean(); // 清空输出缓冲区并关闭
header('content-type:text/html; charset=utf-8');
echo 2;
$ob = ob_get_contents(); // 获取当前ob中的所有内容
file_put_contents('log.txt', $ob);
echo 3;
```
命令行环境
- 输出：23
- 文件：空
浏览器环境
- 输出：23
- 文件：2
### 实例4
```php
ob_start(); // 开启ob缓存，后面的输出都会被放入ob缓存
echo 111;
ob_end_flush(); // 冲刷出输出缓冲区内容并关闭缓冲
// 触发错误！`header()`必须在任何输出（包括空格、换行符）之前调用
header('content-type:text/html; charset=utf-8');
echo 222;
// 获取当前ob中的所有内容
$ob = ob_get_contents();
file_put_contents('log.txt', $ob);
echo 333;
```
命令行环境
- 输出：123
- 文件：空
浏览器环境
- 输出：123
- 文件：12
## 多级缓冲
### 核心机制：栈（Stack）
PHP 的 OB 缓存遵循 **“后进先出” (LIFO)** 的原则。
每调用一次 `ob_start()`，就会在栈顶压入一个新的缓冲区，所有的 `echo` 都会进入 **当前栈顶（最底层）** 的缓冲区。

这个程序实例有三个ob_start(),就意味着他有3个缓冲区A,B,C，而其实php程序本身也有一个最终输出的缓冲区，我们就把他叫做F。
在这个程序中他这几个缓冲区是有一定层次的，C->B->A->F，F层次最高，是程序最终的输出缓冲，我们按上面的程序来进行讲解。
### 实例
```php
ob_start();         // 新建缓冲区A; (A: null -> F: null)
echo 'L1', PHP_EOL; // 程序有输出，输出进入最低的缓冲区A; (A: 'L1' -> F: null)
ob_start();         // 新建缓冲区B; (B: null -> A: 'L1' -> F: null )
echo 'L2', PHP_EOL; // 程序有输出，输出进入最低的缓冲区B; (B: 'L2' -> A: 'L1' -> F: null)
ob_start();         // 新建缓冲区C; (C: null -> B: 'L2 ' -> A: 'L1' -> F: null)
echo 'L3', PHP_EOL; // 程序有输出，输出进入最低的缓冲区C; (C: 'L3 ' -> B: 'L2 ' -> A: 'L1' -> F: null)
ob_end_clean();     // 缓冲区C被清空并关闭; (B: 'L2 ' -> A: 'L1' -> F: null)
ob_end_flush();     // 缓冲区B输出到上一级的缓冲区A并关闭 (A: 'L1 L2 ' -> F: null)
ob_end_clean();     // 缓冲区A被清空并关闭, 此时缓冲区A的东西还没真正输出到最终的F中，因此也就整个程序也就没有任何的输出了。
```
# OB总结
- **OB 的作用：**
    - 控制 PHP 输出的时机和内容。
    - 允许在输出前修改或处理数据。
    - 提高脚本灵活性，例如，在发送 HTTP 头部后仍能修改内容。
- **OB 的开启方式：**
    - **`php.ini` 配置：**
        - `output_buffering = Off`：关闭 OB。
        - `output_buffering = On` 或 `output_buffering = [缓冲区大小]`：开启 OB，可指定缓冲区大小。
        - 作用于所有 PHP 页面。
    - **`ob_start()` 函数：**
        - 在当前 PHP 页面开启 OB。
        - 提供更灵活的控制，如指定回调函数处理输出。
- **OB 的工作流程：**
    - 开启 OB 后，`echo` 等输出函数的数据先进入 OB 缓冲区。
    - 脚本执行完毕或调用 `ob_end_flush()` 时，OB 缓冲区数据才发送到程序缓冲区。
    - 程序缓冲区在通过web服务器发送到浏览器。
    - `header` 信息始终进入程序缓冲区，不受 OB 影响。
- **无 OB 情况：**
    - 输出数据直接进入程序缓冲区。
- **缓冲区大小：**
    - OB 缓冲区有大小限制，超出限制或脚本结束时会发送数据。
    - 可通过 `php.ini` 或 `ob_start()` 指定缓冲区大小。