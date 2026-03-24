---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Algorithm/","title":"Algorithm","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T17:39:26.000+08:00","updated":"2026-03-24T17:47:09.681+08:00"}
---

# 递归获取指定键值
```php
/**
 * @param  array  $array  待过滤数组
 * @param  array|string  $filterKey  过滤键配置
 * @param  string  $prepend  前缀
 * @return array
 */
function filter(array $array, array|string $filterKey = '', string $prepend = ''): array
{
    $filterKey = is_string($filterKey) ? explode('.', $filterKey) : $filterKey;

    foreach ($filterKey as $i => $filter) {
        if (isset($array[$filter])) {
            $array = $array[$filter];
        } elseif (in_array($filter, ['*', '**'])) {
            if ($filter == '*') {
                $filterKey = array_slice($filterKey, ++$i);
            } else {
                $filter = $filterKey[++$i] ?? '';
            }

            $results = [];
            foreach ($array as $key => $value) {
                if (!$filterKey || $filter == $key) {
                    $results += [$prepend.$key => $value];
                } elseif (is_array($value)) {
                    $results += filter($value, $filterKey, $prepend.$key.'.');
                }
            }

            return $results;
        } else {
            return [];
        }
        $prepend .= "$filter.";
    }

    return [
        rtrim($prepend, '.') => $array,
    ];
}
$arr = [
    'nodes' => [
        [
            [
                'userid' => 1
            ],
            [
                'userid' => 2
            ]
        ],
        [
            [
                'userid' => 11
            ],
            [
                'userid' => 22
            ]
        ]
    ]
];
var_export(filter($arr, 'nodes.*.*.userid'));
/*
array (
  'nodes.0.0.userid' => 1,
  'nodes.0.1.userid' => 2,
  'nodes.1.0.userid' => 11,
  'nodes.1.1.userid' => 22,
) 
*/
```
# 数组嵌套赋值
```php
function set(&$arr, $k, $v)
{
    if (is_null($k)) {
        $arr = $v;
        return;
    }
    if (!is_string($k)) {
        $arr[$k] = $v;
        return;
    }
    $exploder = explode('.', $k);
    while (count($exploder) > 1) {
        $arr = &$arr[array_shift($exploder)]; // 传引用(理解为递归嵌套)
    }
    $arr[array_shift($exploder)] = $v;
}
set($a, 'a.b.c', 3);
var_export($a);
/*
array (
  'a' =>
  array (
    'b' =>
    array (
      'c' => 3,
    ),
  ),
)
*/
```
# 常见算法
```php
/*
1、一群猴子排成一圈，按1，2，…，n依次编号。
然后从第1只开始数，数到第m只,把它踢出圈，
从它后面再开始数，再数到第m只，在把它踢出去…，
如此不停的进行下去，直到最后只剩下一只猴子为止，
那只猴子就叫做大王。要求编程模拟此过程，
输入m、n, 输出最后那个大王的编号。
*/
function king($n, $m)
{
    // 创建1到n数组
    $monkeys = range(1, $n);
    $i = 0;
    // 循环条件为猴子数量大于1
    while (count($monkeys) > 1) {
        // $i为数组下标;$i+1为猴子标号
        if (($i + 1) % $m == 0) {
            // 余数等于0表示正好第m个，删除，用unset删除保持下标关系
            unset($monkeys[$i]);
        } else {
            // 如果余数不等于0，则把数组下标为$i的放最后，形成一个圆形结构
            array_push($monkeys, $monkeys[$i]);
            unset($monkeys[$i]);
        }
        // $i 循环+1，不断把猴子删除，或 push到数组
        $i++;
    }
    // 猴子数量等于1时输出猴子标号，得出猴王
    return current($monkeys);
}

echo king(6, 3);

// 2、有一母牛，到4岁可生育，每年一头，所生均是一样的母牛，到15岁绝育，不再能生，20岁死亡，问n年后有多少头牛。
function niu($y)
{
    // 定义静态变量;初始化牛的数量为1
    static $num = 1;
    for ($i = 1; $i <= $y; $i++) {
        // 每年递增来算，4岁开始+1，15岁不能生育
        if ($i >= 4 && $i < 15) {
            $num++;
            // 递归方法计算小牛$num，小牛生长年数为$y-$i
            niu($y - $i);
        } else if ($i == 20) {
            // 20岁死亡减一
            $num--;
        }
        return $num;
    }
}

/*3、杨辉三角*/

/* 默认输出十行，用T(值)的形式可改变输出行数 */

class T
{
    private $num;

    public function __construct($var = 10)
    {
        if ($var < 3) die("值太小啦！");
        $this->num = $var;
    }

    public function display()
    {
        $n = $this->num;
        $arr = array();
        // $arr=array_fill(0,$n+1,array_fill(0,$n+1,0));
        $arr[1] = array_fill(0, 3, 0);
        $arr[1][1] = 1;
        echo str_pad(" ", $n * 12, " ");
        printf("%3d", $arr[1][1]);
        echo "<br/>";
        for ($i = 2; $i <= $n; $i++) {
            $arr[$i] = array_fill(0, ($i + 2), 0);
            for ($j = 1; $j <= $i; $j++) {
                if ($j == 1)
                    echo str_pad(" ", ($n + 1 - $i) * 12, " ");
                printf("%3d", $arr[$i][$j] = $arr[$i - 1][$j - 1] + $arr[$i - 1][$j]);
                echo " ";
            }
            echo "<br/>";
        }
    }
}

$yh = new T('3'); // $yh=new T(数量);
$yh->display();

// 4.冒泡排序【核心思想：将每个元素比较一次】
function pop($arr = [])
{
    $len = count($arr);
    for ($j = 0; $j < $len; $j++) {
        for ($k = $len - 1; $k > $j; $k--) {
            if ($arr[$k] < $arr[$j]) {
                [$arr[$j], $arr[$k]] = [$arr[$k], $arr[$j]];
            }
        }
    }
    return $arr;
}
var_export(pop([8, 4, 6, 1, 2, 3, 9, 5, 7]));
/**
array (
    0 => 1,
    1 => 2,
    2 => 3,
    3 => 4,
    4 => 5,
    5 => 6,
    6 => 7,
    7 => 8,
    8 => 9,
)
 */

// 5.快速排序
function quickSort($arr)
{
    // 先判断是否需要继续进行
    $length = count($arr);
    if ($length <= 1) {
        return $arr;
    }
    // 选择第一个元素作为基准
    $base_num = $arr[0];
    // 遍历除了标尺外的所有元素，按照大小关系放入两个数组内
    // 初始化两个数组
    // 小于基准的
    $left_array = [];
    // 大于基准的
    $right_array = [];
    for ($i = 1; $i < $length; $i++) {
        if ($base_num > $arr[$i]) {
            // 放入左边数组
            $left_array[] = $arr[$i];
        } else {
            // 放入右边
            $right_array[] = $arr[$i];
        }
    }
    // 再分别对左边和右边的数组进行相同的排序处理方式递归调用这个函数
    $left_array = quickSort($left_array);
    $right_array = quickSort($right_array);
    // 合并
    return array_merge($left_array, array($base_num), $right_array);
}

// 6.二分查找算法（折半查找算法）
function binarySearch($x, $arr = [])
{
    $len = count($arr);
    $min = 0;
    $max = $len - 1;
    while ($min <= $max) {
        $middle = intval(($min + $max) / 2);
        if ($arr[$middle] > $x) {
            $max = --$middle;
        } else if ($arr[$middle] < $x) {
            $min = ++$middle;
        } else {
            return $middle;
        }
    }
    return false;
}
echo binarySearch(5, range(1, 9)); // 4

// 7.PHP奇异算法(PHP7以下的版本返回的是 6，PHP7版本返回5 ，还真的算奇异，个人底层算法差，认为是PHP7以下版本的BUG)
function test()
{
    $a = 1;
    $b =& $a;
    echo (++$a) + (++$b);
}
test(); // 5

// 8.字符集合：输入一个字符串，求出该字符串包含的字符集合，并按顺序排序（英文）
function set($str)
{
    // 转化为数组
    $arr = str_split($str);
    // 去除重复
    $arr = array_flip(array_flip($arr));
    // 排序
    sort($arr);
    // 返回字符串
    return implode('', $arr);
}

// 9.遍历一个文件下的所有文件和子文件夹下的文件
function AllFile($dir)
{
    if ($dh = opendir($dir)) {
        while (($file = readdir($dh)) !== false) {
            if ($file != '..' && $file != '.') {
                if (is_dir($dir . '/' . $file)) {
                    // 如果判断还是文件，则递归
                    AllFile($dir . '/' . $file);
                } else {
                    echo $file;     // 输出文件名
                }
            }
        }
    }
}

// 10.从一个标准的Url提取出文件的扩展名
function getExt($url)
{
    $arr = parse_url($url);
    // basename函数返回路径中的文件名部分
    $file = basename($arr['path']);
    $ext = explode('.', $file);
    return $ext[count($ext) - 1];
}

/*
11.有个人想上一个n级的台阶，每次只能迈1级或者迈2级台阶，
问：这个人有多少种方法可以把台阶走完？
例如：总共3级台阶，可以先迈1级再迈2级，或者先迈2级再迈1级，
或者迈3次1级总共3中方式
 */
function jieTi($num)
{
    // 实际上是斐波那契数列
    return $num < 2 ? 1 : jieTi($num - 1) + jieTi($num - 2);
}

// 12.请写一段PHP代码，确保多个进程同时写入同一个文件成功
$fp = fopen("lock.txt", "w+");
if (flock($fp, LOCK_EX)) {
    //获得写锁，写数据
    fwrite($fp, "write something");
    // 解除锁定
    flock($fp, LOCK_UN);
} else {
    echo "file is locking...";
}
fclose($fp);

// 13.无限级分类
function tree($arr, $pid = 0, $level = 0)
{
    static $list = array();
    foreach ($arr as $v) {
        // 如果是顶级分类，则将其存到$list中，并以此节点为根节点，
        // 遍历其子节点
        if ($v['pid'] == $pid) {
            $v['level'] = $level;
            $list[] = $v;
            tree($arr, $v['id'], $level + 1);
        }
    }
    return $list;
}

// 14.获取上个月第一天 和 最后一天
// 获取上个月第一天
date('Y-m-01', strtotime('-1 month'));
// 获取上个月最后一天
date('Y-m-t', strtotime('-1 month'));

// 15.随机输入一个数字能查询到对应的数据区间,把区间换成数组写法，
// 用二分法查找区间
function binSearch($target, $arr)
{
    $c = count($arr);
    $lower = 0;
    $high = $c - 1;
    while ($lower <= $high) {
        $middle = intval(($lower + $high) / 2);
        if ($arr[$middle] >= $target) {
            $high = $middle - 1;
        } elseif ($arr[$middle] <= $target) {
            $lower = $middle + 1;
        }
    }

    return '在区间' . $arr[$high] . '到' . $arr[$lower];
}
$array = ['1', '50', '100', '150', '200', '250', '300'];
echo binSearch('120', $array);
```
# 拖拽排序 
## array_splice写法
 * 思路: 以索引位置取排序值，以排序后的索引位置取排序前索引位置对应的权重
 * 优点: 逻辑相对简单
 * 缺点: 每个都需要更新一次
