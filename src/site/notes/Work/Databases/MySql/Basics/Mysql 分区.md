---
{"dg-publish":true,"permalink":"/Work/Databases/MySql/Basics/Mysql 分区/","title":"Mysql 分区","tags":["#flashcards"],"noteIcon":"","created":"2026-03-10T22:33:54.000+08:00","updated":"2026-06-17T14:43:57.104+08:00","dg-note-properties":{"title":"Mysql 分区","tags":["#flashcards"],"reference linking":"[高性能可扩展mysql](https://www.cnblogs.com/wushaopei/tag/%E9%AB%98%E6%80%A7%E8%83%BD%E5%8F%AF%E6%89%A9%E5%B1%95mysql/)"}}
---

### 概念
在MySQL中，分区是一种优化策略，用于处理非常大的表，通过将数据分布在多个**物理存储**上，以提高查询性能和管理能力。
>分区表在物理上表现为多个文件，在**逻辑上**表现为**一个表**

以下是四种主要分区类型：
- RANGE
- LIST
- HASH
- KEY
### 查询是否支持分区
使用 `SHOW PLUGINS` 在mysql命令行查看是否具有分区表的功能：
查询结果中的"partition | ACTIVE | STORAGE ENGINE | NULL | GPL "这一行代表当前数据库可以进行数据库分区表操作。
### 物理结构区别
左边为普通表的物理结构，右边为分区后的数据库表物理结构。
![Pasted image 20251028183125](https://weichengjun2.dpdns.org/i/2025/10/28/69009b7e667db.png)
### PARTITION BY 的基本概念
`PARTITION BY` 可以根据表中某一列的值或者一组列的值来决定数据应该存储在哪个分区。
### 常见的分区类型
1. **范围分区（RANGE）**：数据根据某列的值范围被分配到不同的分区。
2. **列表分区（LIST）**：数据根据某列的值属于特定列表来分配到分区。
3. **散列分区（HASH）**：数据根据某列值的哈希计算结果被随机分配到分区。
4. **键分区（KEY）**：类似于散列分区，但使用 MySQL 内部的哈希算法。
### 1. 范围分区（RANGE Partitioning）
范围分区是基于属于一个给定连续区间的列值，把多行分配给分区。这种类型的分区特别适合于**基于时间的序列数据**，如日志或交易记录。
#### 用法
RANGE分区基于一个列或表达式的值范围进行分区。数据被分配到不同的分区中，每个分区包含了特定范围内的数据行。通常使用`VALUES LESS THAN`关键字来定义分区边界。
```mysql
CREATE TABLE logs (
  id INT NOT NULL,
  log_time TIMESTAMP,
  message TEXT,
  PRIMARY KEY (id)
)
PARTITION BY RANGE (YEAR(log_time))
(
  PARTITION p2020 VALUES LESS THAN (2021),
  PARTITION p2021 VALUES LESS THAN (2022),
  PARTITION p2022 VALUES LESS THAN (2023),
  PARTITION p2023 VALUES LESS THAN MAXVALUE
);
```
#### 场景
适用于按时间顺序或数值顺序分布的数据，比如订单表中的日期字段或用户表中的等级字段。
#### 优点
- 对于范围查询特别有效，如查询特定时间段内的数据。
- 可以减少磁盘I/O和内存使用，因为不需要扫描整个表。
#### 缺点
- 插入新数据时，如果数据落在分区边界附近，可能会引起热点问题。
- 需要预先知道数据分布情况以正确设置分区边界。
### 2. 列表分区（LIST Partitioning）
列表分区类似于范围分区，但它是基于列值匹配一个离散值集合中的某个值来进行选择。列表分区适用于数据中存在特定的离散值的情况。
#### 用法
LIST分区类似于RANGE分区，但是它基于一个列的离散值列表进行分区，而不是连续的值范围。
```mysql
CREATE TABLE users (
    user_id INT,
    country CHAR(2)
)
PARTITION BY LIST (country)
(
    PARTITION usa VALUES IN ('US'),
    PARTITION eu VALUES IN ('DE', 'FR', 'IT'),
    PARTITION other VALUES IN ('JP', 'CN', 'IN')
);
```
#### 场景
适用于数据具有明确分类的情况，如用户的国家或地区。
#### 优点
- 查询特定值时，可以直接定位到相应的分区，提高查询速度。
#### 缺点
- 需要提前知道所有可能的值。
- 增加新的类别可能需要重新调整分区结构。
### 3. 散列分区（HASH Partitioning）
散列分区允许 DBA 通过对表的一个或多个列的 Hash Key 进行计算，最后通过这个 Hash 码不同数值对应的数据区域进行分区。
这种分区方式适用于**数据分布均匀**的情况。
#### 用法
HASH分区使用用户定义的表达式（通常是哈希函数）的结果进行分区，结果根据分区数取模以确定数据属于哪个分区。
```mysql
# 用法1「通过模运算确保数据均匀分布至100个分区，避免热点，但效果依赖于`id`的分布」
CREATE TABLE customers (
  id INT NOT NULL,
  name VARCHAR(50),
  city VARCHAR(50),
  PRIMARY KEY (id)
)
PARTITION BY HASH (id % 100) PARTITIONS 100;

# 用法2「直接使用`id`的哈希值分配分区，实现简单却可能造成数据分布不均，尤其当`id`分布不规则时」
CREATE TABLE customers (
  id INT NOT NULL,
  name VARCHAR(50),
  city VARCHAR(50),
  PRIMARY KEY (id)
)
PARTITION BY HASH (id) PARTITIONS 100;
```
#### 场景
适用于需要均匀分布数据的场景，尤其是当数据分布未知或者不需要基于特定值查询时。
#### 优点
- 自动均匀分布数据，避免热点问题。
- 提高写入性能，尤其是在大量并发插入的情况下。
#### 缺点
- 查询可能需要扫描多个分区，降低查询效率。
- 不适合范围查询或等值查询。
### 4. 键分区（KEY Partitioning）
键分区类似于散列分区，不同之处在于键分区使用 MySQL 提供的哈希函数。这种分区方式通常用于当列值为整数时。
#### 用法
KEY分区与HASH分区相似，但使用的是MySQL内部的哈希函数，并且可以基于多列进行分区。
```mysql
CREATE TABLE transactions (
  transaction_id INT NOT NULL,
  account_id INT,
  amount DECIMAL(10,2),
  PRIMARY KEY (transaction_id)
)
PARTITION BY KEY (account_id) PARTITIONS 16;
```
#### 场景
适用于需要基于多列数据均匀分布的场景。
#### 优点
- 数据均匀分布，减少热点。
- 支持多列分区，灵活性更高。
#### 缺点
- 查询性能取决于索引和分区的组合，可能不如RANGE或LIST分区针对特定查询高效。
### 注意事项
1. **分区键与唯一性：** 分区键通常应包含在主键或唯一键中，以保证数据在所有分区间的唯一性。这是分区设计的**铁律**。
2. **主键性能代价：** 对 InnoDB 而言，如果分区键选择不当，导致主键过大，会增加**聚簇索引**和二级索引的体积，增加 I/O 开销。
3. **主键与分区键选择：** 关键在于**合理选择分区键**。不应因分区而轻易放弃主键，否则会损失 InnoDB 的性能优势和数据完整性。
4. **查询模式是核心：** 分区可以提高查询性能，**特别是当查询条件涉及分区键时**。同时，分区的数量和类型应根据数据分布、**查询模式**和硬件配置来决定。
5. **查询风险（分区裁剪）：** 必须验证。如果查询条件**不包含分区键**，将导致**分区裁剪 (Partition Pruning) 失败**，查询将扫描所有分区，性能会大幅下降。
6. **分区键类型：** 分区键类型**限制已放宽**，较新版本（5.7/8.0）不再严格要求 `INT`，支持直接使用日期/时间或 `VARCHAR`。
7. **分区数量：** 外键支持的**限制已解除**，InnoDB 分区表支持外键约束。
8. **数量与资源：** 分区数量存在上限（8.0 为 8192），且对于范围和列表分区，必须确保定义覆盖所有可能的值范围。**过多分区会显著消耗系统内存**。
### 分区演示
 **如何为customer_login_log表分区?** 
从以下两个业务场景入手：
- 用户每次登陆都会记录customer_login_log入职
- 用户登录日志保存一年，一年后可以删除
#### 分区类型、分区键确定
> 分区类型： 使用 **RANGE** 分区以 **login_time** 作为分区键
#### 创建分区表
```mysql
CREATE TABLE `crn`.`customer_login_log`
(
    `customer_id` INT UNSIGNED NOT NULL,
    `login_time`  DATETIME     NOT NULL,
    `login_ip`    INT UNSIGNED NOT NULL,
    `login_type`  TINYINT      NOT NULL
) ENGINE = INNODB
    PARTITION BY RANGE (YEAR(`login_time`))(
        PARTITION `p0` VALUES LESS THAN (2015),
        PARTITION `p2` VALUES LESS THAN (2016),
        PARTITION `p3` VALUES LESS THAN (2017));
```
结果截图：
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f952a17c.png)
#### 插入分区数据
```mysql
INSERT INTO `customer_login_log`(`customer_id`, `login_time`, `login_ip`, `login_type`)
VALUES (1001, '2015-01-25', 0, 1),
       (1001, '2015-07-1', 0, 1),
       (1001, '2015-10-1', 0, 1),
       (1001, '2016-3-1', 0, 1),
       (1001, '2016-9-1', 0, 1);
```
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f955ee06.png)
默认匹配规则说明：
**创建2条2020年的数据** 
```mysql
INSERT INTO `customer_login_log`(`customer_id`, `login_time`, `login_ip`, `login_type`)
VALUES (1001, '2020-01-25', 0, 1),
       (1001, '2020-07-1', 0, 1);
```
创建分区范围分别为2019及2021年的分区：
```mysql
ALTER TABLE `customer_login_log` ADD PARTITION (PARTITION `p5` VALUES LESS THAN (2019));  
ALTER TABLE `customer_login_log` ADD PARTITION (PARTITION `p6` VALUES LESS THAN (2021));
```

