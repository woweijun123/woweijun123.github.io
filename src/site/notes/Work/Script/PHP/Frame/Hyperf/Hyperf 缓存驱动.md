---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Hyperf/Hyperf 缓存驱动/","title":"Hyperf 缓存驱动","tags":["flashcards"],"noteIcon":"","created":"2025-09-04T13:50:00.646+08:00","updated":"2026-03-24T17:31:57.486+08:00"}
---

# 优化用户信息查询：避免每次请求读取 Redis
在 Hyperf 中避免每次请求都从 Redis 查询用户信息是一个非常重要的性能优化点。我将为你提供一套完整的最佳实践方案，从简单到高级层层递进。
## 📊 优化策略对比
首先，我们通过一个表格了解不同优化策略的优劣：

| 策略 | 实现难度 | 性能提升 | 内存占用 | 数据一致性 | 适用场景 |
|:---|:---|:---|:---|:---|:---|
| **请求级缓存 (Context)** | ⭐☆☆☆☆ | ⭐⭐⭐⭐⭐ | 低 | 强 | 所有场景，首选方案 |
| **进程级缓存 (APCu)** | ⭐⭐☆☆☆ | ⭐⭐⭐⭐☆ | 中 | 中（进程内强，进程间弱） | 用户数据变更不频繁 |
| **本地缓存层 (两级缓存)** | ⭐⭐⭐☆☆ | ⭐⭐⭐⭐⭐ | 中 | 中（可控制） | 高并发，允许短暂数据不一致 |
| **协程缓存预热** | ⭐⭐⭐⭐☆ | ⭐⭐⭐⭐☆ | 低 | 强 | 已知后续需要的数据 |
## 🏆 最佳实践推荐

对于大多数应用，我推荐 **策略一：请求级缓存 (Context)** 作为基础，结合 **策略三：本地缓存层** 用于高性能场景。这是性能与一致性兼顾的最佳方案。
### 策略一：请求级缓存 (Hyperf Context) - 首选方案
这是最简单且最有效的优化，**强烈推荐作为所有 Hyperf 项目的标准实践**。
**原理**：利用 Hyperf 的 `Context` 在**同一次请求生命周期内**缓存数据，避免一次请求内多次查询 Redis。
**优势**：
- 零额外开销，性能提升显著
- 保证请求内数据一致性
- 实现简单，无需引入额外组件
**代码实现**：
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Hyperf\Redis\Redis;
use Hyperf\Context\Context;
use Psr\Container\ContainerInterface;

class AuthService
{
    private const SESSION_PREFIX = 'user:session:';
    
    protected Redis $redis;

    public function __construct(protected ContainerInterface $container)
    {
        $this->redis = $this->container->get(Redis::class);
    }

    /**
     * 获取用户信息（带请求级缓存）
     */
    public function getUser(string $token): ?array
    {
        // 生成请求内唯一的缓存键
        $cacheKey = 'user_data_' . md5($token);
        
        // 检查当前请求是否已缓存
        if (Context::has($cacheKey)) {
            return Context::get($cacheKey);
        }

        // 从 Redis 获取数据
        $sessionKey = $this->getSessionKey($token);
        $serializedData = $this->redis->get($sessionKey);
        
        if (empty($serializedData)) {
            return null;
        }

        // 反序列化数据
        $userData = unserialize($serializedData);
        
        // 刷新 Redis 中的过期时间
        $this->redis->expire($sessionKey, 604800);
        
        // 存储到当前请求的 Context 中
        Context::set($cacheKey, $userData);
        
        return $userData;
    }

    /**
     * 更新用户信息（同时清除缓存）
     */
    public function updateUser(string $token, array $userData): bool
    {
        $sessionKey = $this->getSessionKey($token);
        $cacheKey = 'user_data_' . md5($token);
        
        // 清除请求级缓存
        Context::destroy($cacheKey);
        
        // 更新 Redis 数据
        return $this->redis->setex(
            $sessionKey, 
            604800, 
            serialize($userData)
        );
    }