```php
/**
 * @param  string  $ids  排序前的ID数组
 * @param  int  $oldIndex  原索引
 * @param  int  $newIndex  新索引
 * @return void
 */
function dragSort(string $ids, int $oldIndex, int $newIndex)
{
    // 查询指定ID集合，并按权重降序
    $link = mysqli_connect('127.0.0.1', 'root', '123456', 'basic', '3306');
    $ret = mysqli_query($link, "SELECT `id`, `weigh` FROM `article` WHERE `id` IN ($ids)ORDER BY `weigh` DESC;");
    // 以ID为键值的二维数组
    $list = array_column(mysqli_fetch_all($ret, MYSQLI_ASSOC), null, 'id');
    // ID、权重集合
    $idArr = $weighArr = [];
    foreach ($list as $item) {
        $idArr[] = $item['id'];
        $weighArr[] = $item['weigh'];
    }
    // 记录要拖动的ID
    $oldId = $idArr[$oldIndex];
    // 从排序前ID集合中，删除要拖动的ID
    unset($idArr[$oldIndex]);
    // 插在新索引前面
    array_splice($idArr, $newIndex, 0, $oldId);
    // 重新设置排序
    for ($i = 0; $i < count($idArr); $i++) {
        // 更新每个元素的权重
        $list[$idArr[$i]]['weigh'] = $weighArr[$i];
    }
    // 保存到数据库省略
    $sql = "`weigh` = CASE ";
    foreach ($list as $v) {
        $sql .= "WHEN `id` = {$v['id']} THEN {$v['weigh']} ";
    }
    $sql .= 'ELSE `weigh` END';
    $sql = "UPDATE `article` SET $sql where `id` in(${ids})";
    // 更新数据的权重
    mysqli_query($link, $sql);
    // 查询重新排序后的数据
    var_export(mysqli_fetch_all(mysqli_query($link, "SELECT `id`, `weigh` FROM `article`")));
}
dragSort('1,2,3,4,5', 3, 1);
```
## Laravel写法
 * 思路: 以索引位置取排序值，以排序后的索引位置取排序前索引位置对应的权重
 * 优点: 逻辑相对简单
 * 缺点: 每个都需要更新一次
