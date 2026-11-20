---
{"dg-publish":true,"permalink":"/Work/Databases/MySql/Basics/Mysql Commands/","title":"Mysql Commands","tags":["#flashcards"],"noteIcon":"","created":"2024-12-20T18:02:33.000+08:00","updated":"2026-06-17T14:43:40.624+08:00","dg-note-properties":{"title":"Mysql Commands","tags":["#flashcards"],"reference linking":null}}
---

# 配置相关
```mysql
# 显示mysql数据目录
SHOW VARIABLES LIKE 'datadir';
```
# SHOW STATUS
```mysql
# 查询当前系统有多少条慢查询记录
show global status like '%slow_queries%';
# 行锁分析
show status like '%row_lock%';
# 分析表锁定
show status like '%table_lock%'
# 查看缓存命中次数
show status like 'qcache_hits';
```
# MySQL架构介绍
```mysql
# 查看字符集
show variables like '%character%';
# 查看所有引擎
show engines;
# 查看当前表引擎
show variables like '%storage_engine%';
```
# 慢Sql查询日志
```mysql
# 查看是否开启慢查询日志
show variables like '%slow_query_log%';
# 开启慢查询日志
set global slow_query_log=1; 
# 关闭慢查询日志
set global slow_query_log=0;
# 查询慢的阙值时间
show variables like '%long_query_time%';
# 设置慢查询值时间
set long_query_time=5
set global long_query_time =5;

# s: 是表示按照何种方式排序;
# c:访问次数
# l:锁定时间
# r:返回记录
# t:查询时间
# al:平均锁定时间
# ar:平均返回记录数
# at:平均查询时间
# t:即为返回前面多少条的数据;
# g:后边搭配一个正则匹配模式,大小写不敏感的;
# 得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /usr/local/mysql/var/localhost-slow.log
# 得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /usr/local/mysql/var/localhost-slow.log
# 另外建议在使用这些命令时结合|和more使用,否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /usr/local/mysql/var/localhost-slow.log | more
```
# Show Profile
```mysql
# 查询状态
show variables like '%profiling%';
# 开启(默认为关闭状态)
set profiling=on;
# 查看结果
show profiles;

# 显示所有的开销信息
ALL              
# 显示块IO相关开销
BLOCK IO         
# 上下文切换相关开销
CONTEXT SWITCHES 
# 显示CPU相关开销信息
CPU              
# 显示发送和接收相关开销信息
IPC              
# 显示内存相关开销信息
MEMORY           
# 显示页面错误相关开销信息
PAGE FAULTS      
# 显示和source_function, source_file, source_line相关的开销信息
SOURCE           
# 显示交换次数相关开销的信息
SWAPS
# 查看sql语句执行过程
show profile cpu, block io for query 17;
```
# 全局查询日志
```mysql
# 查询是否开启
show variables like '%general_log%';
# 开启
set general_log=1;
# 查询
select * from mysql.general_log;
```
# 表锁
```mysql
# 手动增加表锁
lock table `table_name` read, `table_name2` write;
# 查看加过锁的表
show open tables;
# 释放表锁
unlock tables;
```
# 行锁
```mysql
# 排他锁
select * from test_innodb_lock where a=8 for update;
# 共享锁
select * from test_innodb_lock where a=8 lock in share mode;
```
# 解决死锁
### SHOW PROCESSLIST 对比
| 维度                 | `SHOW PROCESSLIST`     | `SHOW FULL PROCESSLIST` |
| ------------------ | ---------------------- | ----------------------- |
| **Info 列（SQL 语句）** | **截断显示**（仅前 100 字符）    | **完整显示**（全部语句）          |
| **适用场景**           | 快速查看连接状态、排除 `Sleep` 进程 | 深度分析 SQL、定位复杂的死锁或慢查询    |
| **易读性**            | 界面整齐，不会被超长 SQL 撑乱      | SQL 较长时屏幕滚动较多，排版较乱      |
### 核心差异直观图示
* **普通版**：`SELECT * FROM users WHERE status = 1 AND create_time > '2023-01-01' AND ...` （末尾被忽略）
* **FULL 版**：`SELECT * FROM users WHERE status = 1 AND create_time > '2023-01-01' AND region = 'CN' ORDER BY id DESC LIMIT 100;` （完整呈现）
### 结论
除非你只想看看有多少人在连数据库，否则**排查具体问题时，请养成直接输入 `SHOW FULL PROCESSLIST;` 的习惯**。
```mysql
# 查询数据库线程, 找出慢SQL
SHOW FULL PROCESSLIST;
# 查询数据库线程「会截断SQL」
SHOW PROCESSLIST;

# 如果你需要对进程列表进行过滤（例如：查找运行时间超过 60 秒的所有 SQL），可以使用系统表，这比肉眼看 `SHOW` 命令更高效：
SELECT *
FROM information_schema.processlist
WHERE command != 'Sleep'
  AND time > 60
ORDER BY time DESC;
```
再去查看innodb的事务表INNODB_TRX，看下里面是否有正在锁定的事务线程，看看ID是否在show full processlist里面的sleep线程中，如果是，就证明这个sleep的线程事务一直没有commit或者rollback而是卡住了，我们需要手动kill掉。
```mysql
SELECT * FROM `information_schema`.`INNODB_TRX`;
KILL 21517; # 杀死sleep的进程
```
# 查询缓存「mysql8已移除」
```mysql
# 查看查询缓存相关参数状态
show variables like '%query_cache%';
# 如果query_cache_type为1, 不用查询缓存中的数据
select sql_no_cache * from my_table where condition;
# 如果query_cache_type为2(demand), 用查询缓存中的数据
select sql_cache * from my_table where condition;
# 设置 query cache 所使用的内存大小
set global query_cache_size = 134217728;

# 清理查询缓存内存碎片。
flush query cache;
# 从查询缓存中移出所有查询。
reset query cache;
# 关闭所有打开的表，同时该操作将会清空查询缓存中的内容。
flush tables;
```
# 修改密码、允许所有连接
## 创建用户
```mysql
create user 'username'@'host' identified by 'password'; 
```
## 用户授权
- **`WITH GRANT OPTION`**：这个子句允许被授权用户将他们自己的权限再授予其他用户。这是一个非常强大的选项，应该谨慎使用。
- **`ALL PRIVILEGES`**：所有可能的权限
- **适用对象**：权限适用于所有数据库和表（`*.*`），并且允许从任何主机连接（`'%'`）。
```mysql
grant all privileges on *.* to 'username'@'%' with grant option;

#收回权限(不包含赋权权限)
REVOKE ALL PRIVILEGES ON *.* FROM user_name;
REVOKE ALL PRIVILEGES ON user_name.* FROM user_name;
#收回赋权权限
REVOKE GRANT OPTION ON *.* FROM user_name;

#操作完后重新刷新权限
flush privileges;
```
## 修改密码
```mysql
# MySQL 5.7 适用
UPDATE USER SET authentication_string = PASSWORD('123456') WHERE USER = 'root' AND HOST = 'localhost';
# MySQL 8.0 适用，因 `PASSWORD()` 函数已被弃用
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```
## 授权从任何主机都允许连接
```mysql
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
```
## 刷新权限
```mysql
flush privileges;
```
# 查看连接总数、活跃数、最大并发数
```mysql
-- 查看当前最大连接数
SHOW VARIABLES LIKE 'max_connections';
-- 查看已用连接数
SHOW STATUS LIKE 'Threads_connected';
-- 查看历史最大连接数峰值
SHOW STATUS LIKE 'Max_used_connections';
# 查看当前线程状态
SHOW STATUS LIKE 'Threads%';
# Threads_connected ：已经和连接绑定的线程
# Threads_cached ：当前线程池空余的线程数
# Threads_running ：正在运行（执行sql等）的线程数
# Threads_created ：已经创建的线程数
```
# 用户管理
## 创建只读用户
```mysql
CREATE USER bytebase_readonly@'%' IDENTIFIED BY '123456';
GRANT SELECT, SHOW DATABASES, SHOW VIEW, USAGE ON *.* to bytebase_readonly@'%';
```
## 创建读写用户
```mysql
CREATE USER bytebase@'%' IDENTIFIED BY '123456';
  
GRANT ALTER, ALTER ROUTINE, CREATE, CREATE ROUTINE, CREATE VIEW,
    DELETE, DROP, EVENT, EXECUTE, INDEX, INSERT, PROCESS, REFERENCES,
    SELECT, SHOW DATABASES, SHOW VIEW, TRIGGER, UPDATE, USAGE,
    RELOAD, LOCK TABLES, REPLICATION CLIENT, REPLICATION SLAVE
    /*!80000 , SET_USER_ID */
    ON *.* to bytebase@'%';
```
# 数据清空
## 批量删除表语句
```mysql
SELECT CONCAT('DROP TABLE ', `TABLE_NAME`, ';')
FROM `information_schema`.`TABLES`
WHERE `TABLE_SCHEMA` = '数据库名';
# 输出如下
+----------------------------------------+
|CONCAT('DROP TABLE ', `TABLE_NAME`, ';')|
+----------------------------------------+
|DROP TABLE article;                     |
|DROP TABLE country;                     |
|DROP TABLE es_news;                     |
|DROP TABLE lock;                        |
|DROP TABLE my;                          |
|DROP TABLE role;                        |
|DROP TABLE user;                        |
|DROP TABLE user_role;                   |
+----------------------------------------+
```
## 批量清空表语句
```mysql
SELECT CONCAT('TRUNCATE TABLE ', table_name, ';')
FROM `information_schema`.`TABLES`
WHERE `TABLE_SCHEMA` = '数据库名';
# 输出如下
+------------------------------------------+
|CONCAT('TRUNCATE TABLE ', table_name, ';')|
+------------------------------------------+
|TRUNCATE TABLE article;                   |
|TRUNCATE TABLE country;                   |
|TRUNCATE TABLE es_news;                   |
|TRUNCATE TABLE lock;                      |
|TRUNCATE TABLE my;                        |
|TRUNCATE TABLE role;                      |
|TRUNCATE TABLE user;                      |
|TRUNCATE TABLE user_role;                 |
+------------------------------------------+
```
## 获取某个表的所有字段名
```mysql
SELECT `COLUMN_NAME`
FROM `information_schema`.`COLUMNS`
WHERE `TABLE_SCHEMA` = '数据库名' AND `TABLE_NAME` = '表名';
```
# BINLOG 相关
## 查询是否开启 binlog
```mysql
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'log_bin_basename';
SHOW VARIABLES LIKE 'server_id';
```
### 开启binlog
```mysql
[mysqld]
server-id=1
log-bin=mysql-bin
binlog_format=ROW
# 若做主从，通常还需要：
# log_slave_updates=ON   # 从库链式复制时
# gtid_mode=ON           # 若用 GTID（MySQL 5.6+）
# enforce_gtid_consistency=ON
```
## 操作 binlog
```mysql
# 查看当前 binlog 状态
SHOW MASTER STATUS;
# 查看所有 binlog 文件
SHOW BINARY LOGS;
# 删除到指定文件之前的 binlog
PURGE BINARY LOGS TO 'binlog.000041';
# 删除指定日期之前的文件（例如删除 2025-07-01 之前的日志）
PURGE BINARY LOGS BEFORE '2025-07-01 00:00:00';
# 验证删除结果
SHOW BINARY LOGS;
# 清除7天内的binlog
PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);
```