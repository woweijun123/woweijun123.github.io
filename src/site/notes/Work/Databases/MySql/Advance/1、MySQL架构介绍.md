---
{"dg-publish":true,"permalink":"/Work/Databases/MySql/Advance/1、MySQL架构介绍/","title":"1、MySQL架构介绍","tags":["flashcards"],"noteIcon":"","created":"2026-03-10T22:33:54.000+08:00","updated":"2026-03-24T11:08:48.288+08:00","dg-note-properties":{"title":"1、MySQL架构介绍","tags":["flashcards"]}}
---

# 官网
https://dev.mysql.com/downloads/mysql/5.5.html#downloads
# 查询
```shell
rpm -qa | grep -i mysql # 查询是否安装过
```
# 安装
```shell
rpm -ivh MySQL-server-5.5.48-1.linux2.6.i386.rpm # mysql服务端
rpm -ivh MySQL-client-5.5.48-1.linux2.6.i386.rpm # mysql客户端
```
# 卸载
```shell
rpm -e mysql # 普通删除模式
rpm -e --nodeps mysql # 强力删除模式
```
# 查看是否安装成功
```shell
# 查看mysql安装时自动生成的mysql用户组,可用以检测是否安装成功
cat /etc/group | grep -i mysql
# 查看mysql安装时自动生成的mysql用户,可用以检测是否安装成功
cat /etc/passwd | grep -i mysql
# mysqladmin 创建软链接到/usr/local/bin 方可全局调用
mysqladmin --version # 打印出消息即为成功
ps -ef | grep -i mysql # 打印出mysql进程即为成功
```
# 设置mysql密码
```shell
# mysqladmin 创建软链接到/usr/local/bin 方可全局调用
mysqladmin -u root -proot password 'root'
```
# 配置mysql自启动
```shell
chkconfig mysql on # 设置开机自启动
chkconfig --list | grep -i mysql # 查看mysql默认启动状态列表
cat /etc/inittab # 查看默认运行级别
ntsysv # 图形化设置自启动，[*]为开机自动启动
```
# mysql配置和数据库存储目录
```shell
/var # 目录是变量的配置文件
/usr/sbin/mysqld --verbose --help | grep -A 1 'Default options' # 查看默认加载的配置文件顺序
ps aux | grep mysql|grep 'my.cnf' # 查看启动的mysql加载的配置文件路径
mysql --help | grep 'my.cnf'  # 查看mysql默认读取my.cnf的目录
```

| 路径 | 解释 | 备注 |
| - | - | - |
| /var/lib/mysql | mysql数据库文件的存放路径 | /var/lib/mysql/atguigu.cloud.pid |
| /usr/share/mysql | 配置文件目录 | mysql.server命令及配置文件 |
| /usr/bin | 相关命令目录 | mysqladmin mysgldump等命令 |
| /etc/init.d/mysql | 启停相关脚本 |  |
## mysql修改配置文件位置

```shell
cp /usr/share/mysql/my-huge.cnf /etc/my.cnf # mysql5.5
cp /usr/share/mysql/my-default.cnf /etc/my.cnf # mysql5.6
```
## mysql修改字符集和数据存储路径
**注：在完成mysql安装后，应先修改字符集编码，再建库，否则乱码**
```shell
[client]
#password=your_password
port=3306
socket=/var/lib/mysql/mysql.sock
# 客户端来源数据使用的字符集
default-character-set=utf8mb4

# The MySQL server
[mysqld]
port=3306
# 数据库存储路径
socket=/var/lib/mysql/mysql.sock 
# 0:区分大小写, 1:不区分大小写
lower_case_table_names=1
# 设置最大连接数,默认为 151, MySQL服务器允许的最大连接数16384
max_connections=1000
skip-external-locking
key_buffer_size=384M
# 设置允许导入的sql文件大小
max_allowed_packet=100M 
# 设置mysql连接时间为24小时
# 使用场景:数据迁移、长时间的执行批量的MYSQL语句
interactive_timeout=86400 
table_open_cache=512
# 排序的缓冲区，用于大数据的group by、order by
sort_buffer_size=2M
read_buffer_size=2M
read_rnd_buffer_size=8M
myisam_sort_buffer_size=64M
thread_cache_size=8
query_cache_size=32M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency=8
# 客户端来源数据使用的字符集
character_set_client=utf8mb4
# 默认的内部操作字符集
character-set-server=utf8mb4
# 数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server=utf8mb4_unicode_ci
# 设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'
# 使用服务端字符集(0:使用服务端字符集, 1:不使用服务端字符集)
character-set-client-handshake=0
# 是否忽略客户端的字符集，使用服务器的设置(0:不忽略, 1:忽略)
skip-character-set-client-handshake=0
# 数据库默认字符集,字符集支持一些特殊表情符号(特殊表情符占用4个字节)

[mysql]
no-auto-rehash
# 客户端来源数据使用的字符集
default-character-set = utf8mb4
```
## 查看字符集
```mysql
show variables like 'character%';
show variables like '%char%';
```
# mysql配置文件
## 主要配置文件
	log-bin 二进制日志(用于主从复制)
	log-error 错误日志 默认关闭,记录严重的警告和错误信息,每次启动和关闭的详细信息等
	log 查询日志
	默认关闭,记录查询的sql语句,如果开启会减低mysql的整体性能,因为记录日志也是需要消耗系统资源

