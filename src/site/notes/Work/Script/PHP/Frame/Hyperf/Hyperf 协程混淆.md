---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Hyperf/Hyperf 协程混淆/","title":"Hyperf 协程混淆","tags":["flashcards","#hyperf","#coroutineConfusion"],"noteIcon":"","created":"2026-04-06T16:46:04.000+08:00","updated":"2026-07-24T10:01:21.545+08:00","dg-note-properties":{"title":"Hyperf 协程混淆","tags":["flashcards","#hyperf","#coroutineConfusion"],"reference linking":null}}
---

# 核心原则：协程内存模型
要理解数据混淆，首先必须明确 Hyperf 在 Swoole 驱动下的内存模型：
- 一个 Worker 进程内包含**多个**协程。
- 多个协程共享该进程的**绝大部分堆内存**。
- 每个协程拥有**独立**的执行栈。
- **全局变量、静态变量、单例对象的普通属性**位于**共享堆**内存；**局部变量**位于协程**独立栈**上。
因此，Hyperf 协程混淆的核心风险是：共享==1;;可变==状态。
一个 Worker 进程可以运行==1;;多个==协程，而协程之间共享进程的堆内存。
局部变量通常存储在==1;;独立栈==上，因此不会因为其他协程改写同名局部变量而混淆。
### Go 与 PHP 协程调度区别
Go 协程由 Go Runtime 调度，PHP 协程由 Swoole 调度。两者都能降低并发任务的切换成本，但调度模型和并行能力不同。

| 对比项      | Go 协程                              | PHP 协程（Swoole/Hyperf）                             |
| :------- | :--------------------------------- | :------------------------------------------------ |
| 创建方式     | `go func() {}`                     | `Coroutine::create()`                             |
| 调度模型     | M:N：多个 Goroutine **复用多个** OS 线程    | 通常**一个** Worker 线程调度**多个**协程                      |
| 并行执行     | 可通过 `GOMAXPROCS` 使用**多核并行**        | **同一 Worker 内通常不能真正并行**，依靠多个 Worker 进程扩展          |
| 调度触发     | 阻塞、Channel、锁、系统调用，也**支持运行时抢占**     | 主要在 I/O、Channel、锁、`Coroutine::sleep()` 等**操作时切换** |
| CPU 密集任务 | 运行时**可抢占长时间运行的 Goroutine**         | 长时间计算**不主动让出**会阻塞同一 Worker 的其他协程                  |
| 共享状态     | Goroutine 之间**共享堆内存**，需要锁或 Channel | 同一 Worker 协程**共享对象属性、静态变量和全局状态**                  |
### Go 示例
```go
go func() {
    doWork()
}()
```
`go` 是 Go 语言关键字，由 Go Runtime 接管调度。
### PHP Swoole 示例
```php
use Swoole\Coroutine;

Coroutine::create(function (): void {
    doWork();
});
```
`Coroutine::create()` 是 Swoole API，不是 PHP 原生关键字。
**核心区别：** Go 协程可以**跨多个 OS 线程==1;;并行==运行**；Swoole 协程通常在**单个 Worker 线程内==1;;并发==执行**，主要依赖 I/O 或同步操作**主动让出执行权**。
<!--SR:!2026-08-07,14,290-->
<?e?>
# 一、什么情况下类的属性会协程混淆？
当一个类实例被==1;;多个==协程==1;;同时==读写其属性时，就可能发生**数据混淆**。最常见的场景是单例服务：Hyperf 依赖注入默认使用单例，所有并发请求访问同一个实例。
## 单例服务
`#[Injectable]` 默认单例的普通属性是 Worker 内的共享状态。下面手动创建一个共享对象，模拟两个并发请求使用同一单例；`Channel` 用于固定执行顺序。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;

use function Swoole\Coroutine\run;

class UnsafeService
{
    public string $id = '';
}