最终匹配结果：
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f958de78.png)
 ​
新创建的2020年的数据都被匹配到了2021年的分区区间，这是由于在没有创建相应分区的情况下，其会**默认匹配**到**最近的规则**的分区区域。
有鉴于此，当创建的时间信息超出当前已定义的范围时，需根据规则及时创建新的分区，已规范数据的管理。
#### 删除分区--同步删除分区内数据：
```mysql
ALTER TABLE `customer_login_log` DROP PARTITION `p6`;
```
分区表被删除：
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f95b688f.png)
 ​
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f9606f37.png)
在这里对过期数据的删除不需要通过在数据库进行查询等操作，提高了对数据的处理效率，减少了不必要的运算操作
#### 分区数据迁移
创建新分区表：arch_customer_login_log
```mysql
CREATE TABLE `arch_customer_login_log`
(
    `customer_id` INT UNSIGNED NOT NULL,
    `login_time`  DATETIME     NOT NULL,
    `login_ip`    INT UNSIGNED NOT NULL,
    `login_type`  TINYINT      NOT NULL
) ENGINE = INNODB
```
当前customer_login_log 分区表中的数据：
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f964244a.png) ​
这里将p3的数据迁移到新表中：
```mysql
ALTER table customer_login_log exchange PARTITION p3 WITH TABLE arch_customer_login_log;
```

迁移后的原表 customer_login_log
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f9668c49.png)
 ​
迁移后的新表arch_customer_login_log
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f968faba.png)
 ​
新表arch_customer_login_log的分区信息：
![](https://weichengjun2.dpdns.org/i/2025/10/28/69009f96b62d9.png)

由截图可知，分区表表名为空、归档规则为空；数据量为2条
实现分区迁移的两个条件：
1. mysql版本要大于5.7；
2. 归档的分区日志表要属于非分区表，归档的分区表和迁移的分区表数据结构必须相同，并且不能有外键约束；

满足以上两个条件的多个分区之间就可以进行分区数据的迁移了.
 **归档分区表到相应的存储引擎：** 
```mysql
ALTER TABLE arch_customer_login_log ENGINE=archive
```
使用分区表的注意事项：
1. 结合业务场景选择分区键，避免跨分区查询；
2. 对分区表进行查询最好在WHERE从句中包含分区键；
3. 具有主键或唯一索引的键，主键或唯一索引必须是分区键的一部分。


