---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Sharding search 分表查询/","title":"Sharding search 分表查询","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:26:31.000+08:00","updated":"2026-03-24T17:47:54.563+08:00","dg-note-properties":{"title":"Sharding search 分表查询","tags":["flashcards"],"reference linking":null}}
---

## 数据库分表（Sharding）的应用
### 1. 分表与分区的区别
* **分区 (Partitioning):** 是将一个**逻辑表**分割成多个**物理文件**，存储在同一数据库中。对应用程序透明。
* **分表 (Sharding):** 是将一个表分成**几个独立的表**，可以存储在同一个数据库的不同表中，或分散到不同的数据库实例中。
### 2. 分表的类型与方法
| 类型       | 定义                                   | 常用场景                   | 切分依据                    |
| :------- | :----------------------------------- | :--------------------- | :---------------------- |
| **垂直切分** | 按照**字段（列）**进行切分，将不常用的或大字段移到新表。       | 优化单表宽度、提升缓存命中率。        | 业务功能、字段热度。              |
| **水平切分** | 按照**行（记录）**进行切分，将一个表的数据分散到多个结构相同的表中。 | **最常用**，处理超大规模数据量和高并发。 | **日期/时间**、**取模（Hash）**。 |
#### 水平切分常用策略：
1.  **取模（Hash/Mod）切分：**
    * **方法：** 根据用户 ID (`uid`) 对一个数（如 10）取模，将数据分散到 `uid_0` 到 `uid_9` 等表中。
    * **关键问题：** **不能使用自增键**。需要使用**序列**或其他方式生成全局唯一的 ID。
2.  **按日期切分：**
    * **方法：** 按年、月、日等时间维度分表，将每日统计信息放入以日期命名的表中。
    * **场景：** 流量统计系统等对历史数据关注度不高的场景。
3.  **按增量/容量切分：**
    * **方法：** 每个表固定存储一定数量的数据（如 100 万条），超过则创建新表。
### 3. 分表后的挑战与解决方案
分表后，查询、分页和统计是主要难题，通常需要**程序处理**或**辅助视图**。

| 挑战场景 | 挑战描述 | 解决方案 |
| :--- | :--- | :--- |
| **已知 UID 查询** | 仅查询一个特定的分表即可，效率高。 | **程序中计算** `uid%10` 确定表名，然后定向查询。 |
| **分页/统计** | 需要在程序中汇总多个分表结果，再进行排序和分页。 | **通用视图/程序逻辑**。在分页临界点上，可允许微小数据差异，不影响业务。 |
| **模糊搜索** | 需要搜索**所有**分表并汇总结果。 | **多次查询**：好的 SQL 语句执行 10 次，也可能比差的 SQL 执行 1 次还快。 |
| **周/月报表** | 需要汇总大量日数据。 | **定期汇总**：设置外部程序或数据库 JOB，将每日数据汇总到 `week` 表或 `month` 表，查询汇总表。 |
| **历史数据查询** | 巨大流量系统无法提供超长期限的详细数据。 | **功能让步（设限）：** 对超出时限（如 90 天/180 天）的数据不再提供详单查询，仅提供统计数据或归档。 |
### 4. 总结原则
* **分表依据：** 应根据实际业务和查询中起主要作用的字段来分表。
* **先定规则：** 必须在分表前**估计好规模**，确定好切分规则。
* **分后操作：** 复杂的分表管理操作通常需要借助联合查询、视图、Merge 引擎或外部工具/存储过程完成。
* **平衡效率与功能：** 在处理庞大数据时，必须接受在**功能上做出让步**（如限制查询时间范围、限制老帖回复）以保证系统的**效率和稳定性**。
# 分表查询实例
```php
// 分表基类
trait Sharding
{
    /**
     * @var int 单表最大条数
     */
    protected $tableMaxNum;

    /**
     * @var int 当前有多少张分表
     */
    protected $totalTableNum;

    /**
     * 根据ID获取表后缀
     * @param $id
     * @return string
     */
    protected function getTableSuffix($id)
    {
        $num = floatval($id / $this->tableMaxNum);
        return ceil($num) - 1;
    }

    /**
     * 获取表名+后缀
     * @param $id
     * @return string
     */
    public function getTableNameById($id)
    {
        if (env('APP_ENV', 'dev') == IndexCode::APP_DEV) {
            return $this->table . '_0';
        }
        return $this->table . '_' . $this->getTableSuffix($id);
    }
}

// 查询类
class TestModel
{
    use Sharding;

    public $table = 'member';

    public function __construct()
    {
        $this->tableMaxNum = env("MEMBER_TABLE_MAX_NUM", 5000000);
        $this->totalTableNum = env("MEMBER_TOTAL_TABLE_NUM", 2);
    }

    /**
     * 通过ID查手机号
     * @param array $ids
     * @param array $where
     * @return array
     */
    public function getTelsById(array $ids, array $where = []): array
    {
        if (empty($ids)) {
            return [];
        }
        $arrayIds = [];
        foreach ($ids as $id) {
            $tableNumber = $this->getTableSuffix($id);
            $arrayIds[$tableNumber][] = (int)$id;
        }
        $allList = $telArr = [];
        foreach ($arrayIds as $tableNumber => $idArr) {
            foreach (array_chunk($idArr, 5000) as $tIds) {
                $list = Db::table($this->table . "_$tableNumber")
                    ->where($where)
                    ->whereIn('id', $tIds)
                    ->where('tel', '<>', '')
                    ->get(['id', 'tel'])
                    ->toArray();
                if (!empty($list)) {
                    $telArr[] = $list;
                }
            }
        }
        if (!empty($telArr)) {
            $allList = array_merge($allList, ...$telArr);
        }
        return $allList;
    }
}
(new TestModel())->getTelsById($ids);
```