    private function getSessionKey(string $token): string
    {
        return self::SESSION_PREFIX . $token;
    }
}
```
### 策略二：进程级缓存 (APCu) - 补充方案
**原理**：使用 APCu 在 Worker 进程级别缓存用户数据，跨请求共享。
**适用场景**：用户数据变更不频繁，读多写少的场景。
**代码实现**：
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Hyperf\Redis\Redis;

class AuthService
{
    // ... 其他代码同上 ...

    /**
     * 获取用户信息（带进程级缓存）
     */
    public function getUserWithProcessCache(string $token): ?array
    {
        $apcuKey = 'user_apcu_' . md5($token);
        
        // 检查 APCu 缓存
        $cached = apcu_fetch($apcuKey, $success);
        if ($success) {
            return $cached;
        }

        // 从 Redis 获取
        $userData = $this->getUserFromRedis($token);
        if (!$userData) {
            return null;
        }

        // 缓存到 APCu（5分钟）
        apcu_store($apcuKey, $userData, 300);
        
        return $userData;
    }

    /**
     * 更新用户信息（清除多级缓存）
     */
    public function updateUserWithCache(string $token, array $userData): bool
    {
        $apcuKey = 'user_apcu_' . md5($token);
        
        // 清除 APCu 缓存
        apcu_delete($apcuKey);
        
        // 更新 Redis
        return $this->updateUserInRedis($token, $userData);
    }
}
```
### 策略三：本地缓存层 (两级缓存) - 高性能方案
**原理**：建立 Redis → 本地内存的两级缓存架构，首先检查本地内存，未命中再查询 Redis。
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Hyperf\Redis\Redis;
use Swoole\Table;

class AuthService
{
    private $localCache;
    
    public function __construct(protected Redis $redis)
    {
        // 创建 Swoole Table 作为本地缓存
        $this->localCache = new Table(1024);
        $this->localCache->column('data', Table::TYPE_STRING, 1024);
        $this->localCache->column('expire', Table::TYPE_INT, 4);
        $this->localCache->create();
    }

    /**
     * 获取用户信息（两级缓存）
     */
    public function getUserWithLocalCache(string $token): ?array
    {
        $cacheKey = 'user_local_' . md5($token);
        
        // 检查本地缓存
        $item = $this->localCache->get($cacheKey);
        if ($item && $item['expire'] > time()) {
            return unserialize($item['data']);
        }

        // 从 Redis 获取
        $userData = $this->getUserFromRedis($token);
        if (!$userData) {
            return null;
        }

        // 更新本地缓存（30秒过期）
        $this->localCache->set($cacheKey, [
            'data' => serialize($userData),
            'expire' => time() + 30
        ]);
        
        return $userData;
    }
}
```
## 🛡️ 缓存一致性保障
无论采用哪种缓存策略，都必须处理缓存一致性问题：
```php
<?php
declare(strict_types=1);

namespace App\Service;

class UserService
{
    // ... 其他代码 ...
    
    /**
     * 更新用户信息并清理所有缓存
     */
    public function updateUserProfile(int $userId, array $profileData): bool
    {
        // 1. 更新数据库
        $result = $this->userModel->where('id', $userId)->update($profileData);
        
        if ($result) {
            // 2. 获取用户所有有效 token
            $tokens = $this->getUserTokens($userId);
            
            // 3. 清理所有相关缓存
            foreach ($tokens as $token) {
                $this->clearUserCache($token);
            }
            
            // 4. 发布缓存失效消息（分布式环境下）
            $this->publishCacheInvalidation($userId);
        }
        
        return $result;
    }
    
    /**
     * 清理用户所有缓存
     */
    private function clearUserCache(string $token): void
    {
        // 清理请求级缓存
        $contextKey = 'user_data_' . md5($token);
        Context::destroy($contextKey);
        
        // 清理进程级缓存
        $apcuKey = 'user_apcu_' . md5($token);
        apcu_delete($apcuKey);
        
        // 清理本地缓存
        $localKey = 'user_local_' . md5($token);
        $this->localCache->del($localKey);
        
        // 注意：不清理 Redis，因为它是数据源
    }
}
```

## 📈 监控与度量
为了确保优化有效，应该添加监控：
```php
<?php
declare(strict_types=1);

namespace App\Middleware;

