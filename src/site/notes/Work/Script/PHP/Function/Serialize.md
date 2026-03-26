---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Serialize/","title":"Serialize","tags":["flashcards"],"noteIcon":"","created":"2026-03-01T11:59:19.000+08:00","updated":"2026-03-24T17:37:53.704+08:00","dg-note-properties":{"title":"Serialize","tags":["flashcards"],"reference linking":null}}
---

# json_encode
对变量进行 JSON 编码,返回字符串，包含了 value 值 JSON 形式的表示。
```php
$arr = ['a'=>1,'b'=>2,'c'=>3,'d'=>4,'e'=>5];
echo json_encode($arr); // {"a":1,"b":2,"c":3,"d":4,"e":5}

$arr = ['a'=>'你','b'=>'好','c'=>'世','d'=>'界'];
// Json不要编码Unicode, 针对中文编码，进行解析
echo json_encode($arr, JSON_UNESCAPED_UNICODE);
echo json_encode($arr, 256);
// {"a":"你","b":"好","c":"世","d":"界"}
```
# json_decode
对 JSON 格式的字符串进行解码，转换为 PHP 变量
```php
$json = '{"a":1,"b":2,"c":3,"d":4,"e":5}';
var_dump(json_decode($json)); // 默认对象
/*
object(stdClass)#1 (5) {
["a"] => int(1)
["b"] => int(2)
["c"] => int(3)
["d"] => int(4)
["e"] => int(5)
}
*/

var_dump(json_decode($json, true)); // 转数组
/*
array(5) {
    ["a"] => int(1)
    ["b"] => int(2)
    ["c"] => int(3)
    ["d"] => int(4)
    ["e"] => int(5)
}
*/
```
# json_last_error
如果有，返回 JSON 编码解码时最后发生的错误
```php
// 无效的 UTF8 序列
$text = "\xB1\x31";
$json  = json_encode($text);
var_dump($json, json_last_error() === JSON_ERROR_UTF8);
/*
bool(false)
bool(true)
*/

// 判断是否为json
function isJson($string): bool  
{  
    json_decode($string);  
    return (json_last_error() == JSON_ERROR_NONE);  
}
```
# serialize()
函数用于序列化对象或数组，并返回一个字符串。
这有利于存储或传递 PHP 的值，同时不丢失其类型和结构
```php
echo serialize(['riven', 'loren', 'gamer']) . PHP_EOL;
/*
a:3:{i:0;s:5:"riven";i:1;s:5:"loren";i:2;s:5:"gamer";}
*/
```
# unserialize()
函数用于将通过 serialize()函数序列化后的对象或数组进行反序列化，
并返回原始的对象结构
```php
var_export(unserialize(
    'a:3:{i:0;s:5:"riven";i:1;s:5:"loren";i:2;s:5:"gamer";}'
));
/*
array (
  0 => 'riven',
  1 => 'loren',
  2 => 'gamer',
)
*/
```
# simplexml_load_string()
将XML字符串解释为对象
```php
$xml       = "<xml><appid>123456</appid></xml>"; // XML文件
$objectXml = simplexml_load_string($xml); // 将文件转换成 对象
$xmlJson   = json_encode($objectXml); // 将对象转换个JSON
$xmlArray  = json_decode($xmlJson, true); // 将json转换成数组
var_dump($xmlArray);
/*
array(1) {
  ["appid"]=>
  string(6) "123456"
}
*/
```
# pack/unpack
[PHP: pack - Manual](https://www.php.net/manual/zh/function.pack.php)
用于将变量按照指定的**二进制格式**打包成字符串，常用于处理二进制数据（如网络协议、文件头、加密数据等）
```
pack(string $format, mixed ...$values): string
```
### format 字符串由格式代码组成，后面跟着一个可选的重复参数
| a   | 以 NUL 字节填充字符串                 |
| --- | ----------------------------- |
| A   | 以 SPACE(空格) 填充字符串             |
| h   | 十六进制字符串，低位在前                  |
| H   | 十六进制字符串，高位在前                  |
| c   | 有符号字符                         |
| C   | 无符号字符                         |
| s   | 有符号短整型(16位，主机字节序)             |
| S   | 无符号短整型(16位，主机字节序)             |
| n   | 无符号短整型(16位，大端字节序)             |
| v   | 无符号短整型(16位，小端字节序)             |
| i   | 有符号整型(机器相关大小字节序)              |
| I   | 无符号整型(机器相关大小字节序)              |
| l   | 有符号长整型(32位，主机字节序)             |
| L   | 无符号长整型(32位，主机字节序)             |
| N   | 无符号长整型(32位，大端字节序)             |
| V   | 无符号长整型(32位，小端字节序)             |
| q   | 有符号长长整型(64位，主机字节序)            |
| Q   | 无符号长长整型(64位，主机字节序)            |
| J   | 无符号长长整型(64位，大端字节序)            |
| P   | 无符号长长整型(64位，小端字节序)            |
| f   | 单精度浮点型(机器相关大小)                |
| g   | 单精度浮点型(机器相关大小，小端字节序)          |
| G   | 单精度浮点型(机器相关大小，大端字节序)          |
| d   | 双精度浮点型(机器相关大小)                |
| e   | 双精度浮点型(机器相关大小，小端字节序)          |
| E   | 双精度浮点型(机器相关大小，大端字节序)          |
| x   | NUL 字节                        |
| X   | 回退一字节                         |
| Z   | 以 NUL 结尾（ASCIIZ）字符串，将用 NUL 填充 |
| @   | NUL 填充到绝对位置                   |
- **大端**：高位字节在前，符合人类阅读习惯。
- **小端**：低位字节在前，更利于处理器处理。
- **主机字节序**：你的电脑使用的字节序，通常是**小端**。
- **网络字节序**：数据在网络上传输时遵循的统一标准，即**大端**。
### 示例1：pack 与 unpack
```php
// 打包 (Pack)
// 将给定的值按照指定的格式指令打包成一个二进制字符串。
// 'n' (无符号短整数，大端序/网络字节序) => 0x1234
// 'V' (无符号长整数，小端序/主机字节序) => 0x56789ABC
// 'C' (无符号字符/字节) => 0xFF
$binary = pack('nVC', 0x1234, 0x56789ABC, 0xFF);
// 解包 (Unpack)
// 从二进制字符串 $binary 中解析数据，并按照指定的格式指令和命名提取数据。
// 'nflag': 2字节无符号短整数（大端序），命名为 flag
// 'Vseq': 4字节无符号长整数（小端序），命名为 seq
// 'Cack': 1字节无符号字符，命名为 ack
$data = unpack('nflag/Vseq/Cack', $binary);
var_export($data);
/*
输出解析后的数组内容
结果示例 (取决于系统字节序，但 V/n 指令确保了结果的一致性):
array (
  'flag' => 4660,      // 0x1234
  'seq' => 1450550428, // 0x56789ABC
  'ack' => 255,        // 0xFF
)
*/
```
### 示例2：利用 pack/unpack 在共享内存中存储序列化数据
`pack('L', $strlen)`的作用是创建一个长度前缀，这样在读取共享内存时就能：
1. 精确知道数据的实际长度
2. 避免读取过多无用数据
3. 正确分割多个连续存储的数据
4. 安全处理包含特殊字符的序列化数据
>如果你直接读取共享内存，你怎么知道这个字符串在哪里结束？所以需要使用pack将原始数据的长度序列化
```php
// 写入数据
$data = ['name' => 'tom', 'age' => 18];
$serialized = serialize($data);
$length = strlen($serialized);

// 格式: [4字节长度][N字节数据]
$lengthInfo = pack('L', $length);
$fullData = $lengthInfo . $serialized;

shmop_write($shmId, $fullData, 0);

// 读取数据
$lengthData = shmop_read($shmId, 0, 4); // 先读取长度信息
$dataLength = unpack('L', $lengthData)[1]; // 解析出实际长度
$serializedData = shmop_read($shmId, 4, $dataLength); // 精确读取数据
$originalData = unserialize($serializedData); // 反序列化

var_dump($originalData); // ['name' => 'tom', 'age' => 18]
```
使用场景:
- 数据通信(通过二进制格式与其它语言通信) 
- 数据加密(如果不告诉第三方你的打包方式，对方解包的难度就相对很大) 
- 节省空间(比较大的数字按字符串储存会浪费很多空间，打包成二进制格式才需要4字节「32位数字」)
# 性能和效率排行（一般情况）
| 维度         | 第一名 (最快/最小)  | 第二名                | 第三名 (最慢/最大)      |
| :--------- | :----------- | :----------------- | :--------------- |
| **序列化速度**  | `pack`       | `json_encode`      | `serialize`      |
| **反序列化速度** | `pack`       | `json_encode`      | `serialize`      |
| **结果数据大小** | `pack` (二进制) | `json_encode` (文本) | `serialize` (文本) |
**注意：** 这里的 `pack` 通常会产生 **最小且最快** 的结果，但它的适用性**最受限**。`json_encode` 在许多现代 PHP 版本中已被高度优化，通常比 `serialize` 更快且结果更小。
## 三者的主要区别和作用
这三个函数代表了三种不同的数据处理需求：
### 1. serialize()、unserialize()
| 特性       | 描述                                                                                            |
| :------- | :-------------------------------------------------------------------------------------------- |
| **主要作用** | 将 **任意** PHP 变量（包括对象、资源、引用等复杂结构）转换为一个 **PHP 特定** 的字符串表示。                                      |
| **数据格式** | **文本格式**。格式冗长，包含大量类型和长度信息（例如 `s:4:"name";`）。                                                  |
| **跨语言性** | **差**。这是 **PHP 专用** 格式，其他语言无法直接解析。                                                            |
| **安全性**  | **最差**。使用 `unserialize()` 处理来自**不可信来源**的数据是 **非常危险的**，可能导致 **PHP 对象注入漏洞 (Object Injection)**。 |
| **适用场景** | 仅用于 **PHP 内部** 存储或传递数据，且需要 **保留对象类型和引用** 的场合（如 Session 存储、Memcached 缓存）。                      |
### 2. json_encode()、json_decode()
| 特性        | 描述                                                               |
| :-------- | :--------------------------------------------------------------- |
| **主要作用**  | 将 PHP 数组或对象转换为 **JSON 格式** 的字符串。                                 |
| **数据格式**  | **文本格式**。结构清晰、**可读性好**。                                          |
| **跨语言性**  | **优秀**。JSON (JavaScript Object Notation) 是 **通用标准**，几乎所有编程语言都支持。 |
| **安全性**   | **较好**。`json_decode()` 不会尝试实例化任意 PHP 对象，安全性更高。                   |
| **性能/大小** | 速度快，生成结果比 `serialize` **小且快**（因为它不存储 PHP 专有信息）。                  |
| **适用场景**  | **Web API 接口**、前后端数据交换、**通用** 数据缓存、日志存储。                         |
### 3. pack()、unpack()
| 特性        | 描述                                                               |
| :-------- | :--------------------------------------------------------------- |
| **主要作用**  | 根据提供的**格式模板**，将数据打包成 **二进制字符串**。                                 |
| **数据格式**  | **二进制格式**。没有分隔符和类型标记，**紧凑且不可读**。                                 |
| **跨语言性**  | **中等**。依赖于精确的格式模板和字节序（endianness），但理论上其他语言可以按照相同的格式模板解析。         |
| **性能/大小** | **最快且最小**。因为它不包含任何额外的元数据，直接将数据类型映射为字节流。                          |
| **适用场景**  | **底层数据存储**、网络协议数据包（如 TCP/UDP）、处理 C 语言结构体或**定长**、**已知结构** 的二进制数据。 |
## 总结：如何选择？
| 需求                         | 推荐方案              | 理由                                                                                                    |
| :------------------------- | :---------------- | :---------------------------------------------------------------------------------------------------- |
| **Web API / 跨语言通信**        | **`json_encode`** | 通用标准、可读性高、安全性好。                                                                                       |
| **PHP 内部复杂对象缓存**           | **`serialize`**   | 必须保留对象的类名、私有/保护属性和引用关系。                                                                               |
| **极致性能和数据大小**              | **`pack`**        | 数据结构简单（如固定长度的整数、浮点数），且只需要在 PHP 内部或底层系统中使用。                                                            |
| **通用数据缓存**                 | **`json_encode`** | 比 `serialize` 更快更小，且避免安全风险。                                                                           |
| **替代方案（MsgPack/Igbinary）** | **扩展库**           | 如果对性能有**极高要求**，可以考虑使用像 **MsgPack** 或 **Igbinary** 这样的第三方二进制扩展，它们通常比原生 `json_encode` 或 `serialize` 更快。 |
**简而言之：**
* **90% 的场景**：使用 **`json_encode`**。它速度快、结果小、通用且安全。
* **需要保留 PHP 对象完整性**：使用 **`serialize`** (注意安全)。
* **处理底层二进制数据**：使用 **`pack`**。