---
{"dg-publish":true,"permalink":"/Work/Databases/PostgreSql/Basics/PostgreSQL Basics/","title":"PostgreSQL Basics","tags":["PostgreSQL","MySQL","flashcards"],"noteIcon":"","created":"2026-06-17T14:30:06.017+08:00","updated":"2026-07-01T11:38:10.580+08:00","dg-note-properties":{"title":"PostgreSQL Basics","tags":["PostgreSQL","MySQL","flashcards"],"reference linking":["[[Work/Databases/MySql/Basics/Mysql Basics\|Mysql Basics]]"]}}
---

# 概念
## 与 MySQL 的核心差异（先建立心智模型）
| 维度 | MySQL | PostgreSQL |
| --- | --- | --- |
| 定位 | 偏 OLTP、Web 应用 | 标准 SQL 最完整、扩展性强 |
| 层级 | 库 → 表 | 库 → **Schema** → 表（默认 `public`） |
| 标识符 | 反引号 `` ` `` | 双引号 `"`（区分大小写）；未加引号自动转小写 |
| 自增 | `AUTO_INCREMENT` | `SERIAL` / `GENERATED … AS IDENTITY` / `SEQUENCE` |
| 布尔 | `TINYINT(1)` 模拟 | 原生 `BOOLEAN`（`true`/`false`/`null`） |
| 枚举 | 列级 `ENUM('a','b')` | 独立类型 `CREATE TYPE … AS ENUM` |
| Upsert | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT … DO UPDATE` |
| 字符串聚合 | `GROUP_CONCAT()` | `string_agg()` |
| 空值函数 | `IFNULL()` / `NULLIF()` | `COALESCE()` / `NULLIF()` |
| 分页 | `LIMIT offset, size` | `LIMIT size OFFSET offset`（MySQL 8 也支持后者） |
| NULL 排序 | 需 `CASE WHEN` 或 `(col IS NOT NULL)` | 原生 `NULLS FIRST` / `NULLS LAST` |
| 全文连接 | 不支持 `FULL OUTER JOIN` | **支持** |
| 默认隔离级别 | `REPEATABLE READ`（InnoDB） | `READ COMMITTED` |
| 读未提交 | 支持（InnoDB 实际等同 RC） | **不支持** |
| TRUNCATE | DDL，隐式提交，难回滚 | 事务内可回滚 |
| 写后返回 | MySQL 8.0.21+ 部分支持 | `INSERT/UPDATE/DELETE … RETURNING` 原生 |
| JSON | `JSON` 类型 | `JSON` + 更常用 **`JSONB`**（二进制、可索引） |
| 数组 | 无原生数组列 | 原生 `ARRAY` 类型 |
| 正则 | `REGEXP` / `RLIKE` | `~` / `~*` / `SIMILAR TO` |
| 客户端 | `mysql` | `psql`（元命令以 `\` 开头） |
| 备份 | `mysqldump` | `pg_dump` / `pg_restore` |
## SQL 分类（与 MySQL 一致）
DQL（`SELECT`）、DML（`INSERT/UPDATE/DELETE`）、DDL（`CREATE/ALTER/DROP/TRUNCATE`）、DCL（`GRANT/REVOKE`）、TCL（`BEGIN/COMMIT/ROLLBACK/SAVEPOINT`）。PostgreSQL 把 `SELECT` 也归入 DML 讨论，但日常分类与 [[Work/Databases/MySql/Basics/Mysql Basics\|Mysql Basics]] 相同。
# 认识
## 下载与安装
[https://www.postgresql.org/download/](https://www.postgresql.org/download/)
macOS：`brew install postgresql@16 && brew services start postgresql@16`
## 连接与 psql
```shell
# 连接（-W 交互输入密码；-p 默认 5432）
psql -h 127.0.0.1 -U postgres -d mydb -p 5432