run(function (): void {
    // 模拟 Hyperf 容器注入给并发请求的同一个单例实例。
    $service = new UnsafeService();
    $a       = new Channel(1);
    $b       = new Channel(1);
    // 1. 协程 A 写入 Alice 后等待。
    Coroutine::create(function () use ($service, $a, $b): void {
        $service->id = 'Alice';
        $a->push(true);
        $b->pop();
        // 3. 协程 A 恢复，读取到 Bob。
        printf("A expected: Alice, actual: %s\n", $service->id);
    });
    // 2. 协程 B 写入 Bob，覆盖共享属性。
    Coroutine::create(function () use ($service, $a, $b): void {
        $a->pop();
        $service->id = 'Bob';
        $b->push(true);
    });
});
```
输出：`A expected: Alice, actual: Bob`。
**结论：** 协程的局部变量隔离，但同一单例的属性共享；B 会覆盖 A 写入的 `id`。请求级数据应放入 `Context`，不要存入单例属性。
## 被长期持有的对象
放在==1;;静态==变量、==1;;全局==变量、==1;;常驻==内存数组中的对象，若被多个协程访问，其属性同样会被**共享**。
## 判断标准
	判断对象是否安全时，优先确认它是否是==1;;共享==实例以及属性是否是==1;;可变==状态。
<!--SR:!2026-07-31,8,250-->
<?e?>
# 二、为什么慎用静态字段？
**静态字段属于类本身**，不属于某个对象实例；它在类加载后存在于整个 Worker 进程生命周期内。因此，所有协程访问的都是同一份静态数据。
静态字段本质上接近==1;;全局==变量，不应存储请求相关状态。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;

use function Swoole\Coroutine\run;

class Ctx
{
    public static string $u = '';
}

$a = new Channel(1);
$b = new Channel(1);

run(function () use ($a, $b): void {
    Coroutine::create(function () use ($a, $b): void {
        // 1. 协程 A 写入自己的用户标识后让出执行权。
        Ctx::$u = 'Alice';
        $a->push(true);

        // 3. 协程 A 恢复，却读到了协程 B 覆盖后的值。
        $b->pop();
        printf("A expected: Alice, actual: %s\n", Ctx::$u);
    });

    Coroutine::create(function () use ($a, $b): void {
        $a->pop();
        // 2. 协程 B 覆盖同一个静态属性。
        Ctx::$u = 'Bob';
        $b->push(true);
    });
});
```
输出：`A expected: Alice, actual: Bob`。
`Ctx::$u` 不是==1;;原子==操作，多个协程执行时可能导致**数据混淆**。
静态字段的安全用途主要是存储==1;;只读==**配置**或**缓存**，而不是请求级状态。
<!--SR:!2026-08-04,11,270-->
<?e?>
# 三、数据混淆的情况有哪些？
## 1. 身份混淆
- 场景：在单例服务中用属性存储当前请求的用户信息。
- 后果：用户 A 的请求可能读取用户 B 的数据，造成越权漏洞。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\run;

class UserService
{
    public string $u = '';
}

run(function (): void {
    $s = new UserService();
    $a = new Channel(1);
    $b = new Channel(1);

    Coroutine::create(function () use ($s, $a, $b): void {
        // 1. 协程 A 写入自己的用户信息后让出执行权。
        $s->u = 'Alice';
        $a->push(true);

        // 3. 协程 A 恢复，读取到协程 B 覆盖后的信息。
        $b->pop();
        printf("A expected: Alice, actual: %s\n", $s->u);
    });

    Coroutine::create(function () use ($s, $a, $b): void {
        $a->pop();
        // 2. 协程 B 覆盖同一个单例属性。
        $s->u = 'Bob';
        $b->push(true);
    });
});
```
输出：`A expected: Alice, actual: Bob`
说明请求身份可能被覆盖，造成**越权**或数据**泄露**。
## 2. 计算错误与状态混乱
计数器、库存扣减、状态标志位等共享状态可能被交错修改，造成**超卖**或**状态错误**。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;

use function Swoole\Coroutine\run;

run(function (): void {
    $n = 0;
    $a = new Channel(1);
    $b = new Channel(1);

    Coroutine::create(function () use (&$n, $a, $b): void {
        // 1. 协程 A 读取 0 后让出执行权。
        $x = $n;
        $a->push(true);

        // 3. 协程 A 恢复，将 0 加 1 后写入 1。
        $b->pop();
        $n = $x + 1;
    });

    Coroutine::create(function () use (&$n, $a, $b): void {
        // 2. 协程 B 也读取 0，并覆盖 A 的计算结果。
        $x = $n;
        $b->push(true);
        $a->pop();
        $n = $x + 1;
    });
    printf("A expected: 2, actual: %d\n", $n);
});
```
输出：`A expected: 2, actual: 1`
最终 `$n` 为 `1`，而不是预期的 `2`，这就是共享状态的更新丢失。
### 典型场景：库存扣减
库存扣减是最常见的真实场景。`Redis` 的 `get` 和 `set` 都是网络 I/O，两个请求可能在两次操作之间发生协程切换。
```php
use Hyperf\Redis\Redis;

class StockService
{
    public function __construct(
        private Redis $redis,
    ) {
    }

    public function take(): bool
    {
        // 1. 请求 A、B 都读取到库存 1。
        $n = (int) $this->redis->get('stock');

        if ($n <= 0) {
            return false;
        }

        // 2. A、B 先后写入库存 0，导致重复扣减。
        $this->redis->set('stock', $n - 1);
        return true;
    }
}
```
并发执行顺序可能是：A 读取 `1`；B 读取 `1`；A 写入 `0`；B 写入 `0`。最终库存为 `0`，但两个请求都返回 `true`，实际卖出了两件商品。
正确做法是使用 Redis 原子操作，或使用数据库条件更新：
```php
public function take(): bool
{
    $n = $this->redis->decr('stock');

    if ($n >= 0) {
        return true;
    }

    $this->redis->incr('stock');
    return false;
}
```
`get`、计算、`set` 是多个步骤，不能依赖单例属性或普通锁来假设操作天然安全；应优先使用 Redis/数据库提供的原子能力。
## 3. 资源混淆
在单例服务中持有数据库连接、文件句柄或 Redis 连接，并假设它们完全隔离，可能导致一个协程拿到另一个协程未完成的状态。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;