use Hyperf\Di\Annotation\Inject;
use Hyperf\Utils\Context as HyperfContext;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class CacheMonitorMiddleware implements MiddlewareInterface
{
    /**
     * @Inject
     * @var \Prometheus\CollectorRegistry
     */
    protected $prometheus;
    
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // 记录缓存查询次数
        HyperfContext::set('cache_queries', [
            'redis' => 0,
            'context' => 0,
            'apcu' => 0
        ]);
        
        $response = $handler->handle($request);
        
        // 收集指标
        $metrics = HyperfContext::get('cache_queries');
        $this->recordMetrics($metrics);
        
        return $response;
    }
    
    private function recordMetrics(array $metrics): void
    {
        $this->prometheus->getCounter('user_cache_queries_total')
            ->withLabels(['redis'])
            ->incBy($metrics['redis']);
            
        $this->prometheus->getCounter('user_cache_queries_total')
            ->withLabels(['context'])
            ->incBy($metrics['context']);
    }
}
```

在 AuthService 中添加统计：
```php
public function getUser(string $token): ?array
{
    $cacheKey = 'user_data_' . md5($token);
    
    if (Context::has($cacheKey)) {
        $this->recordCacheHit('context');
        return Context::get($cacheKey);
    }
    
    $this->recordCacheMiss('redis');
    // ... 从 Redis 查询 ...
}