```php
/**
 * @param  array  $ids 排序后的ID集合
 */
public static function sort(array $ids)
{
    $list = ArticleModel::query()
        ->whereIn(ArticleModel::ID, $ids)
        ->orderBy(ArticleModel::WEIGH, 'desc')
        ->select([ArticleModel::ID, ArticleModel::WEIGH])
        ->get()
        ->keyBy(ArticleModel::ID)
        ->toArray();
    // 权重
    $weighArr = array_column($list, ArticleModel::WEIGH);
    // 重新设置排序
    for ($i = 0; $i < count($ids); $i++) {
        $list[$ids[$i]][ArticleModel::WEIGH] = $weighArr[$i];
    }
    success(Batch::updateBatch(app(ArticleModel::class)->getTable(), $list));
}
```
## 取差集写法 
 * 思路: 取差集, 找出改变过的数据, 根据往前或往后进行索引位置计算
 *  1.往前移动, 被改变数据往后移动一位
 *  2.往后移动, 被改变数据往前移动一位
 * 优点: 更新次数少, 只需要更新改变过的数据
 * 缺点: 逻辑稍微有些不好理解
```php
/**  
 * @param  array  $newIds  修改后的ID数组  
 * @param  int  $changeId  改变的ID  
 */function weigh(array $newIds, int $changeId)  
{  
    // 修改前的ID数组  
    $oldIds = [1, 2, 3, 4, 5];  
    // 修改前的ID、权重关联数组  
    $oldIdWeighMap = [1 => 1, 2 => 2, 3 => 3, 4 => 4, 5 => 5];  
    // 修改后的ID、权重关联数组  
    $newIdWeighMap = [];  
    // 找到新位置对应的原来的ID，然后获取到对应的权重  
    $oldId = $oldIds[array_search($changeId, $newIds)];  
    // 带索引检查计算数组的差集，找出顺序改变过的ID数组  
    $changeIds = array_diff_assoc($newIds, $oldIds);  
    foreach ($changeIds as $index => $id) {  
        if ($id == $changeId) {  
            $offset = $oldId;  
        } else {  
            if ($changeId == reset($changeIds)) {  
                // 元素往右移动  
                $offset = $changeIds[++$index] ?? $changeId;  
            } else {  
                // 元素往左移动  
                $offset = $changeIds[--$index] ?? $changeId;  
            }  
        }  
        $newIdWeighMap[$id] = $oldIdWeighMap[$offset];  
    }  
    var_export($newIdWeighMap);  
}  
weigh([1, 3, 4, 2, 5], 2);
```
## 不用函数写法
 * 思路: 根据往前或往后进行索引位置计算
 *  1.根据往前或往后进行索引位置计算
 *  2.将旧值放置在新位置
 * 优点: 更新次数少, 只需要更新改变过的数据
 * 缺点: 逻辑稍微有些不好理解