use function Swoole\Coroutine\run;

class Db
{
    public string $tx = 'none';

    public function begin(): void
    {
        $this->tx = 'A';
    }
}

run(function (): void {
    $db = new Db();
    $a  = new Channel(1);
    $b  = new Channel(1);

    Coroutine::create(function () use ($db, $a, $b): void {
        // 1. 协程 A 开启事务后让出执行权。
        $db->begin();
        $a->push(true);

        // 3. 协程 A 恢复，却读取到协程 B 提交后的状态。
        $b->pop();
        printf("A expected: A, actual: %s\n", $db->tx);
    });

    Coroutine::create(function () use ($db, $a, $b): void {
        $a->pop();
        // 2. 协程 B 复用同一资源并提交事务。
        $db->tx = 'committed';
        $b->push(true);
    });
});
```
输出：`A expected: A, actual: committed`。
实际数据库和 Redis 客户端应使用==1;;协程连接池==，不要在单例中保存未隔离的连接状态。
## 4. 缓存污染
把类的数组属性作为临时缓存，可能导致一个协程设置的键值被另一个协程覆盖，或读取到无关数据。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;

use function Swoole\Coroutine\run;

class Cache
{
    public static array $m = [];

    public static function set(string $k, string $v): void
    {
        self::$m[$k] = $v;
    }
}

run(function (): void {
    $a = new Channel(1);
    $b = new Channel(1);

    Coroutine::create(function () use ($a, $b): void {
        // 1. 协程 A 写入临时缓存后让出执行权。
        Cache::set('u', 'Alice');
        $a->push(true);

        // 3. 协程 A 恢复，读取到协程 B 覆盖后的值。
        $b->pop();
        printf("A expected: Alice, actual: %s\n", Cache::$m['u']);
    });

    Coroutine::create(function () use ($a, $b): void {
        $a->pop();
        // 2. 协程 B 使用相同键覆盖缓存值。
        Cache::set('u', 'Bob');
        $b->push(true);
    });
});
```
输出为 `A expected: Alice, actual: Bob`。请求级临时缓存不应直接放在**共享**静态缓存中。
<!--SR:!2026-08-01,9,250-->
<?e?>
# 协程混淆的解决方案
## 1. 使用协程上下文
协程上下文是存储请求级数据的首选方案，为每个协程提供独立隔离的存储空间。
```php
use Hyperf\Context\Context; // Hyperf v3.x+
// use Hyperf\Utils\Context; // Hyperf v2.x

// 设置值，仅在当前协程有效
Context::set('user_info', $userInfo);

// 获取值
$userInfo = Context::get('user_info');
```
请求级数据优先使用==1;;Context==，而不是放入单例属性或静态字段。
## 2. 使用短生命周期依赖注入
需要保存状态的服务可以定义为非单例，每次注入时创建新实例。
```php
#[Injectable(scope: Scope::PROTOTYPE)] // 每次依赖注入都创建新实例
class StatefulService {
    public $state;
}
```
原型 Scope 的实例可以降低共享风险，因为它的状态不会默认由所有请求共用。
## 3. 使用同步原语与锁
多个协程同时读写共享状态而没有同步机制，就会导致==1;;Data Race==，进而引发数据混乱、计算错误或程序崩溃。
### 1. 使用协程锁（Lock）
协程锁确保==1;;互斥==访问临界区资源。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use Swoole\Coroutine\Lock;
use function Swoole\Coroutine\run;

