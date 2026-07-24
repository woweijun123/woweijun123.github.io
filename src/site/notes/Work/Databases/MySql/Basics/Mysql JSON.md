---
{"dg-publish":true,"permalink":"/Work/Databases/MySql/Basics/Mysql JSON/","title":"Mysql JSON","tags":["#flashcards"],"noteIcon":"","created":"2024-03-11T11:57:37.000+08:00","updated":"2026-06-17T14:44:06.666+08:00","dg-note-properties":{"title":"Mysql JSON","tags":["#flashcards"],"reference linking":"[(20条消息) MySQL 5.7新增对JSON支持_格子的博客-CSDN博客](https://blog.csdn.net/szxiaohe/article/details/82772881)","reference linking2":"[MySQL · 最佳实践 · 如何索引JSON字段-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/303208)","reference linking3":"[mysql8 json 索引总结 - 郭慕荣 - 博客园](https://www.cnblogs.com/jelly12345/p/17399361.html)"}}
---

# 嵌套数组结构字段查询
## json结构
```json
{
  "general": {
    "prompt_icon": "",
    "button_attr": [
      {
        "jump_type": "1",
        "button_color": "#FFEEC9",
        "button_border": "#FFEEC9"
      }
    ]
  }
}
```
## 查询
```mysql
SELECT *
FROM
    `jc_active_dialog`
WHERE
    JSON_CONTAINS(`popup_ext`->"$.*.button_attr[*]", '{"jump_type":"1"}');
```
# 数组结构字段查询
查询字段数组中是否存在某个值「类似PHP中的in_array()」
## json结构
```shell
["xxx"]
```
## 查询
```mysql
SELECT `purchase_ids`
FROM `gc_invoice_income`
WHERE JSON_SEARCH(`purchase_ids`, 'one', 'xxx') IS NOT NULL;
```
# Multi-Valued Indexes（多值索引）
## 1. 创建多值索引
多值索引语法自 **MySQL 8.0.17** 起由 InnoDB 支持（参见官方 [Multi-Valued Indexes](https://dev.mysql.com/doc/refman/8.0/en/create-index.html#create-index-multi-valued)）。核心是 **`CAST( json_expr AS scalar_type ARRAY)`**：`ARRAY` **必须写上**；被索引列须为 **`JSON`** 类型；标量类型须为 **`CAST()` 所支持的类型**，多值索引中不允许 `BINARY`、`JSON`、`YEAR`（见官方 [CAST](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_cast) 说明）。

假设表名为 `users`，JSON 字段名为 `languages`（值为 JSON 数组，如 `[1,2,3]` 或 `["zh","en"]`），示例：
```mysql
-- 整数数组（无符号，与官方示例一致）
ALTER TABLE `users` ADD INDEX `idx_languages` ((CAST(`languages` AS UNSIGNED ARRAY)));

-- 整数数组（有符号）
ALTER TABLE `users` ADD INDEX `idx_languages_signed` ((CAST(`languages` AS SIGNED ARRAY)));

-- 字符串数组（CHAR 长度需不小于元素最大长度）
ALTER TABLE `users` ADD INDEX `idx_languages_str` ((CAST(`languages` AS CHAR(32) ARRAY)));
```
### 常用数据类型的多值索引写法
| **数据类型**   | **语法示例**                              |
| ---------- | ------------------------------------- |
| **无符号整数**  | `(CAST(field AS UNSIGNED ARRAY))`     |
| **带符号整数**  | `(CAST(field AS SIGNED ARRAY))`       |
| **字符串**    | `(CAST(field AS CHAR(N) ARRAY))`      |
| **日期**     | `(CAST(field AS DATE ARRAY))`         |
| **时间**     | `(CAST(field AS TIME ARRAY))`         |
| **日期时间**   | `(CAST(field AS DATETIME ARRAY))`     |
| **浮点**      | `(CAST(field AS DOUBLE ARRAY))`       |
| **定点小数** | `(CAST(field AS DECIMAL(M,D) ARRAY))` |
### 关键点说明：
- **`CAST(expr AS UNSIGNED ARRAY)`**（或 `SIGNED ARRAY` / `CHAR(n) ARRAY` 等）：多值索引必须把 JSON 数组里**同类型的标量** cast 成 **`标量类型 + ARRAY`**，MySQL 再为数组中每个元素各生成索引项。
- **双括号**：在 `ADD INDEX` 语法中，表达式索引必须包含在额外的括号内。
- **数据类型**：若数组元素为字符串（例如 `["Java", "Python"]`），使用 `CAST(... AS CHAR(n) ARRAY)`。
- **CHAR(n)**：必须指定长度。请确保长度足以容纳数组中**最长**的字符串。
## 2. 如何触发索引（查询方式）
多值索引**不会**在普通的 `=` 或 `IN` 查询中生效。你必须使用特定的 **JSON函数** 才能激活该索引：
### 使用 MEMBER OF()
查询数组中是否包含数字 `3`：
```mysql
SELECT * FROM `users` WHERE 3 MEMBER OF(`languages`);
```
### 使用 JSON_CONTAINS()
查询数组中是否包含 `[1, 2]`：
```mysql
SELECT * FROM `users` WHERE JSON_CONTAINS(`languages`, '[1, 2]');
```
### 使用 JSON_OVERLAPS()
查询数组是否与 `[4, 5]` 有交集（只要命中一个即可）：
```mysql
SELECT * FROM `users` WHERE JSON_OVERLAPS(`languages`, '[4, 5]');
```
## 3. 注意事项
- **MySQL 版本要求**：多值索引功能仅在 **MySQL 8.0.17** 及更高版本中可用。
- **字段限制**：该索引仅适用于 JSON 类型的数组字段。如果字段内容不是数组或格式非法，插入数据时可能会报错。
- **唯一性**：多值索引**可以**定义为 `UNIQUE`；语义为「任一数组元素取值在整张表索引中不得重复」，若表中已存在重复元素则建唯一索引会失败（详见官方示例）。
- **性能**：多值索引通过“一变多”的条目映射**提升查询性能**，但也**成倍增加了写入和维护的成本**。
## 4. 另一种方案：生成列索引（适用于旧版本）
如果你使用的是低于 8.0.17 的版本，或者你只想针对数组中的某个固定位置进行索引，可以使用**虚拟生成列**：
```mysql
-- 提取数组第一个元素并建立索引
ALTER TABLE users 
ADD COLUMN first_lang INT AS (languages->'$[0]') VIRTUAL,
ADD INDEX idx_first_lang (first_lang);
```
## 案例
### 表结构
```mysql
drop table if exists test_json_array;
create table test_json_array
(
    id        bigint unsigned auto_increment comment 'ID' primary key,
    name      varchar(25) default '' not null comment '名称',
    languages json                   null comment '语言列表'
)
    comment '测试json数组表';
ALTER TABLE test_json_array ADD INDEX idx_languages ((CAST(languages AS UNSIGNED ARRAY)));
```
### 生成随机数据
```mysql
DROP PROCEDURE IF EXISTS BatchGenerateTestData;
DELIMITER //
CREATE PROCEDURE BatchGenerateTestData(
    IN totalRows INT,
    IN maxRange INT
)
BEGIN
    -- 关闭自动提交以加速
    SET @old_autocommit = @@autocommit;
    SET autocommit = 0;

    -- 1. 准备基础数字内存表
    DROP TABLE IF EXISTS tmp_t10;
    CREATE TABLE tmp_t10 (n INT) ENGINE = MEMORY;
    INSERT INTO tmp_t10 VALUES (0),(1),(2),(3),(4),(5),(6),(7),(8),(9);

    -- 2. 准备 1000 行的内存表
    DROP TABLE IF EXISTS tmp_t1000;
    CREATE TABLE tmp_t1000 (n INT) ENGINE = MEMORY;
    INSERT INTO tmp_t1000
    SELECT a.n + b.n * 10 + c.n * 100
    FROM tmp_t10 a, tmp_t10 b, tmp_t10 c;

    -- 3. 动态执行插入
    -- 使用 4 个 t1000 连接，最大支持 1000^4 = 1兆条数据，足够覆盖任何 totalRows
    INSERT INTO test_json_array (name, languages)
    SELECT
        LEFT(MD5(RAND()), 12),
        (
            SELECT JSON_ARRAYAGG(val)
            FROM (
                     WITH RECURSIVE seq AS (
                         SELECT 1 AS val
                         UNION ALL
                         SELECT val + 1 FROM seq WHERE val < maxRange
                     )
                     SELECT val FROM seq WHERE RAND() > 0.5
                 ) AS lang_temp
        )
    FROM
        tmp_t1000 d1,
        tmp_t1000 d2,
        tmp_t1000 d3, -- 增加一层，上限升至 10 亿
        tmp_t1000 d4  -- 增加一层，上限升至 1 兆
    LIMIT totalRows; -- 核心：由传参控制最终生成的数量

    COMMIT;

    -- 4. 清理现场
    DROP TABLE tmp_t10;
    DROP TABLE tmp_t1000;

    SET autocommit = @old_autocommit;
END //
DELIMITER ;
```
### 测试
```mysql
# 调用存储过程「分5次构建避免: 1.Undo Log 撑爆：超大事务会占用大量磁盘空间 2.锁表时间过长：其他操作无法进入」
CALL BatchGenerateTestData(1000000, 20);
CALL BatchGenerateTestData(1000000, 20);
CALL BatchGenerateTestData(1000000, 20);
CALL BatchGenerateTestData(1000000, 20);
CALL BatchGenerateTestData(1000000, 20);

# 数据量测试
SELECT COUNT(*) FROM test_json_array;
# 查看特定语言的覆盖率
SELECT JSON_LENGTH(languages), COUNT(*) FROM test_json_array GROUP BY 1;
# 查看每个语言出现的比例  
SELECT JSON_UNQUOTE(JSON_EXTRACT(j.val, ')) as lang_id,
       COUNT(*)                               as count
FROM test_json_array,
     JSON_TABLE(test_json_array.languages, '$[*]' COLUMNS (val JSON PATH ')) j
GROUP BY lang_id;
# 测试索引命中（检查 rows 字段）
EXPLAIN SELECT * FROM test_json_array WHERE 5 MEMBER OF(languages);

# 查询示例
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[1]');
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[2]');
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[3]');
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[4]');
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[5]');
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[2, 4, 5]');
explain SELECT * FROM test_json_array WHERE JSON_CONTAINS(languages, '[4, 5]');
# 这种写法最容易触发多值索引
explain SELECT * FROM test_json_array WHERE 1 MEMBER OF(languages) OR 2 MEMBER OF(languages);
explain SELECT * FROM test_json_array WHERE 4 MEMBER OF(languages);
```