private function recordCacheHit(string $type): void
{
    $metrics = Context::get('cache_queries', ['redis' => 0, 'context' => 0, 'apcu' => 0]);
    $metrics[$type]++;
    Context::set('cache_queries', $metrics);
}
```
## 💡 总结与建议
1. **首要实施**：务必使用 **请求级缓存 (Context)**，这是最简单且效果最显著的优化
2. **按需添加**：对于读多写少的场景，可以考虑添加 **进程级缓存 (APCu)**
3. **高性能场景**：极端性能要求时使用 **本地缓存层**，但要处理好一致性
4. **始终牢记**：任何缓存策略都必须配套相应的 **缓存清理机制**
5. **监控验证**：通过监控确认优化效果，确保 Redis 查询次数显著下降

**推荐方案**：从策略一开始实施，80% 的应用只需要这一级优化就能满足需求。只有在监控数据显示 Redis 仍然是瓶颈时，才考虑添加更复杂的缓存策略。

# Swoole Table 缓存：深度解析与最佳实践
使用 Swoole Table 作为缓存来避免重复查询 Redis 是一个高性能方案，但在内存管理和使用模式上与 Context 缓存有本质区别。
## 📊 Swoole Table 特性一览
### Swoole Table、Context、Redis 缓存方案对比
| 特性维度       | Swoole Table                              | Context (协程上下文)              | Redis                            |
| ---------- | ----------------------------------------- | ---------------------------- | -------------------------------- |
| **数据物理位置** | **同一台服务器的共享内存**                           | 单个 Worker 进程的内存空间            | **独立的 Redis 服务器进程**（可跨服务器）       |
| **数据生命周期** | 与 **Swoole Server** 同生命周期（服务重启，数据丢失）      | 与 **当前请求** 同生命周期（请求结束，自动销毁）  | **独立持久化**（可配置过期，服务重启，数据仍在）       |
| **清理方式**   | **手动管理** 或进程结束                            | **自动回收**                     | 手动删除或等待过期                        |
| **共享范围**   | **单机多进程共享**：<br>同一 Swoole 实例下所有 Worker 进程 | **单请求共享**：<br>单个 Worker 协程内部 | **全局共享**：<br>所有服务器上的所有进程         |
| **性能**     | **极致** (内存操作，无 IO)                        | 极高 (内存操作)                    | 高 (但有网络 IO 开销)                   |
| **数据一致性**  | **强一致性**<br>（通过原子操作保证多进程写入安全）             | 请求内强一致                       | **全局强一致性**<br>（Redis 服务端单线程模型保证） |
| **容量限制**   | **固定**，创建时指定                              | 无硬性限制                        | 极大，可扩展                           |
| **扩展性**    | **差**，受限于单机                               | 不涉及                          | **极佳**，支持主从、集群                   |
### 🔧 技术原理浅析
Swoole Table 能实现真正的进程间共享，底层是通过以下方式实现的：
1.  **共享内存**：它使用 `mmap` 系统调用创建一块内存区域，并映射到多个 Worker 进程的地址空间。这意味着所有 Worker 进程读写的是**同一块物理内存**。
2.  **自旋锁**：为了保证多进程同时读写这块内存时的数据安全（原子性），Swoole Table 为每一行数据内置了锁机制（行锁）。当一个进程在操作某一行时，会自动上锁，阻止其他进程同时操作同一行，从而避免数据错乱。
## 内存管理：不会自动清空
这是最关键的区别：**Swoole Table 中的数据在处理完毕后不会被自动清空**。
### ⚡ 内存持久化原理
1.  **进程级存储**：Swoole Table 基于共享内存实现，数据存储在 **Worker 进程的内存空间**中
2.  **常驻内存**：由于 Hyperf 的 Worker 进程是常驻的，Table 中的数据会一直存在，直到：
    - 你**手动删除**某条数据
    - Worker 进程**重启**（代码发布、异常退出等）
    - 服务器**重启**
3.  **示例代码**：
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Swoole\Table;

class UserTableCache
{
    private Table $table;
    
    public function __construct()
    {
        // 创建 Table，指定最大行数
        $this->table = new Table(1024);
        $this->table->column('data', Table::TYPE_STRING, 1024);
        $this->table->column('expire_time', Table::TYPE_INT, 4);
        $this->table->create();
    }
    
    public function get(string $key): ?array
    {
        $item = $this->table->get($key);
        if (!$item) {
            return null;
        }
        
        // 检查是否过期
        if ($item['expire_time'] > 0 && $item['expire_time'] < time()) {
            $this->table->del($key); // 手动删除过期数据
            return null;
        }
        
        return unserialize($item['data']);
    }
    
    public function set(string $key, array $data, int $ttl = 0): bool
    {
        return $this->table->set($key, [
            'data' => serialize($data),
            'expire_time' => $ttl > 0 ? time() + $ttl : 0
        ]);
    }
    
    // 必须手动清理数据
    public function delete(string $key): bool
    {
        return $this->table->del($key);
    }
    
    // 定期清理过期数据
    public function gc(): void
    {
        $now = time();
        foreach ($this->table as $key => $row) {
            if ($row['expire_time'] > 0 && $row['expire_time'] < $now) {
                $this->table->del($key);
            }
        }
    }
}
```
## ✅ Swoole Table 的优势
### 1. 极致性能
- **内存级速度**：直接内存访问，无序列化/网络开销
- **零IO阻塞**：完全避免网络延迟带来的性能波动
### 2. 进程内数据共享
- **跨进程共享**：不同 Worker 进程处理的所有请求都能访问相同数据
- **减少重复查询**：多个请求可以共享已缓存的数据
### 3. 原子操作保证
- **内置锁机制**：Swoole Table 的操作是原子性的，无需额外加锁
- **线程安全**：多协程并发读写时数据一致性有保障
## ⚠️ Swoole Table 的劣势与挑战
### 1. 内存管理复杂
```php
// 必须手动实现过期清理机制
class TableGC
{
    public function runRegularCleanup(): void
    {
        // 需要定期执行垃圾回收
        swoole_timer_tick(60000, function() {
            $this->userTableCache->gc();
        });
    }
}
```
### 2. 容量限制严格
- **固定大小**：创建时需要预先分配足够空间
- **无法动态扩容**：超出容量会导致写入失败
### 3. 数据持久化缺失
- **服务重启数据丢失**：部署、重启时所有缓存数据消失
- **需要预热机制**：重启后需要重新加载热点数据
## 🛠️ 最佳实践方案
### 1. 多级缓存架构（推荐）
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Hyperf\Redis\Redis;
use Hyperf\Context\Context;