run(function (): void {
    $m = new Lock();
    $n = 0;
    $d = new Channel(2);

    Coroutine::create(function () use ($m, &$n, $d): void {
        // 1. 协程 A 获取锁，独占操作共享计数器。
        $m->lock();
        try {
            $n++;
        } finally {
            $m->unlock();
        }
        $d->push(true);
    });

    Coroutine::create(function () use ($m, &$n, $d): void {
        // 2. 协程 B 等待 A 释放锁后再修改计数器。
        $m->lock();
        try {
            $n++;
        } finally {
            $m->unlock();
        }
        $d->push(true);
    });

    $d->pop();
    $d->pop();
    printf("counter: %d\n", $n);
});
```
输出：`counter: 2`。
使用锁时必须确保异常路径也能释放锁，避免后续协程永久等待。
### 2. 使用原子操作（Atomic）
对于简单的计数器、状态标志等，`Swoole\Atomic` 能保证特定操作的原子性，性能通常比锁更高。
```php
use Swoole\Atomic;
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\run;

run(function (): void {
    $n = new Atomic(0);
    $d = new Channel(2);

    Coroutine::create(function () use ($n, $d): void {
        // 1. 协程 A 原子增加计数器。
        $n->add(1);
        $d->push(true);
    });

    Coroutine::create(function () use ($n, $d): void {
        // 2. 协程 B 原子增加计数器。
        $n->add(1);
        $d->push(true);
    });

    $d->pop();
    $d->pop();
    printf("counter: %d\n", $n->get());
});
```
输出：`counter: 2`。
`Atomic` 更适合==1;;原子更新==，不适合包含多个步骤的复杂临界区。
### 3. 利用 Channel 进行通信
**不要通过共享内存来通信，而应该通过通信来共享内存**。`Channel` 可以在协程之间传递数据，并由底层处理同步。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\run;

run(function (): void {
    $c = new Channel(1);

    Coroutine::create(function () use ($c): void {
        // 1. 生产者协程通过 Channel 发送数据。
        $c->push(42);
    });

    Coroutine::create(function () use ($c): void {
        // 2. 消费者协程等待并接收数据。
        $v = $c->pop();
        printf("received: %d\n", $v);
    });
});
```
输出：`received: 42`。
Channel 的核心用途是==1;;协程通信==，减少直接共享可变内存。
<!--SR:!2026-07-30,7,250-->
<?e?>
# 注意事项表格
| 类别        | 易错点                                    | 关键原因或后果                               | 应对策略                                                                                       |
| :-------- | :------------------------------------- | :------------------------------------ | :----------------------------------------------------------------------------------------- |
| **框架特性**  | `WorkerStopHandler` 在**服务停止时可能报告协程死锁** | 服务关闭时协程可能都在等待，触发 Swoole 死锁检测，通常属于正常现象 | 如确有干扰，可在自定义 `WorkerStopHandler` 中临时设置 `Coroutine::set(['enable_deadlock_check' => false])` |
| **协程上下文** | 子协程**无法直接访问**父协程上下文                    | Hyperf 协程上下文默认隔离                      | 使用 `Context::copy($parentCoroutineId)` 显式复制，或通过 `Coroutine::pid()` 追溯父协程                   |
| **单例注入**  | `@Inject` 注入的单例对象属性在**协程间共享**          | 普通属性位于共享内存，所有协程访问同一份数据                | 将隔离数据放入 Context，或让 `__get()` / `__set()` 代理到协程上下文                                          |
| **内存管理**  | Worker 进程**内存持续增长**                    | 静态全局变量累积、协程未销毁、锁未释放或资源未关闭             | 使用 IDE 分析工具，确保协程、锁、数据库连接和文件句柄正确释放                                                          |
| **资源释放**  | 数据库连接在 `defer` 中使用时**可能被其他协程绑定**       | 协程环境下连接管理复杂                           | 留意框架更新，例如 Hyperf v2.1.2 的相关修复，并规范资源使用                                                      |
| **部署调试**  | Swoole 环境和参数配置复杂                       | 与传统 PHP-FPM 模式不同                      | 熟悉 Swoole 配置，参考官方文档和社区实践                                                                   |
# 最终判断原则
在 Hyperf 开发中，默认认为对象属性==1;;不==安全，除非能证明它只被一个协程访问、属于原型 Scope，或已经被同步原语妥善保护。
请求上下文数据的默认方案是==1;;Context 隔离==；静态字段和全局变量不应保存请求级状态。
<!--SR:!2026-07-27,7,250-->
<?e?>