## 数据文件
windows
`E:\a_work\a_tools\PhpStudy2018\PHPTutorial\MySQL\data 目录下存放数据库`

## linux
```shell
cd /var/lib/mysql # 默认路径
ls -lF | grep ^d # 查询以目录开头的文件
```
frm文件 存放表结构
myd文件 存放表数据
myi文件 存放表索引
## 如何配置
windows my.ini文件
linux /etc/my.cnf文件
# mysql逻辑架构简介
插件式存储引擎，读写分离
和其它数据库相比, MySQL有点与众不同,它的架构可以在多种不同场景中应用并发挥良好作用。
主要体现在存储引擎的架构上,**插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离**。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

![|673](https://weichengjun2.dpdns.org/i/2023/09/20/650ab44115258.png)

## 1. 连接层
最上层是一些客户端和连接服务,包含本地sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。
主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念,为通过认证安全接入的客户端提供线程。
同样在该层上可以实现基于SSL的安全链接。
服务器也会为安全接入的每个客户端验证它所具有的操作权限。
## 2. 服务层
第二层架构主要完成大多数的核心服务功能，如：SQL接口缓存的查询、SQL的分析和优化、部分内置函数的执行、过程、函数，所有跨存储引擎的功能也在这一层实现。
在该层服务器会解析查询并创建相应的内部解析树,并对其完成相应的优化如：确定查询表的顺序、是否利用索引等,最后生成相应的执行操作。
如果是select语句,服务器还会查询内部的缓存。如果缓存空间足够大,这样在解决大量读操作的环境中能够很好的提升系统的性能。
## 3. 引肇层
存储引擎真正的负责了MySQL中数据的存储和提取,服务器通过API与存储引擎进行通信,不同的存储引擎具有的功能不同,这样我们可以根据自己的实际需要进行选取。后面介绍MyISAM和InnoDB
## 4. 存储层
数据存储层,主要是将数据存储在运行于裸设备的文件系统之上,并完成与存储引擎的交互。
# mysql存储引擎简介
## 查看mysql现在已提供什么存储引擎
```mysql
show engines;
# 输出如下
+------------------+-------+--------------------------------------------------------------+------------+----+----------+
|Engine            |Support|Comment                                                       |Transactions|XA  |Savepoints|
+------------------+-------+--------------------------------------------------------------+------------+----+----------+
|ndbcluster        |NO     |Clustered, fault-tolerant tables                              |null        |null|null      |
|FEDERATED         |NO     |Federated MySQL storage engine                                |null        |null|null      |
|MEMORY            |YES    |Hash based, stored in memory, useful for temporary tables     |NO          |NO  |NO        |
|InnoDB            |DEFAULT|Supports transactions, row-level locking, and foreign keys    |YES         |YES |YES       |
|PERFORMANCE_SCHEMA|YES    |Performance Schema                                            |NO          |NO  |NO        |
|MyISAM            |YES    |MyISAM storage engine                                         |NO          |NO  |NO        |
|ndbinfo           |NO     |MySQL Cluster system information storage engine               |null        |null|null      |
|MRG_MYISAM        |YES    |Collection of identical MyISAM tables                         |NO          |NO  |NO        |
|BLACKHOLE         |YES    |/dev/null storage engine (anything you write to it disappears)|NO          |NO  |NO        |
|CSV               |YES    |CSV storage engine                                            |NO          |NO  |NO        |
|ARCHIVE           |YES    |Archive storage engine                                        |NO          |NO  |NO        |
+------------------+-------+--------------------------------------------------------------+------------+----+----------+
```
## 查看mysql当前默认存储引擎
```mysql
show variables like '%storage_engine%';
# 输出如下
+-------------------------------+---------+
|Variable_name                  |Value    |
+-------------------------------+---------+
|default_storage_engine         |InnoDB   |
|default_tmp_storage_engine     |InnoDB   |
|disabled_storage_engines       |         |
|internal_tmp_mem_storage_engine|TempTable|
+-------------------------------+---------+
```
## MyISAM vs InnoDB 核心区别表
| 特性               | MyISAM                    | InnoDB                  |
| ---------------- | ------------------------- | ----------------------- |
| **事务支持**         | **不支持**                   | **支持** (具有回滚、提交、崩溃恢复能力) |
| **锁粒度**          | **表级锁** (不支持行锁)           | **行级锁** (也支持表锁)         |
| **外键约束**         | **不支持**                   | **支持** (保证数据一致性)        |
| **数据安全性**        | **低** (断电或崩溃易损坏索引)        | **高** (具备崩溃后的自动恢复)      |
| **存储结构**         | 索引与数据分开存放 (非聚簇索引)         | 索引与数据存放在一起 (聚簇索引)       |
| **全文索引**         | 支持                        | 支持 (5.6版本后)             |
| **统计信息 (COUNT)** | 直接存储行数 (执行 `COUNT(*)` 极快) | 需要实时扫描全表计算 (较慢)         |
| **自增主键**         | 自增列可以是复合索引的第二列            | 自增列必须是复合索引的第一列          |
| **内存使用**         | 占用内存低                     | 占用内存高 (需维护 Buffer Pool) |
### 1. 为什么 InnoDB 是现在的标准？
InnoDB 的 **MVCC（多版本并发控制）** 和 **行级锁** 决定了它能承受高强度的并发。
* **数据安全**：InnoDB 拥有重做日志（Redo Log），即便服务器突然断电，它也能在下次启动时恢复未提交的数据。
* **并发性能**：MyISAM 只要有一个写操作，整张表就会被锁死，这在现代高并发 Web 应用中是无法接受的瓶颈。
### 2. MyISAM 是否还有用武之地？
虽然 InnoDB 已经非常成熟，但在极个别场景下 MyISAM 仍有优势：
* **只读报表**：如果你的数据是静态的，且需要大量的 `COUNT(*)`，MyISAM 的计数器优势很明显。
* **低内存环境**：在内存极其有限的嵌入式或极小型服务器上，MyISAM 结构更简单。
* **物理备份**：MyISAM 的文件（.frm, .MYD, .MYI）可以直接跨平台拷贝使用，而 InnoDB 虽然也可以通过 `.ibd` 迁移，但逻辑更复杂。
### 3. 如何查看当前表的引擎？
你可以使用以下命令检查现有表的引擎类型：
```sql
SHOW TABLE STATUS LIKE 'your_table_name';
```
**建议：**
除非有极其特殊的理由，否则**永远优先选择 InnoDB**。
### 4.延伸思考：阿里巴巴、淘宝用什么存储引擎？
在了解了 InnoDB 和 MyISAM 后，我们看看顶级互联网公司是如何选择的：
- **技术背景**：阿里巴巴大部分 MySQL 数据库使用的是基于 **Percona** 原型加以修改的定制版本。
- **引擎选择**：Percona 推出了一款名为 **XtraDB** 的存储引擎。
- **核心优势**：XtraDB 完全可以替代 InnoDB，但在高负载情况下，其并发处理能力和性能诊断工具比原生 InnoDB 更加出色。
- **阿里方案**：目前的阿里体系主要使用 **AliSQL**（在开源版本基础上进行了大量内核优化）+ **AliRedis** 的组合。