```php
/**  
 * @param array $oldArr   修改前的数组  
 * @param int   $oldIndex 修改前的索引  
 * @param int   $newIndex 修改后的索引  
 */  
$oldArr = [1, 2, 3, 4, 5];  
function draw($oldArr, $oldIndex, $newIndex)  
{  
    $oldValue = $oldArr[$oldIndex];  
    // 确保索引在有效范围内  
    if (!(0 <= $oldIndex && $oldIndex < count($oldArr)) || !(0 <= $newIndex && $newIndex < count($oldArr))) {  
        throw new OutOfRangeException("Index out of bounds");  
    }  
    if ($oldIndex < $newIndex) {  
        // 元素右移  
        for ($i = $oldIndex; $i < $newIndex; $i++) {  
            $oldArr[$i] = $oldArr[$i + 1];  
        }  
    } else {  
        // 元素左移  
        for ($i = $oldIndex; $i > $newIndex; $i--) {  
            $oldArr[$i] = $oldArr[$i - 1];  
        }  
    }  
    $oldArr[$newIndex] = $oldValue;  
  
    return $oldArr;  
}  
var_export(draw($oldArr, 0, 4));  
/*  
array (
  0 => 2,
  1 => 3,
  2 => 4,
  3 => 5,
  4 => 1,
)
*/
```
# 树型结构
```php
public static function tree(&$arr, $pid = 0)
{
    $tree = [];
    foreach ($arr as $k => $v) {
        if ($v['pid'] == $pid) {
            $v['child'] = self::tree($arr, $v['id']);
            $tree[] = $v;
            unset($arr[$k]);
        }
    }
    return $tree;
}
```
# 计算二维数组的交叉集
(如果一个数组中某个key是唯一的，则通过key操作不适用 in_array())
```php
$arr1 = [
    ['a' => '1', 'b' => 'a'],
    ['a' => '2', 'b' => 'b'],
];
$arr2 = [
    ['a' => '1', 'b' => 'a'],
    ['a' => '5', 'b' => 'c']
];

// 差集
$r = array_filter($arr1, function ($v) use ($arr2) {
    return !in_array($v, $arr2);
});
var_export($r);
/*
array (
  1 =>
  array (
    'a' => '2',
    'b' => 'b',
  ),
)
*/

// 交集
$r = array_filter($arr1, function ($v) use ($arr2) {
    return in_array($v, $arr2);
});
var_export($r);
/*
array (
  0 =>
  array (
    'a' => '1',
    'b' => 'a',
  ),
)
*/
```
# 二维数组去重
## 某一键名的值不能重复，删除重复项
```php
function assoc_unique(&$arr, $key)
{
    $tmp= [];
    foreach ($arr as $k => $v) {
        if (in_array($v[$k], $tmp)) {
            unset($arr[$k]);
        } else {
            $tmp[] = $v[$k];
        }
    }
    sort($arr);
}
$arr = [
    ['id' => 1, 'name' => 'riven'],
    ['id' => 2, 'name' => 'loren'],
    ['id' => 2, 'name' => 'gamer'],
    ['id' => 3, 'name' => 'kris']
];
assoc_unique($arr, 'id');
var_export($arr);

/*
array (
  0 => 
  array (
    'id' => 1,
    'name' => 'riven',
  ),
  1 => 
  array (
    'id' => 2,
    'name' => 'loren',
  ),
  2 => 
  array (
    'id' => 3,
    'name' => 'kris',
  ),
)
*/
```
## 内部的一维数组不能完全相同，而删除重复项
注: 
1. 当去重对象时会报错：Nesting level too deep - recursive dependency?
2. 当去重数组时会报错：Warning: Array to string conversion

