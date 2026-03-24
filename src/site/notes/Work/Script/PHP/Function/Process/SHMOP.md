---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Process/SHMOP/","title":"SHMOP","tags":["flashcards"],"noteIcon":"","created":"2025-09-04T13:45:46.717+08:00","updated":"2026-03-24T17:36:51.613+08:00"}
---

# PHP 主子进程共享内存
- [shmop_open](https://www.php.net/manual/zh/function.shmop-open.php) — 创建或打开共享内存块
	- key
		- 共享内存块的系统 ID。可以作为十进制或十六进制传递。
	- mode
		- **`a`** 表示访问（为 shmat 设置 SHM_RDONLY）当您需要**打开现有共享内存段**进行**只读**时，请使用此标志
		- **`c`** 表示创建（IPC_CREATE 集）当您需要创建**新的共享内存段**时，或者如果存在具有相同键的段，请尝试打开它进行读写时使用此标志
		- **`w`** 表示读写访问，当需要**读取和写入**共享内存段时使用此标志，在大多数情况下使用此标志。
		- **`n`** 创建一个新的内存段（设置 IPC_CREATE|IPC_EXCL） 当您想要创建一个**新的共享内存段**时使用此标志，但如果已经存在**具有相同标志的共享内存段，则失败**。这对于**安全**目的很有用，使用它可以防止竞争条件漏洞利用。
	- permissions
		- 您希望分配给内存段的权限与文件的权限相同。权限需要以八进制形式传递，例如 `0644`
	- size
		- 要创建的共享内存块的大小（以字节为单位）
- [shmop_read](https://www.php.net/manual/zh/function.shmop-read.php) — 从共享内存块中读取数据
- [shmop_size](https://www.php.net/manual/zh/function.shmop-size.php) — 获取共享内存块的大小
- [shmop_write](https://www.php.net/manual/zh/function.shmop-write.php) — 将数据写入共享内存块
- [shmop_delete](https://www.php.net/manual/zh/function.shmop-delete.php) — 删除共享内存块
- [shmop_close](https://www.php.net/manual/zh/function.shmop-close.php) — 关闭共享内存块「本函数已自 PHP 8.0.0 起被_废弃_。强烈建议不要依赖本函数。」
```php
$shmKey = ftok(__FILE__, 't');
// 定义共享内存段大小为 1024 字节（1KB）
// 重要：这是共享内存段的最大容量限制
// 1. 写入的数据超过此大小会被截断
// 2. 读取时指定的长度不能超过此值
// 3. 建议根据实际需要存储的数据大小来设置此值
$shmSize = 1024;

// 创建一个共享内存块
$shmId = shmop_open($shmKey, 'c', 0644, $shmSize);
if (!$shmId) {
    die("无法创建共享内存段\n");
}
echo "主进程已创建共享内存段\n";

// 写入初始数据
writeData($shmId, ['some_data' => 'Hello, World!']);
// 创建子进程
$pid = pcntl_fork();

if ($pid === -1) {
    die("无法创建子进程\n");
} elseif ($pid > 0) {
    // 主进程
    echo "主进程（PID: " . getmypid() . "）等待子进程...\n";
    pcntl_wait($status);

    // 子进程退出后，读取共享内存中的数据
    $dataFromChild = readData($shmId);
    echo "主进程读到子进程修改后的数据: " . json_encode($dataFromChild) . "\n";

    // 只有创建共享内存段的进程才能删除它
    shmop_delete($shmId);
    echo "主进程已删除共享内存段\n";
} else {
    // 子进程
    echo "子进程（PID: " . getmypid() . "）正在运行...\n";

    // 子进程打开同一个共享内存段
    $shmIdChild = shmop_open($shmKey, 'w', 0, 0);
    if (!$shmIdChild) {
        die("子进程无法打开共享内存段\n");
    }

    // 读取主进程写入的数据
    // 读取时先获取长度，再读取实际数据
    $dataFromParent = readData($shmId);
    echo "子进程读到主进程的数据: " . json_encode($dataFromParent) . "\n";

    // 修改数据并写回
    writeData($shmId, array_merge($dataFromParent, ['some_data' => 'Modified by Child!']));
    echo "子进程已完成任务并退出\n";
    exit(0);
}

function writeData(Shmop $shmId, array $data = [])
{
    $initialData = serialize($data);
    $lengthInfo  = pack('L', strlen($initialData)); // 4字节长度信息
    // 写入数据长度信息 + 实际数据
    $fullData = $lengthInfo . $initialData;
    shmop_write($shmId, $fullData, 0);
}

function readData(Shmop $shmId): mixed
{
    $lengthData = shmop_read($shmId, 0, 4);
    $dataLength = unpack('L', $lengthData)[1];
    $actualData = shmop_read($shmId, 4, $dataLength);

    return unserialize($actualData);
}
```
## 多进程共享内存数据竞争问题
```php
$shmId = shmop_open(ftok(__FILE__, 't'), 'c', 0644, 4);
shmop_write($shmId, pack('L', 0), 0);

for ($i = 0; $i < 3; $i++) {
    if (pcntl_fork() == 0) {
        for ($j = 0; $j < 100; $j++) {
            $value = unpack('L', shmop_read($shmId, 0, 4))[1];
            usleep(rand(1, 100));
            shmop_write($shmId, pack('L', $value + 1), 0);
        }
        exit;
    }
}

do {
    $pid = pcntl_waitpid(0, $status, WNOHANG);
} while ($pid != -1);

$finalValue = unpack('L', shmop_read($shmId, 0, 4))[1];
echo "期望: 300, 实际: $finalValue", PHP_EOL; // 期望: 300, 实际: 116

shmop_delete($shmId);
```
执行结果
```
期望: 300, 实际: 116
```
### 解决方案：使用信号量互斥锁
[PHP: sem\_acquire - Manual](https://www.php.net/manual/zh/function.sem-acquire.php)
```php
$key   = ftok(__FILE__, 't');
$shmId = shmop_open($key, 'c', 0644, 4);
// 创建信号量（二进制信号量，用于互斥访问）
$semId = sem_get($key, 1, 0644, 1);
if (!$semId) {
    die("无法创建信号量");
}
shmop_write($shmId, pack('L', 0), 0);

for ($i = 0; $i < 3; $i++) {
    if (pcntl_fork() == 0) {
        // 获取信号量（进入临界区）
        if (!sem_acquire($semId)) {
            echo "子进程 $i 无法获取信号量", PHP_EOL;
            exit(1);
        }
        try {
            for ($j = 0; $j < 100; $j++) {
                $value = unpack('L', shmop_read($shmId, 0, 4))[1];
                usleep(rand(1, 100));
                shmop_write($shmId, pack('L', $value + 1), 0);
            }
        } finally {
            sem_release($semId); // 释放信号量（退出临界区）
        }
        exit;
    }
}

do {
    $pid = pcntl_waitpid(0, $status, WNOHANG);
} while ($pid != -1);

$finalValue = unpack('L', shmop_read($shmId, 0, 4))[1];
echo "期望: 300, 实际: $finalValue", PHP_EOL; // 期望: 300, 实际: 300

shmop_delete($shmId);
```
执行结果
```
期望: 300, 实际: 300
```
# 信号量超时机制详解
PHP 原生的 `sem_acquire()` 函数默认是阻塞的，但我们可以通过几种方式实现超时机制。
### 方法1：使用非阻塞模式 + 循环尝试
```php
$semKey = ftok(__FILE__, 's');
$semId = sem_get($semKey, 1);

// 超时时间（秒）
$timeout = 5;
$startTime = time();
$acquired = false;

while (time() - $startTime < $timeout) {
    // 非阻塞方式尝试获取信号量
    if (sem_acquire($semId, true)) {
        $acquired = true;
        break;
    }
    
    // 等待一段时间再重试
    usleep(100000); // 100ms
}

if ($acquired) {
    try {
        echo "成功获取信号量锁\n";
        // 执行临界区代码
        // ...
    } finally {
        // 确保总是释放信号量
        sem_release($semId);
        echo "释放信号量锁\n";
    }
} else {
    echo "获取信号量锁超时\n";
    // 处理超时逻辑
    // 可能是抛出异常、返回错误码等
}
```
### 方法2：使用 PCNTL 扩展和警报信号
```php
// 需要启用 PCNTL 扩展
$semKey = ftok(__FILE__, 's');
$semId = sem_get($semKey, 1);

// 超时处理函数
function timeoutHandler($signo)
{
    throw new Exception("获取信号量锁超时");
}

// 设置超时处理
pcntl_signal(SIGALRM, "timeoutHandler");
pcntl_alarm(5); // 5秒后发送警报信号

try {
    // 尝试获取信号量（这会阻塞，但会被信号中断）
    if (sem_acquire($semId)) {
        // 取消警报，因为已经成功获取
        pcntl_alarm(0);
        
        echo "成功获取信号量锁\n";
        // 执行临界区代码
        // ...
    }
} catch (Exception $e) {
    echo $e->getMessage() . "\n";
    // 处理超时
} finally {
    // 确保取消警报
    pcntl_alarm(0);
    
    // 如果成功获取了信号量，则释放
    if (sem_release($semId)) {
        echo "释放信号量锁\n";
    }
}

// 恢复默认信号处理
pcntl_signal(SIGALRM, SIG_DFL);
```
### 方法3：封装为可重用的超时信号量类
```php
class TimeoutSemaphore
{
    private $semId;
    private $key;
    
    public function __construct($key, $maxAcquire = 1)
    {
        $this->key = $key;
        $this->semId = sem_get($this->key, $maxAcquire);
    }
    
    /**
     * 尝试获取信号量锁，支持超时
     */
    public function acquire($timeout = 0)
    {
        if ($timeout <= 0) {
            // 无限等待
            return sem_acquire($this->semId);
        }
        
        $startTime = microtime(true);
        $timeoutMs = $timeout * 1000000; // 转换为微秒
        
        while (true) {
            // 非阻塞尝试
            if (sem_acquire($this->semId, true)) {
                return true;
            }
            
            // 检查是否超时
            $elapsed = (microtime(true) - $startTime) * 1000000;
            if ($elapsed >= $timeoutMs) {
                return false;
            }
            
            // 等待一段时间再重试
            usleep(min(100000, $timeoutMs - $elapsed)); // 最多等待100ms
        }
    }
    
    /**
     * 释放信号量锁
     */
    public function release()
    {
        return sem_release($this->semId);
    }
    
    /**
     * 删除信号量
     */
    public function remove()
    {
        return sem_remove($this->semId);
    }
    
    /**
     * 使用锁执行代码（带超时）
     */
    public function synchronized(callable $callback, $timeout = 0)
    {
        if (!$this->acquire($timeout)) {
            throw new RuntimeException("获取锁超时");
        }
        
        try {
            return call_user_func($callback);
        } finally {
            $this->release();
        }
    }
}

// 使用示例
$semaphore = new TimeoutSemaphore(ftok(__FILE__, 'x'));

try {
    $result = $semaphore->synchronized(function() {
        // 临界区代码
        echo "在临界区内执行\n";
        sleep(2); // 模拟耗时操作
        return "操作完成";
    }, 5); // 5秒超时
    
    echo "结果: " . $result . "\n";
} catch (RuntimeException $e) {
    echo "错误: " . $e->getMessage() . "\n";
}
```
## 推荐方案
对于大多数应用场景，我推荐使用**方法1（非阻塞模式+循环尝试）**，因为：
1. **兼容性好**：不依赖 PCNTL 扩展
2. **实现简单**：代码直观易懂
3. **可控性强**：可以灵活调整重试间隔和超时时间
4. **资源友好**：避免了信号处理可能带来的复杂性
# 信号量的原理
对这个计数器的操作不是随意的，只能通过两个标准的、保证原子性的操作「P、V」来进行，它们最初由荷兰计算机科学家 Dijkstra 提出
-  **本质：** 信号量是一个**整数计数器**，代表当前**可用资源的数量**。
-  **目的：** 实现对共享资源的**同步访问**，确保同一时间只有允许数量的进程访问资源。

| 操作                           | 对应 PHP (System V) | 动作 (试图做什么)                   | 逻辑 (导致什么结果)                                 |
| :--------------------------- | :---------------- | :--------------------------- | :------------------------------------------ |
| **P()** Proberen (Wait/尝试)   | `sem_acquire()`   | **获取**资源（计数器 $\mathbf{-1}$）。 | 计数 $\ge 0$：获取成功，继续执行。**计数 $< 0$：进程被阻塞。**    |
| **V()** Verhogen (Signal/增加) | `sem_release()`   | **释放**资源（计数器 $\mathbf{+1}$）。 | 计数 $\le 0$：唤醒一个等待中的阻塞进程。**计数 $> 0$：** 释放完成。 |
**重要记忆点：** 信号量为**负数**时，其**绝对值**等于当前**等待资源的进程数量**。
## 为什么可以解决 IPC 通信的锁机制？
IPC（如共享内存）通信时，最大的问题就是**竞态条件**：多个进程同时读写共享资源，导致数据不一致。
信号量通过一种 **“互斥”** 和 **“同步”** 的机制来解决这个问题。
1.  **互斥 (Mutex) - 二进制信号量**
    *   这是最常用的场景。将信号量初始化为 `1`，代表一把锁。
    *   一个进程在进入**临界区**（访问共享资源的代码段）之前，先执行 `P()` 操作。
        *   如果成功（值由 `1` -> `0`），它拿到锁，进入临界区。
        *   此时如果另一个进程也执行 `P()` 操作，值会变为 `-1`。该进程将被阻塞，在门外等待。
    *   当第一个进程完成操作后，退出临界区时执行 `V()` 操作。
        *   值由 `0` -> `1`。如果有等待的进程（`值 <= 0`），则唤醒其中一个。
    *   这样就保证了**同一时刻，只有一个进程能进入临界区**，就像只有一个管理员允许一个人进入房间一样，从而避免了数据混乱。
2.  **同步 (Synchronization) - 计数信号量**
    *   信号量还可以初始化为 `N`（N > 1），用于控制访问一组 `N` 个 identical 资源（比如 `N` 个打印机端口，`N` 个数据库连接）。
    *   进程仍然通过 `P()` 和 `V()` 来申请和释放资源。这允许多个进程同时访问资源，但总数被限制在 `N` 个，实现了高效的资源池管理。

**总结：** 信号量通过让进程在无法获取资源时“睡眠等待”，在资源可用时被“唤醒”的方式，优雅地实现了进程间的协调，避免了忙等待（busy-waiting）造成的CPU资源浪费，从而解决了IPC中的竞态问题。
## 为什么是原子性的？
这是整个机制能够正确工作的**基石**。原子性的意思是：**操作一旦开始，就要一次性、不可中断地执行完成，中间不能插入任何其他进程的操作。**
如果 `P()` 和 `V()` 操作不是原子性的，就会发生**最糟糕的情况**：即使在有信号量保护的情况下，依然会出现竞态条件。
我们来看一个非原子操作的 `P()` 是如何失败的：
假设信号量 `sem = 1`，进程 A 和进程 B 同时执行 `P()` 操作。
一个非原子的 `P()` 可能被拆解为三步：
1.  `reg = sem`  (从内存读值到寄存器)
2.  `reg = reg - 1` (在寄存器中计算)
3.  `sem = reg`  (将新值写回内存)

| 时间序列 | 进程 A (非原子 P())            | 进程 B (非原子 P())            | 信号量值 (内存中) |
| :--- | :----------------------------- | :----------------------------- | :-------------- |
| t1   | `reg_A = sem` // 读到 `reg_A=1` |                                | 1               |
| t2   | `reg_A = reg_A - 1` // `reg_A=0` |                                | 1               |
| t3   |                                | `reg_B = sem` // **读到 `reg_B=1`** | 1               |
| t4   |                                | `reg_B = reg_B - 1` // `reg_B=0` | 1               |
| t5   | `sem = reg_A` // **写入 `0`**     |                                | 0               |
| t6   |                                | `sem = reg_B` // **写入 `0`**     | **0 (错误!)**   |
结果：两个进程都认为自己成功执行了 `P()`，都进入了临界区！信号量失去了保护作用。
**如何保证原子性？**
操作系统和硬件底层通过以下方式保证 `P()` 和 `V()` 的原子性：
1.  **硬件支持**：现代CPU提供了特殊的**原子指令**，如 **Test-and-Set**、**Compare-and-Swap** 等。这些指令在一个不可中断的总线周期内完成“读-修改-写”回内存的操作，从而确保在执行过程中其他处理器无法访问同一块内存。
2.  **内核实现**：信号量的操作是**系统调用**。当一个进程执行系统调用陷入内核后，在它从内核返回用户空间之前，是**不会被调度器切换掉**的。这就保证了在内核中执行 `P()`/`V()` 操作的完整性和不可中断性。
**内核态的执行流程可以简化为：**
*   进入内核，**禁用中断**（防止被时钟中断强行切换）。
*   执行指令检查并修改信号量值。
*   如果需要阻塞当前进程，内核会安全地将其放入等待队列并切换进程。
*   如果需要唤醒其他进程，内核会从等待队列中取出进程。
*   完成操作，**启用中断**，返回用户态。
正是这种**硬件和操作系统内核级别的共同保障**，使得信号量操作成为原子操作，从而成为构建可靠并发控制的坚实基础。
# 信号量的值变化分析「多进程同时获取」
## 信号量值的变化规则
信号量的值变化遵循一个关键公式：**最终值 = 初始值 - 成功获取信号的进程数**
但更准确的理解是：
- 每个成功的 `P()` 操作会使信号量值减 1
- 每个因资源不足而阻塞的 `P()` 操作也会使信号量值减 1，但此时值为负
- 负值的**绝对值**表示正在**等待**该资源的**进程数量**
## 3个进程同时获取信号量的场景分析
假设有一个初始值为 `N` 的信号量，3个进程同时尝试执行 `P()` 操作：
### 情况1: N ≥ 3 (充足资源)
- 所有3个进程都能成功获取资源
- 每个 `P()` 操作原子性地将信号量值减 1
- **最终信号量值 = N - 3**
- 没有进程被阻塞
**示例**: 如果 N=5，3个进程同时获取后，信号量值变为 2
### 情况2: 1 ≤ N < 3 (资源不足)
- 前 N 个进程成功获取资源
- 剩余 (3-N) 个进程被阻塞并进入等待队列
- 每个 `P()` 操作都会使信号量值减 1，无论成功与否
- **最终信号量值 = N - 3** (负值)
- 负值的绝对值 |N-3| 表示被阻塞的进程数量
**示例**:
- 如果 N=2，则:
	- 2个进程成功获取资源
	- 1个进程被阻塞
	- 信号量值变为 2-3 = -1 (表示有1个进程在等待)
- 如果 N=1，则:
	- 1个进程成功获取资源
	- 2个进程被阻塞
	- 信号量值变为 1-3 = -2 (表示有2个进程在等待)
### 情况3: N = 0 (无可用资源)
- 所有3个进程都被阻塞
- **最终信号量值 = 0 - 3 = -3**
- 表示有3个进程在等待资源
### 情况4: N < 0 (已有进程在等待)
假设初始时信号量值已经是负值 -K (表示有K个进程在等待)
- 所有3个新进程都会被阻塞
- **最终信号量值 = -K - 3**
- 表示总共有 (K+3) 个进程在等待
## 关键点：原子性的重要性
这种值变化行为之所以可靠，完全依赖于 `P()` 操作的**原子性**。原子性确保了：
1. **值的读取、修改和写入**是一个不可中断的单一操作
2. **进程的阻塞决策**与值的修改是原子性的
3. 即使多个进程同时尝试获取，**操作系统也会序列化这些请求**
如果没有原子性，就会出现竞态条件，导致：
- 多个进程可能看到相同的信号量值
- 多个进程可能都认为自己成功获取了资源
- 实际获取资源的进程数可能超过信号量的容量
## 实际执行序列示例
假设初始信号量值 S=1，进程P1、P2、P3同时执行 `P()` 操作：
1. 操作系统内核序列化这些请求（假设顺序为P1、P2、P3）
2. P1执行 `P()`: S = 1-1 = 0 (≥0)，P1成功获取，继续执行
3. P2执行 `P()`: S = 0-1 = -1 (<0)，P2被阻塞，加入等待队列
4. P3执行 `P()`: S = -1-1 = -2 (<0)，P3被阻塞，加入等待队列
5. 最终信号量值 = -2，表示有2个进程在等待

当P1执行 `V()` 操作释放资源时：
1. S = -2+1 = -1 (仍然≤0)
2. 操作系统会唤醒等待队列中的一个进程（比如P2）
3. P2被唤醒后，实际上已经完成了 `P()` 操作，可以继续执行
4. 信号量值保持为 -1，表示还有1个进程(P3)在等待
## 总结
当多个进程同时获取信号量时：
- 信号量的值会减少，减少的数量等于尝试获取的进程数
- 值为负时，其绝对值表示等待的进程数量
- 只有部分进程能立即获取资源（当初始值N≥1时）
- 这种行为的正确性完全依赖于 `P()` 操作的原子性实现

理解这一机制对于设计和调试使用信号量进行同步的并发程序至关重要。