# 单行执行后退出
psql -h 127.0.0.1 -U postgres -d mydb -c "SELECT version();"
```
## psql 常用元命令（对照 MySQL CLI）
| MySQL | PostgreSQL psql | 说明 |
| --- | --- | --- |
| `SHOW DATABASES` | `\l` 或 `\l+` | 列出数据库 |
| `USE db` | `\c dbname` | 切换数据库 |
| `SHOW TABLES` | `\dt` / `\dt+` | 当前 schema 的表 |
| `DESC t` | `\d t` / `\d+ t` | 表结构 |
| `SHOW CREATE TABLE t` | `\d+ t` 或查 `pg_catalog` | 无 `\G`，用 `\x` 展开行 |
| — | `\dn` | 列出 schema |
| — | `\df` | 列出函数 |
| — | `\timing` | 显示每条 SQL 耗时 |
| — | `\copy` | 客户端导入导出（非服务端 `COPY`） |
| `source file.sql` | `\i file.sql` | 执行脚本 |
| `exit` | `\q` | 退出 |
```sql
-- 查看当前连接信息
SELECT current_database(), current_user, current_schema();
-- 切换 schema 搜索路径（类似 MySQL 的 USE + 默认库）
SET search_path TO public, app;
```
# 常用数据类型
## 整数与小数（对照 MySQL）
| MySQL | PostgreSQL | 说明 |
| --- | --- | --- |
| `TINYINT` | `SMALLINT`（2 字节） | PG 无 1 字节整数列类型 |
| `SMALLINT` | `SMALLINT` | 一致 |
| `MEDIUMINT` | 无 | 用 `INTEGER` |
| `INT` / `INTEGER` | `INTEGER` | 4 字节 |
| `BIGINT` | `BIGINT` | 8 字节 |
| `INT UNSIGNED` | `BIGINT` 或 `NUMERIC` | **PG 无 UNSIGNED** |
| `FLOAT` / `DOUBLE` | `REAL` / `DOUBLE PRECISION` | 浮点 |
| `DECIMAL(M,D)` | `NUMERIC(M,D)` / `DECIMAL` | 精确小数，语义一致 |
| `BIT(M)` | `BIT(n)` / `VARBIT` | 位串 |
## 字符与二进制
| MySQL | PostgreSQL | 说明 |
| --- | --- | --- |
| `CHAR(n)` | `CHAR(n)` | 定长，尾部空格会被 trim |
| `VARCHAR(n)` | `VARCHAR(n)` | 变长 |
| `TEXT` | `TEXT` | 无长度上限（受行大小限制） |
| `BINARY` / `VARBINARY` | `BYTEA` | 二进制 |
| `ENUM` | `CREATE TYPE mood AS ENUM (...)` | PG 枚举是**独立类型**，可复用 |
| `SET` | 无 | 用 `TEXT[]` 数组或关联表 |
```sql
CREATE TYPE gender AS ENUM ('男', '女');
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(50) NOT NULL,
    gender      gender,
    tags        TEXT[],              -- 数组，类似 SET 但更灵活
    avatar      BYTEA
);
INSERT INTO users (name, gender, tags)
VALUES ('张三', '男', ARRAY['a', 'b']);
-- 查数组包含
SELECT * FROM users WHERE 'a' = ANY (tags);
```
## 日期时间（对照 MySQL datetime/timestamp）
| MySQL | PostgreSQL | 说明 |
| --- | --- | --- |
| `DATE` | `DATE` | 日期 |
| `TIME` | `TIME` | 时间 |
| `DATETIME` | `TIMESTAMP` / `TIMESTAMP WITHOUT TIME ZONE` | 无时区 |
| `TIMESTAMP` | `TIMESTAMP WITH TIME ZONE`（`timestamptz`） | **推荐**：存 UTC、按会话时区显示 |
| `YEAR` | 无 | 用 `EXTRACT(YEAR FROM d)` |
**差异要点**：PG 的 `timestamptz` 会规范化存储；MySQL `timestamp` 受 `time_zone` 影响但范围仅到 2038。PG 时间范围更大（4713 BC ~ 294276 AD）。
```sql
CREATE TABLE events (
    id        BIGSERIAL PRIMARY KEY,
    happened  TIMESTAMPTZ NOT NULL DEFAULT NOW(),  -- 等价 MySQL CURRENT_TIMESTAMP
    local_day DATE
);
```
## PostgreSQL 独有/常用类型
| 类型 | 用途 |
| --- | --- |
| `BOOLEAN` | 真/假，`WHERE active = true` |
| `UUID` | 分布式主键，需 `CREATE EXTENSION IF NOT EXISTS "uuid-ossp"` 或 PG13+ `gen_random_uuid()` |
| `JSON` / `JSONB` | 文档字段；**JSONB 可 GIN 索引**，查询更快 |
| `ARRAY` | 同列多值 |
| `INET` / `CIDR` | IP 地址 |
| `TSVECTOR` / `TSQUERY` | 全文检索 |
| `RANGE` 类型 | 区间（如 `daterange`、`int4range`） |
```sql
CREATE TABLE docs (
    id    BIGSERIAL PRIMARY KEY,
    meta  JSONB NOT NULL DEFAULT '{}'
);
-- JSONB 查询
SELECT * FROM docs WHERE meta @> '{"status":"published"}';
CREATE INDEX idx_docs_meta ON docs USING GIN (meta);
```
# 基本语法
## SELECT / JOIN / AS / ORDER BY
标准 SQL 与 MySQL 基本一致；标识符用双引号而非反引号。
```sql
SELECT u.id, u.name AS user_name
FROM "user" AS u
LEFT JOIN profile AS p ON p.user_id = u.id
ORDER BY u.id DESC, p.updated_at DESC NULLS LAST;
```
**MySQL → PG 替换**：
- `` `table` `` → `"table"`（或不用引号，全小写）
- `ORDER BY (col IS NOT NULL) DESC` → `ORDER BY col NULLS LAST`
- `IFNULL(a, b)` → `COALESCE(a, b)`
## WHERE
逻辑运算符、`BETWEEN`、`IN`、`IS NULL` 与 MySQL 相同。差异在模式匹配与正则：
```sql
-- LIKE（与 MySQL 相同，默认不区分大小写取决于 collation）
SELECT * FROM users WHERE name LIKE '%关键字%';

