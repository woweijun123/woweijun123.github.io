---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Process/Program execution/","title":"Program execution","tags":["flashcards"],"noteIcon":"","created":"2025-11-03T09:48:07.000+08:00","updated":"2026-03-24T17:36:50.198+08:00"}
---

# 简单的命令执行函数
这些函数通常用于执行一个外部程序并获取其输出。
## exec
- **特点：** 返回**最后一行**输出，可以传入一个**数组参数来收集所有的**输出行。
- **注意：** 如果程序长时间运行或输出大量内容，PHP 进程可能会挂起直到外部程序结束。
- **用途：** 适用于你只需要确认命令是否成功执行，或只需要命令的**摘要结果**。
```php
$command   = 'ls -l'; // 在 Linux/macOS 上列出当前目录文件
$last_line = exec($command, $output, $return_var);
echo "最后一行输出: " . $last_line . PHP_EOL;
echo "--- 完整输出 (数组形式) ---" . PHP_EOL;
var_export($output);
echo "退出状态码: " . $return_var . PHP_EOL;
/*
最后一行输出: t3.php
--- 完整输出 (数组形式) ---
array (
  0 => 't1.php',
  1 => 't2.php',
  2 => 't3.php',
)
*/
```
## system
- **特点：** 返回**最后一行**输出，适合运行需要实时显示输出的简单程序。
- **用途：** 执行命令，并直接将输出**发送到标准输出**（例如浏览器或终端）能**实时看到**命令的输出。
- **行为：** 直接将输出**发送到 PHP 的标准输出**。
```php
// 运行一个简单的 ping 命令，并直接显示结果
echo "开始执行 ping..." . PHP_EOL;
$last_line = system('ping -c 3 127.0.0.1', $return_var);
echo PHP_EOL . "--- 命令结束 ---" . PHP_EOL;
echo "最后一行输出 (通常是 ping 的总结): " . $last_line . "" . PHP_EOL;
echo "退出状态码: " . $return_var . "" . PHP_EOL;
/*
开始执行 ping...
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.068 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.119 ms
*/
```
## passthru
快记忆(通过调和乳)
- **特点：** `void` (无返回值)，当需要执行一个**返回二进制数据**（如图片生成程序）或**处理大量数据**且**不希望 PHP 内部缓冲**时使用。
- **用途：** 执行外部程序，并将原始的二进制数据输出直接传递给调用者。
```php
header('Content-Type: image/jpeg');  
header('Content-Disposition: inline; filename="generated_image.jpg"');  
// 直接将脚本生成的原始 JPEG 数据流输出到浏览器  
passthru('/usr/bin/generate_image.sh --format jpeg');  
// PHP 不会尝试将图像数据存储到变量中
```
## shell_exec(或反引号 `` ` `` 操作符)
- **特点：** 返回**完整的输出**作为单个字符串，类似于在终端执行命令，并将所有输出作为结果返回。
- **用途：** 适用于将命令的**所有输出**作为一个整体的文本块进行处理和存储（例如日志记录、文件内容读取）。
- **行为：** 通过系统的 shell 执行命令。
```php
// 读取一个文件的内容，并将其存储在一个字符串变量中
// $file_content = shell_exec('cat /etc/hosts');
$file_content = `cat /etc/hosts`;
if ($file_content === null) {
    echo "命令执行失败或被禁用。" . PHP_EOL;
} else {
    echo "--- hosts 文件内容 ---" . PHP_EOL;
    echo $file_content;
}
/*
--- hosts 文件内容 ---
127.0.0.1 host.docker.internal
*/
```
# 高级/精细控制的进程管理
需要更复杂的控制，例如与外部程序进行**双向通信**（输入/输出）、**控制其生命周期**或获取更详细的进程信息，应该使用 `proc_open()`。
## proc_open
[PHP: proc\_open - Manual](https://www.php.net/manual/zh/function.proc-open.php)
- **特点：** 通过创建**两个独立**的**单向**管道**来实现父子进程间的**双向通信。
    - 向子进程的 `stdin` 写入数据。
    - 从子进程的 `stdout` 和 `stderr` 读取数据。
    - 控制子进程的运行环境。
    - 使用 `proc_close()` 关闭进程，获取退出码。
    - 使用 `proc_get_status()` 获取进程状态。
- **用途：** **最适合**需要与外部程序进行**交互**或运行**长时间任务**的场景。
- **状态**：可获取进程 ID (PID)、退出码、是否存活等详细信息。
### 参数
#### 1. `command` (命令行)
* **类型：** `string` 或 `array` (推荐)
* **用途：** 定义要执行的外部程序及其参数。
* **注意：**
    * 如果是 `string`，**必须**手动转义特殊字符。
    * 如果是 `array` (PHP 7.4+)，进程将**不通过 shell** 直接打开，PHP 会处理参数转义，更安全推荐。
#### 2. `descriptor_spec` (描述符规格)
* **类型：** 索引数组
* **用途：** 定义子进程的文件描述符如何设置（0=stdin, 1=stdout, 2=stderr 等）。
* **元素类型：**
    * 管道 (`['pipe', 'r'/'w']`): 创建父子进程通信管道。`r` 给子进程读取端 (stdin)，`w` 给子进程写入端 (stdout/stderr)。
    * 文件 (`['file', 'path']`): 重定向子进程的 I/O 到一个文件。
    * 流资源 (`STDOUT`): 继承父进程已打开的流资源（如终端输出）。
#### 3. `pipes` (管道句柄)
* **类型：** 索引数组 (通过引用返回)
* **用途：** 父进程用于**操作管道**的 PHP 文件指针句柄。
* **注意：** 只包含在 `descriptor_spec` 中定义为 `'pipe'` 的描述符句柄。
#### 4. `cwd` (工作目录)
* **类型：** `string` 或 `null`
* **用途：** 设置子进程启动时的初始工作目录。
* **注意：** 路径**必须是绝对路径**。`null` 表示使用当前 PHP 进程的工作目录。
#### 5. `env_vars` (环境变量)
* **类型：** `array` 或 `null`
* **用途：** 为子进程设置特定的环境变量。
* **注意：** `null` 表示继承当前 PHP 进程的环境变量。
```php
$descriptorSpec = [
    0 => ["pipe", "r"],  // 标准输入，子进程从此管道中读取数据
    1 => ["pipe", "w"],  // 标准输出，子进程向此管道中写入数据
    2 => ["file", "/tmp/error-output.txt", "a"], // 标准错误，写入到一个文件
];
// 若子进程需要拿到父进程的输入输出句柄，则通过以下设置
// $descriptor_spec = [
//     1 => STDOUT, // 继承父进程的 STDOUT 文件描述符
//     2 => STDERR  // 继承父进程的 STDERR 文件描述符
// ];
$cwd            = '/tmp';
$env            = ['some_option' => 'aeiou'];
$process        = proc_open('php', $descriptorSpec, $pipes, $cwd, $env);
if (is_resource($process)) {
    // $pipes 现在看起来是这样的：
    // 0 => 可以向子进程标准输入写入的句柄
    // 1 => 可以从子进程标准输出读取的句柄
    // 错误输出将被追加到文件 /tmp/error-output.txt
    fwrite($pipes[0], '<?php print_r($_ENV); ?>');
    echo stream_get_contents($pipes[1]);
    // 切记：在调用 proc_close 之前关闭所有的管道以避免死锁。
    foreach ($pipes as $pipe) {
	    fclose($pipe);
    }
    $return_value = proc_close($process);
    echo "command returned $return_value\n"; // command returned 127
}
```
#### 关闭顺序建议：
1. **stdin 管道** - 通知进程没有更多输入
2. **发送终止信号** - proc_terminate
3. **stdout/stderr 管道** - 读取可能的最后输出
4. **进程句柄** - proc_close
```php
$process = proc_open(...);
// ... 使用进程
proc_terminate($process); // 关闭子进程
foreach ($pipes as $pipe) fclose($pipe); // 管道
proc_close($process); // 关闭进程句柄
```
#### 注意事项
| 概念        | 解释                                                                                |
| :-------- | :-------------------------------------------------------------------------------- |
| **问题核心**  | **父进程退出导致子进程的管道被关闭，当子进程尝试向已关闭的管道写入时，会收到 SIGPIPE 信号而被终止**                          |
| **缓冲区大小** | 多数现代系统默认为 **64 KB**。                                                              |
| **后果**    | 缓冲区满时，子进程的 `echo` (写入操作) 被 **阻塞 (Block)**，导致子进程**挂起 (Hang)**，无法继续执行代码（即使代码是无限循环）。 |
| **触发条件**  | 使用 `proc_open` 并通过 `["pipe", "w"]` 捕获子进程 STDOUT/STDERR，而父进程**未读取**相应管道。           |
| **解决方案**  | **持续读取子进程输出：** 父进程必须定期且以非阻塞模式 (`stream_set_blocking`) 从管道中读取数据。                   |
#### 详细解析
1. 子进程向 stdout/stderr 写入数据
2. 数据进入管道缓冲区
3. 当缓冲区满时，子进程的 write() 系统调用会被阻塞
4. 如果此时**父进程关闭了管道读取端**，子进程才会收到 **SIGPIPE**
	- **阻塞**：父进程还在，只是不读取 → 子进程在缓冲区满时被挂起
	- **SIGPIPE**：父进程关闭了管道 → 子进程收到信号被终止
### 实例
#### 主进程
```php
$dir     = __DIR__;
$pid = posix_getpid();
$command = "php $dir/child.php --pid=$pid";
// 定义 I/O 规范，不需要 STDIN (0)
$spec = [
    0 => ['pipe', 'r'], // STDIN
    1 => ['pipe', 'w'], // STDOUT
    2 => ['pipe', 'w'], // STDERR
];
// 启动进程，exec 已被移除，减少不必要的 shell 替换
$resource = proc_open($command, $spec, $pipes);
if (!is_resource($resource)) {
    die("无法启动子进程\n");
}
// 管道句柄
$stdin = $pipes[0];
$stdout = $pipes[1];
$stderr = $pipes[2];
// 设置管道为非阻塞模式
stream_set_blocking($stdout, 0);
stream_set_blocking($stderr, 0);
// 循环直到子进程退出
while (true) {
    $read   = [$stdout, $stderr];
    $write  = [$stdin];
    $except = null;
    // 使用 stream_select 等待 I/O 事件，超时设为 1 秒 (更高效的等待)
    if (stream_select($read, $write, $except, 1) > 0) {
        // 读取 STDOUT
        if (in_array($stdout, $read)) {
            echo "子进程输出: " . stream_get_contents($stdout);
        }
        // 读取 STDERR
        if (in_array($stderr, $read)) {
            echo "子进程错误: " . stream_get_contents($stderr);
        }
    }
    // 检查子进程状态并判断退出
    $status = proc_get_status($resource);
    if (!$status['running']) {
        // 关闭所有资源
        fclose($stdout);
        fclose($stderr);
        proc_close($resource);
        echo "子进程退出，状态码: {$status['exitcode']}\n";
        break;
    }
}
```
#### 子进程
```php
// 解析 --pid 参数  
$pid = getopt("", ["pid:"])['pid'] ?? 0;  
$myPid = posix_getpid();  
// 循环运行并输出  
while (true) {  
    sleep(1);  
    // 简化输出内容  
    echo "Run P:{$myPid} PP:{$pid}\n";  
}
```
## pcntl_fork()
`PCNTL` 扩展主要用于类 Unix 环境（Linux, macOS），提供了对进程创建和信号处理的更底层控制。
- **特点：** 允许 PHP 脚本将自身分成两个独立的进程（父进程和子进程），这是实现守护进程（Daemon）或多任务处理的基础。
- **用途：** **创建子进程**（Process Forking）。
## pcntl_exec()
- **特点：** 它并**不创建新的进程**，而是用新的程序来**替换**当前的 PHP 脚本进程。在执行 `pcntl_exec()` 后，当前 PHP 脚本的内存、代码和资源都会被新程序取代，PHP 脚本到此结束（除非新程序失败）。
- **用途：** 在**当前进程空间**执行另一个程序。
## Swoole\Process
- **特点：** 提供了**面向对象的 API**，可以方便地管理子进程的生命周期、**通信（管道、消息队列）** 和信号处理。替代 `pcntl` 的更现代且功能更强大的方式
- **用途：** 用于在 PHP 中创建多进程并发程序，如高性能服务器或后台任务处理。
此示例展示了父进程如何启动一个子进程，并通过管道发送数据，最后等待子进程退出。
```php
// 必须启用 Swoole 扩展
$process = new Swoole\Process(function (Swoole\Process $worker) {
    // 【子进程】: 读取父进程发送的数据，并响应
    $data = $worker->pop();
    $worker->write("Pong: $data");
    exit(0);
}, true); // true 启用管道
$process->start();
// 【父进程】: 发送数据给子进程
$process->push("Ping");
// 【父进程】: 读取子进程的响应
$response = $process->read();
echo "父进程收到响应: " . $response . "\n";
// 回收子进程资源
Swoole\Process::wait(true);
```
## parallel\Runtime
- **特点：** 通过面向对象的方式抽象了并发执行的细节，使得实现并行计算更简单，返回一个 `parallel\Future` 对象来获取未来的结果。
- **用途：** 运行一个独立的 PHP 工作者（线程或进程）来执行一个闭包，适用于并行计算。
- **环境：** 需要安装并启用 **parallel 扩展**。
此示例展示了如何在主线程之外启动一个异步任务，并在主线程不被阻塞的情况下等待结果。
```php
// 必须启用 parallel 扩展
use parallel\Runtime;
// 1. 启动运行时环境 (Worker)
$runtime = new Runtime();
// 2. 提交一个异步任务 (闭包)
$future = $runtime->run(function () {
    // 异步工作者执行的代码
    sleep(1); 
    return "异步任务完成！";
});
echo "主线程未被阻塞，继续执行...\n";
// 3. 阻塞并获取异步任务的结果
$result = $future->value();
echo "异步任务结果: " . $result . "\n";
$runtime->close();
```
# 总结
| **方法**                 | **主要用途**      | **返回值/输出方式**          | **适用场景**                               |
| ---------------------- | ------------- | --------------------- | -------------------------------------- |
| `exec()`               | 执行命令          | 返回最后一行输出，所有输出可选传入数组   | **简单**的单行命令，需要获取特定输出                   |
| `system()`             | 执行命令并显示输出     | 直接输出到标准输出 (stdout)    | 需要**实时显示**简单命令的输出                      |
| `passthru()`           | 执行命令并输出原始数据   | 直接输出原始数据              | 处理二进制数据，**避免 PHP 缓冲大数据**，节省内存和时间。      |
| `shell_exec()`、\`(反引号) | 通过 shell 执行命令 | 返回**完整的输出**字符串        | 将完整的命令执行结果**集中在一个变量**中，方便后续处理。         |
| `proc_open()`          | 高级进程管理        | 返回文件指针 (pipes)        | 需要**双向通信**、**精细控制**进程生命周期              |
| `pcntl_fork()`         | 创建子进程         | 返回子进程 ID              | **专门用于类 Unix** 系统，创建**守护进程**、**多任务处理** |
| `Swoole\Process`       | 高性能多进程管理      | 返回 Swoole\Process 实例  | Worker进程池、实现**高效的进程间通信** (IPC)、后台任务处理  |
| `parallel\Runtime`     | 异步并行计算        | 返回 parallel\Future 对象 | 执行CPU密集型任务、阻塞 I/O 任务并行、简单安全的并行计算       |