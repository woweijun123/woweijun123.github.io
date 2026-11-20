---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Process/PHP Process Management/","title":"PHP Process Management","tags":["flashcards"],"noteIcon":"","created":"2026-11-04T14:30:08.680+08:00","updated":"2026-11-04T15:27:18.808+08:00","dg-note-properties":{"title":"PHP Process Management","tags":["flashcards"],"reference linking":null}}
---

# 选型速查
| **方法**             | **主要能力**            | **输出方式**                 | **适用场景**               |
| ------------------ | ------------------- | ------------------------ | ---------------------- |
| `exec()`           | 执行命令并取得**退出码**      | 返回**最后一行**，数组接收**全部文本行** | 简单命令、需要**退出码和文本结果**    |
| `system()`         | 执行命令并**实时转发文本输出**   | **直接写入 PHP 标准输出**，返回最后一行 | CLI 中**实时展示**简单命令输出    |
| `passthru()`       | **原样转发**命令输出        | **直接传递原始数据**，无命令输出返回值    | **二进制数据或大量输出**         |
| `shell_exec()`、反引号 | 取得**完整标准输出**        | 返回**完整 stdout 字符串**      | **一次性获取完整文本**          |
| `proc_open()`      | **自定义标准流并管理生命周期**   | 返回**进程资源和管道**            | **双向通信、长时间任务、精细控制**    |
| `pcntl_fork()`     | **复制当前 PHP 进程**     | 父子进程从**调用点继续执行**         | **Unix 守护进程、多任务处理**    |
| `pcntl_exec()`     | **用新程序替换当前进程**      | **成功后不返回**               | **当前进程切换为另一个程序**       |
| `Swoole\Process`   | 面向对象的**进程与 IPC 封装** | **管道、消息队列**等             | **Swoole Worker、后台任务** |
| `parallel\Runtime` | 在线程中**并行执行任务**      | 返回 `parallel\Future`     | **CPU 密集型并行计算**        |
- 只需要运行一次命令时，优先选择简单命令函数。
- 需要独立控制三个标准流、持续读取或终止子进程时，使用 `proc_open()`。
- 需要复制当前 PHP 执行现场时使用 PCNTL；需要线程并行时使用 `parallel`，不要把它当作子进程 API。
# 简单命令执行
这些函数适合一次性命令，调用方通常会等待外部程序结束。命令字符串可能经过 shell 解析，任何外部输入都必须正确转义；复杂参数优先使用 `proc_open()` 的数组命令形式。
## `exec()`
- **返回值：** 命令输出的最后一行。
- **引用参数：** `$output` 接收所有文本行，`$resultCode` 接收退出码。
- **边界：** 调用会等待命令结束，不适合长驻进程或交互式通信。
```php
$lastLine = exec('ls -l', $output, $resultCode);

echo "最后一行：{$lastLine}", PHP_EOL;
print_r($output);
echo "退出码：{$resultCode}", PHP_EOL;
```
## `system()`
- **行为：** 执行期间将输出直接转发到 PHP 的标准输出。
- **返回值：** 命令输出的最后一行；引用参数接收退出码。
```php
echo "开始执行 ping", PHP_EOL;
$lastLine = system('ping -c 3 127.0.0.1', $resultCode);
echo "最后一行：{$lastLine}", PHP_EOL;
echo "退出码：{$resultCode}", PHP_EOL;
```
## `passthru()`
- **行为：** 将命令的原始输出直接传递给调用者，不把完整内容保存在 PHP 变量中。
- **用途：** 适合二进制输出或大量数据；可通过引用参数取得退出码。
```php
header('Content-Type: image/jpeg');
header('Content-Disposition: inline; filename="generated-image.jpg"');

passthru('/usr/bin/generate_image.sh --format jpeg', $resultCode);
```
## `shell_exec()` 与反引号
- **返回值：** 将完整 stdout 作为字符串返回；不直接提供退出码。
- **注意：** `null` 既可能表示执行失败，也可能表示命令没有输出，不能仅凭返回值可靠判断成功与否。
```php
$fileContent = shell_exec('cat /etc/hosts');
// 等价写法：$fileContent = `cat /etc/hosts`;

if ($fileContent !== null) {
    echo $fileContent;
}
```
# `proc_open()` 进程控制
[`proc_open()`](https://www.php.net/manual/zh/function.proc-open.php) 用描述符规格连接父进程与子进程的标准流，并返回可查询、终止和回收的进程资源。
- **关联原理：** 管道没有消息边界，EOF 依赖所有写端关闭，见 [[Work/Script/PHP/Function/Process/IPC#1. 匿名管道：父子进程的单向字节流\|IPC：匿名管道]]。
- **关联 API：** 复制当前 PHP 执行现场见 [[Work/Script/PHP/Function/Process/PCNTL#pcntl_fork\|pcntl_fork()]]。
## 核心模型
| 子进程描述符 | 子进程视角 | 父进程取得的管道 | 常见用途 |
| --- | --- | --- | --- |
| `0` stdin | 读取 | 可写 | 父进程向子进程发送输入 |
| `1` stdout | 写入 | 可读 | 父进程读取正常输出 |
| `2` stderr | 写入 | 可读 | 父进程独立读取错误输出 |
- `['pipe', 'r']` 中的方向是**子进程视角**：子进程读取，父进程写入。
- `['pipe', 'w']` 表示子进程写入，父进程读取。
- stdout 和 stderr 必须同时排空，否则其中任意管道缓冲区满都可能阻塞子进程。
## 参数
| 参数               | 类型            | 说明                                          |
| ---------------- | ------------- | ------------------------------------------- |
| `command`        | string\|array | 外部程序及参数；PHP 7.4+ 优先使用**数组**绕过 shell 并避免手工转义 |
| `descriptorSpec` | array         | 描述子进程文件描述符如何连接到**管道、文件、现有流**                |
| `pipes`          | array         | 引用返回父进程一侧的**管道资源**，只包含规格中定义为 `pipe` 的描述符    |
| `cwd`            | ?string       | 子进程初始**工作目录**；使用绝对路径，`null` 表示继承当前目录        |
| `envVars`        | ?array        | 子进程**环境变量**；`null` 表示继承当前环境                 |

描述符支持三种常用配置：
- `['pipe', 'r'/'w']`：创建父子进程通信管道。
- `['file', '/path/to/file', 'a']`：将描述符重定向到文件。
- `STDOUT`、`STDERR` 等流资源：让子进程继承现有流。
## 一次性命令：阻塞读取到 EOF
- **输入结束：** 父进程写完脚本后关闭 stdin，子 PHP 才能确认没有后续代码。
- **读取完成：** 子进程执行完毕并关闭 stdout，`stream_get_contents()` 收到 EOF 后返回。
```php
$descriptorSpec = [
    0 => ['pipe', 'r'],
    1 => ['pipe', 'w'],
    2 => ['file', '/tmp/error-output.txt', 'a'],
];

$process = proc_open(
    [PHP_BINARY],
    $descriptorSpec,
    $pipes,
    '/tmp',
    ['some_option' => 'aeiou'],
);

if (!is_resource($process)) {
    throw new RuntimeException('无法启动子进程');
}

// 子 PHP 没有脚本文件参数时从 stdin 读取代码。
fwrite($pipes[0], '<?php echo getenv("some_option"), PHP_EOL;');
// 关闭 stdin 表示输入完成，否则子 PHP 会继续等待后续代码。
fclose($pipes[0]);

// 子进程会正常退出并关闭 stdout，因此这里最终可以读到 EOF。
$output = stream_get_contents($pipes[1]);
fclose($pipes[1]);

$exitCode = proc_close($process);
echo "子进程输出: " . $output, PHP_EOL;
echo "退出码：{$exitCode}", PHP_EOL;
```
输出
```
子进程输出: aeiou

退出码：0
```
- **阻塞读取成立条件：** 子进程退出并关闭 stdout，父进程才能读到 EOF。
- **长驻进程边界：** stdout/stderr 持续打开，不能沿用读取到 EOF 的方式。
## 长驻进程：非阻塞持续读取
- **EOF：** 长驻进程持续输出，**不会主动关闭**标准流。
- **流模式：** 将 stdout/stderr 设为非阻塞。
- **读取边界：** 用 `stream_select()` 等待就绪，每次只读取当前已有数据。
### `stream_select()` 的就绪语义
- **可读：** 表示当前有数据、已到 EOF 或出现异常，不保证任意长度的后续读取可以完成。
- **数组改写：** 返回后，`$read`、`$write` 只保留已就绪的流，应遍历返回后的数组。
- **写集合：** 只有存在待发送数据时才监听 stdin。
- **空转风险：** stdin 通常一直可写，持续监听会让 `stream_select()` 立即返回。
- **错误：** 返回 `false` 表示调用失败，必须与超时返回的 `0` 区分。
- **按行读取风险：** 阻塞模式的 `fgets()` 收到半行且没有换行符时仍会等待。
- **持续流对策：** 优先采用非阻塞读取。
### 主进程
- **终止时机：** 运行十秒后发送终止信号。
- **退出条件：** 子进程退出，且 stdout/stderr 均排空至 EOF。
```php
$dir = __DIR__;
$pid = posix_getpid();
// 定义 I/O 规范，不需要 STDIN (0)
$spec = [
    1 => ['pipe', 'w'], // STDOUT
    2 => ['pipe', 'w'], // STDERR
];
// 启动进程，exec 已被移除，减少不必要的 shell 替换
$process = proc_open(
    [PHP_BINARY, "{$dir}/child.php", "--pid={$pid}"],
    $spec,
    $pipes,
);
if (!is_resource($process)) {
    throw new RuntimeException('无法启动子进程');
}
// 管道句柄
$stdout = $pipes[1];
$stderr = $pipes[2];
// 设置管道为非阻塞模式
stream_set_blocking($stdout, false);
stream_set_blocking($stderr, false);
// 设置超时时间
$deadline      = microtime(true) + 10;
$terminateSent = false;
// 循环直到子进程退出
while (true) {
    // 监听可读事件
    $read = array_values(
        array_filter(
            [$stdout, $stderr],
            static fn($stream) => !feof($stream),
        )
    );
    // 如果有可读事件
    if ($read !== []) {
        $write  = null; // 没有待写数据，不监听可写事件。
        $except = null;
        // 使用 stream_select 等待 I/O 事件，超时设为 1 秒 (更高效的等待)
        $ready = stream_select($read, $write, $except, 1);

        if ($ready === false) {
            throw new RuntimeException('stream_select failed');
        }

        foreach ($read as $stream) {
            $data = stream_get_contents($stream);
            if ($data === false || $data === '') {
                continue;
            }

            echo $stream === $stdout ? "子进程输出：{$data}" : "子进程错误：{$data}";
        }
    }
    // 检查超时时间
    if (!$terminateSent && microtime(true) >= $deadline) {
        proc_terminate($process);
        $terminateSent = true;
    }
    // 检查子进程状态并判断退出
    $status = proc_get_status($process);
    if (!$status['running'] && feof($stdout) && feof($stderr)) {
        break;
    }
}
// 关闭所有资源
fclose($stdout);
fclose($stderr);
$exitCode = proc_close($process);
echo "进程已回收，proc_close 返回：{$exitCode}", PHP_EOL;
```
### 子进程
```php
// 解析 --pid 参数
$pid   = getopt("", ["pid:"])['pid'] ?? 0;
$myPid = posix_getpid();
// 循环运行并输出
while (true) {
    sleep(1);
    // 简化输出内容
    echo "Run P:{$myPid} PP:{$pid}, Time:", date('Y-m-d H:i:s'), PHP_EOL;
}
```
## 生命周期与关闭顺序
正常退出应优先给子进程完成工作的机会，`proc_terminate()` 是超时后的降级手段，不是固定的第二步：
1. 完成写入后关闭 stdin，让等待输入的子进程收到 EOF。
2. 同时排空 stdout 和 stderr，避免管道缓冲区填满。
3. 使用 `proc_get_status()` 等待子进程自然退出。
4. 超过宽限期后再调用 `proc_terminate()`；必要时升级终止信号。
5. 继续读取最后输出，直到管道 EOF，然后关闭所有管道。
6. 最后调用 `proc_close()` 回收进程资源并取得退出状态。
不要先调用 `proc_close()` 再读取管道；它会等待子进程退出，而子进程可能正因输出管道已满而等待父进程读取，双方因此死锁。
## 阻塞、死锁与 SIGPIPE
### 管道缓冲区满
| 状态                | 条件                      | 结果                           |
| ----------------- | ----------------------- | ---------------------------- |
| 写入阻塞              | 父进程**保留读端但不读取，管道缓冲区已满** | 子进程的 `write()`/`echo` 等待可用空间 |
| EOF               | 所有写端都已关闭，读端消费完剩余数据      | 父进程读取返回 EOF                  |
| `SIGPIPE`/`EPIPE` | 所有读端都已关闭，子进程继续写入        | 子进程收到信号或写入错误                 |
- 管道容量由操作系统和运行环境决定，不应依赖固定的 `64 KB` 数值。
- stdout 和 stderr 是独立管道，只读取其中一条仍可能被另一条堵塞。
- **阻塞不等于 SIGPIPE：** 读端仍存在但不消费时通常是写入阻塞；读端全部关闭后继续写才涉及 `SIGPIPE`/`EPIPE`。
### `stream_get_contents()` 等待 EOF
- **实际阻塞点：** `stream_get_contents()`，不是 `stream_select()`。
- **就绪边界：** `stream_select()` 只保证读取当前已有数据的起点不会阻塞，不保证无界读取能够结束。
- **EOF 条件：** 长驻子进程不退出且**不关闭标准流，父进程无法读到 EOF**。
- **影响范围：** stdout 和 stderr 的阻塞式无界读取都存在相同风险。
- **读取 API 对比：** 行边界、长度上限与短读语义见 [[Work/Script/PHP/Function/Socket、Stream#fgets() vs stream_get_contents()\|fgets() vs stream_get_contents()]]。
- **阻塞过程：**
	1. 子进程写入一段数据，stdout 变为可读。
	2. `stream_select()` 返回 stdout 已就绪。
	3. `stream_get_contents()` 读完当前数据后继续等待更多数据或 EOF。
	4. 子进程不退出，EOF 不会到来，父进程永久阻塞。
**最小复现：** 子进程先输出一行，再等待两秒退出。父进程可直接比较两个调用的返回时间。
```php
$process = proc_open(
    [PHP_BINARY, '-r', 'echo "tick\n"; sleep(2);'],
    [1 => ['pipe', 'w']],
    $pipes,
);

if (!is_resource($process)) {
    throw new RuntimeException('无法启动子进程');
}

$stdout = $pipes[1];
$read = [$stdout];
$write = null;
$except = null;
$startedAt = microtime(true);

stream_select($read, $write, $except, 5);
printf("stream_select: %.2fs\n", microtime(true) - $startedAt);

// 已有一行可读，但阻塞式无界读取仍要等待子进程关闭 stdout。
$output = stream_get_contents($stdout);
printf("stream_get_contents: %.2fs\n", microtime(true) - $startedAt);

fclose($stdout);
proc_close($process);
```
输出（耗时因环境略有差异）：
```text
stream_select: 0.01s
stream_get_contents: 2.02s
```
- **实验结论：** `stream_select()` 很快返回，但无界读取仍等待 EOF；若把两秒等待换成长驻循环，`stream_get_contents()` 将一直不返回。
- **正确处理：** stdout/stderr 同时设为非阻塞，每次只读取当前已有数据。
# PCNTL 进程控制
PCNTL 面向类 Unix 系统，提供更接近操作系统的进程创建、程序替换、信号和回收能力。
## `pcntl_fork()`
- **行为：** 复制当前 PHP 进程，父子进程从同一调用点继续执行。
- **用途：** 守护进程、多任务处理和自定义 Worker 模型。
- **继续学习：** 返回值、描述符继承与 COW 见 [[Work/Script/PHP/Function/Process/PCNTL#pcntl_fork\|PCNTL]]；数据交换方案见 [[Work/Script/PHP/Function/Process/IPC#总结：IPC 选型流程\|IPC 选型流程]]。
## `pcntl_exec()`
- **行为：** 用新程序替换当前进程，不会创建新进程。
- **结果：** 执行成功后 PHP 代码不再继续；只有替换失败时才返回。
# 扩展提供的并发方案
## `Swoole\Process`
- **定位：** 面向对象的子进程生命周期、管道、消息队列和信号封装。
- **用途：** Swoole Worker 进程池、高性能服务和后台任务。
```php
$process = new Swoole\Process(function (Swoole\Process $worker) {
    $data = $worker->pop();
    $worker->write("Pong: {$data}");
    exit(0);
}, true);

$process->start();
$process->push('Ping');
echo $process->read(), PHP_EOL;
Swoole\Process::wait(true);
```
## `parallel\Runtime`
- **定位：** `parallel` 扩展在线程中创建独立 PHP Runtime，不是操作系统子进程管理 API。
- **用途：** CPU 密集型并行计算；`Future::value()` 会等待任务完成。
```php
use parallel\Runtime;

$runtime = new Runtime();
$future = $runtime->run(function () {
    sleep(1);
    return '异步任务完成';
});

echo "主线程继续执行", PHP_EOL;
echo $future->value(), PHP_EOL;
$runtime->close();
```
# 总结
- **简单命令：** 使用 `exec()`、`system()`、`passthru()` 或 `shell_exec()`，根据退出码、输出保存和实时转发需求选型。
- **外部进程精细控制：** 使用 `proc_open()`，明确三个标准流的方向、EOF 条件和关闭顺序。
- **持续输出：** 同时非阻塞读取 stdout/stderr，使用 `stream_select()` 等待就绪，不监听没有待写数据的 stdin。
- **Unix 进程模型：** 使用 PCNTL 创建或替换进程；使用 `Swoole\Process` 获取更高层封装。
- **线程并行：** 使用 `parallel\Runtime`，不要与子进程通信模型混淆。