-- 正则（MySQL 的 REGEXP）
SELECT * FROM users WHERE email ~ '^[a-z]+@example\.com
## 表关系（与 MySQL 相同思路）
一对一、一对多、多对多中间表写法与 [[Work/Databases/MySql/Basics/Mysql Basics\|Mysql Basics]] 一致；PG 外键**默认 enforced**，无需选存储引擎。
```sql
CREATE TABLE teacher (
    id   BIGSERIAL PRIMARY KEY,
    name VARCHAR(25)
);
CREATE TABLE student (
    id   BIGSERIAL PRIMARY KEY,
    name VARCHAR(25)
);
CREATE TABLE teacher_student_map (
    teacher_id BIGINT NOT NULL REFERENCES teacher (id),
    student_id BIGINT NOT NULL REFERENCES student (id),
    PRIMARY KEY (teacher_id, student_id)
);
-- 多对多聚合（MySQL GROUP_CONCAT → string_agg）
SELECT t.id, t.name, string_agg(s.name, ',' ORDER BY s.name) AS students
FROM teacher t
JOIN teacher_student_map m ON m.teacher_id = t.id
JOIN student s ON s.id = m.student_id
GROUP BY t.id, t.name;
```
# SQL
## DDL（定义语言）
```sql
-- 库（通常 shell 里 createdb，或 superuser 执行）
CREATE DATABASE mydb ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8';
DROP DATABASE IF EXISTS mydb;
-- 表
CREATE TABLE IF NOT EXISTS users (
    id         BIGSERIAL PRIMARY KEY,
    email      VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
DROP TABLE IF EXISTS users;
TRUNCATE TABLE users RESTART IDENTITY;  -- 重置序列，类似 MySQL TRUNCATE 重置 AUTO_INCREMENT
-- 查看
\d users
\d+ users
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE id = 1;
```
## Schema 与字符集（对照 MySQL CHARSET）
MySQL 在库/表级设 `utf8mb4`；PG 在**数据库创建时**定 encoding/collation，表继承库设置。
```sql
CREATE SCHEMA IF NOT EXISTS app AUTHORIZATION current_user;
CREATE TABLE app.orders (id BIGSERIAL PRIMARY KEY);
ALTER DATABASE mydb SET client_encoding TO 'UTF8';
-- 改表所属 schema
ALTER TABLE public.orders SET SCHEMA app;
-- 改列类型
ALTER TABLE users ADD COLUMN age SMALLINT;
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(320);  -- 类似 MODIFY
ALTER TABLE users RENAME COLUMN email TO mail;           -- 类似 CHANGE 只改名
ALTER TABLE users RENAME TO accounts;                    -- 改表名
COMMENT ON TABLE users IS '用户表';
COMMENT ON COLUMN users.email IS '登录邮箱';
```
## 导入导出（对照 mysqldump）
```shell
# 逻辑备份（类似 mysqldump）
pg_dump -h 127.0.0.1 -U postgres -d mydb -Fc -f mydb.dump      # 自定义格式，可 pg_restore
pg_dump -h 127.0.0.1 -U postgres -d mydb --schema-only > schema.sql
pg_dump -h 127.0.0.1 -U postgres -d mydb --data-only > data.sql

# 恢复
pg_restore -h 127.0.0.1 -U postgres -d mydb_new mydb.dump
psql -h 127.0.0.1 -U postgres -d mydb < schema.sql

# 单表 CSV（服务端，需超级用户或 COPY 权限）
psql -c "\COPY users TO '/tmp/users.csv' CSV HEADER"
psql -c "\COPY users FROM '/tmp/users.csv' CSV HEADER"
```
## DQL（查询语言）
### LIMIT / DISTINCT / 分页
```sql
-- MySQL: LIMIT 0, 20  →  PG:
SELECT DISTINCT job_id FROM employees
ORDER BY job_id
LIMIT 20 OFFSET 0;

-- 标准 SQL 写法（PG / DB2 / SQL Server 2012+）
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```
### JOIN 对照
| 类型 | MySQL | PostgreSQL |
| --- | --- | --- |
| 内连接 | `INNER JOIN` / `JOIN` | 相同 |
| 左/右连接 | `LEFT JOIN` / `RIGHT JOIN` | 相同 |
| 全外连接 | **不支持** | `FULL OUTER JOIN` |
| 交叉连接 | `CROSS JOIN` | 相同 |
| 自然连接 | `NATURAL JOIN` | 相同（仍不推荐） |
| USING | `JOIN t USING(id)` | 相同 |
```sql
-- FULL OUTER JOIN（MySQL 需 UNION 模拟）
SELECT a.id, b.id
FROM table_a a
FULL OUTER JOIN table_b b ON a.id = b.a_id;

-- 自身连接（与 MySQL 相同）
SELECT mgr.last_name AS 上司, emp.last_name AS 员工
FROM employees mgr
JOIN employees emp ON mgr.employee_id = emp.manager_id;
```
### UNION
与 MySQL 相同：`UNION` 去重，`UNION ALL` 保留重复；列数、类型需兼容。
### 子查询 / EXISTS
`WHERE` / `SELECT` / `FROM` / `EXISTS` 子查询语法与 MySQL 高度一致。
```sql
-- EXISTS
SELECT d.department_name
FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.department_id = d.department_id
);
```
### GROUP BY / HAVING
规则与 MySQL 5.7+ `ONLY_FULL_GROUP_BY` 一致：**SELECT 中非聚合列必须出现在 GROUP BY**。
```sql
SELECT department_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 8000;
```
**MySQL `GROUP_CONCAT` → PostgreSQL `string_agg`**：
```sql
-- MySQL: GROUP_CONCAT(name ORDER BY name SEPARATOR ',')
SELECT dept_id, string_agg(name, ',' ORDER BY name) AS names
FROM employees
GROUP BY dept_id;
```
**MySQL `WITH ROLLUP` → PostgreSQL `ROLLUP` / `CUBE` / `GROUPING SETS`**：
```sql
SELECT department_id, job_id, SUM(salary)
FROM employees
GROUP BY ROLLUP (department_id, job_id);
```
### 常用函数对照
| 场景 | MySQL | PostgreSQL |
| --- | --- | --- |
| 空值 | `IFNULL(a,b)` | `COALESCE(a,b)` |
| 条件 | `IF(cond,a,b)` | `CASE WHEN cond THEN a ELSE b END` |
| 字符串拼接 | `CONCAT(a,b)` | `a \|\| b` 或 `CONCAT(a,b)` |
| 截取 | `SUBSTRING(str, pos, len)` | `SUBSTRING(str FROM pos FOR len)` |
| 长度 | `CHAR_LENGTH` / `LENGTH` | `char_length` / `length` / `octet_length` |
| 去空格 | `TRIM` | `TRIM` / `BTRIM` / `LTRIM` / `RTRIM` |
| 替换 | `REPLACE` | `REPLACE` |
| 填充 | `LPAD` / `RPAD` | `LPAD` / `RPAD` |
| 索引 | `LOCATE` / `INSTR` | `POSITION(sub IN str)` / `STRPOS` |
| 聚合拼接 | `GROUP_CONCAT` | `string_agg` |
| 条件聚合 | `SUM(IF(...))` | `SUM(CASE WHEN ... END)` 或 `FILTER` |
| 时间格式化 | `DATE_FORMAT` | `TO_CHAR(ts, 'YYYY-MM-DD')` |
| 字符串转时间 | `STR_TO_DATE` | `TO_TIMESTAMP` |
| 当前时间 | `NOW()` | `NOW()` / `CURRENT_TIMESTAMP` |
| 时间差 | `TIMESTAMPDIFF` | `AGE(t1, t2)` 或 `t1 - t2` |
| MD5 | `MD5()` | `MD5()`（扩展 `pgcrypto` 更安全） |
| UUID | `UUID()` | `gen_random_uuid()` |
**FILTER 子句（PG 特色，替代 SUM(IF)）**：
```sql
SELECT
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE status = 'active') AS active_cnt
FROM users;
```
**FIND_IN_SET 替代**：
```sql
-- MySQL: FIND_IN_SET('1', area)
SELECT * FROM t WHERE '1' = ANY (string_to_array(area, ','));
-- 或数组列: WHERE 1 = ANY (area_ids)
```
## DML（操纵语言）
### 基础 CRUD
```sql
INSERT INTO users (email) VALUES ('a@b.com');
UPDATE users SET email = 'c@d.com' WHERE id = 1;
DELETE FROM users WHERE id = 1;
```
### RETURNING（PG 强项，MySQL 8.0.21+ 才部分支持）
```sql
INSERT INTO users (email) VALUES ('x@y.com') RETURNING id, created_at;
UPDATE users SET email = 'new@x.com' WHERE id = 1 RETURNING *;
DELETE FROM users WHERE id = 1 RETURNING id;
```
### Upsert（对照 replace / on duplicate key）
```sql
-- MySQL: INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO users (id, email, age)
VALUES (1, 'z3@x.com', 18)
ON CONFLICT (id) DO UPDATE
SET email = EXCLUDED.email, age = EXCLUDED.age;

-- 冲突则忽略
INSERT INTO users (email) VALUES ('dup@x.com')
ON CONFLICT (email) DO NOTHING;

-- 指定约束名（复合唯一索引）
ON CONFLICT ON CONSTRAINT users_email_key DO UPDATE SET ...
```
**注意**：必须有 **PRIMARY KEY 或 UNIQUE** 约束；PG **无 `REPLACE INTO`**。
### 多表 UPDATE / DELETE
PG 支持 `UPDATE … FROM` / `DELETE … USING`，比 MySQL 多表语法更标准。
```sql
-- 修改（对照 MySQL UPDATE a JOIN b）
UPDATE beauty b
SET phone = 114
FROM boys bo
WHERE b.boyfriend_id = bo.id AND bo.boy_name = '张无忌';

-- 删除
DELETE FROM beauty b
USING boys bo
WHERE b.boyfriend_id = bo.id AND bo.boy_name = '张无忌';
```
### INSERT … SELECT（复制表）
```sql
-- 仅结构
CREATE TABLE users_copy (LIKE users INCLUDING ALL);
-- 结构 + 全量数据
CREATE TABLE users_copy2 AS SELECT * FROM users;
-- 结构 + 部分数据
CREATE TABLE users_copy3 AS SELECT * FROM users WHERE id IN (1, 2);
```
### DELETE vs TRUNCATE
| 对比项 | DELETE | TRUNCATE |
| --- | --- | --- |
| WHERE | 支持 | 不支持 |
| 速度 | 逐行慢 | 快 |
| 序列重置 | 不重置 | `RESTART IDENTITY` 可重置 |
| 回滚 | 事务内可回滚 | **事务内可回滚**（与 MySQL 不同） |
| 触发器 | 逐行触发 | 不触发 `FOR EACH ROW`（PG 11+ 有 `TRUNCATE` 触发器） |
# 约束
## 对照 MySQL 约束写法
| MySQL | PostgreSQL |
| --- | --- |
| `PRIMARY KEY` | `PRIMARY KEY` |
| `AUTO_INCREMENT` | `SERIAL` / `GENERATED BY DEFAULT AS IDENTITY` |
| `UNIQUE` | `UNIQUE` 或 `CONSTRAINT name UNIQUE` |
| `NOT NULL` | `NOT NULL` |
| `DEFAULT` | `DEFAULT` |
| `FOREIGN KEY … REFERENCES` | 相同，默认 `NO ACTION` |
| `ON DELETE CASCADE` | 相同 |
```sql
CREATE TABLE orders (
    id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id      BIGINT NOT NULL REFERENCES users (id) ON DELETE CASCADE,
    order_no     VARCHAR(32) NOT NULL,
    amount       NUMERIC(12, 2) NOT NULL DEFAULT 0,
    status       SMALLINT NOT NULL DEFAULT 1,
    CONSTRAINT uk_order_no UNIQUE (order_no)
);
-- 事后加外键
ALTER TABLE orders ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE SET NULL;
ALTER TABLE orders DROP CONSTRAINT fk_user;
```
## 自增 / 序列（对照 AUTO_INCREMENT）
```sql
-- 方式 1：BIGSERIAL（语法糖，背后创建 sequence）
CREATE TABLE t1 (id BIGSERIAL PRIMARY KEY);

-- 方式 2：SQL 标准 IDENTITY（推荐新项目）
CREATE TABLE t2 (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY
);

-- 手动改序列起点（对照 ALTER TABLE AUTO_INCREMENT = n）
SELECT setval(pg_get_serial_sequence('users', 'id'), 1943762, false);

-- 查看序列当前值
SELECT currval(pg_get_serial_sequence('users', 'id'));
```
## 索引（对照 MySQL）
```sql
CREATE INDEX idx_users_email ON users (email);
CREATE UNIQUE INDEX uk_users_phone ON users (phone);
CREATE INDEX idx_docs_meta ON docs USING GIN (meta);           -- JSONB
CREATE INDEX idx_users_name_lower ON users (lower(name));     -- 表达式索引
-- 部分索引（PG 特色）
CREATE INDEX idx_active_users ON users (created_at) WHERE status = 1;
-- 并发建索引（不长时间锁写）
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);
```
# 事务
## ACID 与 MySQL 对比
语义相同；实现不同：MySQL InnoDB 用 undo/redo + Read View；PostgreSQL 用 **xmin/x xmax + clog** 判断元组可见性，无 undo log 回滚旧行，而是写新行版本 + vacuum 清理死元组。
## 使用语法
```sql
BEGIN;                    -- 或 START TRANSACTION
-- 多条 DML
COMMIT;
-- 或
ROLLBACK;

SAVEPOINT sp1;
ROLLBACK TO SAVEPOINT sp1;
RELEASE SAVEPOINT sp1;
```
**差异**：
- PG 默认 **`READ COMMITTED`**：每条语句看到已提交快照；MySQL InnoDB 默认 **RR**。
- PG **无 `READ UNCOMMITTED`**。
- `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;` 在 PG 中 RR 也能防幻读（通过 SI / 锁升级）。
- DDL（`CREATE TABLE` 等）**可在事务中回滚**（MySQL 多数 DDL 隐式提交）。
```sql
SHOW transaction_isolation;   -- 查看当前隔离级别
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL READ COMMITTED;
```
## 并发问题（与 MySQL 相同概念）
脏读、不可重复读、幻读；PG 在 RC 下允许不可重复读；RR/SERIALIZABLE 更严格。写冲突可用 `SELECT … FOR UPDATE` / `FOR SHARE` / `SKIP LOCKED` / `NOWAIT`。
```sql
-- 悲观锁
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- 跳过已锁行（队列/任务抢占，MySQL 8+ 也支持 SKIP LOCKED）
SELECT * FROM jobs WHERE status = 'pending' FOR UPDATE SKIP LOCKED LIMIT 1;
```
# 视图
语法与 MySQL 类似；PG 视图默认可更新需满足规则（无聚合、DISTINCT 等）。
```sql
CREATE OR REPLACE VIEW v_active_users AS
SELECT id, email FROM users WHERE status = 1;

SELECT * FROM v_active_users;

DROP VIEW IF EXISTS v_active_users;
```
# 函数与存储过程
## 对照 MySQL 函数/存储过程
| MySQL | PostgreSQL |
| --- | --- |
| `DELIMITER $$` | 用 `$$ … $$` 或 `$func$` 美元引号 |
| `@变量` | `plpgsql` 内变量，或 `SET` 会话参数 |
| `CREATE PROCEDURE` | `CREATE PROCEDURE`（PG11+）或 **`CREATE FUNCTION`** 更常见 |
| 函数必须有返回值 | `RETURNS void` / `RETURNS TABLE` / `RETURNS SETOF` |
```sql
-- 标量函数
CREATE OR REPLACE FUNCTION dept_avg_salary(p_dept TEXT)
RETURNS NUMERIC
LANGUAGE sql
STABLE
AS $
    SELECT AVG(salary) FROM employees WHERE department_name = p_dept;
$;
SELECT dept_avg_salary('IT');

-- plpgsql 过程（无返回值）
CREATE OR REPLACE PROCEDURE reset_stale_orders()
LANGUAGE plpgsql
AS $
BEGIN
    UPDATE orders SET status = 0 WHERE updated_at < NOW() - INTERVAL '7 days';
END;
$;
CALL reset_stale_orders();

-- 返回多行
CREATE OR REPLACE FUNCTION list_users()
RETURNS TABLE (id BIGINT, email VARCHAR)
LANGUAGE sql
AS $
    SELECT id, email FROM users;
$;
SELECT * FROM list_users();
```
## DO 块（匿名代码，MySQL 无直接等价）
```sql
DO $
DECLARE
    v_sum INT := 0;
BEGIN
    v_sum := 1 + 2;
    RAISE NOTICE 'sum = %', v_sum;
END;
$;
```
# CTE 与窗口函数（PG 强项）
MySQL 8.0+ 也支持 CTE；PG 更早、更完整（含递归、写 CTE）。
```sql
-- 递归 CTE（组织树、路径）
WITH RECURSIVE subordinates AS (
    SELECT employee_id, manager_id, last_name, 1 AS depth
    FROM employees WHERE employee_id = 100
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.last_name, s.depth + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.employee_id
)
SELECT * FROM subordinates;

-- 窗口函数
SELECT
    department_id,
    last_name,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
FROM employees;
```
# 权限（DCL）
```sql
CREATE ROLE app_rw LOGIN PASSWORD 'secret';
GRANT CONNECT ON DATABASE mydb TO app_rw;
GRANT USAGE ON SCHEMA public TO app_rw;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_rw;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_rw;  -- 序列权限，MySQL 无此概念
REVOKE INSERT ON users FROM app_rw;
```
# MySQL 迁移 PostgreSQL 速查
| MySQL 写法 | PostgreSQL 写法 |
| --- | --- |
| `` `col` `` | `"col"` 或 `col` |
| `AUTO_INCREMENT` | `BIGSERIAL` 或 `GENERATED AS IDENTITY` |
| `LIMIT 10, 20` | `LIMIT 20 OFFSET 10` |
| `IFNULL(a,b)` | `COALESCE(a,b)` |
| `GROUP_CONCAT(x)` | `string_agg(x::text, ',')` |
| `ON DUPLICATE KEY UPDATE` | `ON CONFLICT … DO UPDATE` |
| `REPLACE INTO` | `INSERT … ON CONFLICT DO UPDATE` |
| `FIND_IN_SET(v, col)` | `v = ANY(string_to_array(col, ','))` |
| `DATE_FORMAT(d, '%Y-%m')` | `TO_CHAR(d, 'YYYY-MM')` |
| `UNIX_TIMESTAMP()` | `EXTRACT(EPOCH FROM NOW())` |
| `JSON_EXTRACT(j, '$.a')` | `j->>'a'` 或 `j #>> '{a,b}'` |
| `REGEXP 'pat'` | `~ 'pat'` 或 `~* 'pat'` |
| `BOOLEAN` 用 0/1 | 用 `true`/`false` |
| `ENGINE=InnoDB` | 删除（无引擎选项） |
| `UNSIGNED` | 换更大类型或 `CHECK (col >= 0)` |
| `TINYINT(1)` 布尔 | `BOOLEAN` |
| `DATETIME` | `TIMESTAMP` 或 `TIMESTAMPTZ` |
| `SHOW TABLES` | `\dt` 或查 `information_schema.tables` |
| `DESCRIBE t` | `\d t` |
# 一句话总结
会 MySQL 的上手 PostgreSQL：**SQL 主干相同**，重点改 **Schema/序列/Upsert/聚合函数/JSONB/RETURNING/隔离级别默认值**，并用 **psql + EXPLAIN ANALYZE** 替代 mysql 习惯；标准 SQL 与高级特性（CTE、窗口、数组、FULL JOIN）在 PG 里更完整，值得优先掌握。
;      -- 区分大小写
SELECT * FROM users WHERE email ~* '^[a-z]+@example\.com
## 表关系（与 MySQL 相同思路）
一对一、一对多、多对多中间表写法与 [[Work/Databases/MySql/Basics/Mysql Basics\|Mysql Basics]] 一致；PG 外键**默认 enforced**，无需选存储引擎。
{{CODE_BLOCK_7}}
# SQL
## DDL（定义语言）
{{CODE_BLOCK_8}}
## Schema 与字符集（对照 MySQL CHARSET）
MySQL 在库/表级设 `utf8mb4`；PG 在**数据库创建时**定 encoding/collation，表继承库设置。
{{CODE_BLOCK_9}}
## 导入导出（对照 mysqldump）
{{CODE_BLOCK_10}}
## DQL（查询语言）
### LIMIT / DISTINCT / 分页
{{CODE_BLOCK_11}}
### JOIN 对照
| 类型 | MySQL | PostgreSQL |
| --- | --- | --- |
| 内连接 | `INNER JOIN` / `JOIN` | 相同 |
| 左/右连接 | `LEFT JOIN` / `RIGHT JOIN` | 相同 |
| 全外连接 | **不支持** | `FULL OUTER JOIN` |
| 交叉连接 | `CROSS JOIN` | 相同 |
| 自然连接 | `NATURAL JOIN` | 相同（仍不推荐） |
| USING | `JOIN t USING(id)` | 相同 |
{{CODE_BLOCK_12}}
### UNION
与 MySQL 相同：`UNION` 去重，`UNION ALL` 保留重复；列数、类型需兼容。
### 子查询 / EXISTS
`WHERE` / `SELECT` / `FROM` / `EXISTS` 子查询语法与 MySQL 高度一致。
{{CODE_BLOCK_13}}
### GROUP BY / HAVING
规则与 MySQL 5.7+ `ONLY_FULL_GROUP_BY` 一致：**SELECT 中非聚合列必须出现在 GROUP BY**。
{{CODE_BLOCK_14}}
**MySQL `GROUP_CONCAT` → PostgreSQL `string_agg`**：
{{CODE_BLOCK_15}}
**MySQL `WITH ROLLUP` → PostgreSQL `ROLLUP` / `CUBE` / `GROUPING SETS`**：
{{CODE_BLOCK_16}}
### 常用函数对照
| 场景 | MySQL | PostgreSQL |
| --- | --- | --- |
| 空值 | `IFNULL(a,b)` | `COALESCE(a,b)` |
| 条件 | `IF(cond,a,b)` | `CASE WHEN cond THEN a ELSE b END` |
| 字符串拼接 | `CONCAT(a,b)` | `a \|\| b` 或 `CONCAT(a,b)` |
| 截取 | `SUBSTRING(str, pos, len)` | `SUBSTRING(str FROM pos FOR len)` |
| 长度 | `CHAR_LENGTH` / `LENGTH` | `char_length` / `length` / `octet_length` |
| 去空格 | `TRIM` | `TRIM` / `BTRIM` / `LTRIM` / `RTRIM` |
| 替换 | `REPLACE` | `REPLACE` |
| 填充 | `LPAD` / `RPAD` | `LPAD` / `RPAD` |
| 索引 | `LOCATE` / `INSTR` | `POSITION(sub IN str)` / `STRPOS` |
| 聚合拼接 | `GROUP_CONCAT` | `string_agg` |
| 条件聚合 | `SUM(IF(...))` | `SUM(CASE WHEN ... END)` 或 `FILTER` |
| 时间格式化 | `DATE_FORMAT` | `TO_CHAR(ts, 'YYYY-MM-DD')` |
| 字符串转时间 | `STR_TO_DATE` | `TO_TIMESTAMP` |
| 当前时间 | `NOW()` | `NOW()` / `CURRENT_TIMESTAMP` |
| 时间差 | `TIMESTAMPDIFF` | `AGE(t1, t2)` 或 `t1 - t2` |
| MD5 | `MD5()` | `MD5()`（扩展 `pgcrypto` 更安全） |
| UUID | `UUID()` | `gen_random_uuid()` |
**FILTER 子句（PG 特色，替代 SUM(IF)）**：
{{CODE_BLOCK_17}}
**FIND_IN_SET 替代**：
{{CODE_BLOCK_18}}
## DML（操纵语言）
### 基础 CRUD
{{CODE_BLOCK_19}}
### RETURNING（PG 强项，MySQL 8.0.21+ 才部分支持）
{{CODE_BLOCK_20}}
### Upsert（对照 replace / on duplicate key）
{{CODE_BLOCK_21}}
**注意**：必须有 **PRIMARY KEY 或 UNIQUE** 约束；PG **无 `REPLACE INTO`**。
### 多表 UPDATE / DELETE
PG 支持 `UPDATE … FROM` / `DELETE … USING`，比 MySQL 多表语法更标准。
{{CODE_BLOCK_22}}
### INSERT … SELECT（复制表）
{{CODE_BLOCK_23}}
### DELETE vs TRUNCATE
| 对比项 | DELETE | TRUNCATE |
| --- | --- | --- |
| WHERE | 支持 | 不支持 |
| 速度 | 逐行慢 | 快 |
| 序列重置 | 不重置 | `RESTART IDENTITY` 可重置 |
| 回滚 | 事务内可回滚 | **事务内可回滚**（与 MySQL 不同） |
| 触发器 | 逐行触发 | 不触发 `FOR EACH ROW`（PG 11+ 有 `TRUNCATE` 触发器） |
# 约束
## 对照 MySQL 约束写法
| MySQL | PostgreSQL |
| --- | --- |
| `PRIMARY KEY` | `PRIMARY KEY` |
| `AUTO_INCREMENT` | `SERIAL` / `GENERATED BY DEFAULT AS IDENTITY` |
| `UNIQUE` | `UNIQUE` 或 `CONSTRAINT name UNIQUE` |
| `NOT NULL` | `NOT NULL` |
| `DEFAULT` | `DEFAULT` |
| `FOREIGN KEY … REFERENCES` | 相同，默认 `NO ACTION` |
| `ON DELETE CASCADE` | 相同 |
{{CODE_BLOCK_24}}
## 自增 / 序列（对照 AUTO_INCREMENT）
{{CODE_BLOCK_25}}
## 索引（对照 MySQL）
{{CODE_BLOCK_26}}
# 事务
## ACID 与 MySQL 对比
语义相同；实现不同：MySQL InnoDB 用 undo/redo + Read View；PostgreSQL 用 **xmin/x xmax + clog** 判断元组可见性，无 undo log 回滚旧行，而是写新行版本 + vacuum 清理死元组。
## 使用语法
{{CODE_BLOCK_27}}
**差异**：
- PG 默认 **`READ COMMITTED`**：每条语句看到已提交快照；MySQL InnoDB 默认 **RR**。
- PG **无 `READ UNCOMMITTED`**。
- `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;` 在 PG 中 RR 也能防幻读（通过 SI / 锁升级）。
- DDL（`CREATE TABLE` 等）**可在事务中回滚**（MySQL 多数 DDL 隐式提交）。
{{CODE_BLOCK_28}}
## 并发问题（与 MySQL 相同概念）
脏读、不可重复读、幻读；PG 在 RC 下允许不可重复读；RR/SERIALIZABLE 更严格。写冲突可用 `SELECT … FOR UPDATE` / `FOR SHARE` / `SKIP LOCKED` / `NOWAIT`。
{{CODE_BLOCK_29}}
# 视图
语法与 MySQL 类似；PG 视图默认可更新需满足规则（无聚合、DISTINCT 等）。
{{CODE_BLOCK_30}}
# 函数与存储过程
## 对照 MySQL 函数/存储过程
| MySQL | PostgreSQL |
| --- | --- |
| `DELIMITER $$` | 用 `$$ … $$` 或 `$func$` 美元引号 |
| `@变量` | `plpgsql` 内变量，或 `SET` 会话参数 |
| `CREATE PROCEDURE` | `CREATE PROCEDURE`（PG11+）或 **`CREATE FUNCTION`** 更常见 |
| 函数必须有返回值 | `RETURNS void` / `RETURNS TABLE` / `RETURNS SETOF` |
{{CODE_BLOCK_31}}
## DO 块（匿名代码，MySQL 无直接等价）
{{CODE_BLOCK_32}}
# CTE 与窗口函数（PG 强项）
MySQL 8.0+ 也支持 CTE；PG 更早、更完整（含递归、写 CTE）。
{{CODE_BLOCK_33}}
# 权限（DCL）
{{CODE_BLOCK_34}}
# MySQL 迁移 PostgreSQL 速查
| MySQL 写法 | PostgreSQL 写法 |
| --- | --- |
| `` `col` `` | `"col"` 或 `col` |
| `AUTO_INCREMENT` | `BIGSERIAL` 或 `GENERATED AS IDENTITY` |
| `LIMIT 10, 20` | `LIMIT 20 OFFSET 10` |
| `IFNULL(a,b)` | `COALESCE(a,b)` |
| `GROUP_CONCAT(x)` | `string_agg(x::text, ',')` |
| `ON DUPLICATE KEY UPDATE` | `ON CONFLICT … DO UPDATE` |
| `REPLACE INTO` | `INSERT … ON CONFLICT DO UPDATE` |
| `FIND_IN_SET(v, col)` | `v = ANY(string_to_array(col, ','))` |
| `DATE_FORMAT(d, '%Y-%m')` | `TO_CHAR(d, 'YYYY-MM')` |
| `UNIX_TIMESTAMP()` | `EXTRACT(EPOCH FROM NOW())` |
| `JSON_EXTRACT(j, '$.a')` | `j->>'a'` 或 `j #>> '{a,b}'` |
| `REGEXP 'pat'` | `~ 'pat'` 或 `~* 'pat'` |
| `BOOLEAN` 用 0/1 | 用 `true`/`false` |
| `ENGINE=InnoDB` | 删除（无引擎选项） |
| `UNSIGNED` | 换更大类型或 `CHECK (col >= 0)` |
| `TINYINT(1)` 布尔 | `BOOLEAN` |
| `DATETIME` | `TIMESTAMP` 或 `TIMESTAMPTZ` |
| `SHOW TABLES` | `\dt` 或查 `information_schema.tables` |
| `DESCRIBE t` | `\d t` |
# 一句话总结
会 MySQL 的上手 PostgreSQL：**SQL 主干相同**，重点改 **Schema/序列/Upsert/聚合函数/JSONB/RETURNING/隔离级别默认值**，并用 **psql + EXPLAIN ANALYZE** 替代 mysql 习惯；标准 SQL 与高级特性（CTE、窗口、数组、FULL JOIN）在 PG 里更完整，值得优先掌握。
;     -- 不区分大小写

