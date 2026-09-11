---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Algorithm/","title":"Algorithm","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T17:39:26.000+08:00","updated":"2026-03-24T17:47:09.000+08:00","dg-note-properties":{"title":"Algorithm","tags":["flashcards"],"reference linking":null}}
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
<div class="excalidraw-svg"><svg version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1286.3446406070886 488.4294632177034" width="1286.3446406070886" height="488.4294632177034" class="excalidraw-svg" style="max-width: 100%; height: auto;"><!-- svg-source:excalidraw --><metadata/><defs><style class="style-fonts">      @font-face { font-family: Virgil; src: url(data:font/woff2;base64,d09GMgABAAAAAATAAAsAAAAAB3wAAARyAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAABmAATBEICoc0hWsLDAABNgIkAxQEIAWDHAcgG+gFUZQtTp7sR0J1ewWzk92iuqxzk+FF1cNDne8jqB/77btvgmZNhEK0rslDJapFsQiRkLR0hkqEpo32xYff33vufX9k0p387TSZjWntx8FJaQtFjmxq4PDvme8VPic5SsUA/MHttq9080t6Qq9nDUDaDMLX7Mff5LOvX+CMPkr62hh4uPw8oMjzMX2LhlkYYKCQbtMJWU8gdESMrhDp6O3rspEHD+JDnMYMAJOqVYNzEOGfYJfnAAQOJK12adZqhR6fIrPhHwn4fQHcAKBjLgdIr5wGYAABAUn0wiFV88jfKHNEUXxCYop2G95pv7Vvtm+0r4AcKeCEXj0HKIMCpFEgrQOALIBAEbTfv9RiRrh32RpWpaMoKVBLr0i1iltGOcPIzyrpviGCaDGxrMJof8oYy6qGTSl9lyVkHApDtAOj6p+O1bK4FnazqtqGKYMPqR/Rd9hVYUgjEtFlPqafcuhLPTFdeEGq9b/kf0vexMiptjaqUzROtVk4uxqr9kM+XQkmOarqwNCEj15CJhhook/hPe6o01E49KUl5EW7xYTEFEDXoE5TCxtVqu7oTrsqxg5V4ZfHqfhzUm2udlyq09nq2EbWsXbau73XjkJVWA9j1+ms7pEgpJfUAVRdjav99BAZiACarqFrIdQs1xD0VUzPXPQZGwug4XE29QyrGjf6+1PVdUBtD9RVoI5xQggRZFcD0cKyi86pdKyt7kQ3pA2GEQkj79lZEN+YLU3f/RD6iviL/OTp6SbtD33yVARNjrcu4fmkF91lby9Rz8KZdmXBw9Z/wuV/rmmxe5v8zc0/wMucM8VeiRM/q+/oNUkIzL4xPtoUrhSM9HG3JA1JMry3hHVbdh3KFtuuNTaEIN4kkmWm68X73qEVHm8VSqZ4FOutrdoeluVvvL1sy62aOpdhqct2qe9LcnNYjG5ZbVufrs4SzL99+a5vBN7GfReTvvzO0vKeqcZs3D0JV3aesA+7ADv/zn/OYCdLmwrsHc05VzNVFsvm4QHyRcWsL/5/czRabtMXkPQjHCGWNoiFVNyzyrH1crPOGsHxhHmOR8zknv52i8aq5bs1NEWYx8064iwO/NXwUMscT0c+N4gHDq545UMPNUJl5O9Qef6cAtpwsyPPyFkFnjkrnKeG9hYIRYIbWs4ayMGeuaYWpbnaPHpO8Zdazcfw3xd6GdIFG9lpxe4rVzA4nA5HpNYBAAlk4Jb+FDvfIulLVOG+BYD3lbXHAPBx/7n1/+Lmr+wXbx9AEQUk+Mgw53VQ7VklANLluZQNEA/C4iLIDDVpLATBCMi5J4DvIeQImIChgkIckhUDOAEnyYSrXabcrTAnVzrz+FOynSABsDcgUtugU50GzRJkqkHqWVShU64anbo0aNOK4eMTSDRlqhqRAmmsrPFAFSmiysYqQvyRVLFKJqN8SaaWmLRpN6BTgzr1ujHRqsRgslRAgKk0gCnQpkcnJk2fVoz1Qa97oYeoPj0YkIgxaNaMuep0baiZVINey1VLxHFTDgAAAA==); }</style></defs><rect x="0" y="0" width="1286.3446406070886" height="488.4294632177034" fill="#ffffff"/><g stroke-linecap="round"><g transform="translate(62.02944625051407 59.62582236842127) rotate(0 61.34218546999131 -0.15748969938840673)"><path d="M-1 -1.16 C19.69 -1.53, 103.17 -1.21, 123.69 -1.07 M0.67 0.85 C21.28 1.19, 102.35 0.99, 122.97 0.48" stroke="#e03131" stroke-width="1" fill="none"/></g></g><mask/><g transform="translate(214.29918309261944 13.626644736842536) rotate(0 6.089996337890625 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">9</text></g><g transform="translate(449.23750546104037 10) rotate(0 13.999992370605469 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">20</text></g><g stroke-linecap="round"><g transform="translate(30.33947914525089 36.31990131578959) rotate(0 303.91668550451874 -3.1559265459839025)"><path d="M1.17 -0.19 C101.88 -1.35, 504.46 -5.35, 605.12 -6.12 M0.32 -1.33 C101.37 -2.38, 506.9 -3.78, 607.51 -4.66" stroke="#1e1e1e" stroke-width="1" fill="none"/></g></g><mask/><g transform="translate(16.47434756630355 18.293585526315837) rotate(0 6.879997253417969 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">0</text></g><g transform="translate(641.1166173031456 14.975328947368325) rotate(0 13.519989013671875 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">24</text></g><g stroke-linecap="round"><g transform="translate(282.3443331489439 65.14515417307211) rotate(0 62.22220601828748 -1.0854199771802087)"><path d="M0.98 0.52 C21.62 0.44, 104.08 -0.34, 124.41 -0.75 M0.04 -0.25 C20.57 -0.68, 103.73 -2.56, 124.06 -2.69" stroke="#e03131" stroke-width="1" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(496.7440042015754 68.6155489099142) rotate(0 61.39868119609987 -0.40735148599242166)"><path d="M-1.17 0.81 C19.49 0.34, 103.04 -1.4, 123.96 -1.62 M0.42 0.18 C20.98 0.38, 102.76 0.09, 123.39 -0.36" stroke="#e03131" stroke-width="1" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(176.5260765699964 127.25864101517845) rotate(0 61.82024659769587 -0.5264859478720609)"><path d="M0.52 0.36 C21.32 -0.03, 103.83 -1.21, 124.3 -1.42 M-0.66 -0.49 C20.11 -0.79, 103.36 -0.22, 123.9 -0.05" stroke="#e03131" stroke-width="1" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(409.8114384121017 126.45271996254678) rotate(0 62.29854362519437 -1.1108211903492702)"><path d="M1.13 0.68 C21.84 0.51, 104.15 -0.75, 124.33 -0.88 M0.27 -0.01 C20.9 -0.58, 103.23 -2.57, 123.94 -2.9" stroke="#e03131" stroke-width="1" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(147.29087920157536 203.36472654149384) rotate(0 209.21749028743372 -1.5832905459467383)"><path d="M-0.08 0.32 C70.07 0.11, 350.09 -1.41, 420.01 -1.84 M-1.58 -0.56 C68.58 -1.05, 349.51 -2.86, 419.59 -3.49" stroke="#e03131" stroke-width="1" fill="none"/></g></g><mask/><g transform="translate(207.8248355263159 358.3696546052638) rotate(0 6.089996337890625 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">9</text></g><g transform="translate(442.7631578947368 354.74300986842127) rotate(0 13.999992370605469 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">20</text></g><g stroke-linecap="round"><g transform="translate(23.865131578947512 381.0629111842113) rotate(0 302.5590112610267 -3.464389451867646)"><path d="M0.82 0 C101.74 -1.1, 504.32 -5.74, 605.33 -6.93 M-0.21 -1.04 C100.53 -1.99, 503.03 -4.99, 604.18 -5.89" stroke="#1e1e1e" stroke-width="1" fill="none"/></g></g><mask/><g transform="translate(10 363.0365953947371) rotate(0 6.879997253417969 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">0</text></g><g transform="translate(634.6422697368421 359.7183388157896) rotate(0 13.519989013671875 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">24</text></g><g transform="translate(822.4872283692188 380.1411109449766) rotate(0 6.089996337890625 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">9</text></g><g transform="translate(1057.4255507376397 376.5144662081341) rotate(0 13.999992370605469 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">20</text></g><g stroke-linecap="round"><g transform="translate(638.5275244218503 402.83436752392413) rotate(0 302.4403787785455 -3.287253294084053)"><path d="M-0.71 1.14 C100.06 0.23, 504.39 -4.74, 605.59 -5.73 M1.12 0.69 C101.63 -0.57, 503.73 -6.51, 604.57 -7.72" stroke="#1e1e1e" stroke-width="1" fill="none"/></g></g><mask/><g transform="translate(624.6623928429028 384.8080517344499) rotate(0 6.879997253417969 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">0</text></g><g transform="translate(1249.304662579745 381.4897951555024) rotate(0 13.519989013671875 12.5)"><text x="0" y="17.619999999999997" font-family="Virgil, sans-serif, Segoe UI Emoji" font-size="20px" fill="#1e1e1e" text-anchor="start" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">24</text></g><g transform="translate(220.25782351863114 411.01705791467293) rotate(0 309.0317234848485 33.70620265151524)" stroke="none"><path fill="#e03131" d="M 2.47,0.13 Q 2.47,0.13 2.19,2.54 1.91,4.96 1.83,6.72 1.74,8.49 1.78,9.62 1.82,10.75 1.72,12.10 1.62,13.45 1.65,15.05 1.68,16.65 1.75,17.77 1.82,18.89 6.05,20.63 10.28,22.37 14.58,23.17 18.88,23.96 24.47,24.85 30.06,25.74 38.16,26.98 46.26,28.21 51.35,28.76 56.44,29.31 60.99,29.83 65.54,30.35 75.20,31.14 84.85,31.92 91.67,32.54 98.48,33.15 105.06,33.42 111.64,33.69 117.46,33.81 123.29,33.93 130.17,34.21 137.05,34.49 144.37,34.87 151.68,35.25 157.13,35.41 162.59,35.57 169.31,35.92 176.04,36.26 181.93,36.68 187.82,37.11 193.74,37.77 199.66,38.44 205.20,38.95 210.75,39.45 215.85,39.91 220.95,40.37 225.48,40.83 230.01,41.29 233.98,41.48 237.94,41.67 243.05,42.01 248.15,42.34 251.70,42.72 255.25,43.09 260.13,43.48 265.01,43.88 270.14,44.29 275.28,44.70 279.47,44.88 283.65,45.05 289.19,45.39 294.72,45.74 301.45,46.13 308.19,46.51 312.73,46.68 317.28,46.84 321.60,46.91 325.93,46.98 328.67,47.01 331.42,47.04 333.62,47.05 335.82,47.06 338.17,47.07 340.52,47.08 344.58,47.08 348.64,47.08 351.73,47.08 354.82,47.08 358.86,47.08 362.91,47.08 365.99,47.08 369.07,47.08 371.80,47.08 374.52,47.08 375.88,46.96 377.23,46.84 378.83,46.97 380.42,47.10 381.17,48.45 381.92,49.80 382.38,51.06 382.85,52.32 383.35,53.50 383.86,54.67 384.19,55.84 384.53,57.01 384.86,58.16 385.18,59.32 385.38,60.42 385.57,61.53 385.81,62.59 386.04,63.65 386.40,64.66 386.75,65.68 385.03,65.11 383.32,64.54 384.75,61.51 386.18,58.48 388.50,55.64 390.83,52.80 392.47,50.83 394.12,48.86 395.63,47.33 397.15,45.80 398.91,44.41 400.67,43.03 401.81,41.96 402.95,40.89 404.31,40.49 405.66,40.08 406.76,40.24 407.85,40.40 410.71,41.10 413.56,41.80 415.68,42.26 417.79,42.72 419.46,43.05 421.13,43.37 422.86,43.64 424.59,43.92 426.74,44.30 428.88,44.68 431.21,45.04 433.54,45.41 435.59,45.78 437.64,46.15 440.16,46.33 442.68,46.50 445.19,46.82 447.71,47.14 450.45,47.28 453.19,47.42 456.10,47.49 459.01,47.56 461.29,47.59 463.57,47.62 465.57,47.61 467.57,47.60 470.50,47.62 473.42,47.64 478.02,47.65 482.62,47.66 486.32,47.67 490.02,47.68 493.01,47.68 496.00,47.69 498.04,47.84 500.08,47.99 503.00,48.28 505.91,48.58 513.21,48.95 520.51,49.31 524.68,49.68 528.86,50.05 533.65,50.20 538.43,50.36 543.60,50.63 548.77,50.91 551.66,51.02 554.56,51.14 558.04,51.19 561.53,51.24 564.05,51.26 566.57,51.28 569.39,51.29 572.20,51.30 574.76,51.31 577.32,51.31 580.81,51.31 584.29,51.32 586.81,51.32 589.33,51.32 591.82,51.32 594.30,51.32 596.39,51.14 598.48,50.97 599.59,50.76 600.70,50.56 601.79,50.20 602.87,49.85 604.71,49.00 606.56,48.16 607.98,47.60 609.40,47.04 610.47,46.48 611.53,45.92 612.42,44.91 613.31,43.90 614.14,42.05 614.98,40.20 615.17,39.02 615.37,37.84 615.54,36.57 615.71,35.30 615.41,33.97 615.10,32.65 614.79,31.49 614.48,30.33 613.61,29.38 612.74,28.43 611.94,27.67 611.13,26.91 610.39,25.76 609.65,24.61 609.43,21.11 609.22,17.60 609.42,17.07 609.62,16.54 609.98,16.10 610.34,15.67 610.83,15.37 611.31,15.07 611.86,14.94 612.41,14.81 612.98,14.86 613.54,14.91 614.06,15.14 614.58,15.37 615.00,15.76 615.42,16.14 615.69,16.64 615.96,17.14 616.06,17.70 616.16,18.26 616.08,18.82 616.00,19.38 615.74,19.89 615.48,20.39 615.07,20.79 614.67,21.18 614.15,21.43 613.64,21.67 613.08,21.74 612.51,21.81 611.96,21.70 611.40,21.58 610.91,21.30 610.42,21.01 610.05,20.59 609.67,20.16 609.46,19.63 609.24,19.11 609.20,18.54 609.16,17.98 609.31,17.43 609.45,16.88 609.76,16.40 610.07,15.93 610.52,15.58 610.97,15.23 611.50,15.04 612.04,14.85 612.61,14.84 613.17,14.84 613.71,15.01 614.25,15.18 614.71,15.52 615.17,15.86 615.49,16.32 615.82,16.79 615.98,17.33 616.14,17.88 616.11,18.45 616.09,19.01 616.09,19.01 616.09,19.01 615.94,19.75 615.79,20.48 616.11,21.62 616.43,22.77 617.56,23.82 618.70,24.88 619.84,26.74 620.98,28.61 621.29,30.22 621.60,31.83 621.85,33.35 622.10,34.86 621.94,36.16 621.79,37.46 621.47,39.08 621.14,40.69 620.41,42.46 619.67,44.24 619.18,45.36 618.69,46.48 617.76,47.64 616.84,48.81 615.88,49.64 614.92,50.48 613.67,51.00 612.42,51.51 610.78,52.03 609.15,52.54 607.38,52.85 605.60,53.16 604.48,53.41 603.35,53.65 602.14,53.67 600.92,53.69 599.81,53.64 598.70,53.59 596.50,53.77 594.30,53.95 591.82,53.95 589.33,53.95 586.81,53.95 584.29,53.95 580.80,53.95 577.32,53.95 574.75,53.94 572.19,53.94 569.37,53.93 566.55,53.92 564.02,53.90 561.49,53.88 557.97,53.84 554.45,53.79 551.54,53.68 548.63,53.56 543.49,53.30 538.35,53.03 533.49,52.88 528.63,52.73 524.50,52.38 520.37,52.02 513.01,51.67 505.65,51.31 502.74,51.05 499.83,50.78 497.91,50.57 496.00,50.37 493.01,50.38 490.02,50.38 486.32,50.39 482.62,50.39 478.02,50.40 473.42,50.41 470.49,50.42 467.56,50.43 465.55,50.40 463.54,50.37 461.25,50.36 458.97,50.35 456.01,50.32 453.06,50.28 450.22,50.18 447.37,50.07 444.94,49.82 442.51,49.57 439.82,49.46 437.12,49.35 435.11,49.05 433.11,48.76 430.75,48.52 428.40,48.29 426.20,48.09 424.00,47.89 422.17,47.64 420.34,47.39 418.74,47.06 417.15,46.74 414.90,46.51 412.66,46.28 410.02,45.92 407.39,45.55 406.51,44.90 405.62,44.25 404.33,45.03 403.04,45.82 401.57,47.16 400.11,48.50 398.87,49.98 397.64,51.47 396.25,53.61 394.85,55.75 392.95,58.67 391.05,61.59 389.55,64.54 388.05,67.49 386.39,68.77 384.72,70.05 383.15,69.55 381.58,69.05 380.85,67.50 380.11,65.94 379.96,64.67 379.81,63.40 379.70,62.21 379.59,61.01 379.33,59.78 379.08,58.54 378.99,57.31 378.91,56.08 378.57,54.76 378.23,53.44 377.74,51.70 377.24,49.95 375.88,49.83 374.52,49.71 371.80,49.71 369.07,49.71 365.99,49.71 362.91,49.71 358.86,49.71 354.82,49.71 351.73,49.71 348.64,49.71 344.58,49.70 340.52,49.70 338.16,49.70 335.81,49.69 333.60,49.68 331.39,49.67 328.64,49.64 325.88,49.61 321.53,49.54 317.18,49.47 312.61,49.30 308.04,49.14 301.30,48.75 294.56,48.36 289.05,48.02 283.55,47.68 279.31,47.50 275.07,47.32 269.93,46.91 264.79,46.50 259.89,46.10 254.98,45.70 251.48,45.34 247.98,44.97 242.90,44.63 237.82,44.30 233.78,44.10 229.75,43.91 225.23,43.45 220.71,43.00 215.61,42.54 210.51,42.09 204.94,41.58 199.36,41.07 193.50,40.42 187.63,39.76 181.77,39.34 175.90,38.93 169.21,38.59 162.51,38.26 157.03,38.11 151.55,37.95 144.25,37.59 136.95,37.23 130.09,36.97 123.24,36.71 117.38,36.62 111.53,36.53 104.88,36.30 98.23,36.07 91.42,35.51 84.62,34.95 74.92,34.24 65.21,33.53 60.67,33.12 56.12,32.71 50.93,32.29 45.74,31.87 37.63,30.83 29.51,29.79 23.86,29.15 18.21,28.52 13.57,28.00 8.94,27.48 4.59,26.55 0.25,25.62 -0.81,25.01 -1.87,24.39 -2.62,23.35 -3.38,22.31 -3.48,20.57 -3.58,18.83 -3.48,17.66 -3.39,16.49 -3.19,14.71 -2.99,12.94 -2.82,11.80 -2.64,10.66 -2.56,9.51 -2.47,8.36 -2.45,6.54 -2.42,4.73 -2.45,2.30 -2.47,-0.13 -2.42,-0.42 -2.37,-0.71 -2.25,-0.99 -2.13,-1.26 -1.94,-1.50 -1.76,-1.73 -1.53,-1.92 -1.29,-2.11 -1.02,-2.23 -0.75,-2.36 -0.46,-2.41 -0.16,-2.47 0.12,-2.45 0.42,-2.44 0.71,-2.35 0.99,-2.26 1.25,-2.11 1.51,-1.96 1.72,-1.75 1.93,-1.54 2.09,-1.28 2.25,-1.03 2.34,-0.75 2.43,-0.46 2.45,-0.16 2.47,0.13 2.47,0.13 L 2.47,0.13 Z"/></g><g transform="translate(462.14140543151 341.8062979714912) rotate(0 181.46898674242425 -7.836174242424136)" stroke="none"><path fill="#e03131" d="M -2.07,-1.27 Q -2.07,-1.27 -1.22,-2.61 -0.37,-3.94 0.27,-5.08 0.91,-6.22 1.73,-7.51 2.55,-8.80 3.50,-9.99 4.44,-11.17 5.24,-12.21 6.04,-13.24 7.24,-14.58 8.44,-15.92 9.36,-16.87 10.28,-17.82 11.60,-19.13 12.91,-20.45 13.94,-21.25 14.98,-22.05 17.73,-22.68 20.48,-23.31 29.77,-23.49 39.07,-23.68 47.02,-23.69 54.98,-23.70 60.80,-23.44 66.62,-23.17 70.47,-22.80 74.33,-22.42 77.28,-22.24 80.23,-22.07 83.65,-21.55 87.06,-21.04 90.25,-20.43 93.44,-19.81 96.79,-19.29 100.13,-18.78 103.93,-18.35 107.74,-17.92 111.07,-17.48 114.41,-17.05 118.20,-16.69 122.00,-16.33 124.68,-15.96 127.37,-15.60 130.88,-15.22 134.39,-14.84 136.95,-14.47 139.51,-14.09 141.31,-14.00 143.12,-13.91 145.33,-13.64 147.54,-13.38 148.84,-13.39 150.14,-13.40 152.07,-13.35 154.00,-13.31 155.31,-13.38 156.62,-13.45 157.89,-13.60 159.17,-13.75 160.34,-13.86 161.51,-13.98 162.74,-14.17 163.98,-14.36 165.02,-14.60 166.06,-14.83 166.86,-15.99 167.65,-17.16 168.30,-18.33 168.95,-19.51 169.62,-20.55 170.29,-21.59 171.04,-23.19 171.79,-24.78 172.43,-26.14 173.07,-27.49 173.49,-28.54 173.91,-29.60 174.27,-30.61 174.63,-31.62 175.21,-32.81 175.79,-33.99 177.30,-35.65 178.80,-37.31 180.55,-36.93 182.30,-36.55 183.42,-34.63 184.54,-32.70 184.79,-31.30 185.05,-29.90 185.62,-28.25 186.18,-26.60 186.50,-25.36 186.82,-24.13 188.15,-23.41 189.47,-22.70 191.47,-22.23 193.46,-21.77 195.37,-21.10 197.28,-20.44 200.31,-19.86 203.34,-19.29 206.96,-18.97 210.58,-18.65 214.91,-18.23 219.23,-17.81 223.75,-17.59 228.26,-17.38 231.23,-17.25 234.19,-17.13 237.52,-17.06 240.84,-16.99 244.05,-16.74 247.26,-16.49 249.29,-16.39 251.33,-16.29 254.25,-16.23 257.17,-16.16 260.85,-15.92 264.53,-15.68 268.09,-15.58 271.65,-15.47 275.55,-15.42 279.46,-15.37 283.53,-15.35 287.59,-15.32 290.35,-15.31 293.11,-15.30 295.75,-15.12 298.38,-14.93 301.20,-14.85 304.02,-14.77 305.93,-14.79 307.84,-14.80 309.41,-14.87 310.98,-14.95 312.81,-14.80 314.65,-14.65 315.75,-14.68 316.85,-14.70 318.38,-14.71 319.90,-14.72 322.04,-14.45 324.18,-14.18 325.43,-14.18 326.68,-14.18 328.44,-13.84 330.20,-13.50 331.47,-13.32 332.73,-13.14 334.45,-13.03 336.17,-12.92 337.26,-12.79 338.34,-12.67 339.58,-12.43 340.81,-12.19 342.07,-11.72 343.33,-11.25 344.38,-10.58 345.43,-9.92 346.16,-8.98 346.90,-8.04 347.34,-6.89 347.79,-5.75 348.59,-4.26 349.39,-2.76 350.08,-1.94 350.78,-1.11 352.00,0.09 353.21,1.31 354.07,2.25 354.92,3.19 355.90,4.02 356.88,4.85 357.67,5.89 358.47,6.92 359.35,7.97 360.23,9.02 360.91,10.03 361.59,11.05 362.49,12.39 363.39,13.72 364.61,15.35 365.83,16.97 366.10,17.49 366.37,18.01 366.46,18.59 366.55,19.17 366.45,19.75 366.35,20.32 366.07,20.84 365.79,21.35 365.36,21.75 364.93,22.14 364.39,22.38 363.86,22.62 363.28,22.68 362.70,22.74 362.13,22.60 361.56,22.47 361.06,22.17 360.56,21.86 360.19,21.41 359.82,20.96 359.60,20.41 359.39,19.87 359.37,19.28 359.34,18.70 359.51,18.14 359.67,17.58 360.00,17.10 360.34,16.61 360.81,16.27 361.28,15.92 361.83,15.74 362.39,15.56 362.97,15.56 363.56,15.57 364.11,15.76 364.66,15.96 365.12,16.31 365.58,16.67 365.91,17.16 366.23,17.65 366.38,18.21 366.53,18.78 366.49,19.36 366.45,19.95 366.23,20.49 366.01,21.03 365.63,21.47 365.24,21.91 364.74,22.21 364.23,22.50 363.66,22.62 363.09,22.74 362.51,22.67 361.93,22.60 361.40,22.35 360.87,22.10 360.45,21.69 360.03,21.28 360.03,21.28 360.03,21.29 358.59,19.11 357.15,16.93 356.50,16.09 355.85,15.25 355.23,14.13 354.61,13.00 353.69,11.90 352.78,10.81 351.84,9.63 350.91,8.46 350.02,7.51 349.13,6.56 348.35,5.73 347.56,4.90 346.75,3.99 345.93,3.08 344.78,1.49 343.63,-0.10 342.79,-1.72 341.95,-3.33 341.03,-4.87 340.11,-6.40 338.83,-6.99 337.56,-7.57 336.44,-7.90 335.33,-8.23 333.63,-8.61 331.94,-8.99 330.66,-9.29 329.39,-9.60 327.90,-9.93 326.42,-10.27 325.14,-10.43 323.86,-10.59 321.84,-10.71 319.82,-10.82 318.21,-10.89 316.60,-10.97 315.45,-11.15 314.30,-11.33 312.62,-11.54 310.95,-11.76 309.37,-11.87 307.79,-11.97 305.87,-12.05 303.94,-12.12 301.07,-12.20 298.20,-12.27 295.65,-12.45 293.11,-12.62 290.34,-12.63 287.58,-12.63 283.51,-12.64 279.44,-12.65 275.50,-12.68 271.57,-12.71 267.97,-12.79 264.36,-12.88 260.75,-13.08 257.13,-13.29 254.16,-13.30 251.19,-13.32 249.12,-13.40 247.05,-13.49 243.94,-13.66 240.82,-13.84 237.47,-13.82 234.13,-13.79 231.15,-13.78 228.16,-13.77 223.56,-13.81 218.96,-13.85 214.67,-14.03 210.37,-14.20 206.54,-14.23 202.70,-14.25 199.29,-14.49 195.89,-14.73 193.93,-15.01 191.98,-15.28 190.81,-15.50 189.64,-15.72 187.30,-16.41 184.95,-17.10 183.66,-17.92 182.36,-18.74 181.73,-19.62 181.10,-20.49 180.47,-22.03 179.85,-23.56 179.59,-24.75 179.34,-25.93 178.59,-28.08 177.84,-30.23 177.94,-31.33 178.05,-32.43 179.49,-32.09 180.93,-31.75 181.90,-32.37 182.88,-33.00 182.06,-31.56 181.25,-30.12 180.67,-28.68 180.10,-27.23 179.58,-26.06 179.07,-24.89 178.65,-23.91 178.22,-22.93 177.57,-21.81 176.92,-20.70 175.90,-18.82 174.88,-16.95 174.13,-15.58 173.37,-14.20 172.77,-13.31 172.16,-12.42 171.11,-11.24 170.06,-10.05 168.85,-9.77 167.64,-9.49 166.21,-9.38 164.78,-9.28 163.14,-9.27 161.50,-9.26 160.32,-9.38 159.14,-9.50 157.85,-9.67 156.57,-9.84 155.25,-9.95 153.92,-10.06 151.90,-10.10 149.89,-10.14 148.56,-10.36 147.23,-10.58 145.05,-10.80 142.87,-11.02 141.00,-11.24 139.13,-11.46 136.62,-11.82 134.11,-12.18 130.56,-12.56 127.01,-12.93 124.38,-13.28 121.75,-13.63 117.91,-13.98 114.07,-14.33 110.75,-14.73 107.44,-15.14 103.57,-15.55 99.71,-15.95 96.31,-16.44 92.91,-16.92 89.77,-17.47 86.64,-18.01 83.37,-18.43 80.10,-18.85 77.08,-18.92 74.06,-19.00 70.29,-19.22 66.51,-19.43 60.79,-19.50 55.06,-19.56 47.15,-19.28 39.24,-19.00 30.29,-18.50 21.33,-18.00 19.26,-17.50 17.18,-16.99 16.23,-16.38 15.27,-15.78 14.47,-15.04 13.67,-14.31 12.86,-13.59 12.06,-12.88 11.00,-11.51 9.93,-10.15 9.08,-9.11 8.24,-8.08 7.44,-7.13 6.65,-6.19 5.88,-4.97 5.11,-3.76 4.40,-2.59 3.69,-1.42 2.88,-0.07 2.07,1.27 1.88,1.50 1.70,1.73 1.47,1.91 1.23,2.09 0.97,2.21 0.70,2.33 0.41,2.38 0.12,2.43 -0.16,2.40 -0.46,2.38 -0.74,2.29 -1.02,2.20 -1.26,2.05 -1.51,1.90 -1.72,1.69 -1.93,1.48 -2.07,1.22 -2.22,0.97 -2.31,0.69 -2.39,0.41 -2.41,0.12 -2.42,-0.17 -2.37,-0.45 -2.31,-0.74 -2.19,-1.01 -2.07,-1.27 -2.07,-1.27 L -2.07,-1.27 Z"/></g></svg></div>
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