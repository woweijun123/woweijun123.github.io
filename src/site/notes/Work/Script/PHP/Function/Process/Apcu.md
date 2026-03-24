---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Process/Apcu/","title":"Apcu","tags":["flashcards"],"noteIcon":"","created":"2026-01-09T11:53:19.605+08:00","updated":"2026-03-24T17:36:44.569+08:00"}
---

APCu（APC User Cache）是一个为 PHP 设计的**内存驻留级键值存储扩展**。它是早期 APC（Alternative PHP Cache）扩展的延续，但剥离了过时的操作码缓存（Opcode Cache）功能，专注于**用户数据缓存**。
在 PHP 8+ 的现代开发中，Opcode 缓存已由内置的 Opcache 负责，而 APCu 则专门填补了“进程间共享内存数据”的空白。
# APCu 的核心特性
* **内存存储**：数据直接存储在系统的共享内存（RAM）中，读写速度极快。
* **进程间共享**：在同一个 PHP-FPM 进程池下，所有工作进程（Worker Processes）都可以访问同一份 APCu 缓存数据。
* **生命周期**：数据的生命周期独立于请求。只要 PHP-FPM 进程不重启，数据就会一直存在（除非达到过期时间或被手动删除）。
* **轻量化**：相比 Redis 或 Memcached，它不需要网络 IO，没有 TCP 连接开销，是**单机性能最强**的缓存方案。
# 主要使用场景
APCu 并非要取代 Redis，它最适合处理**高频读取、低频修改、且仅限单机使用**的数据。
#### A. 配置信息与元数据缓存
许多框架（如 Symfony）使用 APCu 缓存路由表、注解元数据或翻译文件。
* **场景**：不再每请求都去解析巨大的 YAML 或 XML 配置文件。
* **优势**：显著降低磁盘 I/O。
#### B. 本地热点数据缓存（L1 Cache）
在分布式系统中，可以将 APCu 作为一级缓存。
* **架构**：APCu (L1) -> Redis (L2) -> 数据库 (L3)。
* **场景**：对于极其频繁访问的数据（如权限角色列表），先看本地 APCu，没有再去 Redis 取。
#### C. 计数器与锁（简单场景）
利用 `apcu_inc()` 和 `apcu_cas()` 等原子操作，可以实现简单的单机计数器或防止并发冲突的锁。
# APCu vs. Redis：如何选择？
| 特性        | APCu      | Redis                      |
| --------- | --------- | -------------------------- |
| **存储介质**  | 共享内存 (本地) | 内存 (远程或本地服务)               |
| **访问开销**  | 极低（函数调用级） | 较高（网络/套接字 IO）              |
| **数据持久化** | 无（重启丢数据）  | 支持 RDB/AOF 持久化             |
| **分布式支持** | 不支持（仅限单机） | 完美支持（集群/哨兵）                |
| **数据结构**  | 仅支持简单的键值对 | 支持 List, Set, Hash, ZSet 等 |
# 核心 API 示例
使用 APCu 非常直观，类似于简单的数组操作：
```php
/**
 * APCu 核心 API 综合示例手册
 * 适用版本：PHP 7.4 / 8.x
 */
class ApcuExpertDemo
{
    /**
     * 1. 基础 CRUD 操作
     */
    public function basicOps()
    {
        // 存储 (Store)：如果 key 已存在则覆盖
        // 参数：key, value, ttl (秒)
        apcu_store('user_name', '张三', 3600);

        // 检查是否存在 (Exists)
        if (apcu_exists('user_name')) {
            // 获取 (Fetch)
            $name = apcu_fetch('user_name');
        }

        // 删除 (Delete)
        apcu_delete('user_name');
    }

    /**
     * 2. 条件写入与防击穿
     */
    public function conditionalOps()
    {
        // 仅在不存在时添加 (Add)：如果 key 已存在，则返回 false
        $isAdded = apcu_add('lock_key', 'locked', 10);

        // 自动计算并获取 (Entry)：【最推荐】
        // 如果 key 不存在，执行回调函数并存储结果，返回结果
        // 它可以防止多个请求同时计算同一个耗时任务（防击穿）
        $data = apcu_entry('heavy_task_result', function($key) {
            // 模拟复杂计算或数据库查询
            return ['time' => time(), 'data' => 'System Report'];
        }, 3600);

        return $data;
    }

    /**
     * 3. 原子操作 (计数器与并发控制)
     */
    public function atomicOps()
    {
        // 原子自增 (Increment)：返回增加后的新值
        // 步长默认为 1
        $count = apcu_inc('counter', 1, $success);

        // 原子自减 (Decrement)
        $count = apcu_dec('counter', 1);

        // 比较并交换 (CAS - Compare And Swap)：实现乐观锁
        // 语法：apcu_cas(key, 旧值, 新值)
        // 只有当当前值为 10 时，才将其改为 11
        $isUpdated = apcu_cas('version', 10, 11);
    }

    /**
     * 4. 批量操作 (提升性能)
     */
    public function batchOps()
    {
        $payload = [
            'config.host' => '127.0.0.1',
            'config.port' => 3306,
            'config.user' => 'root'
        ];

        // 批量存储
        apcu_store($payload, null, 7200);

        // 批量获取
        $configs = apcu_fetch(['config.host', 'config.port']);

        // 批量删除
        apcu_delete(['config.host', 'config.port']);
    }

    /**
     * 5. 监控与系统维护
     */
    public function systemOps()
    {
        // 获取缓存详细列表及元数据
        $info = apcu_cache_info();

        // 获取内存分配详情（可用空间、片段化情况）
        $smaInfo = apcu_sma_info();

        // 获取特定 Key 的详细信息（创建时间、访问次数等）
        $keyDetail = apcu_key_info('heavy_task_result');

        // 清空所有缓存 (谨慎使用)
        // apcu_clear_cache();
        
        return [
            'slots' => $info['num_slots'],
            'free_mem' => $smaInfo['avail_mem']
        ];
    }
}

// --- 实用工具函数：检查环境 ---
if (!extension_loaded('apcu')) {
    exit("请先安装并启用 APCu 扩展。\n");
}

if (PHP_SAPI === 'cli' && !ini_get('apc.enable_cli')) {
    echo "警告：CLI 模式下未开启 apc.enable_cli，脚本可能无法持久化存储。\n";
}
```
# 注意事项
1. **单机限制**：如果你的应用部署在多台服务器上，每台机器的 APCu 缓存是不互通的。同步这些缓存会非常麻烦，这时应优先考虑 Redis。
2. **内存限制**：需在 `php.ini` 中配置 `apc.shm_size`。如果存储的数据超过该值，会触发逐出机制或导致写入失败。
# 实战场景
### 1. 热点配置/元数据缓存 (性能提升最明显)
如果你的应用需要解析大型配置文件（如 YAML、XML）或从数据库频繁读取系统设置，APCu 可以消除磁盘 I/O 和数据库查询。
```php
function get_system_config() {
    $cacheKey = 'app_global_config';
    // 尝试从 APCu 获取
    $config = apcu_fetch($cacheKey, $success);
    if (!$success) {
        // 模拟从数据库或文件加载
        $config = [
            'api_version' => 'v2.1',
            'maintenance_mode' => false,
            'features' => ['login', 'payment', 'search']
        ];
        
        // 存入缓存，有效期 1 小时 (3600秒)
        apcu_store($cacheKey, $config, 3600);
    }
    return $config;
}
```
### 2. 原子计数器 (限制访问频率/统计)
APCu 提供了原子性的操作函数，这意味着在高并发下，计数器依然准确，不会出现由于竞争导致的读写覆盖。
```php
$ip = $_SERVER['REMOTE_ADDR'];
$rateKey = "rate_limit:" . $ip;

// 如果不存在则初始化为 0，存在则自增 1
$currentRequests = apcu_inc($rateKey, 1, $success);

if ($success && $currentRequests > 100) {
    die("请求过于频繁，请稍后再试。");
}

// 第一次请求时设置过期时间，确保每分钟重置
if ($currentRequests === 1) {
    apcu_store($rateKey, 1, 60); 
}
```
### 3. “防击穿”式的数据缓存 (使用 `apcu_entry`)
`apcu_entry` 是一个非常强大的函数。它能确保在高并发请求同一个 Key 时，如果缓存失效，**只有一个进程**会执行回调函数生成数据，其他进程会等待该结果，防止数据库被瞬间击穿。
```php
$data = apcu_entry('heavy_report_data', function($key) {
    // 这里执行非常耗时的操作，例如复杂的 SQL 聚合
    sleep(2); 
    return [
        'time' => date('Y-m-d H:i:s'),
        'result' => '聚合数据内容'
    ];
}, 3600);

print_r($data);
```
### 4. 本地 L1 级缓存 (配合 Redis 使用)
在分布式架构中，将 APCu 作为第一级缓存，Redis 作为第二级。这样即使 Redis 响应很快，也可以省去网络请求的延迟。
```php
class MultiLevelCache {
    public function get($key) {
        // 1. 先看本地 APCu
        $data = apcu_fetch($key, $success);
        if ($success) return $data;

        // 2. 本地没有，再看远程 Redis (假设已有 $redis 对象)
        $data = $this->redis->get($key);
        if ($data) {
            // 3. 回填到本地 APCu，加速下次访问
            apcu_store($key, $data, 60); 
            return $data;
        }

        return null;
    }
}
```
### 5. 常用管理小技巧
#### 检查内存使用情况
你可以写一个简单的脚本来监控 APCu 的健康状况：
```php
$info = apcu_cache_info();
echo "缓存条目数: " . $info['num_entries'] . "\n";
echo "内存占用: " . ($info['mem_size'] / 1024 / 1024) . " MB\n";
```
#### 清理缓存
* **代码清理**：`apcu_clear_cache();` (清理所有用户缓存)
* **命令行清理**：由于 CLI 和 FPM 进程通常不共享内存，在终端运行 `php -r "apcu_clear_cache();"` 无法清理网页端的缓存。你需要创建一个 Web 入口或使用 `cachetool` 之类的工具。
### 注意事项
* **对象存储**：APCu 存储对象时会序列化。如果对象包含 `Closure`（闭包）或未初始化的资源（如数据库连接），会报错或失效。
* **分片锁**：虽然 APCu 处理了并发，但存储超大数据块（几百 MB）仍可能导致 FPM 进程在写入时短暂阻塞。
# `shmop VS apcu VS shm_attach`
| **特性**    | **APCu (推荐)**                  | **SysV Shared Memory**           | **shmop**                       |
| --------- | ------------------------------ | -------------------------------- | ------------------------------- |
| **扩展名称**  | `ext-apcu`                     | `ext-sysvshm`                    | `ext-shmop`                     |
| **抽象层级**  | **应用级/键值对**。最接近现代开发习惯。         | **系统级/变量式**。System V IPC 封装。     | **底层/字节流**。类似 C 语言内存管理。         |
| **数据操作**  | 类似 `get/set`。自动处理序列化。          | 基于 `Var ID`。自动处理序列化。             | 基于**偏移量(Offset)**。需手动处理序列化。     |
| **核心函数**  | `apcu_store()`, `apcu_fetch()` | `shm_put_var()`, `shm_get_var()` | `shmop_write()`, `shmop_read()` |
| **并发安全**  | **内置读写锁**。支持原子自增 `apcu_inc`。   | **无内置保护**。通常需配合 `sysvsem` (信号量)。 | **无内置保护**。需手动实现互斥锁/信号量。         |
| **过期机制**  | **支持 TTL**（自动过期回收）。            | 不支持（除非手动删除变量）。                   | 不支持（必须手动释放内存段）。                 |
| **跨语言能力** | 弱。数据结构由 PHP 内部引擎维护。            | 中等。遵循 System V 标准，但序列化格式特定。      | **强**。存储的是原始字节，适合跨进程/语言。        |
| **主要用途**  | 高性能应用缓存、单机热点数据。                | 较老旧项目中简单的跨进程变量共享。                | 极致性能需求、多语言通信、二进制数据处理。           |
### 关键差异深度解析
#### 1. 内存管理逻辑：Key-Value vs Segment-Offset
* **APCu** 像是一个内存中的 Redis。你给它一个字符串 Key（如 `user_1`），它帮你打理好一切。
* **SysV (`shm_attach`)** 要求你先创建一个内存段，然后在里面分配变量 ID。它虽然也支持变量存储，但管理起来比 APCu 沉重。
* **shmop** 就像在内存中读写文件。你必须知道数据从第几个字节开始，到第几个字节结束。
#### 2. 并发安全的代价
* 在使用 **APCu** 时，你不需要担心两个 PHP 进程同时写入导致内存崩溃，因为它底层实现了细粒度的锁。
* 使用 **SysV** 或 **shmop** 时，如果两个 PHP 进程同时向同一个偏移量写入数据，数据会发生**损坏（Corruption）**。你必须引入 `sysvsem` 扩展来手动锁定：
```php
$sem_id = sem_get($key); // 获取信号量
sem_acquire($sem_id);   // 加锁
// 执行 shmop 读写...
sem_release($sem_id);   // 释放锁
```
#### 3. 序列化的开销
* **APCu** 和 **SysV** 在存储非字符串数据（如数组、对象）时，会在底层自动调用 `serialize()`。
* **shmop** 强制你面对原始字节。如果你存的是大数组，手动 `igbinary_serialize()` 配合 `shmop` 有时能获得比 APCu 更极致的控制力，但开发复杂度极高。
### 总结建议
* **95% 的场景**：请直接使用 **APCu**。它在性能、安全性和易用性之间取得了完美的平衡，且支持 TTL 过期。
* **需要与 C/Go 等其他语言共享内存**：选择 **shmop**。
* **维护 10 年以上的旧项目**：你可能会看到 **SysV (shm_attach)**，但在新项目中已不推荐使用。