---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/ArrayAccess/","title":"ArrayAccess","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:25:08.000+08:00","updated":"2026-03-24T17:47:12.174+08:00","dg-note-properties":{"title":"ArrayAccess","tags":["flashcards"],"reference linking":null}}
---

# ArrayAccess(数组式访问)接口
提供像访问数组一样访问对象的能力的接口。
>继承 ArrayObject 可以直接拥有数组能力，无需自己实现
## 示例1
```php
class Obj implements ArrayAccess
{
    private $container = [];
    public function __construct()
    {
        $this->container = [
            'one' => 1,
            'two' => 2,
            'three' => 3
        ];
    }
    // 获取一个偏移位置的值 $obj['two'];
    public function offsetGet($offset)
    {
        return isset($this->container[$offset]) ? $this->container[$offset] : null;
    }
    // 设置一个偏移位置的值 $obj['two'] = 1;
    public function offsetSet($offset, $value)
    {
        if (is_null($offset)) {
            $this->container[] = $value;
        } else {
            $this->container[$offset] = $value;
        }
    }
    // 检查一个偏移位置是否存在 isset()
    public function offsetExists($offset)
    {
        return isset($this->container[$offset]);
    }
    // 复位一个偏移位置的值 unset()
    public function offsetUnset($offset)
    {
        unset($this->container[$offset]);
    }
}

$obj = new Obj;

var_dump(isset($obj['two']));
var_dump($obj['two']);

unset($obj['two']);
var_dump(isset($obj['two']));

$obj['two'] = 'A value';
var_dump($obj['two']);

$obj[] = 'Append 1';
$obj[] = 'Append 2';
$obj[] = 'Append 3';
var_dump($obj);

/*
bool(true)

int(2)

bool(false)

string(7) "A value"

class Obj#1 (1) {
  private $container =>
  array(6) {
    'one' =>
    int(1)
    'three' =>
    int(3)
    'two' =>
    string(7) "A value"
    [0] =>
    string(8) "Append 1"
    [1] =>
    string(8) "Append 2"
    [2] =>
    string(8) "Append 3"
  }
}
*/
```
## 示例2
```php
<?php

namespace XxlJob\Dto;

class RunRequestDto
{
    private array $container;

    /**
     * @param int $jobId 任务ID
     * @param string $executorHandler 执行器Handler
     * @param ?string $executorParams 执行器参数, Nullable string.
     * @param string $executorBlockStrategy 执行器阻塞策略
     * @param int $executorTimeout 执行器超时时间 (秒)
     * @param int $logId 日志ID
     * @param int $logDateTime 日志时间 (Unix timestamp)
     * @param string $glueType Glue类型
     * @param ?string $glueSource Glue源码, Nullable string.
     * @param int $glueUpdatetime Glue更新时间 (Unix timestamp)
     * @param int $broadcastIndex 广播索引
     * @param int $broadcastTotal 广播总数
     */
    public function __construct(
        public int     $jobId = 0,
        public string  $executorHandler = '',
        public ?string $executorParams = null,
        public string  $executorBlockStrategy = '',
        public int     $executorTimeout = 0,
        public int     $logId = 0,
        public int     $logDateTime = 0,
        public string  $glueType = '',
        public ?string $glueSource = null,
        public int     $glueUpdatetime = 0,
        public int     $broadcastIndex = 0,
        public int     $broadcastTotal = 0
    )
    {
        $this->container = get_object_vars($this);
    }

    public function offsetGet($offset): mixed
    {
        return $this->container[$offset] ?? null;
    }

    public function offsetSet($offset, $value): void
    {
        if (is_null($offset)) {
            $this->container[] = $value;
        } else {
            $this->container[$offset] = $value;
        }
    }

    public function offsetExists($offset): bool
    {
        return isset($this->container[$offset]);
    }

    public function offsetUnset($offset): void
    {
        unset($this->container[$offset]);
    }
}

```
# 获取不带命名空间的类名
## Laravel 源码里扒出来的 class_basename 辅助函数
```php
basename(str_replace('\\', '/', $class));
# substr 实现

```php
substr(strrchr($class, "\\"), 1);
// or
substr($class, strrpos($class, '\\') + 1);
```
## explode 实现
```php
array_pop(explode('\\', $class));
```
## ReflectionClass 实现
其中，ReflectionClass 是最快最保险的方案，但此类必须实际存在，不存在则会抛出 ReflectionException: Class \Foo\Bar does not exist。
```php
(new \ReflectionClass($class))->getShortName();
```
# 定时执行脚本
```php
while($change_records = $query->offset($offset)->limit($limit)->all()){
    foreach($change_records as $change_record){
        $log = [
            'tag' => 'RunChangeRecord',
            'attributes' => $change_record->attributes
        ];
        try{
            $change_record->run();
            $log['message'] = 'success';
        }catch(\Exception $e){
            $log['exception'] = [
                'code' => $e->getCode(),
                'message' => $e->getMessage(),
                'trace' => ErrException::getTraceString($e)
            ];
        }
        Yii::info($log, 'log');
    }
    $offset += $limit;
    usleep(10000); // 0.01秒一次
}
```