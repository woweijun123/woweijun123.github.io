---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Concurrent/","title":"Concurrent","tags":["flashcards"],"noteIcon":"","created":"2026-07-30T18:45:57.000+08:00","updated":"2026-07-30T18:45:57.000+08:00","dg-note-properties":{"title":"Concurrent","tags":["flashcards"],"reference linking":null}}
---

# 参考资料
 - [php结合redis实现高并发下的抢购、秒杀功能](https://blog.csdn.net/nuli888/article/details/51865401#commentBox)
 - [PHP 并发场景的几种解决方案](https://www.jb51.net/article/163114.htm)
 - [网站大规模并发处理方案-电商秒杀与抢购](https://www.awaimai.com/348.html)
 - [PHP高并发问题](https://www.cnblogs.com/walblog/articles/8476579.html)
 - [单据锁](https://blog.csdn.net/zhangliangzi/article/details/79874910)
 - [lua脚本解锁](https://www.cnblogs.com/youjiaxing/p/10437674.html)
 - [PHP+Redis实现分布式锁 ](https://juejin.cn/post/7163834723200925709)
## 相关概念
- [[Work/Script/PHP/Swoole/同步、异步，并发、并行，线程、进程、协程#并发、并行\|并发与并行]]：本页处理的是**并发下的共享资源竞争**。
- [[Work/Databases/Redis/Redis basic#WATCH\|`WATCH`]]、[[Work/Databases/Redis/Redis basic#MULTI\|`MULTI`]]、[[Work/Databases/Redis/Redis basic#EXEC\|`EXEC`]]：Redis 乐观事务的命令语义。
- [[Work/Databases/Redis/Redis Lua 脚本#9.1 分布式锁（SET NX + 安全释放）\|Redis Lua 分布式锁]]：`SET NX EX/PX`、token 校验和 Lua 原子释放。
- [[Work/Script/PHP/Swoole/锁竞争状态\|锁竞争状态]]：竞争程度、性能影响与优化方向。
# Redis 并发控制
## Redis 队列
**1. 初始化数据**
```php
try {
    // 连接redis
    $redis = new Redis();
    $result = $redis->connect('redis', 6379);
    $redis->auth('riven');
    $redis->select(1);
    $res = $redis->llen('goods_store');

    // 连接mysql
    $conn = mysqli_connect('mysql', 'root', '123456', 'big', '3306');
    if (!$conn) {
        throw new Exception('connection failed');
    }

    // 清空redis、mysql库存
    $redis->flushDB();
    if (!mysqli_query($conn, 'TRUNCATE TABLE `goods`;')) {
        throw new Exception('清空 goods 表失败');
    }
    if (!mysqli_query($conn, 'TRUNCATE TABLE `log`;')) {
        throw new Exception('清空 log 表失败');
    }
    if (!mysqli_query($conn, 'TRUNCATE TABLE `order`;')) {
        throw new Exception('清空 order 表失败');
    }
    if (!mysqli_query($conn, 'TRUNCATE TABLE `store`;')) {
        throw new Exception('清空 store 表失败');
    }

    $store = 100;
    // 添加redis库存
    for ($i = 0; $i < $store; $i++) {
        $redis->lpush('goods_store', 1);
    }
    echo '添加redis库存', $redis->llen('goods_store'), PHP_EOL;

    // 添加mysql库存
    $ret = mysqli_query($conn, "INSERT INTO `goods` (`goods_name`) VALUES('Iphone 12')");
    if (!$ret) {
        throw new Exception('添加商品失败');
    }
    $ret = mysqli_query($conn, "INSERT INTO `store` (`goods_id`, `sku_id`, `number`) VALUES(1, 1, ${store})");
    if (!$ret) {
        throw new Exception('添加mysql库存失败');
    }
    echo '添加mysql库存', $store, PHP_EOL;
} catch (Exception $e) {
    echo $e->getMessage(), PHP_EOL;
}
```

**2. 进行并发测试**
```php
$conn = mysqli_connect('mysql', 'root', '123456', 'big');
if (!$conn) {
    exit('connect failed');
}

// 生成唯一订单号
function build_order_no()
{
    return date('ymd') . substr(implode(NULL, array_map('ord', str_split(substr(uniqid(), 7, 13), 1))), 0, 8);
}

// 记录日志
function insertLog($event, $type = 0)
{
    global $conn;
    mysqli_query($conn, "INSERT INTO `log` (`event`, `type`) values('${event}', '${type}')");
}

try {
    // 模拟下单操作
    // 下单前判断redis队列库存量
    $redis = new Redis();
    $result = $redis->connect('redis', 6379);
    $redis->auth('riven');
    $redis->select(1);
    $count = $redis->lpop('goods_store');
    if (!$count) {
        throw new Exception('库存不足');
    }

    // 模拟订单数据
    $price = 10;
    $userId = 1;
    $goodsId = 1;
    $skuId = 1;
    $number = 1;

    // 库存减少
    $store_rs = mysqli_query($conn, "UPDATE `store` SET `number` = `number` - ${number} WHERE `sku_id` = '${skuId}' AND `number` > 0");
    if (!$store_rs) {
        throw new Exception('库存减少失败');
    }

    // 生成订单
    $orderSn = build_order_no();
    $order_rs = mysqli_query($conn, "INSERT INTO `order` (order_sn, user_id, goods_id, sku_id, price) VALUES ('${orderSn}', '${userId}', '${goodsId}', '${skuId}', '${price}') ");
    if (!$order_rs) {
        throw new Exception('下单失败');
    }

    $msg = '库存减少成功';
    echo $msg, PHP_EOL;
    insertLog($msg);
} catch (Exception $e) {
    echo $e->getMessage(), PHP_EOL;
    insertLog($e->getMessage());
}
```

## Redis 事务
`WATCH` 监测库存 → `MULTI` 入队 → `EXEC` 提交；若监测的 key 在提交前被改动，`EXEC` 返回失败，业务应重试。命令细节见 [[Work/Databases/Redis/Redis basic#WATCH\|Redis 事务]]。
**1. 入库**
```php
// 模拟唯一用户ID
$uid = uniqid('uid-', TRUE);
$redis = new Redis();
// 连接 redis
$redis->connect('redis', 6379);
$redis->select(1);
$redis->flushDB();
echo '重置成功' . PHP_EOL;
// 设置100库存
$redis->set('rest_count', 100);
echo "库存设置为: " . $redis->get('rest_count');
```

**2. 消费库存**
```php
// 模拟唯一用户ID
$uid = uniqid('uid-', TRUE);
$redis = new Redis();
// 连接 redis
$redis->connect('redis', 6379);
$redis->select(1);
// 监测 rest_count 是否被其它的进程更改
$redis->watch('rest_count');
// 模拟唯一订单ID
$rest_count = intval($redis->get('rest_count'));
if ($rest_count > 0) {
    // 表示当前订单，被当前用户抢到了
    $value = "{$rest_count}-{$uid}";
    // do something ... 主要是模拟用户抢到单后可能要进行的一些密集运算
    $rand = rand(100, 1000000);
    $sum = 0;
    for ($i = 0; $i < $rand; $i++) {
        $sum += $i;
    }

    $redis->multi(); // 开始redis事务
    $redis->lPush('uid', $value);
    $redis->decr('rest_count');
    $replies = $redis->exec(); // 执行以上redis事务

    // 若 rest_count 被其他进程修改，EXEC 返回失败；队列中的命令不会执行
    if (!$replies) {
        echo "订单 ${value} 回滚" . PHP_EOL;
    }
}
$redis->unwatch();
echo '剩余库存: ' . $redis->get('rest_count') . PHP_EOL;
```

# Redis 分布式锁
## Lua 原子释放
**核心：** `SET NX EX` 获取带过期时间的锁；释放时以 token 比对后再 `DEL`，并用 Lua 把“比对 → 删除”做成一个原子操作。完整原理、误删时序和 `NX` / `PX` 语义见 [[Work/Databases/Redis/Redis Lua 脚本#9.1 分布式锁（SET NX + 安全释放）\|Redis Lua 分布式锁]]。
```php
/**
 * redis 分布式锁
 */
class Lock
{

    private mixed $config;
    private Redis $redis;

    public function __construct($config = [])
    {
        $this->config = $config ?: ['host' => '127.0.0.1', 'port' => 6379];
        $this->redis  = $this->connect();
    }

    public function connect(): Redis
    {
        $redis = new Redis();
        try {
            $redis->connect($this->config['host'], $this->config['port']);
        } catch (RedisException $e) {
            var_dump($e->getMessage());
        }

        return $redis;
    }

    /**
     * @param $key        [键]
     * @param int $expire [锁有效期]
     */
    public function lock($key = null, int $expire = 10): bool|string|Redis
    {
        if (!$key || !$expire) {
            return false;
        }

        // 生成随机值，锁标识
        $lockId = md5(uniqid());
        try {
            // $lockId 必须全局唯一、"NX" 仅在key不存在时加锁、"EX" 设置锁过期时间
            $result = $this->redis->set($key, $lockId, ['NX', 'EX' => $expire]);
        } catch (RedisException $e) {
            var_dump($e->getMessage());
        }

        if ($result) {
            return $lockId;
        } else {
            return $result;
        }
    }

    /**
     * 解锁
     */
    public function unLock($scene, $lockId)
    {
        $lua = <<<EOF
if redis.call("get", KEYS[1]) == ARGV[1]
then
    return redis.call("del", KEYS[1])
end
EOF;
        try {
            // 返回结果 >0 表示解锁成功
            return $this->redis->eval($lua, [$scene, $lockId], 1);
        } catch (RedisException $e) {
            var_dump($e->getMessage());
        }
    }
}

$lock = new Lock();
// 第一次加锁
$lockId = $lock->Lock("test", 30);
var_dump($lockId); // 返回lockId
// 第二次加锁
var_dump($lock->Lock("test", 25)); // 加锁失败 false
// 解锁
if ($lockId) {
    var_dump($lock->unLock("test", $lockId));
}
```

## `WATCH` + `MULTI` / `EXEC`
这是 Lua 方案的替代示例：`WATCH` 保护“读取锁 ID → 删除锁”的乐观事务；锁释放的首选仍是上方 Lua 方案，见 [[Work/Databases/Redis/Redis Lua 脚本#9.1 分布式锁（SET NX + 安全释放）\|Redis Lua 分布式锁]]。
```php
/**
 * Class Lock_Service 单据锁服务
 */
class RedisLock
{
    // Redis配置：IP
    const REDIS_CONFIG_HOST = 'host.docker.internal';
    // Redis配置：端口
    const REDIS_CONFIG_PORT = 6379;
    // 单据锁redis key模板
    const REDIS_LOCK_KEY_TEMPLATE = 'lock_%s';
    // 单据锁默认超时时间3600秒(1小时)
    const REDIS_LOCK_DEFAULT_EXPIRE_TIME = 3600;
    // 用于生成唯一的锁ID的redis key
    const REDIS_LOCK_UNIQUE_ID_KEY = 'lock_unique_id';

    /**
     * 获取Redis连接（简易版本，可用单例实现）
     * @param string $strIp IP
     * @param int $intPort 端口
     * @return object Redis连接
     */
    public static function instance($strIp = self::REDIS_CONFIG_HOST, $intPort = self::REDIS_CONFIG_PORT)
    {
        $objRedis = new Redis();
        $objRedis->connect($strIp, $intPort);
        return $objRedis;
    }

    /**
     * 生成锁唯一ID（通过Redis incr指令实现简易版本，可结合日期、时间戳、取余、字符串填充、随机数等函数，生成指定位数唯一ID）
     * @return mixed
     */
    public static function generateUniqueLockId()
    {
        return self::instance()->incr(self::REDIS_LOCK_UNIQUE_ID_KEY);
    }

    /**
     * 加单据锁
     * @param int $intOrderId 单据ID
     * @param int $intExpireTime 锁过期时间（秒）
     * @return bool|int 加锁成功返回唯一锁ID，加锁失败返回false
     */
    public static function lock($intOrderId, $intExpireTime = self::REDIS_LOCK_DEFAULT_EXPIRE_TIME)
    {
        // 参数校验
        if (empty($intOrderId) || $intExpireTime <= 0) {
            return false;
        }
        // 获取Redis连接
        $objRedisConn = self::instance();
        // 生成唯一锁ID，解锁需持有此ID
        $intUniqueLockId = self::generateUniqueLockId();
        // 根据模板，结合单据ID，生成唯一Redis key（一般来说，单据ID在业务中系统中唯一的）
        $strKey = sprintf(self::REDIS_LOCK_KEY_TEMPLATE, $intOrderId);
        // 加锁（通过Redis setnx指令实现，从Redis 2.6.12开始，通过set指令可选参数也可以实现setnx，同时可原子化地设置超时时间）
        $bolRes = $objRedisConn->set($strKey, $intUniqueLockId, ['nx', 'ex' => $intExpireTime]);
        // 加锁成功返回锁ID，加锁失败返回false
        return $bolRes ? $intUniqueLockId : $bolRes;
    }

    /**
     * 解单据锁
     * @param int $intOrderId 单据ID
     * @param int $intLockId 锁唯一ID
     * @return bool
     */
    public static function unLock($intOrderId, $intLockId)
    {
        // 参数校验
        if (empty($intOrderId) || empty($intLockId)) {
            return false;
        }
        // 获取Redis连接
        $objRedisConn = self::instance();
        // 生成Redis key
        $strKey = sprintf(self::REDIS_LOCK_KEY_TEMPLATE, $intOrderId);
        // 监听Redis key防止在【比对lock id】与【解锁事务执行过程中】被修改或删除，提交事务后会自动取消监控，其他情况需手动解除监控
        $objRedisConn->watch($strKey);
        if ($intLockId == $objRedisConn->get($strKey)) {
            $objRedisConn->multi();
            $objRedisConn->del($strKey);
            $objRedisConn->exec();
            return true;
        }
        $objRedisConn->unwatch();
        return false;
    }
}
$res1 = RedisLock::lock('666666');
var_dump($res1); // 返回lock id，加锁成功
$res2 = RedisLock::lock('666666');
var_dump($res2); // false，加锁失败
$res3 = RedisLock::unLock('666666', $res1);
var_dump($res3); // true，解锁成功
$res4 = RedisLock::unLock('666666', $res1);
var_dump($res4); // false，解锁失败
```

## Laravel 原子锁
```php
public function atom()
{
    $lock = Cache::lock('product', 10);
    try {
        // 最多等待5秒，5秒后未获取到锁，则抛出异常
        $lock->block(5);
        $number = DB::table('ih_store')->where('id', 1)->value('number');
        if ($number <= 0) {
            throw new CustomException('卖光啦！');
        }
        DB::table('ih_store')->where('id', 1)->decrement('number'); // 库存-1
        success();
    } catch (LockTimeoutException $e) {
        throw new CustomException('当前人数过多！');
    } finally {
        $lock->release();
    }
}
```

## Redis Cluster / Redlock
**Redlock：** 在多个独立 Master 上取得多数锁才算成功；失败则尽力释放已取得的锁。它的适用边界与争议见 [[Work/Databases/Redis/Redis Lua 脚本#9.1 分布式锁（SET NX + 安全释放）\|Redis Lua 分布式锁]]。

# MySQL 锁
## 悲观锁与乐观锁
悲观锁通过 `FOR UPDATE` 排他；乐观锁通过条件更新检测冲突。与 Redis `WATCH` 的乐观重试思路关联，见 [[Work/Databases/Redis/Redis basic#WATCH\|`WATCH`]]。
```php
// 悲观锁
public function pessimistic()
{
    try {
        DB::beginTransaction();
        $num = DB::table('ih_store')->where('id', 1)->lockForUpdate()->value('number');
        if ($num <= 0) {
            throw new CustomException('卖光了！');
        }
        DB::table('ih_store')->where('id', 1)->decrement('number'); // 库存-1
        DB::commit();
    } catch (Exception $e) {
        DB::rollBack();
        throw $e;
    }
    return $num;
}

// 乐观锁
public function optimistic()
{
    try {
        DB::beginTransaction();
        $num = DB::table('ih_store')->where('id', 1)->value('number');
        if ($num <= 0) {
            throw new CustomException('卖光了！');
        }
        if (!DB::table('ih_store')->where('id', 1)->where('number', '>', 0)->decrement('number')) {  // 库存-1
            throw new CustomException('当前人数过多！');
        }
        DB::commit();
    } catch (Exception $e) {
        DB::rollBack();
        throw $e;
    }
    return $num;
}
```

# 文件锁
仅适用于**单机共享文件系统**；多实例部署应使用 Redis / 数据库方案。竞争与性能取舍见 [[Work/Script/PHP/Swoole/锁竞争状态#文件锁优化\|锁竞争状态 · 文件锁优化]]。
## 排它锁（阻塞模式）
**1. 入库**
```php
$redis = new Redis();
$redis->connect('redis', 6379);
$redis->select(1);
$key = 'rest_count';
$redis->set($key, 100);
echo '剩余次数:', $redis->get($key), PHP_EOL;
```

**2. 消费库存**
```php
$redis = new Redis();
$redis->connect('redis', 6379);
$redis->select(1);

$uid = uniqid('uid-', TRUE);
$fp = fopen('lock.txt', 'w+');
// 阻塞(等待)模式, 要取得独占锁定(写入的程序)
if (flock($fp, LOCK_EX))
{
    // 成功取得锁后，放心处理订单
    $rest_count = intval($redis->get('rest_count'));
    $value = "{$rest_count}-{$uid}";
    if ($rest_count > 0) {
        // do something ...
        $rand = rand(100, 1000000);
        $sum = 0;
        for ($i = 0; $i < $rand; $i++) {
            $sum += $i;
        }
        $redis->lPush('uid', $value);
        $redis->decr('rest_count');
    }
    // 订单处理完成后，再释放锁
    flock($fp, LOCK_UN);
}
fclose($fp);
```

## 排它锁（非阻塞模式）
**1. 入库**
```php
$redis = new Redis();
$redis->connect('redis', 6379);
$redis->select(1);
$key = 'rest_count';
$redis->set($key, 100);
echo '剩余次数:', $redis->get($key), PHP_EOL;
```

**2. 消费库存**
```php
$redis = new Redis();
$redis->connect('redis', 6379);
$redis->select(1);

$uid = uniqid('uid-', TRUE);
$fp = fopen('lock.txt', 'w+');
// 非阻塞模式, 如果不希望 flock() 在锁定时堵塞, 则给 lock 加上 LOCK_NB
if (flock($fp, LOCK_EX | LOCK_NB)) {
    // 成功取得锁后，放心处理订单
    $rest_count = intval($redis->get('rest_count'));
    $value = "${rest_count}-${uid}";
    if ($rest_count > 0) {
        // do something ...
        $rand = rand(100, 1000000);
        $sum = 0;
        for ($i = 0; $i < $rand; $i++) {
            $sum += $i;
        }

        $redis->lPush('uid', $value);
        $redis->decr('rest_count');
    }

    // 订单处理完成后，再释放锁
    flock($fp, LOCK_UN);
} else {
    // 如果获取锁失败，马上进入这里执行
    echo "${uid} - 系统繁忙，请稍后再试" . PHP_EOL;
}
fclose($fp);
```
