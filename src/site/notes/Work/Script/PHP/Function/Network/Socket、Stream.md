---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Network/Socket、Stream/","title":"Socket、Stream","tags":["flashcards","#select","#epoll","#kqueue"],"noteIcon":"","created":"2026-03-10T22:33:54.000+08:00","updated":"2026-03-24T17:35:21.043+08:00","dg-note-properties":{"title":"Socket、Stream","tags":["flashcards","#select","#epoll","#kqueue"],"reference linking":null}}
---

# Stream vs Socket
## Socket（套接字）
Socket 是操作系统提供的一种**通信机制**。它位于传输层和应用层之间，是网络通信的**底层接口**。
- **定义：** 一种抽象的通信终端，允许应用程序通过网络发送和接收数据。
- **层次：** 网络层/传输层（底层）。
- **通信类型：** 明确区分协议和类型，例如：
    - `AF_INET` (IPv4) + `SOCK_STREAM` (TCP)
    - `AF_INET` (IPv4) + `SOCK_DGRAM` (UDP)
- **PHP 实现：** 通过 `socket_*` 系列函数（例如 `socket_create`, `socket_bind`, `socket_connect`）直接操作底层套接字。
- **用途：** 需要对网络通信的底层细节（如绑定、监听、连接建立、错误处理）进行**细粒度控制**时使用。
## Stream（流）
Stream 是 PHP 引入的一种**统一的 I/O 抽象接口**。它将文件、网络连接、压缩文件、内存等各种不同的 I/O 源都统一为可以通过一组相同函数（如 `fread`, `fwrite`）来操作的接口。
- **定义：** 一种资源，代表了一个可读、可写或双向的 I/O 通道。
- **层次：** 应用层（高层抽象）。
- **通信类型：** 抽象了底层细节，通过 **包装器（Wrapper）** 来处理不同的 I/O 源。例如：
    - `file://` (本地文件)
    - `http://` (HTTP 请求)
    - `ftp://` (FTP 连接)
    - `tcp://` 或 `udp://` (网络 Socket)