- SORT_REGULAR - 按照通常方法比较（不修改类型）
- SORT_NUMERIC - 按照数字形式比较
- SORT_STRING - 按照字符串形式比较
- SORT_LOCALE_STRING - 根据当前的本地化设置，按照字符串比较。
```php
# array_unique 去重
$arr = [
    ['id' => 1, 'name' => 'riven'],
    ['id' => 2, 'name' => 'loren'],
    ['id' => 2, 'name' => 'loren'],
    ['id' => 4, 'name' => 'kris']
];
var_export(array_unique($arr, SORT_REGULAR));

/*
array (
  0 => 
  array (
    'id' => 1,
    'name' => 'riven',
  ),
  1 => 
  array (
    'id' => 2,
    'name' => 'loren',
  ),
  3 => 
  array (
    'id' => 4,
    'name' => 'kris',
  ),
)
*/

# serialize 去重
var_export(array_map('unserialize', array_unique(array_map('serialize', $arr))));
/*
array (
  0 =>
  array (
    'id' => 1,
    'name' => 'riven',
  ),
  1 =>
  array (
    'id' => 2,
    'name' => 'loren',
  ),
  3 =>
  array (
    'id' => 4,
    'name' => 'kris',
  ),
)
*/
```
# 进制转换
自增主键,看起来像是随机字母+数字
>慎用：15位数字以上会有问题
```php
// 十进制转62进制以内的任意进制「最高支持62位」  
function systemConvert($int, $format = 58): string  
{  
    $dic = [  
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',  
        'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j',  
        'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't',  
        'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D',  
        'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N',  
        'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X',  
        'Y', 'Z'  
    ];  
    $arr = [];  
    $loop = true;  
    while ($loop) {  
        $int = scToNum($int);  
        $arr[] = $dic[bcmod($int, $format)];  
        $int   = floor(bcdiv($int, $format));  
        if ($int == 0) {  
            $loop = false;  
        }  
    }  
    return implode('', array_reverse($arr));  
}  
  
function scToNum($num, $double = 0){  
    if(false !== stripos($num, "e")){  
        $a = explode("e",strtolower($num));  
        return bcmul($a[0], bcpow(10, $a[1], $double), $double);  
    }else{  
        return $num;  
    }  
}  
echo systemConvert('999999999999999999');
```
# 随机字符串
```php
function uuid($lenght = 24)
{
    // uniqid gives 13 chars, but you could adjust it to your needs.
    if (function_exists("random_bytes")) {
        $bytes = random_bytes(ceil($lenght / 2));
    } elseif (function_exists("openssl_random_pseudo_bytes")) {
        $bytes = openssl_random_pseudo_bytes(ceil($lenght / 2));
    } else {
        throw new \Exception("no cryptographically secure random function available");
    }
    return substr(bin2hex($bytes), 0, $lenght);
}
uuid(); // 0adad0e79c314bf621c80cee
```
# 计算两个日期相差天数
```php
echo (new DateTime('2009-10-11'))->diff(new DateTime('2009-10-13'))->format('%a'); // 2
```
# 判断当前时间(时分)是否在指定的时间区间的函数
```php
function time_section($start, $finish): bool  
{  
    $now   = date('Y-m-d', time());  
    $begin = strtotime("$now $start");  
    $end   = strtotime("$now $finish");  
    $time  = time();  
    if ($begin <= $time && $time <= $end) {  
        return true;  
    } else {  
        return false;  
    }  
}  
var_export(time_section('9:00:00', '15:58:30'));
```
# 计算俩个时间的时间差, 不包含某个时间段
[如何计算俩个时间的时间差？其中不含包含某个时间段 | Laravel | Laravel China 社区](https://learnku.com/laravel/t/78309?order_by=vote_count&#reply286088)

<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">




==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Excalidraw Data

## Text Elements
9 
20 
0 
24 
9 
20 
0 
24 
9 
20 
0 
24 


</div></div>

```php
/**  
 * 计算有效时间  
 * @param string|int $startTime  @comment 开始时间  
 * @param string|int $endTime    @comment 结束时间  
 * @param int        $validStart @comment 有效开始时间  
 * @param int        $validEnd   @comment 有效结束时间  
 * @param int        $interval  
 * @return float|int|mixed  
 */  
function calculateTime(string|int $startTime, string|int $endTime = '', int $validStart = 9, int $validEnd = 20, int $interval = 0): mixed  
{  
    $startTime = is_numeric($startTime) ? $startTime : strtotime($startTime);  
    if ($endTime) {  
        $endTime = is_numeric($endTime) ? $startTime : strtotime($endTime);  
    } else {  
        $endTime = time();  
    }  
    // 开始时间 第一个时间点 时间戳  
    $startFirstValidTime = strtotime(date("Y-m-d $validStart:00:00", $startTime));  
    // 开始时间 第二个时间点 时间戳  
    $startSecondValidTime = strtotime(date("Y-m-d $validEnd:00:00", $startTime));  
    // 开始时间 第三个时间点 时间戳  
    $startThirdValidTime = strtotime(date("Y-m-d $interval:00:00", $startTime));  
    // 结束时间 第一个时间点 时间戳  
    $endFirstValidTime = strtotime(date("Y-m-d $validStart:00:00", $endTime));  
    // 结束时间 第二个时间点 时间戳  
    $endSecondValidTime = strtotime(date("Y-m-d $validEnd:00:00", $endTime));  
    // 结束时间 第三个时间点 时间戳  
    $endThirdValidTime = strtotime(date("Y-m-d $interval:00:00", $endTime));  
    $begin             = max($startTime, $startFirstValidTime);  
    $begin             = min($begin, $startSecondValidTime);  
    $now               = max($endTime, $endFirstValidTime);  
    $now               = min($now, $endSecondValidTime);  
    // 实际相差天数  
    $diffDays = intval(floor(($endThirdValidTime - $startThirdValidTime) / 86400));  
  
    return $now - $begin - (($validStart + 24 - $validEnd) * 3600 * $diffDays);  
}
echo calculateTime('2024-01-16 19:00:00', '2024-01-17 01:00:00');
```
# 雪花算法「yii」

[雪花算法原理介绍及基于php的雪花算法(snowflake) - 温柔的风 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wt645631686/p/13173602.html)

```javascript
<?php
namespace common\components;

use Yii;
use yii\redis\Connection;
use yii\base\Component;

/**
 * 雪花算法
 * Class SnowFlake
 * @package common\components
 */
class SnowFlake extends Component
{

    /**
     * 符号标识位长度
     */
    const SIGN_BITS = 1;

    /**
     * 毫秒时间戳长度
     */
    const TIMESTAMP_BITS = 41;

    /**
     * 数据中心序号长度
     */
    const DATACENTER_BITS = 5;

    /**
     * 机器序号长度
     */
    const MACHINE_ID_BITS = 5;

    /**
     * 当前实例同一毫秒内的自增序号长度
     */
    const SEQUENCE_BITS = 12;

    /**
     * 偏移起始时间戳
     */
    public $epochOffset;

    /**
     * @var int 当前实例 ID
     */
    public $instanceId;

    /**
     * @var int 最大实例 ID
     */
    public $maxInstanceId;

    /**
     * @var string 自增的实例 ID 的 KEY
     */
    public $instanceIdKey;

    /**
     * @var string 实例 ID 集合的 KEY
     */
    public $instanceSetKey;

    /**
     * @var string 实例 ID 缓存的 KEY
     */
    public $instanceCacheKey;

    /**
     * @var int 实例 ID 缓存的过期时间，秒级别
     */
    public $instanceExpire;

    /**
     * @var int 上一次取值的时间戳
     */
    public $lastTimestamp;

    /**
     * @var int 当前实例同一毫秒内的自增序号
     */
    public $sequence;

    /**
     * @var int 自增序号的最大值
     */
    public $maxSequence;

    /**
     * @var bool 自增序号是否是通过 Redis 自增
     */
    public $sequenceRedisIncr = false;

    /**
     * @var string 自增序号的 KEY
     */
    public $sequenceKey;

    /**
     * @var int 自增序号的过期时间
     */
    public $sequenceExpire;

    /**
     * @var string Redis 组件名称
     */
    public $redisComponent;

    /**
     * @var Connection
     */
    public $redis;

    /**
     * @var bool 是否已完成初始化
     */
    public $inited = false;

    /**
     * @throws \Exception
     */
    public function initInstance()
    {
        $this->sequenceKey = 'snow_flake:sequence';
        $this->redis = Yii::$app->get($this->redisComponent);
        $this->maxInstanceId = -1 ^ (-1 << (self::DATACENTER_BITS + self::MACHINE_ID_BITS));
        $this->instanceId = $this->generateInstanceId();
        $this->sequence = 1;
        if($this->sequenceRedisIncr){
            $this->maxSequence = -1 ^ (-1 << (self::DATACENTER_BITS + self::MACHINE_ID_BITS + self::SEQUENCE_BITS));
        }else{
            $this->maxSequence = -1 ^ (-1 << (self::SEQUENCE_BITS));
        }
        $this->lastTimestamp = 0;
        $this->inited = true;
    }

    /**
     * 生成一个实例 ID
     * @return int
     * @throws \Exception
     */
    public function generateInstanceId()
    {
        if($this->sequenceRedisIncr){
            return 1;
        }
        $instance_id = null;
        while(true){
            $instance_id = $this->redis->incr($this->instanceIdKey);
            if(!is_numeric($instance_id)){
                usleep(1000);
                continue;
            }
            if($instance_id > $this->maxInstanceId){
                $instance_id = '1';
                if(!$this->redis->set($this->instanceIdKey, $instance_id)){
                    usleep(1000);
                    continue;
                }
            }
            if($this->redis->scard($this->instanceSetKey) >= $this->maxInstanceId){
                $members = $this->redis->smembers($this->instanceSetKey);
                if(!is_array($members)){
                    usleep(1000);
                    continue;
                }
                foreach($members as $member){
                    if($this->redis->get($this->instanceCacheKey.':'.$member) === null){
                        $this->redis->srem($this->instanceSetKey, $member);
                    }
                }
                if($this->redis->scard($this->instanceSetKey) >= $this->maxInstanceId){
                    throw new \Exception('实例数量超过上限');
                }
            }
            if($this->redis->sadd($this->instanceSetKey, $instance_id) && $this->redis->setex($this->instanceCacheKey.':'.$instance_id, $this->instanceExpire, $instance_id)){
                break;
            }
        }
        return intval($instance_id);
    }

    /**
     * 返回当前毫秒时间戳
     * @return int
     */
    public function getTimestamp()
    {
        return floor(microtime(true) * 1000);
    }

    /**
     * 返回执行了偏移的时间戳
     * @param int|null $time
     * @return int
     */
    public function getMockTimestamp($time = null)
    {
        if($time === null){
            $time = $this->getTimestamp();
        }
        return $time - $this->epochOffset;
    }

    /**
     * 实例的心跳
     * @param int $timestamp 毫秒时间戳
     * @throws \Exception
     */
    public function heartbeat($timestamp)
    {
        if($this->sequenceRedisIncr){
            return;
        }
        $diff_time = ($timestamp - $this->lastTimestamp) / 1000;
        if($diff_time >= ($this->instanceExpire - 10) && $diff_time <= ($this->instanceExpire - 5)){
            $this->redis->setex($this->instanceCacheKey.':'.$this->instanceId, $this->instanceExpire, $this->instanceId);
        }elseif($diff_time > ($this->instanceExpire - 5)){
            $this->instanceId = $this->generateInstanceId();
            $this->lastTimestamp = 0;
        }
    }

    /**
     * 生成自增序号
     * @param int $timestamp
     */
    public function generateSequence(&$timestamp)
    {
        if($this->sequenceRedisIncr){
            $key = $this->sequenceKey.':'.floor($timestamp / 1000);
            $sequence = null;
            while(true){
                $sequence = $this->redis->incr($key);
                if(!is_numeric($sequence)){
                    usleep(1000);
                    continue;
                }
                if($sequence > $this->maxSequence){
                    $timestamp++;
                    $sequence = 1;
                    if(!$this->redis->set($key, $sequence)){
                        usleep(1000);
                        continue;
                    }
                }
                break;
            }
            $this->redis->expire($key, $this->sequenceExpire);
            $this->sequence = $sequence;
        }else{
            if($this->lastTimestamp && $timestamp <= $this->lastTimestamp){
                $this->sequence++;
                if($this->sequence >= $this->maxSequence){
                    $this->sequence = 1;
                    $this->lastTimestamp = ++$timestamp;
                }
            }else{
                $this->sequence = 1;
            }
        }
    }

    /**
     * 生成 ID
     * @return int
     * @throws \Exception
     */
    public function generateId()
    {
        if(!$this->inited){
            $this->initInstance();
        }
        $timestamp = $this->getTimestamp();
        $this->heartbeat($timestamp);
        $this->generateSequence($timestamp);
        $this->lastTimestamp = $timestamp;
        $timestamp_bin = str_pad(decbin($this->getMockTimestamp($timestamp)), self::SIGN_BITS + self::TIMESTAMP_BITS, '0', STR_PAD_LEFT);
        if($this->sequenceRedisIncr){
            $machine_id_bin = '';
        }else{
            $machine_id_bin = str_pad(decbin($this->instanceId), self::DATACENTER_BITS + self::MACHINE_ID_BITS, '0', STR_PAD_LEFT);
        }
        if($this->sequenceRedisIncr){
            $sequence_bin = str_pad(decbin($this->sequence), self::DATACENTER_BITS + self::MACHINE_ID_BITS + self::SEQUENCE_BITS, '0', STR_PAD_LEFT);
        }else{
            $sequence_bin = str_pad(decbin($this->sequence), self::SEQUENCE_BITS, '0', STR_PAD_LEFT);
        }
        $id_bin = $timestamp_bin.$machine_id_bin.$sequence_bin;
        return bindec($id_bin);
    }

}
```
# 获取eth0的IP地址
```php
/**
 * 获取eth0的IP地址
 * @return string|null
 */
function getEth0Ip(): ?string
{
    // 你提供的命令
    $command = "ip -4 addr show eth0";
    // 执行命令并获取输出
    $pattern = '/inet\s(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})\//';
    $matches = [];
    if (preg_match($pattern, shell_exec($command), $matches)) {
        return $matches[1];
    }

    return '';
}

```