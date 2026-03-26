---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Redis 位图实现签到/","title":"Redis 位图实现签到","tags":["flashcards"],"noteIcon":"","created":"2025-01-06T16:13:54.000+08:00","updated":"2026-03-24T17:47:51.134+08:00","dg-note-properties":{"title":"Redis 位图实现签到","tags":["flashcards"],"reference linking":null}}
---

# 需求
记录用户签到，查询用户签到
# 技术方案
## 使用mysql(max_time字段为连续签到天数)
```mysql
CREATE TABLE `sign_in`
(
    `id`          BIGINT(20) UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT 'ID',
    `user_id`     BIGINT(20) UNSIGNED NOT NULL COMMENT '用户ID',
    `create_time` DATETIME            NULL COMMENT '创建时间',
    `max_time`    DATETIME            NULL COMMENT '连续签到天数'
)
```
思路：
1. 用户签到，插入一条记录，根据create_time查询昨日是否签到，有签到则max_time在原基础+1，否则，max_time=0
2. 检测签到，根据user_id、create_time查询记录是否存在，不存在则表示未签到
## 使用redis位图功能
思路：
1. 每个用户每个月单独一条redis记录，如`00101010101010`，从左往右代表01-31天（每月有几天，就到几天）
2. 每月8号凌晨，统一将redis的记录，搬至mysql，记录如下

