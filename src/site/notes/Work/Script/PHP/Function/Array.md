---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Array/","title":"Array","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T11:34:34.000+08:00","updated":"2026-03-24T17:38:16.961+08:00","dg-note-properties":{"title":"Array","tags":["flashcards"],"reference linking":null}}
---

# array_pad
将数组填充到具有值的指定长度
参数说明：
	$array：原始数组。
	$size：
		如果为负数，数组将被扩展到指定长度（左侧填充）。
		如果为正数，数组将被扩展到指定长度（右侧填充）。
		如果绝对值小于原数组长度，会截断数组。
	$value：用来填充的值。
```php
# 向数组左侧填充默认值
$array = [1, 2];
$padded = array_pad($array, -5, 0);
// 输出: [0, 0, 0, 1, 2]

# 向数组右侧填充默认值
$array = [1, 2];
$padded = array_pad($array, 5, 0);
// 输出: [1, 2, 0, 0, 0]
```
# array_replace
使用传递的数组替换第一个数组的元素，后面的数组里的值会覆盖前面的值。
```php
$base = ['orange', 'banana', 'apple', 'raspberry'];
$replace1 = [0 => 'pineapple', 4 => 'cherry'];
$replace2 = [0 => 'grape'];
$basket = array_replace($base, $replace1, $replace2);
var_export($basket);
/*
array (
    0 => 'grape',
    1 => 'banana',
    2 => 'apple',
    3 => 'raspberry',
    4 => 'cherry',
)
*/
```
# array_replace_recursive
使用传递的数组递归替换第一个数组的元素
```php
$base = [
    'citrus' => ['orange'],
    'berries' => [
        'blackberry',
        'raspberry',
    ],
];
$replacements = [
    'citrus' => ['pineapple'],
    'berries' => ['blueberry'],
];

$basket = array_replace_recursive($base, $replacements);
var_export($basket);
/*
array (
  'citrus' =>
  array (
    0 => 'pineapple',
  ),
  'berries' =>
	  array (
	    0 => 'blueberry',
	    1 => 'raspberry',
	  ),
)
*/

$basket = array_replace($base, $replacements);
var_export($basket);
/*
array (
  'citrus' => 
  array (
    0 => 'pineapple',
  ),
  'berries' => 
  array (
    0 => 'blueberry',
  ),
)
*/
```
# array_slice
从数组中取出一段
```php
$input = ['a', 'b', 'c', 'd', 'e'];
var_export(array_slice($input, 2)); // length省略，则序列将从 offset 开始一直到 array 的末端
/*
array (
  0 => 'c',
  1 => 'd',
  2 => 'e',
)
 */
var_export(array_slice($input, -2, 1)); // length 为正，则序列中将具有这么多的单元「包含」
/*
array (
  0 => 'd',
)
 */
var_export(array_slice($input, 0, 3)); // length 为正，则序列中将具有这么多的单元「包含」
/*
array (
  0 => 'a',
  1 => 'b',
  2 => 'c',
)
 */
// 注意数组键的差异
var_export(array_slice($input, 2, -1)); // length 为负，则序列将终止在距离数组末端这么远的地方「不包含」
/*
array (
0 => 'c',
1 => 'd',
)
 */
var_export(array_slice($input, 2, -1, true)); // 保留字符串的键
/*
array (
2 => 'c',
3 => 'd',
)
 */
```
# array_splice
去掉数组中的某一部分并用其它值取代
参数 [¶](https://www.php.net/manual/zh/function.array-splice.php#refsect1-function.array-splice-parameters)
**`array`**
	输入的数组。
**`offset`**
	如果 `offset` 为正，则从 `array` 数组中该值指定的偏移量开始移除。
	如果 `offset` 为负，则从 `array` 末尾倒数该值指定的偏移量开始移除。
**`length`**
	如果省略 `length`，则移除数组中从 `offset` 到结尾的所有部分。
	如果指定了 `length` 并且为正值，则移除这么多单元。
	如果指定了 `length` 并且为负值，则移除部分停止于数组末尾该数量的单元。
	如果设置了 `length` 为零，不会移除单元。
**小技巧**
	当给出了 `replacement` 时要移除从 `offset` 到数组末尾所有单元时，用 `count($input)` 作为 `length`。
**`replacement`**
	如果给出了 `replacement` 数组，则被移除的单元被此数组中的单元替代。
	如果 `offset` 和 `length` 的组合结果是不会移除任何值，则 `replacement` 数组中的单元将被插入到 `offset` 指定的位置。
> **注意**:
> 不保留替换数组 `replacement` 中的键名。

如果用来替换 `replacement` 只有一个单元，那么不需要给它加上 `array()` 或方括号，除非该单元本身就是一个数组、一个对象或者 **null**。
```php
$input = ['red', 'green', 'blue', 'yellow'];
array_splice($input, 2);
var_export($input);
/*
array (
  0 => 'red',
  1 => 'green',
)
*/

# 不包含截止的索引
$input = ['red', 'green', 'blue', 'yellow'];
array_splice($input, 1, -1);
var_export($input);
/*
array (
  0 => 'red',
  1 => 'yellow',
)
*/

$input = ['red', 'green', 'blue', 'yellow'];
array_splice($input, 1, count($input), 'orange');
var_export($input);
/*
array (
  0 => 'red',
  1 => 'orange',
)
*/

$input = ['red', 'green', 'blue', 'yellow'];
array_splice($input, -1, 1, ['black', 'maroon']);
var_export($input);
/*
array (
  0 => 'red',
  1 => 'green',
  2 => 'blue',
  3 => 'black',
  4 => 'maroon',
)
*/

// 添加两个新元素到 $input
array_push($input, $x, $y);
array_splice($input, count($input), 0, array($x, $y));

// 移除 $input 中的最后一个元素
array_pop($input);
array_splice($input, -1);

// 移除  $input 中第一个元素
array_shift($input);
array_splice($input, 0, 1);

// 在 $input 的开头插入一个元素
array_unshift($input, $x, $y);
array_splice($input, 0, 0, array($x, $y));

// 在 $input  的索引  $x 处替换值
$input[$x] = $y; // 对于键名和偏移量等值的数组
array_splice($input, $x, 1, $y);
```
# array_fill
用给定的值填充数组
```php
var_export(array_fill(0, 5, '?'));
/*
array (
  0 => '?',
  1 => '?',
  2 => '?',
  3 => '?',
  4 => '?',
)
*/
```
# array_chunk
将一个数组分割成多个
```php
$input_array = ['a', 'b', 'c', 'd', 'e'];
var_export(array_chunk($input_array, 2));
var_export(array_chunk($input_array, 2, true));
/*
array (
    0 =>
        array (
            0 => 'a',
            1 => 'b',
        ),
    1 =>
        array (
            0 => 'c',
            1 => 'd',
        ),
    2 =>
        array (
            0 => 'e',
        ),
)
array (
    0 =>
        array (
            0 => 'a',
            1 => 'b',
        ),
    1 =>
        array (
            2 => 'c',
            3 => 'd',
        ),
    2 =>
        array (
            4 => 'e',
        ),
)
 */
```
# array_column
返回数组中指定的一列
```php
// #1 返回数组中指定的一列
$name = array_column([
    ['id' => 1, 'user_id' => 1, 'name' => '张三'],
    ['id' => 2, 'user_id' => 2, 'name' => '李四'],
    ['id' => 3, 'user_id' => 1, 'name' => '王五'],
], 'name');
var_export($name);
/*
array (
    0 => '张三',
    1 => '李四',
    2 => '王五',
)
*/
// #2 空数组、键名不存在时, 返回空数组
var_export(array_column([], 'number'));
/*
array (
)
*/

// #3 获取指定值为键名的二维数组，相同键值则保留最后的键值对
var_export(array_column($name, null, 'user_id'));
/*
array (
    1 =>
        array (
            'id' => 3,
            'user_id' => 1,
            'name' => '王五',
        ),
    2 =>
        array (
            'id' => 2,
            'user_id' => 2,
            'name' => '李四',
        ),
)
*/
```
# array_combine
创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值 
```php
$a = ['01:00' => 1, '02:00' => 2, '03:00' => 3];
$b = [4, 5, 6];
$combine = array_combine(array_keys($a), array_values($b));
var_export($combine);
/*
array (
  '01:00' => 4,
  '02:00' => 5,
  '03:00' => 6,
)
*/
```
# array_unique
移除数组中重复的值, 返回过滤后的数组
先将值作为字符串排序, 如果相同值出现多次, 保留首位值, 注意键名保留不变,
```php
var_export(array_unique([
    'a' => 'green', 'b' => 'green', 'blue', 'red', 'red'
]));
/*
array (
  'a' => 'green',
  0 => 'blue',
  1 => 'red',
)
 */
 
 // php判断检测一个数组里有没有重复的值  
$array = [1, 1, 2, 3];  
if (count($array) != count(array_unique($array))) {  
    echo '该数组有重复值';  
}
```

# array_reverse
返回单元顺序相反的数组
```php
$input     = ["php", 4.0, ["green", "red"]];
$reversed  = array_reverse($input);
// 如果设置为 TRUE 会保留数字的键。 非数字的键则不受这个设置的影响，总是会被保留。
$preserved = array_reverse($input, true);

var_export($reversed);
var_export($preserved);
/*
array (
  0 =>
  array (
    0 => 'green',
    1 => 'red',
  ),
  1 => 4.0,
  2 => 'php',
)
array (
  2 =>
  array (
    0 => 'green',
    1 => 'red',
  ),
  1 => 4.0,
  0 => 'php',
)
*/
```
# array_flip
交换数组中的键和值
注意 array 中的值需要能够作为合法的键名(例如需要是 integer 或者 string) 
如果类型不对，将出现一个警告，并且有问题的键／值对将不会出现在结果里。
如果相同值出现多次，**保留末位值，并且替换位置**，其它值会被丢弃。
```php
$array1 = ['a' => 1, 'b' => 2, 'c' => 3, 'd' => 2];
var_export(array_flip($array1));
/*
array (
  1 => 'a',
  2 => 'd',
  3 => 'c',
)
 */
```
# +
合并数组,hash索引、数字索引都**只保留第一个数组的同键名的元素**，第二个数组中的元素将会被忽略
```php
var_export(
    [0 => 'a', 1 => 'b', 2 => 'c']
    +
    [1 => 'd', 2 => 'e', 3 => 'f']
    +
    ['a' => 'a1', 'b' => 'b1', 'c' => 'c1']
    +
    ['a' => 'a2', 'b' => 'b2', 'c' => 'c2']
);
/*
array (
  0 => 'a',
  1 => 'b',
  2 => 'c',
  3 => 'f',
  'a' => 'a1',
  'b' => 'b1',
  'c' => 'c1',
)
 */
```
# array_merge
合并一个或多个数组
1. 字符串索引: **保留最后数组的同键名的元素**
2. 数字索引: **相同键追加不覆盖**「若只给了一个数组且该数组是数字索引的,则键名会以连续方式重新索引」
3. 切记不用作合并数字键名的数组！
```php
var_export(array_merge(
    [1, 2, 'a' => 'a1', 'b' => 'b1'],
    [1, 2, 'a' => 'a2', 'b' => 'b3']
));
/*
array (
  0 => 1,
  1 => 2,
  'a' => 'a2',
  'b' => 'b3',
  2 => 1,
  3 => 2,
)
 */

// 如果只给了一个数组并且该数组是数字索引的,则键名会以连续方式重新索引
var_export(array_merge([1 => 'data']));
/*
array (
  0 => 'data',
)
 */
```
# array_merge_recursive
递归地合并一个或多个数组
1. 字符串索引: 相同键合并到一个数组「递归」
2. 数字索引: 相同键追加
```php
$ar1 = [
    1,
    'fruits' => 'orange',
    'color' => [
        'favorite' => 'red'
    ],
];
$ar2 = [
    1,
    2,
    'fruits' => 'strawberry',
    'color' => [
        'favorite' => 'green',
        'blue'
    ]
];
var_export(array_merge_recursive($ar1, $ar2));
/*
array (
  0 => 1,
  'fruits' => 
  array (
    0 => 'orange',
    1 => 'strawberry',
  ),
  'color' => 
  array (
    'favorite' => 
    array (
      0 => 'red',
      1 => 'green',
    ),
    0 => 'blue',
  ),
  1 => 1,
  2 => 2,
) 
*/
```
# array_flip
效率高于 array_unique()
## 第一种方式，使用 **array_merge** 修复数组的keys
添加array_flip之后的函数，将会对数组的键值排序并且让它们恢复到正常的序列，如：0,1,2,3…
```php
$array = array_flip(['green','blue','orange','blue']);
print_r($array);
/** 
Array
(
    [green] => 0
    [blue] => 3
    [orange] => 2
)
*/

$array = array_flip($array);
print_r($array);
/** 
Array
(
    [0] => green
    [3] => blue
    [2] => orange
)
*/

// 如果只给了一个数组并且该数组是数字索引的,则键名会以连续方式重新索引
print_r(array_merge($array));
/** 
Array
(
    [0] => green
    [1] => blue
    [2] => orange
)
*/
```
## 第二种方式，使用 **array_keys** 修复数组的keys
注意，这种修复数组键值的方法**比使用`array_merge`函数稍微快了一点**。
你也可以在最后一步结合使用array_keys()函数(此函数返回翻转后的值)。然后当你翻转数组的值，键值就会根据顺序创建。
```php
$array = array_flip(['green','blue','orange','blue']);
print_r($array);
/** 
Array
(
    [green] => 0
    [blue] => 3
    [orange] => 2
)
*/

/* 跟第一个例子一样，但是现在我们先提取数组的键值 */
$array = array_keys($array);
print_r($array);
/** 
Array
(
    [0] => green
    [1] => blue
    [2] => orange
)
*/
```
# array_keys
返回数组中部分的或所有的键名
如果指定了可选参数 search_value，则只返回该值的键名
否则 input 数组中的所有键名都会被返回。
```php
print_r(array_keys([0 => 100, 'color' => 'red']));
/*
Array
(
    [0] => 0
    [1] => color
)
 */

// 搜索指定值键名
print_r(array_keys([
    'blue', 'red', 'green', 'blue', 'blue'
], 'blue'));
/*
Array
(
    [0] => 0
    [1] => 3
    [2] => 4
)
 */

// 二维数据只返回一维数组键名
print_r(array_keys([
    'color' => ['blue', 'red', 'green'],
    'size' => ['small', 'medium', 'large']
]));
/*
Array
(
    [0] => color
    [1] => size
)
 */
```
# array_reduce
## 功能
用回调函数迭代地将数组简化为单一的值
需要提供一个初始值（可选）。
每次回调接收当前累加值和当前元素。
适用于**聚合计算**、**结构转换**等。
## 使用场景
如果你要从数组中提取汇总信息（如总和、平均值、结构重组），用 array_reduce。
## 参数
$array：要处理的数组。
$callback：每次迭代执行的回调函数，接收两个参数：
第一个参数是累计值（carry）
第二个参数是当前元素（item）
$initial：初始累计值（可选）

### 数组求和 / 求积
```php
function sum($carry, $item)
{
	// 携带上次迭代的返回值； 如果本次迭代是第一次，那么这个值是 `initial`
	// 携带了本次迭代的值。
    $carry += $item;
    return $carry;
}

function product($carry, $item)
{
    $carry *= $item;
    return $carry;
}

$a = [1, 2, 3, 4, 5];
$x = [];

var_dump(array_reduce($a, "sum")); // int(15)
var_dump(array_reduce($a, "product", 10)); // int(1200), 因为：10*1*2*3*4*5
var_dump(array_reduce($x, "sum", "No data to reduce")); // string(17) "No data to reduce"
```
### 拼接字符串
```php
$words = ['hello', 'world', 'php'];
$sentence = array_reduce($words, fn($carry, $word) => $carry . ' ' . $word, '');
// $sentence = " hello world php"
```
### 构建键值对映射表（字典）
```php
$users = [
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
];

$userMap = array_reduce($users, function ($carry, $user) {
    $carry[$user['id']] = $user['name'];
    return $carry;
}, []);

// $userMap = [1 => 'Alice', 2 => 'Bob']
```
### 多维数组合并去重
```php
$data = [
    [1, 2, 3],
    [3, 4, 5],
    [5, 6],
];

$unique = array_reduce($data, function ($carry, $arr) {
    return array_merge($carry, $arr);
}, []);

$unique = array_unique($unique);

// $unique = [1, 2, 3, 4, 5, 6]
```
# array_walk
使用用户自定义函数对数组中的每个元素做回调处理
## 功能
对数组中的每个元素执行回调函数。
直接**修改原始数组**（通过引用传递）。
回调函数接收当前元素的引用，可以修改其值。
## 使用场景
**需要直接修改原数组内容。**
操作键值对（如添加前缀、后缀、格式化等）。
执行副作用（如记录日志、发送请求等）。
```php
$fruits = [
    'a' => 'orange', 'b' => 'banana',
    'c' => 'apple', 'd' => 'lemon'
];
function get($v, $k)
{
    echo $k, '=>', $v, PHP_EOL;
}

function alter(&$v, $k, $prefix)
{
    $v = $prefix . ': ' . $v;
}

array_walk($fruits, 'get');
/*
a=>orange
b=>banana
c=>apple
d=>lemon
*/
array_walk($fruits, 'alter', 'fruit');
array_walk($fruits, 'get');
/*
a=>fruit: orange
b=>fruit: banana
c=>fruit: apple
d=>fruit: lemon
*/
```
# array_walk_recursive
对数组中的每个成员递归地应用用户函数
```php
$fruits = [
    'sweet' => [
        'a' => 'apple',
        'b' => 'banana'
    ],
    'sour' => 'lemon'
];

function test_print(&$item, $key)
{
    $item = strtoupper($item);
    echo $key, ' holds ', $item, PHP_EOL;
}

array_walk_recursive($fruits, 'test_print');
var_export($fruits);
/*
a holds APPLE
b holds BANANA
sour holds LEMON
array (
  'sweet' => 
  array (
    'a' => 'APPLE',
    'b' => 'BANANA',
  ),
  'sour' => 'LEMON',
)
 */
```
# array_map
## 功能
对数组中的每个元素应用回调函数。
返回一个新的数组，原数组不变。
支持多个数组作为输入，按顺序传入回调。
## 使用场景
数组元素转换（如字符串转小写、数值运算等）。
合并多个数组的对应元素进行处理。
**不希望修改原数组时使用。**
```php
// #1 单数组遍历
function myFunction($v) {
    return ($v * $v);
}
var_export(array_map("myFunction", [1, 2, 3, 4, 5]));
/*
array (
  0 => 1,
  1 => 4,
  2 => 9,
  3 => 16,
  4 => 25,
)
*/
 
 // #2 多数组遍历
$a = range(1, 3);
$b = ['one', 'two', 'three'];
function show($n, $m) {
    return "The number ${n} is called ${m} in English";
}
function map($n, $m) {
    return [$n => $m];
}
$c = array_map('show', $a, $b);
var_export($c);
/*
array (
  0 => 'The number 1 is called one in English',
  1 => 'The number 2 is called two in English',
  2 => 'The number 3 is called three in English',
)
*/
$d = array_map('map', $a, $b);
var_export($d);
/*
array (
  0 => array (
    1 => 'one',
  ),
  1 => array (
    2 => 'two',
  ),
  2 => array (
    3 => 'three',
  )
)
*/
 
// #3 使用匿名函数 (PHP 5.3.0 起)
$func = function ($value) {
    return $value * 2;
};
var_export(array_map($func, range(1, 3)));
/*
array (
  0 => 2,
  1 => 4,
  2 => 6
)
*/

// #4 调用对象方法
class test
{
    public final function add($key, $value)
    {
        if (is_array($value)) {
            array_map([$this, __FUNCTION__], array_keys($value), array_values($value));
        } else {
            echo $key, ' => ', $value, PHP_EOL;
        }
    }
}
(new test)->add('', [
    'one' => 'value1',
    'two' => 'value2',
    'three' => 'value3'
]);
/*
one => value1
two => value2
three => value3
*/
```
# array_change_key_case
将数组中的所有键名修改为全大写或小写
```php
// CASE_UPPER 或 CASE_LOWER(默认值)
var_export(array_change_key_case(
    ['first' => 1, 'second' => 4],
    CASE_UPPER
));
/*
array (
  'FIRST' => 1,
  'SECOND' => 4,
)
 */
```
# current
输出数组中的当前元素的值(每个数组中都有一个内部的指针指向它的"当前"元素，初始指向插入到数组中的第一个元素)
不推荐原因：
https://stackoverflow.com/questions/15446602/php-current-returns-false-when-passing-a-valid-array-to-it
```php
echo current(['riven', 'gamer']); // riven
```
# reset
将数组的内部指针指向第一个单元(获取数组中的第一个元素)
```php
$arr = ['riven', 'gamer'];
echo reset($arr); // riven
```
# end
输出数组中的最后一个元素的值
注:参数只能传变量, 不能直接传数组
```php
echo end(['riven', 'gamer']); // error, 只有变量可以通过引用传递
$end = ['riven', 'gamer'];
echo end($end); // gamer
```
# array_unshift
向数组头部插入 "joe" 和 "hank"(头部入栈)
```php
$names = ['3'];
array_unshift($names, '1', '2');
var_export($names);
/*
array (
  0 => 1,
  1 => 2,
  2 => 3,
)
*/
```
# array_shift
将数组开头的单元移出数组(头部出栈)
```php
$names = ['a', 'b', 'c'];
var_export(array_shift($names)); // a
var_export($names);
/*
array (
  0 => 'b',
  1 => 'c',
)
*/
```
# array_push
将一个或多个单元压入数组的末尾(尾部入栈)
```php
$a = ['a', 'b'];
array_push($a, 'c', 'd');
var_export($a);
/*
array (
0 => 'a',
1 => 'b',
2 => 'c',
3 => 'd',
)
*/
```
# array_pop
弹出数组最后一个单元(尾部出栈)
```php
$a = ['a', 'b'];
var_export(array_pop($a)); // b
var_export($a);
/*
array (
0 => 'a',
)
*/
```
# array_sum
返回数组中所有值的和
```php
$a = [1, 2, 3];
echo array_sum($a);//输出结果: 6
```
# range
根据范围创建数组，包含指定的元素
```php
var_export(range(1, 5));
/*
array (
  0 => 1,
  1 => 2,
  2 => 3,
  3 => 4,
  4 => 5,
)
*/
var_export(range(10, 50, 10));
/*
array (
  0 => 10,
  1 => 20,
  2 => 30,
  3 => 40,
  4 => 50,
)
*/

var_export(range('a', 'e'));
/*
array (
  0 => 'a',
  1 => 'b',
  2 => 'c',
  3 => 'd',
  4 => 'e',
)
*/

var_export(range('e', 'a'));
/*
array (
  0 => 'e',
  1 => 'd',
  2 => 'c',
  3 => 'b',
  4 => 'a',
)
*/
```
# array_multisort
对多个数组或多维数组进行排序
[PHP数组排序函数array\_multisort()函数详解 - hiyanxu - 博客园](https://www.cnblogs.com/WuNaiHuaLuo/p/5794669.html)
```php
$arr = [
    [
        "name" => "小A",
        "sort" => 7,
        "money" => 123,
    ],
    [
        "name" => "小B",
        "sort" => 2,
        "money" => 321,
    ],
    [
        "name" => "小C",
        "sort" => 3,
        "money" => 23,
    ],
    [
        "name" => "小D",
        "sort" => 9,
        "money" => 23,
    ],
    [
        "name" => "小E",
        "sort" => 1,
        "money" => 22,
    ],
];

// 根据money降序、sort升序
array_multisort(
    array_column($arr, 'money'), SORT_DESC, 
    array_column($arr, 'sort'), SORT_ASC, 
    $arr
);
var_export($arr);
/*
array (
  0 => 
  array (
    'name' => '小B',
    'sort' => 2,
    'money' => 321,
  ),
  1 => 
  array (
    'name' => '小A',
    'sort' => 7,
    'money' => 123,
  ),
  2 => 
  array (
    'name' => '小C',
    'sort' => 3,
    'money' => 23,
  ),
  3 => 
  array (
    'name' => '小D',
    'sort' => 9,
    'money' => 23,
  ),
  4 => 
  array (
    'name' => '小E',
    'sort' => 1,
    'money' => 22,
  ),
)
*/

$arr = [
    [
        "name" => "小A",
        "sort" => 7,
        "money" => 123,
    ],
    [
        "name" => "小B",
        "sort" => 2,
        "money" => 321,
    ],
    [
        "name" => "小C",
        "sort" => 3,
        "money" => 23,
    ]
];

// 根据money升序
array_multisort(array_column($arr, 'money'), SORT_ASC, $arr);
var_export($arr);
/*
array (
  0 => 
  array (
    'name' => '小C',
    'sort' => 3,
    'money' => 23,
  ),
  1 => 
  array (
    'name' => '小A',
    'sort' => 7,
    'money' => 123,
  ),
  2 => 
  array (
    'name' => '小B',
    'sort' => 2,
    'money' => 321,
  ),
)
 */
```
# usort
使用用户自定义的比较函数对数组中的值进行排序
排序完成后，它会**丢弃**原有的键，并为排序后的数组分配新的、从 `0` 开始的数字键。
**特点：**
- **重新索引**：排序后，所有元素都会得到新的数字键。
- **忽略原有键**：函数只关注值，不保留键值对的关联性。
- **使用场景**：当你不关心数组原有键，只需要对值进行排序时，`usort()` 非常适用，例如对一个简单的数字或字符串列表进行排序。
```php
// #1
// 在第一个参数小于，等于或大于第二个参数时，
// 该比较函数必须相应地返回一个小于，等于或大于 0 的整数。
$a = [3, 2, 4, 5, 1];
usort($a, function ($a, $b) {
    if ($a == $b) return 0;
    return ($a < $b) ? -1 : 1;
});
foreach ($a as $k => $v) {
    echo $k, ':', $v, PHP_EOL;
}
/*
0:1
1:2
2:3
3:4
4:5
*/

// #2
class Test
{
    var $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public static function cmpObj($a, $b)
    {
        $al = strtolower($a->name);
        $bl = strtolower($b->name);
        if ($al == $bl) return 0;
        return ($al > $bl) ? 1 : -1;
    }
}

$a[] = new Test('c');
$a[] = new Test('b');
$a[] = new Test('d');

usort($a, ['Test', 'cmpObj']);

foreach ($a as $item) {
    echo $item->name, PHP_EOL;
}
/*
b
c
d
*/

// #3
$array = [
    ['key_a' => 'z', 'key_b' => 'c'],
    ['key_a' => 'x', 'key_b' => 'b'],
    ['key_a' => 'y', 'key_b' => 'a']
];
function build_sorter($key)
{
    return function ($a, $b) use ($key) {
        return strnatcmp($a[$key], $b[$key]);
    };
}

usort($array, build_sorter('key_b'));

foreach ($array as $item) {
    echo $item['key_a'], ' , ', $item['key_b'], PHP_EOL;
}
/*
y , a
x , b
z , c
*/
```
# uasort
使用用户定义的比较函数对数组进行排序并**保持原有键**
**特点：**
- **保留键**：排序后，键值对的关联关系不会被破坏。
- **不重新索引**：数组的键保持不变，只是元素的顺序发生了变化。
- **使用场景**：当你需要对一个关联数组或对象数组进行排序，并且需要保持键值对的完整性时，`uasort()` 是正确的选择，例如按用户 ID 对用户数据进行排序。
```php
// 比较函数
function cmp($a, $b) {
    if ($a == $b) {
        return 0;
    }
    return ($a < $b) ? -1 : 1;
}

// 要排序的数组
$array = array('a' => 4, 'b' => 8, 'c' => -1, 'd' => -9, 'e' => 2, 'f' => 5, 'g' => 3, 'h' => -4);
print_r($array);

// 排序并打印排序后的数组
uasort($array, 'cmp');
print_r($array);
?>
```
# compact
创建一个包含变量名和它们的值的数组
```php
$hello = 'hello';
$world = 'world';
var_export(compact('hello', 'world'));
/*
array (
  'hello' => 'hello',
  'world' => 'world',
)
*/
```
# extract
从数组中将变量导入到当前的符号表
```php
$size = 'large';
$var_array = [
    'color' => 'blue',
    'shape' => 'sphere',
    'size' => 'medium'
];

// EXTR_PREFIX_SAME 如果有冲突，在变量名前加上前缀
extract($var_array, EXTR_PREFIX_SAME, 'riven');

echo $color, PHP_EOL, $shape, PHP_EOL, $size, PHP_EOL, $riven_size;
// blue sphere large medium
```
# list
把数组中的值赋给一组变量
注:PHP 5 里，list() 从最右边的参数开始赋值； PHP 7 里，list() 从最左边的参数开始赋值。
```php
$info = ['coffee', 'brown', 'caffeine'];

// 列出所有变量
list($drink, $color, $power) = $info;
// [$drink, $color, $power] = $info; PHP7.1以上支持这种写法
echo "$drink, $color, $power", PHP_EOL; // coffee, brown, caffeine

list($drink, , $power) = $info;
echo "$drink, $power", PHP_EOL; // coffee, caffeine

list(, , $power) = $info;
echo $power, PHP_EOL; // caffeine

// list() 不能对字符串起作用
list($bar) = 'abcde';
var_dump($bar); // NULL
```