- **PHP 实现：** 通过 `stream_socket_*` 系列函数或高层文件 I/O 函数（例如 `fopen`, `fsockopen`）创建和操作。
- **用途：** 进行**简单、便捷**的 I/O 操作，无需关心底层是文件、HTTP 请求还是网络连接。
## 核心对比
| **特性**   | **Socket (套接字)**                                        | **Stream (流)**                                               |
| -------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| **抽象级别** | **底层** (OS 通信机制)                                        | **高层** (PHP I/O 抽象)                                          |
| **主要用途** | **网络通信**：精确控制协议、生命周期和底层细节。                              | **统一 I/O**：提供对**所有 I/O** (文件、网络等) 的通用读写接口。                   |
| **实现方式** | `socket_*` 系列函数。                                        | `file_get_contents`、`fopen`、`fsockopen`、`stream_socket_*` 等。 |
| **协议处理** | 需要显式设置协议类型 (如 `SOCK_STREAM` / `SOCK_DGRAM`)。            | 通过**包装器** (Wrapper) 抽象 (如 `tcp://`, `http://`, `file://`)。   |
| **易用性**  | **复杂**，多步创建，适用于底层控制。                                    | **简单快捷**，通常一步创建，更符合 PHP 的开发习惯。                               |
| **典型应用** | **高性能/异步网络服务** (如 Swoole/ReactPHP 的核心)。                 | **日常客户端连接、文件读写、简单的服务端**。                                     |
| **创建示例** | `socket_create()` + `socket_bind()` + `socket_listen()` | `stream_socket_client('tcp://...')`                          |
## 核心关联
**Socket** 是操作系统提供的**底层网络通信机制**。
**Stream** 是 PHP 对 I/O 操作提供的**统一抽象接口**。
**结论：** Stream 是一种更高级的接口，它利用包装器（如 `tcp://`）在底层封装和操作实际的 Socket 资源。
# Socket
**PHP操作MySQL数据库**也是通过**Socket**进行的，这正是由于Socket屏蔽了底层的协议，使得网络服务之间的互联互通变得简单。
## 常用函数
- [socket_accept](https://www.php.net/manual/zh/function.socket-accept.php) — **接受套接字上的连接**
- [socket_addrinfo_bind](https://www.php.net/manual/zh/function.socket-addrinfo-bind.php) — 从给定的 addrinfo 创建并绑定一个套接字
- [socket_addrinfo_connect](https://www.php.net/manual/zh/function.socket-addrinfo-connect.php) — 指定 addrinfo 创建并连接套接字
- [socket_addrinfo_explain](https://www.php.net/manual/zh/function.socket-addrinfo-explain.php) — 获取有关 addrinfo 的信息
- [socket_addrinfo_lookup](https://www.php.net/manual/zh/function.socket-addrinfo-lookup.php) — 获取数组，包含有关给定主机名的 getaddrinfo 内容
- [socket_atmark](https://www.php.net/manual/zh/function.socket-atmark.php) — 确认 socket 是否处于带外数据标记
- [socket_bind](https://www.php.net/manual/zh/function.socket-bind.php) — **给套接字绑定名字**
- [socket_clear_error](https://www.php.net/manual/zh/function.socket-clear-error.php) — 清除套接字或者最后的错误代码上的错误
- [socket_close](https://www.php.net/manual/zh/function.socket-close.php) — **关闭 Socket 实例**
- [socket_cmsg_space](https://www.php.net/manual/zh/function.socket-cmsg-space.php) — Calculate message buffer size
- [socket_connect](https://www.php.net/manual/zh/function.socket-connect.php) — 开启一个套接字连接
- [socket_create](https://www.php.net/manual/zh/function.socket-create.php) — **创建一个套接字（通讯节点）**
- [socket_create_listen](https://www.php.net/manual/zh/function.socket-create-listen.php) — 在端口上打开一个套接字以接受连接
- [socket_create_pair](https://www.php.net/manual/zh/function.socket-create-pair.php) — **创建一对彼此连接的套接字，并用数组存储**
- [socket_export_stream](https://www.php.net/manual/zh/function.socket-export-stream.php) — Export a socket into a stream that encapsulates a socket
- [socket_get_option](https://www.php.net/manual/zh/function.socket-get-option.php) — 获取套接字的套接字选项
- [socket_getopt](https://www.php.net/manual/zh/function.socket-getopt.php) — 别名 socket_get_option
- [socket_getpeername](https://www.php.net/manual/zh/function.socket-getpeername.php) — 获取套接字远端名字
- [socket_getsockname](https://www.php.net/manual/zh/function.socket-getsockname.php) — 获取套接字本地端的名字，返回主机名和端口号或是 Unix 文件系统路径，具体取决于套接字类型
- [socket_import_stream](https://www.php.net/manual/zh/function.socket-import-stream.php) — 导入 stream
- [socket_last_error](https://www.php.net/manual/zh/function.socket-last-error.php) — 返回套接字上的最后一个错误
- [socket_listen](https://www.php.net/manual/zh/function.socket-listen.php) — **监听套接字的连接**
- [socket_read](https://www.php.net/manual/zh/function.socket-read.php) — **从套接字中读取最大长度的数据**
- [socket_recv](https://www.php.net/manual/zh/function.socket-recv.php) — 从已连接的 socket 接收数据
- [socket_recvfrom](https://www.php.net/manual/zh/function.socket-recvfrom.php) — 从套接字接收数据，无论它是否是面向连接的
- [socket_recvmsg](https://www.php.net/manual/zh/function.socket-recvmsg.php) — Read a message
- [socket_select](https://www.php.net/manual/zh/function.socket-select.php) — 从给定套接字数组运行带指定超时时间的 select() 系统调用
- [socket_send](https://www.php.net/manual/zh/function.socket-send.php) — 向已连接的套接字发送数据
- [socket_sendmsg](https://www.php.net/manual/zh/function.socket-sendmsg.php) — Send a message
- [socket_sendto](https://www.php.net/manual/zh/function.socket-sendto.php) — 向套接字发送消息，无论它是否已建立连接
- [socket_set_block](https://www.php.net/manual/zh/function.socket-set-block.php) — **设置套接字为阻塞模式**
- [socket_set_nonblock](https://www.php.net/manual/zh/function.socket-set-nonblock.php) — 设置套接字为非阻塞模式
- [socket_set_option](https://www.php.net/manual/zh/function.socket-set-option.php) — 为套接字设置套接字选项
- [socket_setopt](https://www.php.net/manual/zh/function.socket-setopt.php) — 别名 socket_set_option
- [socket_shutdown](https://www.php.net/manual/zh/function.socket-shutdown.php) — 关闭套接字接收或发送，或两者都关闭
- [socket_strerror](https://www.php.net/manual/zh/function.socket-strerror.php) — 返回描述套接字错误的字符串
- [socket_write](https://www.php.net/manual/zh/function.socket-write.php) — **向套接字写数据**
- [socket_wsaprotocol_info_export](https://www.php.net/manual/zh/function.socket-wsaprotocol-info-export.php) — 导出 WSAPROTOCOL_INFO 结构体
- [socket_wsaprotocol_info_import](https://www.php.net/manual/zh/function.socket-wsaprotocol-info-import.php) — 从另一个进程导入套接字
- [socket_wsaprotocol_info_release](https://www.php.net/manual/zh/function.socket-wsaprotocol-info-release.php) — 释放已导出的 WSAPROTOCOL_INFO 结构体
## 实例
```php
// 配置服务器地址和端口
$host = "127.0.0.1";
$port = 8080;
// 设置脚本执行时间无限制（推荐在 CLI 模式下运行）
set_time_limit(0);
// 1. 创建 Socket
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP) or die("Could not create socket\n");
// 2. 绑定 Socket 到指定地址和端口
$result = socket_bind($socket, $host, $port) or die("Could not bind to socket\n");
// 3. 开始监听连接 (设置等待队列长度为 3)
$result = socket_listen($socket, 3) or die("Could not set up socket listener\n");
// 4. 接受连接请求
// 接收连接，并用一个新的子 Socket ($spawn) 来处理客户端通信
$spawn = socket_accept($socket) or die("Could not accept incoming connection\n");
// 5. 读取客户端输入
// 从客户端 Socket 读取最多 1024 字节的数据
$input = socket_read($spawn, 1024) or die("Could not read input\n");
// 6. 清理和处理数据
$input = trim($input); // 清理输入字符串两端的空白
// 将客户端输入数据反转，准备作为输出
$output = strrev($input) . "\n";
// 7. 返回数据给客户端
// 将反转后的字符串写入客户端 Socket
socket_write($spawn, $output, strlen($output)) or die("Could not write output\n");
// 8. 关闭 sockets
// 先关闭用于客户端通信的子 Socket
socket_close($spawn);
// 最后关闭用于监听的主 Socket
socket_close($socket);
```
## fsockopen
打开 Internet 或者 Unix 套接字连接
[PHP: fsockopen - Manual](https://www.php.net/manual/zh/function.fsockopen.php)
fsockopen()函数的好处是**把Socket连接绑定到一个流上**，然后使用**各种操作流的函数操作这个Socket连接**。
```php
function request($url, $param = [])
{
    $urlInfo = parse_url($url);
    $host  = $urlInfo['host'];
    $path  = $urlInfo['path'];
    // 将关联数组转换为 URL 编码的查询字符串
    $data = isset($param) ? http_build_query($param) : '';
    try {
        // 1.建立 Socket 连接「连接目标服务器的 88 端口，超时时间为 5 秒」
        $fp = @fsockopen($host, 88, $errno, $errstr, 5);
        // 检查连接是否成功
        if (!$fp) {
            // 连接失败，输出错误信息
            die("Connection Error: [$errno] $errstr" . PHP_EOL);
        }
        stream_set_blocking($fp, 0); // 开启非阻塞模式
        stream_set_timeout($fp, 1);  // 设置超时时间, 单位秒
        // 2.构造完整的 HTTP GET 请求报文
        $out = "GET {$path} HTTP/1.1\r\n";
        $out .= "Host: {$host}\r\n";
        // 模拟 User-Agent
        $out .= "User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; zh-CN; rv:1.9.2.13) Gecko/20101203 Firefox/3.6.13\r\n";
        // 声明请求体的数据类型
        $out .= "Content-Type: application/x-www-form-urlencoded\r\n";
        // 模拟 Referer
        $out .= "Referer: http://{$host}/archives/54/\r\n";
        // 携带 PHPSESSID，如果需要模拟登录或特定会话
        $out .= "PHPSESSID=082b0cc33cc7e6df1f87502c456c3eb0\r\n";
        // 关键字段：声明请求体的数据长度
        $out .= "Content-Length: " . strlen($data) . "\r\n";
        // 通知服务器处理完请求后关闭连接
        $out .= "Connection: Close\r\n";
        // 必须有两个 \r\n 才能结束头部，开始请求体
        $out .= "\r\n";
        // 附加请求体数据
        $out .= $data . "\r\n";
        // 3.发送请求
        fwrite($fp, $out);
        // 4.读取服务器响应
        $response = '';
        while (!feof($fp)) {
            // 每次读取 1280 字节数据
            $response .= fgets($fp, 1280);
        }
        // 5.关闭连接
        fclose($fp);
        return $response;
    } catch (Exception $e) {
        echo $e->getMessage(), PHP_EOL;
    }
}
$response = request('https://weichengjun2.dpdns.org/i/2025/11/19/691d5aaa9c83e.png');
// 输出完整的 HTTP 响应
echo "--- Server Response ---" . PHP_EOL;
echo $response . PHP_EOL;
echo "-----------------------" . PHP_EOL;
```
## pfsockopen
[PHP: pfsockopen - Manual](https://www.php.net/manual/zh/function.pfsockopen.php)
打开**持久的** Internet 或 Unix 套接字连接
这个函数的作用与 [fsockopen()](https://www.php.net/manual/zh/function.fsockopen.php) 完全一样，不同的地方在于当在脚本执行完后，连接一直不会关闭。可以说它是 [fsockopen()](https://www.php.net/manual/zh/function.fsockopen.php) 的**长连接**版本。
# Stream
[PHP: Stream 函数 - Manual](https://www.php.net/manual/zh/ref.stream.php)
## stream_socket_pair
创建**一对完全一样**的**网络套接字连接**，这个函数通常会被用在进程间通信（Inter-Process Communication）。
```php
// stream_socket_pair 创建了两个连接的套接字，可以双向通信
// $sockets[0] 和 $sockets[1]
$sockets = stream_socket_pair(STREAM_PF_UNIX, STREAM_SOCK_STREAM, STREAM_IPPROTO_IP);
$pid     = pcntl_fork();

if ($pid) {
    /* 父进程 */
    // 因为父进程只使用 $sockets[1] 来通信，所以关闭 $sockets[0]
    fclose($sockets[0]);

    // 父进程向子进程发送消息（通过 $sockets[1]）
    fwrite($sockets[1], "child PID: $pid\n");
    // 父进程从子进程读取消息（通过 $sockets[1]）
    echo 'parent: ' . fgets($sockets[1]);

    fclose($sockets[1]);
} elseif ($pid == 0) {
    /* 子进程 */
    // 因为子进程只使用 $sockets[0] 来通信，所以关闭 $sockets[1]
    fclose($sockets[1]);

    // 子进程向父进程发送消息（通过 $sockets[0]）
    fwrite($sockets[0], "message from child\n");
    // 子进程从父进程读取消息（通过 $sockets[0]）
    echo 'child: ' . fgets($sockets[0]);

    fclose($sockets[0]);
} else {
    die('could not fork');
}
```
### 父子进程通信解析
#### 1. 通道创建
* `stream_socket_pair()`：在 `pcntl_fork()` **前**创建。
* **结果**：一个**双向**通信管道，有 `$sockets[0]` 和 `$sockets[1]` 两个端点。
#### 2. 进程分叉
* `pcntl_fork()`：创建子进程。
* **结果**：父进程和子进程都**各自继承**了这两个端点 (`$sockets[0]` 和 `$sockets[1]`) 的文件描述符。
#### 3. 通信策略
* 为了避免混乱，父子进程各自只使用**一个**端点。
* **父进程**：使用 `$sockets[1]`，**关闭** `$sockets[0]`。
* **子进程**：使用 `$sockets[0]`，**关闭** `$sockets[1]`。
#### 4. 通信流程
* **发送**：`fwrite()` 写入各自持有的套接字。
* **接收**：`fgets()` 读取各自持有的套接字。
#### 5. 核心原理
* **`父进程的$sockets[1]`** 始终与 **`子进程的$sockets[0]`** 连接。
* 这是**全双工**通信，父子双方都可以同时发送和接收消息。
### 总结
`stream_socket_pair` 创建了一个共享的管道，父子进程通过各自关闭一端、使用另一端的方式，实现高效、清晰的双向通信。
## stream_set_blocking
为资源流设置阻塞或者阻塞模式
该参数的设置将会影响到像 `fgets()` 和 `fread()` 这样的函数从资源流里读取数据。
- 阻塞模式：将会一直等到从资源流里面获取到数据才能返回。
- 非阻塞模式：调用 fgets() 总是会立即返回；
```php
/*
1.$mode=0, 资源流将会被转换为非阻塞模式；
2.$mode=1, 资源流将会被转换为阻塞模式。
*/ 
stream_set_blocking($stream, $mode);
stream_set_blocking($fp, 0); // 开启非阻塞模式
```
## stream_socket_server
创建一个Internet或Unix域服务器socket
## stream_socket_client
开放的互联网或Unix域socket连接
```php
$fp = stream_socket_client("tcp://www.example.com:80", $errno, $errMsg, 30);
if (!$fp) {
    echo "$errMsg ($errno)", PHP_EOL;
} else {
    fwrite($fp, "GET / HTTP/1.0\r\nHost: www.example.com\r\nAccept: */*\r\n\r\n");
    while (!feof($fp)) {
        echo fgets($fp, 1024);
    }
    fclose($fp);
}
```
>[GitHub - php-amqplib/php-amqplib: The most widely used PHP client for RabbitMQ](https://github.com/php-amqplib/php-amqplib) 
>用到 stream_socket_client 实现 AMQP 协议连接
```php
...省略其他代码
try {
	$this->sock = stream_socket_client(
		$remote,
		$errno,
		$errstr,
		$this->connection_timeout,
		STREAM_CLIENT_CONNECT,
		$context
	);
	$this->throwOnError();
} catch (\ErrorException $e) {
	throw new AMQPIOException($e->getMessage(), $e->getCode(), $e->getPrevious());
} finally {
	$this->restoreErrorHandler();
}
...
```
## stream_socket_accept
接受由 [stream_socket_server()](https://www.php.net/manual/zh/function.stream-socket-server.php) 创建的套接字连接
[¶参数](https://www.php.net/manual/zh/function.stream-socket-accept.php#refsect1-function.stream-socket-accept-parameters)
`socket`
需要接受的服务器创建的套接字连接。
`timeout`
覆盖默认的套接字接受的**超时时限**。输入的时间需以秒为单位。默认情况下，使用 [default_socket_timeout](https://www.php.net/manual/zh/filesystem.configuration.php#ini.default-socket-timeout) 作为超时时限。
- 0 表示**非阻塞模式**，即使socket本身是阻塞的
- 如果**使用 -1 或省略**，则会**阻塞**等待
`peer_name`
如果已选的传输器存在且有效的已连接客户端，则将该值设置为已连接客户端名称（地址）。
> **注意**:
> 也可以之后通过 [stream_socket_get_name()](https://www.php.net/manual/zh/function.stream-socket-get-name.php) 来确定。

[¶返回值](https://www.php.net/manual/zh/function.stream-socket-accept.php#refsect1-function.stream-socket-accept-returnvalues)
返回接受套接之后的**资源流** 或者在失败时返回 **false**。
## stream_select「多路复用 I/O」
### 1. 原理
`stream_select` 是 PHP 中实现**多路复用 I/O**（I/O Multiplexing）的核心函数，其**底层依赖**于操作**系统的 `select`** 系统调用。
它允许程序**同时监听多个文件描述符**（如 socket、管道等）的可读、可写或异常事件，并在任意一个事件就绪时触发回调。
#### (1) 位图（Bitmask）与文件描述符集合
`select` 的核心是通过 **位图**（bitmap）管理文件描述符（fd）。每个位图对应一个文件描述符的状态：
- **读集合（readfds）**：监听文件描述符是否有数据可读。
- **写集合（writefds）**：监听文件描述符是否可写。
- **异常集合（exceptfds）**：监听文件描述符是否有异常（如 TCP 错误）。
位图的本质是一个整数数组，每个位（bit）代表一个文件描述符的状态。
例如，`fd_set` 在 Linux 中通常是一个 `1024` 位的数组（最大支持 1024 个 fd，默认可通过 `FD_SETSIZE` 扩展）。
#### (2) 用户态与内核态的交互
`select` 的执行分为 **用户态** 和 **内核态** 两阶段：
1. **用户态准备**：
   - 将需要监听的文件描述符集合（`readfds`、`writefds`、`exceptfds`）从用户空间复制到内核空间。
   - 设置超时时间（`timeout`）。
2. **内核态处理**：
   - 内核轮询所有文件描述符，检查是否有事件就绪。
   - 如果事件未就绪，进程进入休眠状态（阻塞），等待事件触发。
   - 事件触发后，内核修改位图并返回就绪的文件描述符数量。
#### (3) 轮询机制
`select` 的核心逻辑是 **轮询**（Polling）：
- 内核需要**遍历**所有传入的文件**描述符**，逐个**检查其状态**。
- 如果没有就绪事件，进程会阻塞直到超时或事件发生。
- 返回时，内核将修改后的位图复制回用户空间，用户程序通过遍历位图确定哪些文件描述符就绪。
### 2. 实现
[PHP: stream\_select - Manual](https://www.php.net/manual/zh/function.stream-select.php)
PHP 的 `stream_select` 函数是对 `select` 系统调用的封装，其关键特性如下：
#### (1) 参数解析
```php
int stream_select(array &$read, array &$write, array &$except, int $tv_sec, int $tv_usec)
```
- **`$read`、`$write`、`$except`**：需监听的流数组（如 socket 资源）。
- **`$tv_sec`、`$tv_usec`**：超时时间（秒和微秒）。
##### 关键点
- **`stream_select()` 会在流准备就绪时立即返回**，而不会等到超时时间耗尽。
- **超时时间**是一个**最大等待时长**。
- 将超时值设为 `0` 会立即返回，这在循环中使用会**大量消耗 CPU**，不推荐。通常建议设置一个几秒或至少 `200000` 微秒的超时值来降低 CPU 占用。
##### 返回值
Stream Select 函数返回修改后的流资源数量。
- 有I/O活动时返回 > 0
- 超时后返回 0
- 出错时返回 false
#### (2) 非阻塞模式
PHP 流默认是阻塞的，需通过 `stream_set_blocking($stream, 0)` 将流设置为非阻塞模式，以避免 `fread()`/`fwrite()` 阻塞整个程序。
#### (3) 事件驱动模型
PHP 通过 `stream_select` 实现事件驱动模型：
1. **注册监听**：将流加入 `$read`/`$write`/`$except` 数组。
2. **等待事件**：调用 `stream_select` 阻塞等待事件。
3. **处理事件**：遍历返回的就绪流，调用回调函数处理（如读取数据、发送响应）。
### 3. 核心流程
#### 关键点解析
- **流复制**：每次调用 `stream_select` 前需复制数组（`$readCopy`），因为 `stream_select` 会修改原数组。
- **事件处理**：通过遍历 `$readCopy` 确定哪些流有数据可读。
- **非阻塞操作**：`stream_set_blocking($stream, 0)` 避免 `fread()`/`fwrite()` 阻塞。
### 4. 优缺点分析
#### (1) 优点
- **并发处理**：单线程监听多个流，避免线程切换开销。
- **简单易用**：PHP 提供的封装函数（`stream_select`）易于实现基本的异步 I/O。
- **跨平台**：兼容 Windows 和 Unix 系统。
#### (2) 缺点
- **性能瓶颈**：
	- **位图复制开销**：每次调用 `select` 需将文件描述符集合从**用户态复制到内核态**，且**内核需轮询所有 fd**。
	- **文件描述符限制**：默认最大支持 1024 个 fd（可通过 `FD_SETSIZE` 扩展，但受内存限制）。
- **事件通知延迟**：`select` 是**轮询机制**，无法实时通知事件（如 `epoll` 的边缘触发模式）。
- **不适用于高并发场景**：在连接数超过数千时，性能显著下降。
### 5. `select`/`epoll`/`kqueue` 的对比
| **特性**        | **select / poll (通用)**                          | **epoll (Linux 专有)**               | **kqueue (BSD/macOS 专有)**     |
| ------------- | ----------------------------------------------- | ---------------------------------- | ----------------------------- |
| **底层原理**      | **位图 (select)** / **链表 (poll)** 轮询              | **红黑树 + 双向链表**                     | **内核事件过滤器** (基于事件链表)          |
| **最大 FD 数量**  | **`select`:** 默认 1024 (受限)<br>**`poll`:** 无硬性限制 | **无限制** (仅受系统资源限制)                 | **无限制** (仅受系统资源限制)            |
| **事件注册**      | **临时注册**：每次调用都需要传递整个 FD 集合                      | **持久注册**：FD 集合一次性注册到内核             | **持久注册**：事件一次性注册到内核           |
| **性能复杂度**     | **$O(N)$** (每次调用都需要遍历所有监听的 FD 集合)               | **$O(1)$** (直接从就绪链表返回事件)           | **$O(1)$** (直接从事件队列返回事件)      |
| **用户态/内核态交互** | **高**：每次调用复制**整个** FD 集合                        | **低**：一次性注册 FD，后续仅传递**就绪事件**       | **低**：一次性注册事件，后续仅传递**就绪事件**   |
| **通知机制**      | **水平触发 (LT)**：会反复通知 FD 上的数据**存在**。              | 默认 **LT**，可选 **边缘触发 (ET)**。        | 默认 **ET** (某些事件可配置为 LT)。      |
| **数据回调模型**    | **轮询 (Polling)**：用户需要遍历 FD 集合查找就绪者。             | **通知/回调**：内核直接返回就绪事件列表。            | **通知/事件驱动**：内核直接返回就绪事件列表。     |
| **适用场景**      | **通用性强**、**小规模并发** (<1000 连接)。                  | **高性能**、**大规模并发** (高并发 Web/游戏服务器)。 | **高性能**、**大规模并发** (跨平台兼容性更好)。 |
#### 示例 1: 性能瓶颈（$O(N)$ vs $O(1)$）
假设一个 Web 服务器有 **10,000 个** 活跃连接 (FD)。
##### 使用 `stream_select` (或 `select`/`poll`)
1.  **用户空间到内核空间：** 每次调用 `select()`，服务器需要将包含 10,000 个 FD 的整个集合拷贝给内核。
2.  **内核遍历：** 内核需要 **遍历这 10,000 个 FD** 来检查哪些是就绪的。
3.  **结果拷贝：** 内核将就绪的 FD 集合（假设只有 5 个）拷贝回用户空间。
4.  **用户空间遍历：** 服务器程序再次 **遍历这 10,000 个 FD** 集合，找出那 5 个就绪的 FD，并处理它们。
**结论：** 即使只有极少数连接就绪，程序和内核都要付出检查 **10,000 个 FD** 的代价（$O(N)$）。
##### 使用 `epoll`
1.  **注册 FD (一次性)：** 服务器程序启动时，通过 `epoll_ctl` 将这 10,000 个 FD **一次性** 注册到内核的 epoll 实例中。
2.  **等待就绪：** 调用 `epoll_wait`。
3.  **内核就绪列表：** 当有数据到达时，内核会将就绪的 FD 挂载到 **就绪列表** 中。
4.  **返回结果：** `epoll_wait` **直接返回** 包含 5 个就绪 FD 的列表。
**结论：** 程序和内核只需要关注那 5 个就绪的 FD。FD 数量对性能的影响极小（接近 $O(1)$）。在高并发场景下，`epoll` 具有压倒性的性能优势。
#### 示例 2: 通知模式（LT vs ET）
**情景：** 客户端发送了 500 字节数据，但服务器程序只从 socket 中读取了 100 字节。
##### 水平触发 (Level Triggered, LT) - `select` 默认及 `epoll` 默认
1.  **第一次通知：** `select/epoll_wait` 返回，通知该 FD 就绪。
2.  **处理 100 字节：** 服务器读取了 100 字节，还有 400 字节留在内核缓冲区。
3.  **循环：** 服务器再次进入 `select/epoll_wait` 等待。
4.  **第二次通知：** 因为内核缓冲区中 **仍有 400 字节数据存在**，`select/epoll` **会再次立即返回**，通知该 FD 仍然就绪。
5.  **反复通知：** 只要缓冲区中还有数据，它就会**反复被通知**。
**优点：** 简单安全，不会丢失事件，但可能导致服务器重复唤醒，降低效率。
##### 边缘触发 (Edge Triggered, ET) - `epoll` 特有
1.  **第一次通知：** `epoll_wait` 返回，通知该 FD **状态发生了变化**（从不就绪到就绪）。
2.  **处理 100 字节：** 服务器读取了 100 字节，还有 400 字节留在内核缓冲区。
3.  **循环：** 服务器再次进入 `epoll_wait` 等待。
4.  **第二次通知：** **不会立即返回**。因为自从第一次通知后，该 FD 的就绪状态**没有再次发生改变**（没有新的数据包到达）。
5.  **处理：** 如果服务器不读完剩下的 400 字节，它就永远不会再收到通知，可能导致数据滞留。
**优点：** 效率极高，避免了重复唤醒，但要求程序必须**一次性读/写完所有数据**（或读到 EAGAIN/EWOULDBLOCK 为止），编程复杂度更高。高性能框架如 Swoole 通常在协程模型下利用 `epoll` 的 ET 模式来达到极致性能。
### 6. PHP 中的替代方案
####  Swoole 协程
- 提供协程支持，通过 `go()` 函数实现非阻塞 I/O。
- 底层使用 `epoll`/`kqueue` 实现高性能并发。
- 示例：
```php
go(function () {
	$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM);
	$socket->bind('127.0.0.1', 8000);
	$socket->listen();
	
	while (true) {
		$client = $socket->accept();
			go(function () use ($client) {
				$data = $client->recv();
				$client->send("Echo: $data");
				$client->close();
			});
	}
});
```
### 7. 实际应用场景
#### (1) 异步消息队列（AMQP）
- PHP 使用 `php-amqplib` 时，通过 `stream_select` 实现异步发布确认：
```php
// PhpAmqpLib/Wire/IO/StreamIO.php:364
protected function do_select(?int $sec, int $usec)
{
	if ($this->sock === null || !is_resource($this->sock)) {
		$this->sock = null;
		throw new AMQPConnectionClosedException('Broken pipe or closed connection', 0);
	}

	$read = array($this->sock);
	$write = null;
	$except = null;

	if ($sec === null && PHP_VERSION_ID >= 80100) {
		$usec = 0;
	}

	return stream_select($read, $write, $except, $sec, $usec);
}
```
#### (2) WebSocket 服务器
- 使用 `stream_select` 监听多个客户端连接，实现单进程多客户端通信。
#### (3) 实时日志监控
- 同时监听多个日志文件，实时推送更新内容。
### 8. 总结
- `stream_select` 的核心原理是基于 `select` 系统调用的位图轮询机制，通过用户态与内核态的交互实现多路复用 I/O。尽管其性能受限于轮询和文件描述符数量，但在**小规模并发场景**中仍具有实用价值。
- 对于**高性能需求**，建议使用 `epoll`/`kqueue` 基础的替代方案（如 `ReactPHP` 或 `Swoole`）。理解其底层原理有助于优化 PHP 程序的并发能力和资源管理。
## stream_socket... 实例
### 同步阻塞
#### 服务端
```php
// 服务端 接收消息
function receive_message($ip, $port, $timeout)
{
    // 创建socket
    $socket = stream_socket_server("tcp://$ip:$port", $errno, $errMsg);
    if (!$socket) {
        echo "$errMsg ($errno)", PHP_EOL; // 如果创建socket失败输出内容
    }
    // 如果创建成功则接受socket连接并获取信息, 连接在$timeout秒内没有收到消息则断开
    while ($conn = stream_socket_accept($socket, $timeout)) {
        if ($conn) {
            $clientName = stream_socket_get_name($conn, true);

            $message = fread($conn, 1024);
            echo "客户端 $clientName: ", $message, PHP_EOL;
            fclose($conn);
        } else {
            echo $timeout, '秒未收到消息, 自动断开', PHP_EOL;
        }
    }
}
// 服务端,接收消息,直到CTRL-C
receive_message('127.0.0.1', '32100', 30); // 持续接收消息
```
#### 客户端
```php
// 客户端 发送消息
function send_message($ip, $port)
{
    while (true) {
        $fp         = stream_socket_client("tcp://$ip:$port", $errno, $errMsg);
        $clientName = stream_socket_get_name($fp, true);
        if (!$fp) {
            echo "ERREUR : $errno - $errMsg", PHP_EOL;
        }
        echo '请输入消息:', PHP_EOL;
        // 发送消息
        fwrite($fp, fgets(fopen('php://stdin', 'r')));
        $message = fread($fp, 1024);
        echo "服务端 $clientName: ", $message, PHP_EOL;
        fclose($fp); // 关闭socket连接
    }
}

// 客户端, 发送一个消息,并读取响应…
send_message('127.0.0.1', '32100');
```
### 同步非阻塞
#### 服务端
```php
class StreamSelectServer
{
    private $socket;
    private $clients   = [];
    private $isRunning = true;
    private $ip;
    private $port;

    public function __construct($ip = '127.0.0.1', $port = 32100)
    {
        $this->ip   = $ip;
        $this->port = $port;

        // 注册信号处理器
        pcntl_async_signals(true);
        pcntl_signal(SIGINT, [$this, 'shutdown']);
        pcntl_signal(SIGTERM, [$this, 'shutdown']);
    }

    public function start()
    {
        // 创建服务器socket
        $address      = "tcp://$this->ip:$this->port";
        $this->socket = stream_socket_server($address, $errno, $errorMessage, STREAM_SERVER_BIND | STREAM_SERVER_LISTEN);
        if (!$this->socket) {
            echo "创建服务器失败: $errorMessage ($errno)" . PHP_EOL;

            return;
        }
        echo "服务器启动成功，监听 $this->ip:$this->port" . PHP_EOL;
        echo "按 Ctrl+C 停止服务器" . PHP_EOL;
        echo "========================================" . PHP_EOL;
        // 主事件循环
        while ($this->isRunning) {
            // 准备读取数组
            $read = [$this->socket];
            foreach ($this->clients as $client) {
                $read[] = $client['socket'];
            }
            $write  = null;
            $except = null;
            // 使用 stream_select 等待I/O事件
            $ready = stream_select($read, $write, $except, 1); // 1秒超时
            if ($ready === false) {
                echo "stream_select 错误" . PHP_EOL;
                break;
            }
            if ($ready > 0) {
                // 处理可读的sockets
                foreach ($read as $socket) {
                    // 新的客户端连接「当新的客户端连接时，stream_select检测到IO活动,且活动的事件内容=服务器的socket,从而可以区分出是否为新的客户端连接」
                    if ($socket === $this->socket) {
                        $this->handleNewConnection();
                    } else {
                        // 已连接客户端有数据「stream_select检测到IO活动，且活动的事件内容=客户端的socket」
                        $this->handleClientData($socket);
                    }
                }
            }
            // 清理已断开的连接
            $this->cleanupDisconnectedClients();
        }
        $this->stop();
    }

    private function handleNewConnection()
    {
        $conn = stream_socket_accept($this->socket, 0); // 非阻塞接受
        if ($conn) {
            $clientId   = (int)$conn;
            $clientName = stream_socket_get_name($conn, true);

            $this->clients[$clientId] = [
                'socket'        => $conn,
                'name'          => $clientName,
                'connected_at'  => time(),
                'buffer'        => '',
                'last_activity' => time(),
            ];

            echo "[连接] 客户端 $clientName 已连接 (ID: $clientId)" . PHP_EOL;

            // 发送欢迎消息
            $welcomeMessage = "欢迎连接到服务器！当前时间: " . date('Y-m-d H:i:s') . "\n";
            fwrite($conn, $welcomeMessage);
        }
    }

    private function handleClientData($socket)
    {
        $clientId   = (int)$socket;
        $clientName = $this->clients[$clientId]['name'] ?? 'Unknown';

        // 读取数据
        $data = fread($socket, 1024);

        if ($data === false || strlen($data) === 0) {
            // 客户端断开连接
            echo "[断开] 客户端 {$clientName} 断开连接" . PHP_EOL;
            $this->disconnectClient($clientId);

            return;
        }

        // 更新最后活动时间
        $this->clients[$clientId]['last_activity'] = time();
        $this->clients[$clientId]['buffer']        .= $data;

        // 处理完整的消息（以换行符结尾）
        while (($pos = strpos($this->clients[$clientId]['buffer'], "\n")) !== false) {
            $message                            = substr($this->clients[$clientId]['buffer'], 0, $pos);
            $this->clients[$clientId]['buffer'] = substr($this->clients[$clientId]['buffer'], $pos + 1);

            $this->processMessage($clientId, $message);
        }
    }

    private function processMessage($clientId, $message)
    {
        $client     = $this->clients[$clientId];
        $clientName = $client['name'];

        echo "[消息] 来自 {$clientName}: $message" . PHP_EOL;

        // 处理特殊命令
        if (strtolower(trim($message)) === ':q') {
            echo "[退出] 客户端 {$clientName} 请求退出" . PHP_EOL;
            $this->disconnectClient($clientId);

            return;
        }

        if (strtolower(trim($message)) === ':clients') {
            // 显示当前连接的客户端列表
            $clientList = "当前在线客户端:\n";
            foreach ($this->clients as $id => $c) {
                $clientList .= "  - {$c['name']} (ID: $id, 连接时间: " . date('H:i:s', $c['connected_at']) . ")\n";
            }
            fwrite($client['socket'], $clientList);

            return;
        }

        if (strtolower(substr(trim($message), 0, 5)) === ':echo') {
            // 回显消息
            $echoMessage = substr(trim($message), 5);
            fwrite($client['socket'], "回显: $echoMessage\n");

            return;
        }

        // 广播消息给所有客户端
        $broadcastMessage = "[广播] {$clientName}: $message\n";
        $this->broadcastMessage($broadcastMessage, $clientId);

        // 给发送者单独回复
        $response = "服务器确认收到: $message (" . date('H:i:s') . ")\n";
        fwrite($client['socket'], $response);
    }

    private function broadcastMessage($message, $excludeClientId = null)
    {
        foreach ($this->clients as $clientId => $client) {
            if ($excludeClientId !== null && $clientId === $excludeClientId) {
                continue;
            }

            $result = @fwrite($client['socket'], $message);
            if ($result === false) {
                // 发送失败，标记为断开连接
                $this->disconnectClient($clientId);
            }
        }
    }

    private function disconnectClient($clientId)
    {
        if (isset($this->clients[$clientId])) {
            $clientName = $this->clients[$clientId]['name'];
            fclose($this->clients[$clientId]['socket']);
            unset($this->clients[$clientId]);
            echo "[清理] 已清理客户端 $clientName 的连接" . PHP_EOL;
        }
    }

    private function cleanupDisconnectedClients()
    {
        $currentTime = time();
        $timeout     = 300; // 5分钟超时

        foreach ($this->clients as $clientId => $client) {
            if (($currentTime - $client['last_activity']) > $timeout) {
                echo "[超时] 客户端 {$client['name']} 连接超时，自动断开" . PHP_EOL;
                $this->disconnectClient($clientId);
            }
        }
    }

    public function shutdown()
    {
        echo PHP_EOL . "正在关闭服务器..." . PHP_EOL;
        $this->isRunning = false;
    }

    private function stop()
    {
        // 关闭所有客户端连接
        foreach ($this->clients as $clientId => $client) {
            fclose($client['socket']);
        }
        $this->clients = [];

        // 关闭服务器socket
        if ($this->socket) {
            fclose($this->socket);
        }

        echo "服务器已停止" . PHP_EOL;
    }
}

// 启动服务器
$server = new StreamSelectServer('127.0.0.1', 32100);
$server->start();

```
#### 客户端
```php
class StreamlinedClient
{
    private $socket;
    private $isRunning = true;
    private $clientName;

    public function __construct($clientName = 'Client')
    {
        $this->clientName = $clientName;

        // 注册信号处理器以实现优雅退出
        pcntl_async_signals(true);
        pcntl_signal(SIGINT, [$this, 'shutdown']);
        pcntl_signal(SIGTERM, [$this, 'shutdown']);
    }

    public function start($serverIp, $serverPort)
    {
        $address      = "tcp://$serverIp:$serverPort";
        $this->socket = stream_socket_client($address, $errno, $errorMessage, 30);

        if (!$this->socket) {
            echo "连接服务器失败: $errorMessage ($errno)" . PHP_EOL;

            return;
        }

        echo "已连接到服务器 $serverIp:$serverPort" . PHP_EOL;
        echo "客户端名称: {$this->clientName}" . PHP_EOL;
        echo "输入消息并按回车发送，输入 ':q' 退出" . PHP_EOL;
        echo "========================================" . PHP_EOL;

        $readSockets = [$this->socket, STDIN];

        while ($this->isRunning) {
            $readyToRead = $readSockets;
            $write       = null;
            $except      = null;

            // 同时监听socket和STDIN
            if (stream_select($readyToRead, $write, $except, 1) === false) {
                echo "stream_select 错误" . PHP_EOL;
                break;
            }

            if (in_array($this->socket, $readyToRead)) {
                // 读取服务器消息
                $message = fread($this->socket, 1024);
                if ($message === false || $message === '') {
                    echo "服务器断开连接" . PHP_EOL;
                    $this->isRunning = false;
                } else {
                    echo $message;
                }
            }

            if (in_array(STDIN, $readyToRead)) {
                // 读取用户输入
                $input = trim(fgets(STDIN));
                if ($input === ':q') {
                    $this->isRunning = false;
                } else {
                    $this->sendToServer($input);
                }
            }
        }

        $this->disconnect();
    }

    private function sendToServer($message)
    {
        if (!empty($message)) {
            $result = @fwrite($this->socket, $message . "\n");
            if ($result === false) {
                echo "发送消息失败" . PHP_EOL;
                $this->isRunning = false;
            }
        }
    }

    public function shutdown()
    {
        echo PHP_EOL . "正在关闭客户端..." . PHP_EOL;
        $this->isRunning = false;
    }

    private function disconnect()
    {
        if ($this->socket) {
            fclose($this->socket);
        }
        echo "客户端已断开连接" . PHP_EOL;
    }
}

// 启动客户端
$serverIp   = $argv[1] ?? '127.0.0.1';
$serverPort = $argv[2] ?? 32100;
$clientName = $argv[3] ?? 'Client_' . getmypid();
$client     = new StreamlinedClient($clientName);
$client->start($serverIp, $serverPort);

```
## PHP 行过滤器 (Stream Filters)
PHP 流过滤器是一种**中继加工机制**，允许在数据从源（文件、网络、内存）流向目的地时，在不一次性加载全部内容的情况下，对数据进行分块实时处理。
### 核心优势
* **低内存占用**：采用“分块处理”模式，处理 GB 级大文件时内存始终保持在极低水平。
* **高解耦性**：处理逻辑（过滤器）与输入源（文件或网络）完全分离。
* **实时性**：数据在读写的瞬间即完成转换。
### 常用操作
#### 伪协议使用
| 分类      | 示例名称                               | 功能         |
| ------- | ---------------------------------- | ---------- |
| **字符串** | `string.toupper` / `tolower`       | 大小写转换      |
| **转换**  | `convert.base64-encode` / `decode` | Base64 编解码 |
| **压缩**  | `zlib.deflate` / `bzip2.compress`  | 实时压缩数据流    |
| **加密**  | `mcrypt.*` (已废弃) / `openssl.*`     | 读写时加解密     |
```php
// 读取文件并自动转为大写
$content = file_get_contents("php://filter/read=string.toupper/resource=log.txt");
// 读取文件并自动转为base64
$content = file_get_contents("php://filter/read=convert.base64-encode/resource=log.txt");
```
#### 自定义过滤器模板
##### 给文件添加行号
编写自定义过滤器必须继承 `php_user_filter` 并重写 `filter` 方法。
```php
class LineFilter extends php_user_filter
{
    // 记录行号的内部状态，在流的生命周期内保持
    private $lineNumber = 1;

    /**
     * 核心过滤方法
     * $in: 输入数据桶（源数据）
     * $out: 输出数据桶（加工后的数据）
     * $consumed: 引用参数，记录已处理的字节数
     * $closing: 布尔值，标识是否为流的最后一个数据块
     */
    public function filter($in, $out, &$consumed, $closing): int
    {
        // 循环从输入队列中提取可写的“桶”（数据块）
        while ($bucket = stream_bucket_make_writeable($in)) {
            // 将当前桶内的字符串按换行符拆分为数组
            $lines = explode(PHP_EOL, $bucket->data);
            foreach ($lines as &$line) {
                // 仅对非空行进行处理
                if (!empty($line)) {
                    // 核心逻辑：在行首拼接自增行号
                    $line = "[" . ($this->lineNumber++) . "] " . $line;
                }
            }
            // 将处理后的数组重新合并回字符串并存入桶中
            $bucket->data = implode(PHP_EOL, $lines);
            // 更新已消耗的字节计数（必须包含原始数据的长度）
            $consumed += $bucket->datalen;
            // 将加工完的桶推送到输出队列，供下游读取
            stream_bucket_append($out, $bucket);
        }

        // 返回常量：通知系统处理完毕，数据可传递给下一个过滤器或应用层
        return PSFS_PASS_ON;
    }
}

// 1. 将自定义类注册为逻辑名称 "line_processor"
stream_filter_register("line_processor", "LineFilter");
// 2. 以只读模式打开目标文件句柄
$fp = fopen("log.txt", "r");
// 3. 将注册好的过滤器挂载到该文件流上
stream_filter_append($fp, "line_processor");
// 4. 循环读取：此时 fgets 触发过滤器逻辑，获取到的是带行号的数据
while (!feof($fp)) {
    echo fgets($fp);
}
// 5. 关闭句柄，过滤器随之销毁
fclose($fp);
```
##### 大文件写入并自动脱敏手机号
```php
class SensitiveDataFilter extends php_user_filter
{
    public function filter($in, $out, &$consumed, $closing): int
    {
        while ($bucket = stream_bucket_make_writeable($in)) {
            // 逻辑：使用正则匹配手机号，并替换为 138****8888 格式
            // 匹配模式：(前3位)(中间4位)(后4位)
            $pattern      = '/(1[3-9]\d)(\d{4})(\d{4})/';
            $replacement  = '$1****$3';
            $bucket->data = preg_replace($pattern, $replacement, $bucket->data);
            // 更新处理字节数
            $consumed += $bucket->datalen;
            // 将处理后的数据推送到下行（存入磁盘）
            stream_bucket_append($out, $bucket);
        }

        return PSFS_PASS_ON;
    }
}

// 1. 注册过滤器
stream_filter_register("mask_sensitive", "SensitiveDataFilter");
// 2. 以追加写入模式打开日志文件
$fp = fopen("access_log.txt", "w");
// 3. 附加过滤器到写入流 (stream_filter_append 的默认模式包含写入)
stream_filter_append($fp, "mask_sensitive");
// 4. 模拟百次数据写入
for ($i = 0; $i < 100; $i++) {
    // 原始数据包含明文手机号
    $logEntry = "User_ID: {$i} | Phone: 13812345678 | Time: " . date('Y-m-d H:i:s') . PHP_EOL;
    // 写入时，数据会自动经过 SensitiveDataFilter 处理后再落盘
    fwrite($fp, $logEntry);
}
fclose($fp);
```
### 注意事项
* **执行顺序**：`append` 按添加顺序执行，`prepend` 优先执行。
* **作用域**：一旦附加，所有后续的 `fread` 或 `fwrite` 都会经过该过滤器。
* **关闭**：随文件句柄关闭自动释放。
# register_shutdown_function
`register_shutdown_function` 是 PHP 中用于注册 **脚本执行结束后** 自动调用的函数。其关闭时机与 PHP 脚本的生命周期紧密相关，具体如下：
### 1. 触发时机
`register_shutdown_function` 注册的函数会在以下 **所有情况下** 被调用（除非 PHP 进程异常终止）：
#### (1) 脚本正常执行完毕
- 当脚本代码全部执行完毕（包括所有 `return`、`include/require` 等）后，PHP 会进入关闭阶段，调用所有通过 `register_shutdown_function` 注册的函数。
#### (2) 脚本通过 `exit()` 或 `die()` 退出
- 即使脚本通过 `exit()` 或 `die()` 提前终止，`register_shutdown_function` 注册的函数仍会执行。  
```php
register_shutdown_function(fn() => print 'shutdown!!!');
exit(); // 脚本提前退出，但 shutdownFunc 仍会被调用
```
#### (3) 抛出未捕获的异常
- 如果脚本中抛出了未被捕获的异常（包括 `Error` 和 `Exception`），PHP 会先处理异常，然后进入关闭阶段并调用 `register_shutdown_function` 注册的函数。  
  **注意**：定义 register_shutdown_function 函数必须在抛出异常之前「越早越好」
```php
register_shutdown_function(fn() => print 'shutdown!!!');
throw new Exception('致命错误'); // 未捕获的异常，但 shutdownFunc 仍会被调用
```
#### (4) PHP-FPM 的请求处理结束
- 在 PHP-FPM 环境中，每个请求由一个独立的 PHP 子进程处理。当请求处理完成后（无论成功或失败），该子进程会进入关闭阶段，调用所有注册的 `shutdown` 函数。  
  **注意**：如果 PHP-FPM 配置了 `pm = dynamic` 或 `pm = ondemand`，子进程可能被保留以处理后续请求，但 `shutdown` 函数仅在当前请求处理结束时调用一次。
### 2. 不触发的情况
- **PHP 进程崩溃**：如果 PHP 进程因**内存溢出**、**段错误**（Segmentation Fault）等异常崩溃，`register_shutdown_function` 不会被调用。
- **`fastcgi_finish_request()`**：在 FPM 模式下，如果使用了 `fastcgi_finish_request()` 提前结束响应，`shutdown` 函数仍会在脚本完全执行完毕后调用（不依赖于 HTTP 响应是否已发送）。
### 3. 执行顺序
- `register_shutdown_function` 注册的函数会按照 **后进先出（LIFO）** 的顺序执行。  
  **示例**：
```php
register_shutdown_function('funcA'); // 先注册
register_shutdown_function('funcB'); // 后注册
// 执行顺序：funcB -> funcA
```
### 4. 在 PHP-FPM 中的应用
- **资源释放的可靠性**：  
  在 PHP-FPM 环境中，`register_shutdown_function` 是释放资源（如 AMQP 连接、数据库连接）的常用手段之一，尤其适合处理 **单次请求内多次操作资源** 的场景。  
  **示例**：
```php
$connection = new AMQPConnection($config);
register_shutdown_function([$connection, 'close']); // 请求结束时自动关闭连接
```
- **注意事项**：
  1. **避免依赖 `__destruct`**：  
     如果仅依赖对象的 `__destruct` 方法关闭资源，可能因垃圾回收机制延迟而导致资源泄露。结合 `register_shutdown_function` 可提升可靠性。
  2. **极端情况下的健壮性**：  
     如果 PHP 进程因异常崩溃，`register_shutdown_function` 不会执行。此时需通过 PHP-FPM 的 `pm.max_requests` 参数控制子进程生命周期，强制重启进程以释放资源。
### 5. 总结
| 场景                         | 是否触发 `register_shutdown_function` |
| -------------------------- | --------------------------------- |
| 脚本正常执行完毕                   | ✅                                 |
| `exit()` / `die()`         | ✅                                 |
| 未捕获的异常                     | ✅                                 |
| PHP 进程崩溃                   | ❌                                 |
| `fastcgi_finish_request()` | ✅（脚本仍需执行完毕）                       |
**推荐用法**：  
在 PHP-FPM 环境中，使用 `register_shutdown_function` 作为统一的资源释放机制，结合显式关闭逻辑（如 `try-finally`），可确保资源在请求结束时及时释放，避免内存泄漏。
# set_time_limit()
设置允许脚本运行的时间，单位为秒。如果超过了此设置，脚本返回一个致命的错误。默认值为30秒
或者是在php.ini的max_execution_time被定义的值，如果此值存在。
```php
set_time_limit(0); // 允许脚本运行无时间限制
set_time_limit(20); // 允许脚本运行的时间为20秒
```
# ignore_user_abort()
设置客户端断开连接时是否中断脚本的执行
```php
// 默认情况下是设置为 FALSE，与客户机断开会导致脚本停止运行
// 如果设置为 TRUE，则忽略与用户的断开(脚本将继续运行)
ignore_user_abort(true); // 忽略用户中止并允许脚本永久运行
```