| **id** | **user_id** | **create_time** | **update_time** | **year** | **month** | **bit_log**                     |
| ------ | ----------- | --------------- | --------------- | -------- | --------- | ------------------------------- |
| 7      | 2           | 1570192777      | 1570192777      | 2019     | 10        | 0001000000000000000000000000000 |
3. 查询当月，从redis查，上月则从mysql获取
# 方案对比
举例：一万个用户签到365天
## 方案1：mysql 插入365万条记录
- 频繁请求数据库做一些日志记录浪费服务器开销。
- 随着时间推移数据急剧增大
- 海量数据检索效率也不高，同时只能用时间create_time作为区间查询条件，数据量大肯定慢
## 方案2：redis 插入12w条记录
- 节省空间，每个用户每天只占用1bit空间 1w个用户每天产生 10000bit=1050byte 大概为**1kb**的数据
- 内存操作快
# 实现
## redis key
`前缀_年份_月份:用户id -- sign_2019_10:01`
**查询：**
- 单个：`keys sign_2019_10_01`
- 全部：`keys sign_*`
- 月份：`keys sign_2019_10:*`
## mysql 表
```mysql
CREATE TABLE `sign_in`
(
    `id`          BIGINT(20) UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT 'ID',
    `user_id`     BIGINT(20) UNSIGNED  NOT NULL COMMENT '用户ID',
    `create_time` DATETIME             NULL COMMENT '创建时间',
    `update_time` DATETIME             NULL COMMENT '更新时间',
    `year`        SMALLINT(4) UNSIGNED NOT NULL COMMENT '年',
    `month`       TINYINT(2) UNSIGNED  NOT NULL COMMENT '月',
    `bit_log`     BIT(31)              NOT NULL DEFAULT b'0' COMMENT '位图日志'
	-- 备用方案：使用 INT (4字节, 32位) 存储，足以覆盖31天
    -- `bit_log`     INT UNSIGNED         NOT NULL DEFAULT 0 COMMENT '位图日志(1-31位对应日期)'
)
```
## redis、mysql 比特操作
### Redis 操作 Bit 常用示例
Redis 的位图操作是基于 String 类型的，最大支持  位（512MB）。在签到场景中，通常以 `sign:user_id:yyyyMM` 作为 Key。
#### 1. 用户签到 (SETBIT)
假设用户 ID 为 2，在 2019 年 10 月 3 日签到（偏移量通常从 0 开始，即日期 - 1）：
```bash
# SETBIT key offset value
SETBIT sign:2:201910 2 1
```
#### 2. 检查某天是否签到 (GETBIT)
检查用户在 10 月 3 日是否打卡：
```bash
GETBIT sign:2:201910 2
# 返回 1 表示已签到，0 表示未签到
```
#### 3. 统计当月签到总次数 (BITCOUNT)
```bash
BITCOUNT sign:2:201910
```
#### 4. 获取指定范围的签到数据 (BITFIELD)
获取本月前 31 天的状态，返回一个十进制整数：
```bash
# 获取从第 0 位开始的 31 位无符号整数
BITFIELD sign:2:201910 GET u31 0
```
#### 5. 多用户活跃统计 (BITOP)
计算 10 月 1 日和 10 月 2 日都活跃的用户（交集）：
```bash
BITOP AND result_key sign:20191001 sign:20191002
```
### MySQL 操作 Bit 常用示例
在 MySQL 中，如果 `bit_log` 字段定义为 `INT UNSIGNED` 或 `BIT(31)`，我们使用位运算符 `|`, `&`, `<<`, `>>`。
#### 1. 用户签到 (位或运算)
假设用户在 10 月 10 日签到（将第 10 位置为 1）：
```mysql
-- 公式：bit_log = bit_log | (1 << (day - 1))
UPDATE sign_in 
SET bit_log = bit_log | (1 << 9), 
    update_time = NOW()
WHERE user_id = 2 AND year = 2019 AND month = 10;
```
#### 2. 检查某天是否签到 (位与运算)
判断用户在 10 月 10 日是否签到：
```mysql
SELECT (bit_log >> 9) & 1 AS is_signed
FROM sign_in 
WHERE user_id = 2 AND year = 2019 AND month = 10;
```
#### 3. 统计当月签到次数 (BIT_COUNT)
MySQL 内置函数 `BIT_COUNT` 会返回二进制中 1 的个数：
```mysql
SELECT BIT_COUNT(bit_log) AS total_checkins
FROM sign_in 
WHERE user_id = 2 AND year = 2019 AND month = 10;
```
#### 4. 查找某天所有签到的用户
查询 10 月 10 日所有签到的用户 ID：
```mysql
SELECT user_id 
FROM sign_in 
WHERE year = 2019 AND month = 10 AND (bit_log & (1 << 9));
```
### 两种方案的对比总结
| 特性       | Redis (Bitmap)          | MySQL (Bitwise)          |
| -------- | ----------------------- | ------------------------ |
| **存储成本** | 极低（直接按位存储）              | 低（每个月占一条记录）              |
| **查询速度** | 极快（内存操作）                | 较快（磁盘/索引）                |
| **适用场景** | 高频实时签到、亿级用户统计           | 业务系统持久化、历史报表查询           |
| **优势**   | 支持复杂的跨 Key 位运算 (AND/OR) | 方便与其他业务字段（如 user_id）关联查询 |
## 代码（列出1个调用方法，与三个类）
### 签到方法
```php
public static function userSignIn($userId)
{
	$time = Time();
	$today = date('d', $time);
	$year = date('Y', $time);
	$month = date('m', $time);
	$signModel = new Sign($userId, $year, $month);
	// 1、查询用户今日签到信息
	$todaySign = $signModel->getSignLog($today);
	if ($todaySign) {
		return self::jsonArr(-1, '您已经签到过了', []);
	}
	try {
		Db::startTrans();
		$signModel->setSignLog($today);
		// 4、赠送积分
		if (self::SING_IN_SCORE > 0) {
			$dataScore['order_id'] = $userId . '_' . $today;
			$dataScore['type'] = 2; //2、签到
			$dataScore['remark'] = '签到获得积分';
			Finance::updateUserScore(Finance::OPT_ADD, $userId, self::SING_IN_SCORE, $dataScore);
		}
		$code = '0';
		$msg = '签到成功';
		$score = self::SING_IN_SCORE;
		Db::commit();
	} catch (\Exception $e) {
		Db::rollback();
		$code = '-2';
		$msg = '签到失败';
		$score = 0;
	}
	return self::jsonArr($code, $msg, ['score' => $score]);
}
```
### redis基类
```php
namespace app\common\redis\db1;
/**
 * redis操作类
 */
class RedisAbstract
{
    /**
     * 连接的库
     * @var int
     */
    protected $_db = 1;//数据库名
    protected $_tableName = '';//表名
    static $redis = null;

    public function __construct()
    {
        return $this->getRedis();
    }

    public function _calcKey($id)
    {
        return $this->_tableName . $id;
    }

    /**
     * 查找key
     * @param $key
     * @return array
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function keys($key)
    {
        return $this->getRedis()->keys($this->_calcKey($key));
    }

    /**
     * 获取是否开启缓存的设置参数
     *
     * @return boolean
     */
    public function _getEnable()
    {
        $conf = Config('redis');
        return $conf['enable'];
    }

    /**
     * 获取redis连接
     *
     * @staticvar null $redis
     * @return \Redis
     * @throws \Exception
     */
    public function getRedis()
    {
        if (!self::$redis) {
            $conf = Config('redis');
            if (!$conf) {
                throw new \Exception('redis连接必须设置');
            }
            self::$redis = new \Redis();
            self::$redis->connect($conf['host'], $conf['port']);
            self::$redis->select($this->_db);
        }
        return self::$redis;
    }

    /**
     * 设置位图
     * @param $key
     * @param $offset
     * @param $value
     * @param int $time
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function setBit($key, $offset, $value, $time = 0)
    {
        if (!$this->_getEnable()) {
            return null;
        }
        $result = $this->getRedis()->setBit($key, $offset, $value);
        if ($time) {
            $this->getRedis()->expire($key, $time);
        }
        return $result;
    }

    /**
     * 获取位图
     * @param $key
     * @param $offset
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function getBit($key, $offset)
    {
        if (!$this->_getEnable()) {
            return null;
        }
        return $this->getRedis()->getBit($key, $offset);
    }

    /**
     * 统计位图
     * @param $key
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function bitCount($key)
    {
        if (!$this->_getEnable()) {
            return null;
        }
        return $this->getRedis()->bitCount($key);
    }

    /**
     * 位图操作
     * @param $operation
     * @param $retKey
     * @param mixed ...$key
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function bitOp($operation, $retKey, ...$key)
    {
        if (!$this->_getEnable()) {
            return null;
        }
        return $this->getRedis()->bitOp($operation, $retKey, $key);
    }

    /**
     * 计算在某段位图中 1或0第一次出现的位置
     * @param $key
     * @param $bit 1/0
     * @param $start
     * @param null $end
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function bitPos($key, $bit, $start, $end = null)
    {
        if (!$this->_getEnable()) {
            return null;
        }
        return $this->getRedis()->bitpos($key, $bit, $start, $end);
    }

    /**
     * 删除数据
     * @param $key
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function del($key)
    {
        if (!$this->_getEnable()) {
            return null;
        }
        return $this->getRedis()->del($key);
    }
}
```
### 签到redis操作类
```php
namespace app\common\redis\db1;
class Sign extends RedisAbstract
{
    public $keySign = 'sign'; //签到记录key

    public function __construct($userId, $year, $month)
    {
        parent::__construct();
        // 设置当前用户 签到记录的key
        $this->keySign = $this->keySign . '_' . $year . '_' . $month . ':' . $userId;
    }

    /**
     * 用户签到
     * @param $day
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function setSignLog($day)
    {
        return $this->setBit($this->keySign, $day, 1);
    }

    /**
     * 查询签到记录
     * @param $day
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function getSignLog($userId, $day)
    {
        return $this->getBit($this->keySign, $day);
    }

    /**
     * 删除签到记录
     * @return int|null
     * @throws \Exception
     * @author wenzhen-chen
     */
    public function delSignLig()
    {
        return $this->del($this->keySign);
    }
}
```
### 定时更新至mysql的类
```php
namespace app\common\business;

use app\common\mysql\SignLog;
use app\common\redis\db1\Sign;

class Cron
{
    /**
     * 同步用户签到记录
     * @throws \Exception
     */
    public static function addUserSignLogToMysql()
    {
        $data = [];
        $time = Time();
        // 1、计算上月的年份、月份
        $dataTime = Common::getMonthTimeByKey(0);
        $year = date('Y', $dataTime['start_time']);
        $month = date('m', $dataTime['start_time']);
        // 2、查询签到记录的key
        $signModel = new Sign(0, $year, $month);
        $keys = $signModel->keys('sign_' . $year . '_' . $month . ':*');
        foreach ($keys as $key) {
            $bitLog = '';//用户当月签到记录
            $userData = explode(':', $key);
            $userId = $userData[1];
            // 3、循环查询用户是否签到（这里没按每月天数存储，直接都存31天了）
            for ($i = 1; $i <= 31; $i++) {
                $isSign = $signModel->getBit($key, $i);
                $bitLog .= $isSign;
            }
            $data[] = [
                'user_id' => $userId,
                'year' => $year,
                'month' => $month,
                'bit_log' => $bitLog,
                'create_time' => $time,
                'update_time' => $time
            ];
        }
        //4、插入日志
        if ($data) {
            $logModel = new SignLog();
            $logModel->insertAll($data, '', 100);
        }
    }
}
```