-- ILIKE：不区分大小写的 LIKE（MySQL 需 COLLATE 或 LOWER）
SELECT * FROM users WHERE name ILIKE '%abc%';
```
## 表关系（与 MySQL 相同思路）
一对一、一对多、多对多中间表写法与 [[Work/Databases/MySql/Basics/Mysql Basics\|Mysql Basics]] 一致；PG 外键**默认 enforced**，无需选存储引擎。
{{CODE_BLOCK_7}}
# SQL
## DDL（定义语言）
{{CODE_BLOCK_8}}
## Schema 与字符集（对照 MySQL CHARSET）
MySQL 在库/表级设 `utf8mb4`；PG 在**数据库创建时**定 encoding/collation，表继承库设置。
{{CODE_BLOCK_9}}
## 导入导出（对照 mysqldump）
{{CODE_BLOCK_10}}
## DQL（查询语言）
### LIMIT / DISTINCT / 分页
{{CODE_BLOCK_11}}
### JOIN 对照
| 类型 | MySQL | PostgreSQL |
| --- | --- | --- |
| 内连接 | `INNER JOIN` / `JOIN` | 相同 |
| 左/右连接 | `LEFT JOIN` / `RIGHT JOIN` | 相同 |
| 全外连接 | **不支持** | `FULL OUTER JOIN` |
| 交叉连接 | `CROSS JOIN` | 相同 |
| 自然连接 | `NATURAL JOIN` | 相同（仍不推荐） |
| USING | `JOIN t USING(id)` | 相同 |
{{CODE_BLOCK_12}}
### UNION
与 MySQL 相同：`UNION` 去重，`UNION ALL` 保留重复；列数、类型需兼容。
### 子查询 / EXISTS
`WHERE` / `SELECT` / `FROM` / `EXISTS` 子查询语法与 MySQL 高度一致。
{{CODE_BLOCK_13}}
### GROUP BY / HAVING
规则与 MySQL 5.7+ `ONLY_FULL_GROUP_BY` 一致：**SELECT 中非聚合列必须出现在 GROUP BY**。
{{CODE_BLOCK_14}}
**MySQL `GROUP_CONCAT` → PostgreSQL `string_agg`**：
{{CODE_BLOCK_15}}
**MySQL `WITH ROLLUP` → PostgreSQL `ROLLUP` / `CUBE` / `GROUPING SETS`**：
{{CODE_BLOCK_16}}
### 常用函数对照
| 场景 | MySQL | PostgreSQL |
| --- | --- | --- |
| 空值 | `IFNULL(a,b)` | `COALESCE(a,b)` |
| 条件 | `IF(cond,a,b)` | `CASE WHEN cond THEN a ELSE b END` |
| 字符串拼接 | `CONCAT(a,b)` | `a \|\| b` 或 `CONCAT(a,b)` |
| 截取 | `SUBSTRING(str, pos, len)` | `SUBSTRING(str FROM pos FOR len)` |
| 长度 | `CHAR_LENGTH` / `LENGTH` | `char_length` / `length` / `octet_length` |
| 去空格 | `TRIM` | `TRIM` / `BTRIM` / `LTRIM` / `RTRIM` |
| 替换 | `REPLACE` | `REPLACE` |
| 填充 | `LPAD` / `RPAD` | `LPAD` / `RPAD` |
| 索引 | `LOCATE` / `INSTR` | `POSITION(sub IN str)` / `STRPOS` |
| 聚合拼接 | `GROUP_CONCAT` | `string_agg` |
| 条件聚合 | `SUM(IF(...))` | `SUM(CASE WHEN ... END)` 或 `FILTER` |
| 时间格式化 | `DATE_FORMAT` | `TO_CHAR(ts, 'YYYY-MM-DD')` |
| 字符串转时间 | `STR_TO_DATE` | `TO_TIMESTAMP` |
| 当前时间 | `NOW()` | `NOW()` / `CURRENT_TIMESTAMP` |
| 时间差 | `TIMESTAMPDIFF` | `AGE(t1, t2)` 或 `t1 - t2` |
| MD5 | `MD5()` | `MD5()`（扩展 `pgcrypto` 更安全） |
| UUID | `UUID()` | `gen_random_uuid()` |
**FILTER 子句（PG 特色，替代 SUM(IF)）**：
{{CODE_BLOCK_17}}
**FIND_IN_SET 替代**：
{{CODE_BLOCK_18}}
## DML（操纵语言）
### 基础 CRUD
{{CODE_BLOCK_19}}
### RETURNING（PG 强项，MySQL 8.0.21+ 才部分支持）
{{CODE_BLOCK_20}}
### Upsert（对照 replace / on duplicate key）
{{CODE_BLOCK_21}}
**注意**：必须有 **PRIMARY KEY 或 UNIQUE** 约束；PG **无 `REPLACE INTO`**。
### 多表 UPDATE / DELETE
PG 支持 `UPDATE … FROM` / `DELETE … USING`，比 MySQL 多表语法更标准。
{{CODE_BLOCK_22}}
### INSERT … SELECT（复制表）
{{CODE_BLOCK_23}}
### DELETE vs TRUNCATE
| 对比项 | DELETE | TRUNCATE |
| --- | --- | --- |
| WHERE | 支持 | 不支持 |
| 速度 | 逐行慢 | 快 |
| 序列重置 | 不重置 | `RESTART IDENTITY` 可重置 |
| 回滚 | 事务内可回滚 | **事务内可回滚**（与 MySQL 不同） |
| 触发器 | 逐行触发 | 不触发 `FOR EACH ROW`（PG 11+ 有 `TRUNCATE` 触发器） |
# 约束
## 对照 MySQL 约束写法
| MySQL | PostgreSQL |
| --- | --- |
| `PRIMARY KEY` | `PRIMARY KEY` |
| `AUTO_INCREMENT` | `SERIAL` / `GENERATED BY DEFAULT AS IDENTITY` |
| `UNIQUE` | `UNIQUE` 或 `CONSTRAINT name UNIQUE` |
| `NOT NULL` | `NOT NULL` |
| `DEFAULT` | `DEFAULT` |
| `FOREIGN KEY … REFERENCES` | 相同，默认 `NO ACTION` |
| `ON DELETE CASCADE` | 相同 |
{{CODE_BLOCK_24}}
## 自增 / 序列（对照 AUTO_INCREMENT）
{{CODE_BLOCK_25}}
## 索引（对照 MySQL）
{{CODE_BLOCK_26}}
# 事务
## ACID 与 MySQL 对比
语义相同；实现不同：MySQL InnoDB 用 undo/redo + Read View；PostgreSQL 用 **xmin/x xmax + clog** 判断元组可见性，无 undo log 回滚旧行，而是写新行版本 + vacuum 清理死元组。
## 使用语法
{{CODE_BLOCK_27}}
**差异**：
- PG 默认 **`READ COMMITTED`**：每条语句看到已提交快照；MySQL InnoDB 默认 **RR**。
- PG **无 `READ UNCOMMITTED`**。
- `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;` 在 PG 中 RR 也能防幻读（通过 SI / 锁升级）。
- DDL（`CREATE TABLE` 等）**可在事务中回滚**（MySQL 多数 DDL 隐式提交）。
{{CODE_BLOCK_28}}
## 并发问题（与 MySQL 相同概念）
脏读、不可重复读、幻读；PG 在 RC 下允许不可重复读；RR/SERIALIZABLE 更严格。写冲突可用 `SELECT … FOR UPDATE` / `FOR SHARE` / `SKIP LOCKED` / `NOWAIT`。
{{CODE_BLOCK_29}}
# 视图
语法与 MySQL 类似；PG 视图默认可更新需满足规则（无聚合、DISTINCT 等）。
{{CODE_BLOCK_30}}
# 函数与存储过程
## 对照 MySQL 函数/存储过程
| MySQL | PostgreSQL |
| --- | --- |
| `DELIMITER $$` | 用 `$$ … $$` 或 `$func$` 美元引号 |
| `@变量` | `plpgsql` 内变量，或 `SET` 会话参数 |
| `CREATE PROCEDURE` | `CREATE PROCEDURE`（PG11+）或 **`CREATE FUNCTION`** 更常见 |
| 函数必须有返回值 | `RETURNS void` / `RETURNS TABLE` / `RETURNS SETOF` |
{{CODE_BLOCK_31}}
## DO 块（匿名代码，MySQL 无直接等价）
{{CODE_BLOCK_32}}
# CTE 与窗口函数（PG 强项）
MySQL 8.0+ 也支持 CTE；PG 更早、更完整（含递归、写 CTE）。
{{CODE_BLOCK_33}}
# 权限（DCL）
{{CODE_BLOCK_34}}
# MySQL 迁移 PostgreSQL 速查
| MySQL 写法 | PostgreSQL 写法 |
| --- | --- |
| `` `col` `` | `"col"` 或 `col` |
| `AUTO_INCREMENT` | `BIGSERIAL` 或 `GENERATED AS IDENTITY` |
| `LIMIT 10, 20` | `LIMIT 20 OFFSET 10` |
| `IFNULL(a,b)` | `COALESCE(a,b)` |
| `GROUP_CONCAT(x)` | `string_agg(x::text, ',')` |
| `ON DUPLICATE KEY UPDATE` | `ON CONFLICT … DO UPDATE` |
| `REPLACE INTO` | `INSERT … ON CONFLICT DO UPDATE` |
| `FIND_IN_SET(v, col)` | `v = ANY(string_to_array(col, ','))` |
| `DATE_FORMAT(d, '%Y-%m')` | `TO_CHAR(d, 'YYYY-MM')` |
| `UNIX_TIMESTAMP()` | `EXTRACT(EPOCH FROM NOW())` |
| `JSON_EXTRACT(j, '$.a')` | `j->>'a'` 或 `j #>> '{a,b}'` |
| `REGEXP 'pat'` | `~ 'pat'` 或 `~* 'pat'` |
| `BOOLEAN` 用 0/1 | 用 `true`/`false` |
| `ENGINE=InnoDB` | 删除（无引擎选项） |
| `UNSIGNED` | 换更大类型或 `CHECK (col >= 0)` |
| `TINYINT(1)` 布尔 | `BOOLEAN` |
| `DATETIME` | `TIMESTAMP` 或 `TIMESTAMPTZ` |
| `SHOW TABLES` | `\dt` 或查 `information_schema.tables` |
| `DESCRIBE t` | `\d t` |
# 一句话总结
会 MySQL 的上手 PostgreSQL：**SQL 主干相同**，重点改 **Schema/序列/Upsert/聚合函数/JSONB/RETURNING/隔离级别默认值**，并用 **psql + EXPLAIN ANALYZE** 替代 mysql 习惯；标准 SQL 与高级特性（CTE、窗口、数组、FULL JOIN）在 PG 里更完整，值得优先掌握。