class MultiLevelCache
{
    public function getUser(string $token): ?array
    {
        // 1. 检查请求级缓存 (Context)
        if (Context::has($token)) {
            return Context::get($token);
        }
        
        // 2. 检查进程级缓存 (Swoole Table)
        $tableData = $this->tableCache->get($token);
        if ($tableData) {
            Context::set($token, $tableData); // 填充请求级缓存
            return $tableData;
        }
        
        // 3. 检查分布式缓存 (Redis)
        $redisData = $this->redis->get('user:' . $token);
        if ($redisData) {
            $userData = unserialize($redisData);
            $this->tableCache->set($token, $userData, 300); // 缓存到 Table
            Context::set($token, $userData); // 缓存到 Context
            return $userData;
        }
        
        // 4. 查询数据库
        $userData = $this->fetchFromDatabase($token);
        if ($userData) {
            $this->redis->setex('user:' . $token, 3600, serialize($userData));
            $this->tableCache->set($token, $userData, 300);
            Context::set($token, $userData);
        }
        
        return $userData;
    }
}
```
### 2. 合适的应用场景
- **高频读取**：秒杀计数器、实时排行榜
- **小规模数据**：配置信息、功能开关
- **临时状态**：用户会话、游戏房间状态
### 3. 内存监控与管理
```php
class TableMonitor
{
    public function checkMemoryUsage(): array
    {
        $usage = [];
        foreach ($this->table as $key => $row) {
            $usage[$key] = [
                'size' => strlen(serialize($row)),
                'age' => time() - ($row['create_time'] ?? 0)
            ];
        }
        return $usage;
    }
    
