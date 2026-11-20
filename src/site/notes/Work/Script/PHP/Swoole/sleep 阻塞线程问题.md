---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Swoole/sleep 阻塞线程问题/","title":"sleep 阻塞线程问题","tags":["flashcards"],"noteIcon":"","created":"2025-12-15T13:38:05.842+08:00","updated":"2026-03-24T17:50:11.252+08:00","dg-note-properties":{"title":"sleep 阻塞线程问题","tags":["flashcards"],"reference linking":null}}
---

# 写在前面
1. 在`demoNormalSleep()`**非协程环境**中执行`sleep()`会**阻塞整个线程**，导致后续协程无法切换
2. 在`demoCoroutineSleep()`**协程环境**中执行 `sleep()` 效果和协程sleep语法 `Coroutine::sleep();` **一致**，**因为`swoole`在底层hook了`sleep`**
# 实例
```php
// -----------------------------------------------------------
// 演示 1: 普通 sleep (阻塞行为)
// -----------------------------------------------------------
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;

function demoNormalSleep(): void
{
    echo "=== 普通 sleep 演示 (阻塞/串行) ===" . PHP_EOL;
    $start     = microtime(true);
    $taskCount = 2;
    // 在原生 PHP 进程中，sleep() 会阻塞整个进程，协程无法并发调度。
    for ($i = 0; $i < $taskCount; $i++) {
        $taskStart = microtime(true);
        echo "任务 $i 开始执行" . PHP_EOL;
        sleep(2); // !!! 阻塞整个 PHP 进程 !!!
        echo "任务 $i 完成，耗时: " . (microtime(true) - $taskStart) . " 秒" . PHP_EOL;
    }
    // 此处协程会被上面的 sleep() 阻塞，等到 sleep() 结束后才会继续执行。
    demoCoroutineSleep();
    echo "总耗时: " . number_format(microtime(true) - $start, 3) . " 秒" . PHP_EOL;
    echo "=== 普通 sleep 演示结束 ===" . PHP_EOL . PHP_EOL;
}

// -----------------------------------------------------------
// 演示 2: 协程化 sleep (非阻塞/并发行为)
// -----------------------------------------------------------
function demoCoroutineSleep(): void
{
    echo "=== 协程化 sleep 演示 (非阻塞/并发) ===" . PHP_EOL;
    $start     = microtime(true);
    $taskCount = 2;
    // 使用 Swoole\Coroutine\run 包裹，提供协程环境
    Swoole\Coroutine\run(function () use ($start, $taskCount) {
        // 使用 Channel 来等待所有协程完成
        $chan = new Channel($taskCount);

        for ($i = 0; $i < $taskCount; $i++) {
            // 使用 go() 创建新的协程
            Coroutine::create(function () use ($i, $chan) {
                $taskStart = microtime(true);
                echo "协程任务 $i 开始执行" . PHP_EOL;

                // !!! 关键点: 协程 sleep 只挂起当前协程，让出 CPU 给其他协程 !!!
                // sleep(2); // 协程环境下使用原生PHP函数sleep() 和 Coroutine::sleep() 效果一样
                Coroutine::sleep(2);

                echo "协程任务 $i 完成，耗时: " . number_format(microtime(true) - $taskStart, 3) . " 秒" . PHP_EOL;
                $chan->push(true); // 通知完成
            });
        }

        // 等待所有任务完成
        for ($i = 0; $i < $taskCount; $i++) {
            $chan->pop();
        }

        echo "总耗时: " . number_format(microtime(true) - $start, 3) . " 秒" . PHP_EOL;
    });

    echo "=== 协程化 sleep 演示结束 ===" . PHP_EOL;
}
// 运行演示
demoNormalSleep();    // 线程+协程「总耗时约 6 秒」
demoCoroutineSleep(); // 协程「总耗时约 2 秒」
```
## 执行结果
```shell
=== 普通 sleep 演示 (阻塞/串行) ===
任务 0 开始执行
任务 0 完成，耗时: 2.0050489902496 秒
任务 1 开始执行
任务 1 完成，耗时: 2.00470495224 秒
=== 协程化 sleep 演示 (非阻塞/并发) ===
协程任务 0 开始执行
协程任务 1 开始执行
协程任务 0 完成，耗时: 2.002 秒
协程任务 1 完成，耗时: 2.002 秒
总耗时: 2.003 秒
=== 协程化 sleep 演示结束 ===
总耗时: 6.013 秒
=== 普通 sleep 演示结束 ===

=== 协程化 sleep 演示 (非阻塞/并发) ===
协程任务 0 开始执行
协程任务 1 开始执行
协程任务 0 完成，耗时: 2.002 秒
协程任务 1 完成，耗时: 2.002 秒
总耗时: 2.003 秒
=== 协程化 sleep 演示结束 ===
```