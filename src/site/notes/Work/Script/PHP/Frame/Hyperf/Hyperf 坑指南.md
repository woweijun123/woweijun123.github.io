---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Hyperf/Hyperf 坑指南/","title":"Hyperf 坑指南","tags":["flashcards"],"noteIcon":"","created":"2026-04-06T16:37:49.000+08:00","updated":"2026-04-06T16:37:49.000+08:00","dg-note-properties":{"title":"Hyperf 坑指南","tags":["flashcards"],"reference linking":null}}
---

# 注意事项表格
| 类别          | 易错点                                               | 关键原因/后果                                    | 应对策略                                                                                        |
| :---------- | :------------------------------------------------ | :----------------------------------------- | :------------------------------------------------------------------------------------------ |
| **框架特性相关**  | WorkerStopHandler 在服务停止时可能报告协程死锁                  | 服务关闭时所有协程都在等待，触发 Swoole 死锁检测（这通常是**正常现象**） | 如感干扰，可在自定义 WorkerStopHandler 中临时禁用死锁检测 `Coroutine::set(['enable_deadlock_check' => false])` |
| **协程上下文管理** | 子协程**无法直接访问**父协程的上下文数据                            | Hyperf 的协程上下文默认是隔离的                        | 使用 `Context::copy($parentCoroutineId)` 显式复制 或通过 `Coroutine::pid()` 向上追溯父协程                  |
|             | 通过 `@Inject` 注入的**单例对象**的属性在协程间共享，可能导致数据混淆        | 单例对象的普通属性存储在全局内存中，所有协程访问同一份数据              | 将需隔离的数据存入协程上下文，或利用对象的 `__get()` 和 `__set()` 魔术方法代理到协程上下文                                    |
| **内存管理**    | **内存泄漏**，表现为 Worker 进程内存持续增长（如 Worker0 内存占用过高）    | 静态全局变量累积、协程未正常销毁、锁未释放、资源未及时关闭（如数据库连接、文件句柄） | 使用 IDE 分析工具检查、确保协程正确销毁 (`Coroutine::create()` 和 `Coroutine::close()`)、确保锁释放、监控内存            |
| **资源释放与连接** | 数据库连接 (如 `hyperf/db` 组件) 在 `defer` 中使用时，可能被其他协程绑定 | 协程环境下连接管理复杂                                | 留意框架更新和修复（如 Hyperf v2.1.2 修复了相关问题），并规范资源使用                                                  |
| **部署与调试**   | 部署相对复杂，需配置 Swoole 环境和参数                           | 与传统 PHP-FPM 模式不同                           | 熟悉 Swoole 配置，参考官方文档和社区最佳实践                                                                  |
# 协程竞态加锁
多个协程**同时读写**同一个全局变量、静态变量或对象的属性，而没有适当的同步机制，就会导致**数据竞争（Data Race）**，进而引发数据混乱、计算错误或程序崩溃。以下是保证协程原子性的几种方式：
### 1.  使用协程锁 (Mutex)
协程锁机制来确保**同一时间只有一个协程**能访问临界区资源。
```php
use Swoole\Coroutine\Mutex;

$mutex = new Mutex();
$sharedVariable = 0;

co(function () use ($mutex, &$sharedVariable) {
	$mutex->lock(); // 加锁
	$sharedVariable++; // 安全地操作共享变量
	$mutex->unlock(); // 解锁
});
```
### 2.  使用原子操作 (Atomic)
对于简单的**计数器**、状态标志等，使用 `Swoole\Atomic` 是最佳选择。它能保证特定操作的原子性，**性能通常比锁更高**。
```php
use Swoole\Atomic;

$counter = new Atomic(0);

co(function () use ($counter) {
	$counter->add(1); // 原子性地增加
});
```
### 3.  利用 Channel 进行通信
“不要通过共享内存来通信，而应该通过通信来共享内存”。这是 Go 语言倡导的理念，在 Hyperf 中也同样适用。你可以使用 `Swoole\Coroutine\Channel` 在协程之间传递数据，Channel 底层会自动处理同步问题，从而避免显式的锁操作。
```php
use Swoole\Coroutine\Channel;

$chan = new Channel(1); // 创建一个容量为1的Channel
$sharedValue = 42;

co(function () use ($chan, $sharedValue) {
	$chan->push($sharedValue); // 生产者协程推送数据
});

co(function () use ($chan) {
	$value = $chan->pop(); // 消费者协程取出数据，如果Channel为空则会挂起等待
	// 处理 $value
});
```
### 4.  避免共享状态
**重新思考你的设计**，看是否能避免共享状态。例如，将数据封装在对象实例中，并通过参数传递，而不是使用全局变量。