    public function enforceMemoryLimits(): void
    {
        if ($this->table->count() > $this->maxItems) {
            $this->evictOldestItems();
        }
    }
}
```
## 💡 总结
**Swoole Table 是一个强大的工具，但需要谨慎使用：**
### 适用场景：
- ✅ 极致性能要求的临时数据
- ✅ 进程内多请求共享数据
- ✅ 小规模、固定结构的数据缓存
### 需要避免：
- ❌ 大规模、不确定容量的数据
- ❌ 需要持久化的重要数据
- ❌ 对数据一致性要求极高的场景
**推荐方案**：采用 **多级缓存架构**，用 Swoole Table 作为 Redis 和 Context 之间的中间层，既能享受性能优势，又能通过 Redis 保证数据的持久性和一致性。同时建立完善的内存监控和清理机制，避免内存无限增长。

# Hyperf 中实现跨进程共享内存的最佳实践（不使用 Redis）
在 Hyperf 中不使用 Redis 实现跨进程共享内存，确实有一些成熟的方案。我将为你提供完整的最佳实践，从基础实现到高级用法。
## 📊 跨进程共享内存方案对比
| 方案               | 实现难度  | 性能    | 数据一致性     | 容量灵活性     | 功能丰富性    |
| :--------------- | :---- | :---- | :-------- | :-------- | :------- |
| **Swoole Table** | ⭐⭐☆☆☆ | ⭐⭐⭐⭐⭐ | 高 (原子操作)  | 低 (固定大小)  | 中 (键值存储) |
| **Shmop 扩展**     | ⭐⭐⭐⭐☆ | ⭐⭐⭐⭐☆ | 低 (需手动同步) | 中 (可动态分配) | 低 (原始字节) |
| **System V IPC** | ⭐⭐⭐⭐☆ | ⭐⭐⭐⭐☆ | 中 (信号量控制) | 高 (可扩展)   | 中 (多种结构) |
## 🏆 最佳实践推荐：Swoole Table
对于大多数 Hyperf 应用，**Swoole Table 是最佳选择**，因为它与 Swoole 深度集成，使用简单且性能极高。
### ✅ 方案一：Swoole Table (首选)
#### 1. 创建 Table 服务
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Swoole\Table;
use Hyperf\Contract\ConfigInterface;

class SharedMemoryService
{
    private Table $table;
    
    // 定义表结构
    private const TABLE_COLUMNS = [
        'data' => [Table::TYPE_STRING, 1024],
        'created_at' => [Table::TYPE_INT, 8],
        'updated_at' => [Table::TYPE_INT, 8],
        'expires_at' => [Table::TYPE_INT, 8],
    ];
    
    public function __construct(private ConfigInterface $config)
    {
        $this->initTable();
    }
    
    private function initTable(): void
    {
        // 从配置获取表大小，默认 1024 行
        $tableSize = $this->config->get('shared_memory.table_size', 1024);
        
        $this->table = new Table($tableSize);
        
        // 动态添加列
        foreach (self::TABLE_COLUMNS as $name => $column) {
            $this->table->column($name, $column[0], $column[1]);
        }
        
        $this->table->create();
    }
    
    public function set(string $key, $data, int $ttl = 0): bool
    {
        $time = time();
        $expires = $ttl > 0 ? $time + $ttl : 0;
        
        return $this->table->set($key, [
            'data' => serialize($data),
            'created_at' => $time,
            'updated_at' => $time,
            'expires_at' => $expires
        ]);
    }
    
    public function get(string $key)
    {
        $item = $this->table->get($key);
        
        if (!$item) {
            return null;
        }
        
        // 检查过期时间
        if ($item['expires_at'] > 0 && $item['expires_at'] < time()) {
            $this->delete($key);
            return null;
        }
        
        return unserialize($item['data']);
    }
    
    public function delete(string $key): bool
    {
        return $this->table->del($key);
    }
    
    public function has(string $key): bool
    {
        $item = $this->table->get($key);
        
        if (!$item) {
            return false;
        }
        
        // 检查过期时间
        if ($item['expires_at'] > 0 && $item['expires_at'] < time()) {
            $this->delete($key);
            return false;
        }
        
        return true;
    }
    
    public function incr(string $key, int $value = 1): int
    {
        // 对于计数操作，可以使用原子性操作
        $current = $this->get($key) ?? 0;
        $newValue = $current + $value;
        $this->set($key, $newValue);
        
        return $newValue;
    }
    
    // 获取表统计信息
    public function stats(): array
    {
        return [
            'count' => $this->table->count(),
            'memory_size' => $this->table->getMemorySize(),
        ];
    }
    
    // 清理过期数据
    public function gc(): int
    {
        $count = 0;
        $now = time();
        
        foreach ($this->table as $key => $row) {
            if ($row['expires_at'] > 0 && $row['expires_at'] < $now) {
                $this->table->del($key);
                $count++;
            }
        }
        
        return $count;
    }
}
```
#### 2. 配置服务
在 `config/autoload/dependencies.php` 中注册服务：
```php
return [
    App\Service\SharedMemoryService::class => App\Service\SharedMemoryService::class,
];
```
在 `config/autoload/shared_memory.php` 中添加配置：
```php
<?php
return [
    'table_size' => env('SHARED_MEMORY_TABLE_SIZE', 2048),
];
```
#### 3. 使用示例
```php
<?php
declare(strict_types=1);

namespace App\Controller;

use App\Service\SharedMemoryService;
use Hyperf\HttpServer\Annotation\AutoController;
use Hyperf\HttpServer\Contract\RequestInterface;

#[AutoController]
class MemoryController extends AbstractController
{
    public function __construct(private SharedMemoryService $sharedMemory)
    {
    }
    
    public function setValue(RequestInterface $request)
    {
        $key = $request->input('key');
        $value = $request->input('value');
        $ttl = (int)$request->input('ttl', 0);
        
        $result = $this->sharedMemory->set($key, $value, $ttl);
        
        return $this->response->json([
            'success' => $result,
            'message' => $result ? 'Value set successfully' : 'Failed to set value'
        ]);
    }
    
    public function getValue(RequestInterface $request)
    {
        $key = $request->input('key');
        $value = $this->sharedMemory->get($key);
        
        return $this->response->json([
            'exists' => !is_null($value),
            'value' => $value
        ]);
    }
    
    public function stats()
    {
        return $this->response->json($this->sharedMemory->stats());
    }
}
```
#### 4. 定期清理过期数据
在 `config/autoload/processes.php` 中添加一个进程来定期清理：
```php
<?php
declare(strict_types=1);

return [
    App\Process\SharedMemoryGCProcess::class,
];
```
创建清理进程：
```php
<?php
declare(strict_types=1);

namespace App\Process;

use App\Service\SharedMemoryService;
use Hyperf\Process\AbstractProcess;
use Hyperf\Process\Annotation\Process;

#[Process(name: "shared-memory-gc")]
class SharedMemoryGCProcess extends AbstractProcess
{
    public function __construct(private SharedMemoryService $sharedMemory)
    {
        parent::__construct();
    }
    
    public function handle(): void
    {
        while (true) {
            // 每5分钟执行一次垃圾回收
            sleep(300);
            
            $cleaned = $this->sharedMemory->gc();
            
            if ($cleaned > 0) {
                echo "Cleaned $cleaned expired items from shared memory\n";
            }
        }
    }
    
    // 可选：设置进程配置
    public function isEnable(): bool
    {
        return $this->config->get('shared_memory.gc_enabled', true);
    }
}
```
### ✅ 方案二：Shmop 扩展 (备用方案)
如果 Swoole Table 不能满足需求，可以使用 PHP 的 shmop 扩展：
```php
<?php
declare(strict_types=1);

namespace App\Service;

class ShmopService
{
    private $projectId;
    private $permissions;
    
    public function __construct()
    {
        // 使用当前文件路径生成项目ID
        $this->projectId = ftok(__FILE__, 't');
        $this->permissions = 0644;
    }
    
    public function create(int $size): int
    {
        $shmId = shmop_open($this->projectId, "c", $this->permissions, $size);
        
        if (!$shmId) {
            throw new \RuntimeException("Failed to create shared memory segment");
        }
        
        return $shmId;
    }
    
    public function write(int $shmId, string $data, int $offset = 0): int
    {
        return shmop_write($shmId, $data, $offset);
    }
    
    public function read(int $shmId, int $offset = 0, int $size = 0): string
    {
        if ($size === 0) {
            $size = shmop_size($shmId) - $offset;
        }
        
        return shmop_read($shmId, $offset, $size);
    }
    
    public function delete(int $shmId): bool
    {
        return shmop_delete($shmId);
    }
    
    public function close(int $shmId): bool
    {
        return shmop_close($shmId);
    }
}
```
## 🛡️ 数据一致性保障
在多进程环境下，数据一致性至关重要：
```php
<?php
declare(strict_types=1);

namespace App\Service;

use Swoole\Lock;

class ConcurrentSharedMemoryService extends SharedMemoryService
{
    private Lock $lock;
    
    public function __construct(ConfigInterface $config)
    {
        parent::__construct($config);
        
        // 创建互斥锁
        $this->lock = new Lock(SWOOLE_MUTEX);
    }
    
    public function safeIncr(string $key, int $value = 1): int
    {
        $this->lock->lock();
        
        try {
            $current = $this->get($key) ?? 0;
            $newValue = $current + $value;
            $this->set($key, $newValue);
            
            return $newValue;
        } finally {
            $this->lock->unlock();
        }
    }
    
    public function getAndSet(string $key, $value)
    {
        $this->lock->lock();
        
        try {
            $oldValue = $this->get($key);
            $this->set($key, $value);
            
            return $oldValue;
        } finally {
            $this->lock->unlock();
        }
    }
}
```
## 📈 监控与维护
```php
<?php
declare(strict_types=1);

namespace App\Command;

use App\Service\SharedMemoryService;
use Hyperf\Command\Command as HyperfCommand;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class SharedMemoryInfoCommand extends HyperfCommand
{
    protected $name = 'shared-memory:info';
    
    public function __construct(private SharedMemoryService $sharedMemory)
    {
        parent::__construct();
    }
    
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $stats = $this->sharedMemory->stats();
        
        $output->writeln("Shared Memory Statistics:");
        $output->writeln("Items: " . $stats['count']);
        $output->writeln("Memory Usage: " . $this->formatBytes($stats['memory_size']));
        
        return 0;
    }
    
    private function formatBytes($bytes, $precision = 2): string
    {
        $units = ['B', 'KB', 'MB', 'GB', 'TB'];
        
        $bytes = max($bytes, 0);
        $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
        $pow = min($pow, count($units) - 1);
        
        $bytes /= pow(1024, $pow);
        
        return round($bytes, $precision) . ' ' . $units[$pow];
    }
}
```
## 💡 最佳实践总结
1.  **首选 Swoole Table**：它与 Hyperf/Swoole 深度集成，性能最佳，使用最简单
2.  **合理规划容量**：根据业务需求预先分配足够空间，避免运行时空间不足
3.  **实现过期机制**：为数据添加 TTL 支持，避免内存无限增长
4.  **定期清理**：使用后台进程定期清理过期数据
5.  **保障数据一致性**：对关键操作使用锁机制确保原子性
6.  **监控内存使用**：实现监控命令，定期检查内存使用情况
7.  **优雅降级方案**：准备好当共享内存不足时的处理策略
## ⚠️ 重要注意事项
1.  **数据丢失风险**：服务器重启会导致所有数据丢失，重要数据应有持久化备份
2.  **容量限制**：Swoole Table 大小固定，需要合理预估最大需求
3.  **进程隔离**：不同服务器的进程无法共享内存，仅限单机多进程
4.  **数据类型限制**：Swoole Table 只支持有限的数据类型