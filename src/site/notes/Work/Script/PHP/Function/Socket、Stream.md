---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Socket、Stream/","title":"Socket、Stream","tags":["flashcards","select","epoll","kqueue"],"noteIcon":"","created":"2026-11-02T15:26:25.897+08:00","updated":"2026-11-04T15:27:18.809+08:00","dg-note-properties":{"title":"Socket、Stream","tags":["flashcards","select","epoll","kqueue"],"reference linking":null}}
---

# Socket vs Stream
## 核心区别
| **维度**      | **Socket**                   | **Stream**                               |
| ----------- | ---------------------------- | ---------------------------------------- |
| **定义**      | 操作系统提供的**通信端点抽象**            | PHP 提供的**统一 I/O 抽象**                     |
| **对象范围**    | TCP、UDP、Unix Domain Socket 等 | 文件、内存、管道、压缩数据和网络连接等                      |
| **PHP API** | `socket_*`                   | `fread()`、`fwrite()`、`stream_socket_*` 等 |
| **控制能力**    | 直接控制地址族、Socket 类型、协议、选项和收发标志 | 统一使用读写函数、wrapper、context、filter 和超时设置    |
| **优先场景**    | 需要精细控制底层 Socket 行为           | 需要通用 I/O 接口或快速创建网络连接                     |
- **Socket 不是协议层**：它是应用程序使用内核通信能力的 API 和端点对象；底层对象、文件描述符与收发缓冲区见 [[Work/Linux/Network/Socket-到底是什么？\|Socket 的内核结构]]。
- **Stream 不等于 Socket**：网络 Stream 通常由 Socket 支撑，但文件、内存和进程管道 Stream 并不是 Socket。
- **两套 API 可转换**：`socket_import_stream()` 可导入底层由 Socket 支撑的 Stream；`socket_export_stream()` 将 `Socket` 导出为 Stream。
- **IPC 入口**：Socket 与管道、消息队列和共享内存的选型边界见 [[Work/Script/PHP/Function/Process/IPC#7. Socket：同机和跨机的双向通信\|IPC：Socket]]。
## Wrapper、Transport 与底层对象
| **名称** | **职责** | **示例** |
| --- | --- | --- |
| **Wrapper** | 解释 URL，并创建对应 Stream | `file://`、`http://`、`ftp://`、`php://` |
| **Socket transport** | 指定网络传输方式 | `tcp://`、`udp://`、`unix://`、`ssl://` |
| **Stream resource** | 暴露统一读写接口 | `fread()`、`fwrite()`、`fgets()` |
| **底层对象** | 承担实际文件或网络 I/O | 文件描述符、Socket、管道等 |
# Socket
[PHP：Sockets 扩展](https://www.php.net/manual/zh/book.sockets.php)
## 函数索引
以下按用途覆盖 Sockets 扩展函数；别名和平台特定函数单独标注。
- **创建与生命周期**
    - [socket_create](https://www.php.net/manual/zh/function.socket-create.php) — 按地址族、类型和协议创建 Socket。
    - [socket_create_listen](https://www.php.net/manual/zh/function.socket-create-listen.php) — 创建、绑定并监听指定 TCP 端口，适合快速建立服务端监听 Socket。
    - [socket_create_pair](https://www.php.net/manual/zh/function.socket-create-pair.php) — 创建一对已连接的 Socket，常用于本机双向 IPC。
    - [socket_close](https://www.php.net/manual/zh/function.socket-close.php) — 关闭 Socket 并释放其资源。
    - [socket_shutdown](https://www.php.net/manual/zh/function.socket-shutdown.php) — 停止接收、发送或双向传输，但不立即销毁 Socket 对象。
- **服务端流程**
    - [socket_bind](https://www.php.net/manual/zh/function.socket-bind.php) — 把 Socket 绑定到本地地址和端口。
    - [socket_listen](https://www.php.net/manual/zh/function.socket-listen.php) — 将面向连接的 Socket 切换为监听状态，并设置等待连接队列。
    - [socket_accept](https://www.php.net/manual/zh/function.socket-accept.php) — 接受一个待处理连接，返回用于通信的新 Socket。
- **客户端连接**
    - [socket_connect](https://www.php.net/manual/zh/function.socket-connect.php) — 向远端地址和端口发起连接。
- **基础 I/O**
    - [socket_read](https://www.php.net/manual/zh/function.socket-read.php) — 从 Socket 读取指定上限的字节，可选择二进制或按行读取模式。
    - [socket_write](https://www.php.net/manual/zh/function.socket-write.php) — 向 Socket 写入数据，返回实际写入字节数，可能发生短写。
    - [socket_recv](https://www.php.net/manual/zh/function.socket-recv.php) — 按标志位接收数据，并把结果写入引用缓冲区。
    - [socket_send](https://www.php.net/manual/zh/function.socket-send.php) — 按标志位发送指定长度的数据。
    - [socket_recvfrom](https://www.php.net/manual/zh/function.socket-recvfrom.php) — 接收数据报，并取得发送端地址和端口。
    - [socket_sendto](https://www.php.net/manual/zh/function.socket-sendto.php) — 把数据报发送到指定地址和端口。
- **消息与控制信息**
    - [socket_recvmsg](https://www.php.net/manual/zh/function.socket-recvmsg.php) — 接收消息及地址、辅助数据等控制信息。
    - [socket_sendmsg](https://www.php.net/manual/zh/function.socket-sendmsg.php) — 发送由消息头描述的数据、地址和辅助数据。
    - [socket_cmsg_space](https://www.php.net/manual/zh/function.socket-cmsg-space.php) — 计算某类辅助数据所需的控制消息缓冲区大小。
    - [socket_atmark](https://www.php.net/manual/zh/function.socket-atmark.php) — 判断流式 Socket 是否位于带外数据标记处。
- **多路复用与模式**
    - [socket_select](https://www.php.net/manual/zh/function.socket-select.php) — 等待一组 Socket 进入可读、可写或异常就绪状态，并原地筛选数组。
    - [socket_set_block](https://www.php.net/manual/zh/function.socket-set-block.php) — 把 Socket 设置为阻塞模式。
    - [socket_set_nonblock](https://www.php.net/manual/zh/function.socket-set-nonblock.php) — 把 Socket 设置为非阻塞模式。
- **地址解析**
    - [socket_addrinfo_lookup](https://www.php.net/manual/zh/function.socket-addrinfo-lookup.php) — 按主机名、服务名和提示条件解析地址候选列表。
    - [socket_addrinfo_explain](https://www.php.net/manual/zh/function.socket-addrinfo-explain.php) — 把地址候选展开为地址族、协议和地址等可读信息。
    - [socket_addrinfo_bind](https://www.php.net/manual/zh/function.socket-addrinfo-bind.php) — 根据地址候选创建并绑定 Socket。
    - [socket_addrinfo_connect](https://www.php.net/manual/zh/function.socket-addrinfo-connect.php) — 根据地址候选创建 Socket 并建立连接。
- **端点信息**
    - [socket_getpeername](https://www.php.net/manual/zh/function.socket-getpeername.php) — 取得已连接对端的地址和端口。
    - [socket_getsockname](https://www.php.net/manual/zh/function.socket-getsockname.php) — 取得本地 Socket 的地址和端口。
- **选项**
    - [socket_get_option](https://www.php.net/manual/zh/function.socket-get-option.php) — 读取指定层级的 Socket 选项值。
    - [socket_set_option](https://www.php.net/manual/zh/function.socket-set-option.php) — 设置指定层级的 Socket 选项值。
    - [socket_getopt](https://www.php.net/manual/zh/function.socket-getopt.php) — `socket_get_option()` 的别名。
    - [socket_setopt](https://www.php.net/manual/zh/function.socket-setopt.php) — `socket_set_option()` 的别名。
- **错误处理**
    - [socket_last_error](https://www.php.net/manual/zh/function.socket-last-error.php) — 取得指定 Socket 或最近一次全局 Socket 操作的错误码。
    - [socket_clear_error](https://www.php.net/manual/zh/function.socket-clear-error.php) — 清除指定 Socket 或全局保存的错误码。
    - [socket_strerror](https://www.php.net/manual/zh/function.socket-strerror.php) — 把 Socket 错误码转换为可读错误信息。
- **Stream 互操作**
    - [socket_import_stream](https://www.php.net/manual/zh/function.socket-import-stream.php) — 把底层由 Socket 支撑的 Stream 导入为 `Socket` 对象。
    - [socket_export_stream](https://www.php.net/manual/zh/function.socket-export-stream.php) — 把 `Socket` 导出为可用 Stream 函数操作的资源。
- **Windows 专用**
    - [socket_wsaprotocol_info_export](https://www.php.net/manual/zh/function.socket-wsaprotocol-info-export.php) — 导出可供目标 Windows 进程复制 Socket 的协议描述信息。
    - [socket_wsaprotocol_info_import](https://www.php.net/manual/zh/function.socket-wsaprotocol-info-import.php) — 在 Windows 目标进程中根据协议描述信息导入 Socket。
    - [socket_wsaprotocol_info_release](https://www.php.net/manual/zh/function.socket-wsaprotocol-info-release.php) — 释放此前导出的 Windows Socket 协议描述信息。
## 最小单连接回显流程
`SOCK_STREAM` 提供**字节流**，不保留发送端的消息边界。示例使用换行符分包，并循环处理短写；完整服务器还需增加并发、超时、请求上限和优雅退出。
```php
function socketWriteAll($socket, string $data): void
{
    // Socket 可能短写，按实际写入字节数截断并继续发送。
    while ($data !== '') {
        $written = socket_write($socket, $data, strlen($data));
        if ($written === false || $written === 0) {
            throw new RuntimeException(socket_strerror(socket_last_error($socket)));
        }
        $data = substr($data, $written);
    }
}

// 创建 IPv4 TCP 监听 Socket，并允许服务重启后快速复用地址。
$server = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($server === false) {
    throw new RuntimeException(socket_strerror(socket_last_error()));
}

$client = null;
try {
    socket_set_option($server, SOL_SOCKET, SO_REUSEADDR, 1);
    if (!socket_bind($server, '127.0.0.1', 8080) || !socket_listen($server, 16)) {
        throw new RuntimeException(socket_strerror(socket_last_error($server)));
    }

    // 阻塞等待一个客户端，再按换行符读取一条业务消息。
    $client = socket_accept($server);
    if ($client === false) {
        throw new RuntimeException(socket_strerror(socket_last_error($server)));
    }

    $line = socket_read($client, 1024, PHP_NORMAL_READ);
    if ($line === false) {
        throw new RuntimeException(socket_strerror(socket_last_error($client)));
    }

    // 回写时重新补上换行符，保持请求与响应使用相同的分包规则。
    socketWriteAll($client, 'echo: ' . rtrim($line, "\r\n") . "\n");
} finally {
    // 无论正常返回还是抛出异常，都关闭已创建的 Socket。
    if ($client !== null && $client !== false) {
        socket_close($client);
    }
    socket_close($server);
}
```
- **读取边界**：一次 `socket_read()` 不保证得到一条完整业务消息；这里仅因协议明确使用换行符，才可按行读取。
- **写入边界**：`socket_write()` 可能只写入部分数据，因此必须根据返回字节数继续写。
- **MySQL 联想**：PHP MySQL 驱动可通过 TCP 或 Unix Domain Socket 通信；Socket 只提供字节传输，握手、认证、查询和结果集仍由驱动按 MySQL 协议处理。
# Stream
[PHP：Stream 函数](https://www.php.net/manual/zh/ref.stream.php)
Stream 将不同 I/O 来源统一成可读、可写或双向的资源，并提供相同的读写、超时、过滤和多路复用接口。
## 函数索引
- **Bucket**
    - [stream_bucket_append](https://www.php.net/manual/zh/function.stream-bucket-append.php) — 把 Bucket 追加到 Bucket Brigade 尾部。
    - [stream_bucket_make_writeable](https://www.php.net/manual/zh/function.stream-bucket-make-writeable.php) — 从 Brigade 取出一个可修改的 Bucket。
    - [stream_bucket_new](https://www.php.net/manual/zh/function.stream-bucket-new.php) — 为指定 Stream 创建包含给定数据的新 Bucket。
    - [stream_bucket_prepend](https://www.php.net/manual/zh/function.stream-bucket-prepend.php) — 把 Bucket 插入 Bucket Brigade 头部。
- **Context**
    - [stream_context_create](https://www.php.net/manual/zh/function.stream-context-create.php) — 使用 wrapper 选项和通知参数创建 Stream context。
    - [stream_context_get_default](https://www.php.net/manual/zh/function.stream-context-get-default.php) — 取得默认 Stream context，并可合并新的默认选项。
    - [stream_context_get_options](https://www.php.net/manual/zh/function.stream-context-get-options.php) — 读取 context 中按 wrapper 分组的选项。
    - [stream_context_get_params](https://www.php.net/manual/zh/function.stream-context-get-params.php) — 读取 context 的选项和通知回调等参数。
    - [stream_context_set_default](https://www.php.net/manual/zh/function.stream-context-set-default.php) — 设置默认 context 选项，并返回默认 context。
    - [stream_context_set_option](https://www.php.net/manual/zh/function.stream-context-set-option.php) — 设置一个或一组 context wrapper 选项。
    - [stream_context_set_options](https://www.php.net/manual/zh/function.stream-context-set-options.php) — 批量合并 context wrapper 选项。
    - [stream_context_set_params](https://www.php.net/manual/zh/function.stream-context-set-params.php) — 设置 context 的通知回调等参数。
- **复制与过滤**
    - [stream_copy_to_stream](https://www.php.net/manual/zh/function.stream-copy-to-stream.php) — 从当前游标开始，把一个 Stream 的数据复制到另一个 Stream。
    - [stream_filter_append](https://www.php.net/manual/zh/function.stream-filter-append.php) — 把 Filter 追加到 Stream 的过滤链尾部。
    - [stream_filter_prepend](https://www.php.net/manual/zh/function.stream-filter-prepend.php) — 把 Filter 插入 Stream 的过滤链头部。
    - [stream_filter_register](https://www.php.net/manual/zh/function.stream-filter-register.php) — 注册自定义 Filter 名称与实现类。
    - [stream_filter_remove](https://www.php.net/manual/zh/function.stream-filter-remove.php) — 从 Stream 过滤链移除指定 Filter，并触发剩余数据冲刷。
- **读取与元数据**
    - [stream_get_contents](https://www.php.net/manual/zh/function.stream-get-contents.php) — 从当前游标读取剩余内容或指定长度的数据。
    - [stream_get_filters](https://www.php.net/manual/zh/function.stream-get-filters.php) — 列出当前环境已注册的 Filter 名称。
    - [stream_get_line](https://www.php.net/manual/zh/function.stream-get-line.php) — 读取到指定结束分隔符或长度上限，但不包含分隔符。
    - [stream_get_meta_data](https://www.php.net/manual/zh/function.stream-get-meta-data.php) — 取得阻塞、超时、EOF、URI 和 wrapper 等运行状态。
    - [stream_get_transports](https://www.php.net/manual/zh/function.stream-get-transports.php) — 列出当前环境支持的 Socket transport。
    - [stream_get_wrappers](https://www.php.net/manual/zh/function.stream-get-wrappers.php) — 列出当前环境已注册的 Stream wrapper。
- **判断与路径**
    - [stream_is_local](https://www.php.net/manual/zh/function.stream-is-local.php) — 判断 Stream 或 URL 是否属于本地资源。
    - [stream_isatty](https://www.php.net/manual/zh/function.stream-isatty.php) — 判断 Stream 是否连接到交互式终端。
    - [stream_resolve_include_path](https://www.php.net/manual/zh/function.stream-resolve-include-path.php) — 按 `include_path` 解析文件并返回实际路径。
- **通知回调签名**
    - [stream_notification_callback](https://www.php.net/manual/zh/function.stream-notification-callback.php) — PHP 手册用于描述通知回调参数的占位名称，不是可直接调用的内置函数；实际回调通过 context 的 `notification` 参数注册。
- **多路复用与配置**
    - [stream_select](https://www.php.net/manual/zh/function.stream-select.php) — 等待多组 Stream 可读、可写或异常就绪，并原地筛选数组。
    - [stream_set_blocking](https://www.php.net/manual/zh/function.stream-set-blocking.php) — 切换 Stream 的阻塞或非阻塞模式。
    - [stream_set_chunk_size](https://www.php.net/manual/zh/function.stream-set-chunk-size.php) — 设置 Stream 每次底层读写使用的块大小。
    - [stream_set_read_buffer](https://www.php.net/manual/zh/function.stream-set-read-buffer.php) — 设置 Stream 的读取缓冲策略和大小。
    - [stream_set_timeout](https://www.php.net/manual/zh/function.stream-set-timeout.php) — 设置阻塞 Stream 的读取超时。
    - [stream_set_write_buffer](https://www.php.net/manual/zh/function.stream-set-write-buffer.php) — 设置 Stream 的写入缓冲大小。
- **Socket Stream**
    - [stream_socket_accept](https://www.php.net/manual/zh/function.stream-socket-accept.php) — 接受监听流上的连接并返回客户端 Stream。
    - [stream_socket_client](https://www.php.net/manual/zh/function.stream-socket-client.php) — 创建客户端 Socket Stream，并支持 context、持久连接和异步连接标志。
    - [stream_socket_enable_crypto](https://www.php.net/manual/zh/function.stream-socket-enable-crypto.php) — 在已连接 Stream 上启用、协商或关闭 TLS 加密。
    - [stream_socket_get_name](https://www.php.net/manual/zh/function.stream-socket-get-name.php) — 取得 Socket Stream 的本地或对端地址。
    - [stream_socket_pair](https://www.php.net/manual/zh/function.stream-socket-pair.php) — 创建一对已连接的 Socket Stream，常用于本机 IPC。
    - [stream_socket_recvfrom](https://www.php.net/manual/zh/function.stream-socket-recvfrom.php) — 从 Socket Stream 接收数据，并可取得发送端地址。
    - [stream_socket_sendto](https://www.php.net/manual/zh/function.stream-socket-sendto.php) — 向已连接对端或指定地址发送数据。
    - [stream_socket_server](https://www.php.net/manual/zh/function.stream-socket-server.php) — 创建 TCP、UDP 或 Unix Domain Socket 服务端 Stream。
    - [stream_socket_shutdown](https://www.php.net/manual/zh/function.stream-socket-shutdown.php) — 关闭 Socket Stream 的读取、写入或双向通道。
- **锁支持**
    - [stream_supports_lock](https://www.php.net/manual/zh/function.stream-supports-lock.php) — 判断 Stream 是否支持 `flock()` 文件锁。
- **Wrapper 管理**
    - [stream_wrapper_register](https://www.php.net/manual/zh/function.stream-wrapper-register.php) — 为协议名注册自定义 Stream wrapper 类。
    - [stream_register_wrapper](https://www.php.net/manual/zh/function.stream-register-wrapper.php) — `stream_wrapper_register()` 的别名。
    - [stream_wrapper_restore](https://www.php.net/manual/zh/function.stream-wrapper-restore.php) — 恢复被注销或覆盖的内置 wrapper。
    - [stream_wrapper_unregister](https://www.php.net/manual/zh/function.stream-wrapper-unregister.php) — 注销指定协议的 Stream wrapper。
## 可创建或操作 Stream 的相关函数
以下函数来自 Filesystem、Network 等不同扩展，但都能创建或操作 Stream。
- **创建**
    - [fopen](https://www.php.net/manual/zh/function.fopen.php) — 按模式打开文件或 URL，并返回 Stream。
    - [fsockopen](https://www.php.net/manual/zh/function.fsockopen.php) — 建立 Internet 或 Unix Domain Socket 连接并返回 Stream。
    - [pfsockopen](https://www.php.net/manual/zh/function.pfsockopen.php) — 建立可由同一 PHP Worker 后续请求复用的持久 Socket Stream。
    - [popen](https://www.php.net/manual/zh/function.popen.php) — 启动进程并打开连接其标准输入或输出的单向管道。
    - [tmpfile](https://www.php.net/manual/zh/function.tmpfile.php) — 创建关闭时自动删除的临时文件 Stream。
- **关闭**
    - [fclose](https://www.php.net/manual/zh/function.fclose.php) — 关闭 Stream 并冲刷待写数据。
    - [pclose](https://www.php.net/manual/zh/function.pclose.php) — 关闭 `popen()` 管道，等待子进程并返回其退出状态。
- **读取**
    - [feof](https://www.php.net/manual/zh/function.feof.php) — 判断 Stream 是否已经到达 EOF。
    - [fgetc](https://www.php.net/manual/zh/function.fgetc.php) — 从 Stream 读取一个字节。
    - [fgets](https://www.php.net/manual/zh/function.fgets.php) — 读取一行，遇到换行符、EOF 或长度上限时结束。
    - [fgetcsv](https://www.php.net/manual/zh/function.fgetcsv.php) — 读取一行并按 CSV 规则解析为字段数组。
    - [fread](https://www.php.net/manual/zh/function.fread.php) — 从 Stream 最多读取指定字节数。
    - [fscanf](https://www.php.net/manual/zh/function.fscanf.php) — 按格式字符串解析 Stream 中的一行输入。
    - [fpassthru](https://www.php.net/manual/zh/function.fpassthru.php) — 把当前游标后的剩余内容直接输出，并返回输出字节数。
- **写入**
    - [fwrite](https://www.php.net/manual/zh/function.fwrite.php) — 向 Stream 写入数据并返回实际写入字节数。
    - [fputs](https://www.php.net/manual/zh/function.fputs.php) — `fwrite()` 的别名。
    - [fputcsv](https://www.php.net/manual/zh/function.fputcsv.php) — 把字段数组编码为 CSV 记录并写入 Stream。
    - [fflush](https://www.php.net/manual/zh/function.fflush.php) — 把 PHP 写缓冲中的数据冲刷到底层资源。
- **游标**
    - [fseek](https://www.php.net/manual/zh/function.fseek.php) — 按起点和偏移量移动可定位 Stream 的游标。
    - [ftell](https://www.php.net/manual/zh/function.ftell.php) — 返回当前 Stream 游标位置。
    - [rewind](https://www.php.net/manual/zh/function.rewind.php) — 把 Stream 游标移动到开头，并清除 EOF 状态。
- **状态与文件操作**
    - [fstat](https://www.php.net/manual/zh/function.fstat.php) — 取得 Stream 对应资源的文件状态信息。
    - [ftruncate](https://www.php.net/manual/zh/function.ftruncate.php) — 把文件 Stream 截断或扩展到指定长度。
    - [flock](https://www.php.net/manual/zh/function.flock.php) — 对支持锁的 Stream 获取或释放咨询式文件锁。
- **别名**
    - [set_file_buffer](https://www.php.net/manual/zh/function.set-file-buffer.php) — `stream_set_write_buffer()` 的别名，用于设置写缓冲大小。
# 网络 Stream
## `fsockopen()`
`fsockopen()` 创建 Internet 或 Unix Domain Socket 连接，并返回 Stream。连接建立超时由 `$timeout` 控制；阻塞读取超时需另用 `stream_set_timeout()`。
以下代码仅演示在 TCP Stream 上发送**原始 HTTP/1.0 文本**。它依靠 `Connection: close` 和对端关闭连接确定响应结束，不是完整 HTTP 客户端；生产代码应使用 cURL、Guzzle 等成熟实现处理 TLS、重定向、分块编码和协议边界。
```php
// 连接超时只约束建立 TCP 连接，不约束后续阻塞读取。
$fp = fsockopen('example.com', 80, $errno, $errorMessage, 5);
if ($fp === false) {
    throw new RuntimeException("connect failed: {$errorMessage} ({$errno})");
}

// 单独设置读取超时，并构造依靠连接关闭结束响应的 HTTP/1.0 请求。
stream_set_timeout($fp, 5);
$request = "GET / HTTP/1.0\r\n"
    . "Host: example.com\r\n"
    . "Connection: close\r\n\r\n";

// fwrite() 可能短写，必须发送完完整请求头。
while ($request !== '') {
    $written = fwrite($fp, $request);
    if ($written === false || $written === 0) {
        fclose($fp);
        throw new RuntimeException('write failed');
    }
    $request = substr($request, $written);
}

// HTTP/1.0 + Connection: close：持续读取到对端关闭连接。
while (!feof($fp)) {
    $chunk = fread($fp, 8192);
    if ($chunk === false) {
        break;
    }
    echo $chunk;
}

// 关闭前取得元数据，用于区分正常 EOF 与读取超时。
$meta = stream_get_meta_data($fp);
fclose($fp);
if ($meta['timed_out']) {
    throw new RuntimeException('read timed out');
}
```
## `pfsockopen()`
`pfsockopen()` 是 `fsockopen()` 的**持久连接**版本。
- **生命周期**：连接可在当前请求结束后保留，并由同一 PHP Worker 的后续请求复用。
- **不是连接池**：连接不会自动跨所有进程共享。
- **复用条件**：上层协议必须允许安全复用，并处理残留数据、失效连接和服务端连接数限制。
## `stream_socket_client()`
`stream_socket_client()` 支持更多连接标志、Stream context、异步连接和 TLS 配置，通常比 `fsockopen()` 更适合网络客户端。
```php
// stream_socket_client() 可进一步接收 context 和连接标志。
$fp = stream_socket_client('tcp://example.com:80', $errno, $errorMessage, 5);
if ($fp === false) {
    throw new RuntimeException("connect failed: {$errorMessage} ({$errno})");
}

// 读取超时与连接超时独立设置。
stream_set_timeout($fp, 5);
$request = "GET / HTTP/1.0\r\nHost: example.com\r\nConnection: close\r\n\r\n";
// 循环处理短写，确保请求完整到达对端。
while ($request !== '') {
    $written = fwrite($fp, $request);
    if ($written === false || $written === 0) {
        fclose($fp);
        throw new RuntimeException('write failed');
    }
    $request = substr($request, $written);
}
// 按行读取仅适合这个文本协议示例，不代表 TCP 自带行边界。
while (($line = fgets($fp)) !== false) {
    echo $line;
}

$meta = stream_get_meta_data($fp);
fclose($fp);
if ($meta['timed_out']) {
    throw new RuntimeException('read timed out');
}
```
php-amqplib 使用 `stream_socket_client()` 建立 AMQP 传输连接，并在库内补充 context、超时和错误转换；其等待 Socket 就绪的实现也会使用 `stream_select()`。
## `stream_socket_server()` 与 `stream_socket_accept()`
- **创建监听端点**：`stream_socket_server()` 返回服务器 Stream。
- **等待连接**：`stream_socket_accept()` 返回已连接的客户端 Stream；失败或超时时返回 `false`。
- **默认超时**：省略 `$timeout` 或传 `null` 时使用 `default_socket_timeout`，不代表无限等待。
- **立即尝试**：事件循环通常先用 `stream_select()` 等待监听流可读，再调用 `stream_socket_accept($server, 0)`。
- **UDP 边界**：UDP 不使用 `stream_socket_accept()`，应调用 `stream_socket_recvfrom()` 和 `stream_socket_sendto()`。
```php
// 创建 TCP 监听流；失败时错误号与错误文本由引用参数返回。
$server = stream_socket_server('tcp://127.0.0.1:32100', $errno, $errorMessage);
if ($server === false) {
    throw new RuntimeException("listen failed: {$errorMessage} ({$errno})");
}

// 最多等待 30 秒，并取得客户端地址。
$client = stream_socket_accept($server, 30, $peerName);
if ($client !== false) {
    $request = fgets($client);
    if ($request !== false) {
        $response = 'echo: ' . rtrim($request, "\r\n") . "\n";
        // 网络流同样可能短写，按返回长度继续发送。
        while ($response !== '') {
            $written = fwrite($client, $response);
            if ($written === false || $written === 0) {
                break;
            }
            $response = substr($response, $written);
        }
    }
    fclose($client);
}
fclose($server);
```
## `stream_socket_pair()`
`stream_socket_pair()` 创建两个**已互连、地位对等**的 Socket Stream，常用于本机 IPC。`STREAM_PF_UNIX + STREAM_SOCK_STREAM` 得到全双工 Unix Domain Socket pair，不是匿名管道，也不经过外部网络。
以下示例依赖 PCNTL，适用于安装 PCNTL 的 Unix-like CLI 环境。
```php
function streamWriteAll($stream, string $data): void
{
    // 循环处理短写，保证一条应用层消息完整发送。
    while ($data !== '') {
        $written = fwrite($stream, $data);
        if ($written === false || $written === 0) {
            throw new RuntimeException('stream write failed');
        }
        $data = substr($data, $written);
    }
}

// fork 前创建一对已连接的本地全双工 Stream。
$pair = stream_socket_pair(STREAM_PF_UNIX, STREAM_SOCK_STREAM, STREAM_IPPROTO_IP);
if ($pair === false) {
    throw new RuntimeException('stream_socket_pair failed');
}

$pid = pcntl_fork();
if ($pid === -1) {
    fclose($pair[0]);
    fclose($pair[1]);
    throw new RuntimeException('pcntl_fork failed');
}

if ($pid > 0) {
    // 父进程关闭不用的端点，发送一行请求后等待一行响应。
    fclose($pair[0]);
    streamWriteAll($pair[1], "ping\n");
    echo 'parent received: ' . fgets($pair[1]);
    fclose($pair[1]);
    pcntl_waitpid($pid, $status);
} else {
    // 子进程只保留另一端；换行符是本例的消息边界。
    fclose($pair[1]);
    $request = fgets($pair[0]);
    if ($request !== false) {
        echo 'child received: ' . $request;
        streamWriteAll($pair[0], "pong\n");
    }
    fclose($pair[0]);
    exit(0);
}
```
- **创建时机**：在 `pcntl_fork()` 前创建 pair，fork 后父子进程都会继承两个端点。
- **端点管理**：父子进程分别关闭不用的一端，只通过保留端点通信。
- **通信语义**：两个保留端点组成全双工字节流；消息边界仍需由应用协议定义。
- **进程回收**：父进程调用 `pcntl_waitpid()`，避免子进程成为僵尸进程。
- **完整 IPC 场景**：进程关系和其他 IPC 方案见 [[Work/Script/PHP/Function/Process/IPC#socket.php：UDS socketpair 双向请求／响应\|UDS socketpair 请求／响应]]。
# Stream I/O 边界
## `fgets()` vs `stream_get_contents()`
| **函数**                                  | **读取边界**                         | **适合场景**         | **关键风险**                       |
| --------------------------------------- | -------------------------------- | ---------------- | ------------------------------ |
| `fgets($stream, $length)`               | **换行符**、EOF，或最多 `$length - 1` 字节 | 明确使用**换行符分包**的协议 | 返回值包含换行符；一行可能被长度上限拆开           |
| `stream_get_contents($stream, $length)` | 当前游标后的剩余内容，或最多 `$length` 字节      | 文件、已知长度或确定会结束的输入 | 不识别协议边界；**可能等待 EOF、短读或占用大量内存** |
- **socketpair 为何用 `fgets()`**：`"ping\n"` 提供明确行边界，双方读完一条消息后即可继续响应，不必先关闭连接。
- **直接替换的风险**：
    - **等待 EOF**：不指定长度的 `stream_get_contents()` 会持续读取到 EOF。
    - **互等阻塞**：双方都等待读完后再响应和关闭，因而无法产生 EOF；应使用换行符、固定长度或长度前缀分包。
- **进程管道实例**：`proc_open()` 中 `stream_select()` 已返回、无界读取仍等待 EOF 的实测见 [[Work/Script/PHP/Function/Process/PHP Process Management#stream_get_contents() 等待 EOF\|进程管道的 EOF 阻塞实验]]。
- **读取上限**：`$length` 只限制最多读取多少字节，单次调用仍可能少读。
- **非阻塞流**：读取可能只返回当前可用字节，也可能暂时没有数据。
- **消息边界**：一次读取、指定长度或当前可用字节，都不代表自动得到一条完整业务消息。
- **游标联想**：有限响应体读到 EOF 后，游标停在末尾；日志先读取而未复位时，业务再次读取可能得到空字符串，详见 [[Work/Script/PHP/Learn/Guzzle/Guzzle Stream 与 fopen 映射#游标语义对照\|Guzzle Stream 的游标与 EOF 语义]]。
## 阻塞、非阻塞与超时
- **阻塞模式**：`stream_set_blocking($stream, true)`；读取会等待数据、EOF、超时或错误。
- **非阻塞模式**：`stream_set_blocking($stream, false)`；读取立即返回，暂时无数据不代表连接关闭。
- **读取超时**：`stream_set_timeout()` 主要设置阻塞读取超时；发生后检查 `stream_get_meta_data($stream)['timed_out']`。
- **推荐组合**：事件循环中的 Socket Stream 通常设为非阻塞，并配合 `stream_select()`，避免忙轮询。
```php
// 阻塞读取最多等待 1 秒；超时后须检查流元数据。
stream_set_blocking($stream, true);
stream_set_timeout($stream, 1);

$data = fread($stream, 8192);
$meta = stream_get_meta_data($stream);
if ($meta['timed_out']) {
    // 处理读取超时
}
```
# `stream_select()`
`stream_select()` 提供与 `select()` 类似的**流就绪等待**语义。它同步返回就绪 Stream，由调用方自行分发处理，不会直接触发回调。
底层事件机制与 Reactor／Proactor 的进一步区别见 [[Work/Linux/Core/IO 并发模型#epoll、io_uring 原理\|I/O 并发模型]]。
## 调用语义
```text
stream_select(
    ?array &$read,
    ?array &$write,
    ?array &$except,
    ?int $seconds,
    ?int $microseconds = null
): int|false
```
- **`$read`**：读取不会阻塞的 Stream；到达 EOF 的 Stream 也会被报告为可读。
- **`$write`**：当前可执行部分写入的 Stream，不保证一次写完全部数据。
- **`$except`**：高优先级异常条件，典型是 TCP 带外数据，不是通用错误队列。
- **原地修改**：返回时三个数组只保留就绪 Stream，原始键会被保留；循环中必须重新构造或复制监听数组。
- **返回值**：就绪总数为正整数；超时返回 `0`；失败返回 `false`，必须使用 `=== false` 判断。
- **无限等待**：`$seconds === null` 时可一直等待，直到事件发生或被信号中断。
- **即时轮询**：`0, 0` 会立即返回；在紧密循环中使用会持续占用 CPU。
- **短读短写**：就绪只保证当前至少可执行部分 I/O，实际可能只处理一个字节。
- **PHP 缓冲**：若 PHP 层已缓存可读数据，函数可能不咨询操作系统就返回，并清空未检查的 `$write`／`$except` 数组。
## `select`、`poll`、`epoll`、`kqueue`
| **特性** | **`select`** | **`poll`** | **`epoll`** | **`kqueue`** |
| --- | --- | --- | --- | --- |
| **平台** | 广泛支持 | POSIX 系统广泛支持 | Linux | BSD、macOS 等 |
| **监听集合** | 每轮传入 `fd_set` | 每轮传入 `pollfd[]` | 持久注册 interest list | 持久注册 kevent |
| **FD 边界** | FD 数值受 `FD_SETSIZE` 限制 | 不受 `FD_SETSIZE` 限制 | 受系统资源限制 | 受系统资源限制 |
| **等待成本** | 通常随监听范围 `O(N)` | 通常随传入数量 `O(N)` | 主要随就绪事件数 `K` 增长 | 随变更数和就绪事件数增长 |
| **触发语义** | 类似 LT | 类似 LT | 默认 LT，可选 ET | 依 filter 和 flags；`EV_CLEAR` 可提供 edge-like 行为 |
| **返回方式** | 修改传入集合 | 设置 `revents` | 返回就绪事件数组 | 返回就绪事件数组 |
- **不是无限 FD**：`poll`、`epoll`、`kqueue` 仍受 `RLIMIT_NOFILE`、内核内存和进程资源限制。
- **不是回调 API**：`epoll_wait()` 和 `kevent()` 返回事件列表，上层事件循环再调用业务回调。
- **跨平台方案**：Swoole、libevent、libuv 或事件扩展负责在不同系统选择合适后端；ReactPHP 未安装额外事件扩展时仍可能使用 `stream_select()`。
## LT vs ET
假设客户端写入 **500 字节**，应用第一次只读取 **100 字节**：

| **阶段** | **LT** | **ET** |
| --- | --- | --- |
| **第一次通知** | FD 可读，返回就绪 | 状态由不可读变为可读，返回就绪 |
| **读取后剩余** | 内核缓冲区仍有 **400 字节** | 内核缓冲区仍有 **400 字节** |
| **再次等待** | 条件仍满足，会继续报告 | 状态未重新变化，可能不再报告 |
| **处理要求** | 可分多轮读取，但可能重复唤醒 | 使用非阻塞 FD，持续读取到 `EAGAIN`／`EWOULDBLOCK` |
- **LT 优点**：只要就绪条件仍成立，下一轮仍可收到通知，编程更直接。
- **ET 优点**：减少重复通知，但必须正确排空当前可用数据，并兼顾单连接处理预算，避免其他连接饥饿。
## 最小事件循环
示例使用换行符分包、每连接读缓冲和写缓冲，展示 `stream_select()` 的核心结构。它是教学版，未包含 TLS、认证、限流和生产级异常恢复。
```php
// 监听流和客户端流都设为非阻塞，避免单个连接卡住事件循环。
$server = stream_socket_server('tcp://127.0.0.1:32100', $errno, $errorMessage);
if ($server === false) {
    throw new RuntimeException("listen failed: {$errorMessage} ({$errno})");
}
stream_set_blocking($server, false);

// 每个连接分别维护未组成完整行的读缓冲、待发送数据和半关闭状态。
$clients = [];
$readBuffers = [];
$writeBuffers = [];
$readClosed = [];
$maxBuffer = 64 * 1024;
$maxWriteBuffer = 64 * 1024;

while (true) {
    // stream_select() 会原地改写数组，因此每轮都重新构造监听集合。
    $read = [$server];
    $write = [];
    foreach ($clients as $id => $client) {
        if (!$readClosed[$id]) {
            $read[] = $client;
        }
        if ($writeBuffers[$id] !== '') {
            $write[] = $client;
        }
    }
    $except = null;

    // 最多等待 1 秒；返回后数组中只保留已就绪的流。
    $ready = stream_select($read, $write, $except, 1);
    if ($ready === false) {
        break;
    }

    foreach ($read as $stream) {
        if ($stream === $server) {
            // 监听流可读时接受连接；超时为 0，避免 accept 再次阻塞。
            $client = stream_socket_accept($server, 0);
            if ($client !== false) {
                stream_set_blocking($client, false);
                $id = (int) $client;
                $clients[$id] = $client;
                $readBuffers[$id] = '';
                $writeBuffers[$id] = "connected\n";
                $readClosed[$id] = false;
            }
            continue;
        }

        $id = (int) $stream;
        // 非阻塞读取可能短读，也可能暂时返回空字符串。
        $chunk = fread($stream, 8192);
        if ($chunk === false) {
            fclose($stream);
            unset($clients[$id], $readBuffers[$id], $writeBuffers[$id], $readClosed[$id]);
            continue;
        }
        if ($chunk === '' && feof($stream)) {
            // 对端停止发送后，先冲刷已有写缓冲，再关闭连接。
            $readClosed[$id] = true;
            if ($writeBuffers[$id] === '') {
                fclose($stream);
                unset($clients[$id], $readBuffers[$id], $writeBuffers[$id], $readClosed[$id]);
            }
            continue;
        }
        if ($chunk === '') {
            continue;
        }

        // 未收到换行符的连接不得无限占用内存。
        $readBuffers[$id] .= $chunk;
        if (strlen($readBuffers[$id]) > $maxBuffer) {
            fclose($stream);
            unset($clients[$id], $readBuffers[$id], $writeBuffers[$id], $readClosed[$id]);
            continue;
        }

        // 一次读取可能包含多行，也可能只包含半行。
        while (($pos = strpos($readBuffers[$id], "\n")) !== false) {
            $line = substr($readBuffers[$id], 0, $pos);
            $readBuffers[$id] = substr($readBuffers[$id], $pos + 1);
            $writeBuffers[$id] .= 'echo: ' . rtrim($line, "\r") . "\n";
            if (strlen($writeBuffers[$id]) > $maxWriteBuffer) {
                fclose($stream);
                unset($clients[$id], $readBuffers[$id], $writeBuffers[$id], $readClosed[$id]);
                break;
            }
        }
    }

    foreach ($write as $stream) {
        $id = (int) $stream;
        if (!isset($clients[$id])) {
            continue;
        }
        // 只移除本次实际写出的字节，剩余数据留到下一轮。
        $written = fwrite($stream, $writeBuffers[$id]);
        if ($written === false) {
            fclose($stream);
            unset($clients[$id], $readBuffers[$id], $writeBuffers[$id], $readClosed[$id]);
            continue;
        }
        $writeBuffers[$id] = substr($writeBuffers[$id], $written);
        if ($writeBuffers[$id] === '' && $readClosed[$id]) {
            fclose($stream);
            unset($clients[$id], $readBuffers[$id], $writeBuffers[$id], $readClosed[$id]);
        }
    }
}

// 事件循环退出后统一释放仍存活的客户端和监听流。
foreach ($clients as $client) {
    fclose($client);
}
fclose($server);
```
- **监听流可读**：通常表示有连接等待 `accept`。
- **客户端可读**：可能有数据，也可能已到 EOF。
- **写缓冲**：只把有待发送数据的客户端放入 `$write`，并根据实际写入字节数截短缓冲区。
- **缓冲上限**：读写缓冲都设置上限，防止不发送换行符或不读取响应的客户端无限占用内存。
- **升级路径**：连接规模增大时，可由 Swoole 基于平台事件后端管理 Reactor，并在 Socket 长连接上实现 [[Work/Script/PHP/Swoole/利用Swoole实现PHP+websocket直播、及时通讯#WebSocket 原理\|WebSocket 握手、分帧与消息广播]]。
# Stream Filter
Stream Filter 在数据经过 Stream 时按块转换。配合 `fread()`、`fgets()` 或 `stream_copy_to_stream()` 可增量处理；若最终调用 `file_get_contents()`，结果仍会整体进入内存。
## 内置 Filter
| **类别** | **示例** | **作用** |
| --- | --- | --- |
| **字符串** | `string.toupper`、`string.tolower`、`string.rot13` | 字符转换 |
| **编码** | `convert.base64-encode`、`convert.base64-decode` | Base64 编解码 |
| **压缩** | `zlib.deflate`、`zlib.inflate`、`bzip2.compress` | 流式压缩与解压；取决于已安装扩展 |
- **环境核对**：使用 `stream_get_filters()` 查看当前 PHP 实际支持的 Filter。
- **加密边界**：历史 `mcrypt.*` 已淘汰；现代代码使用 TLS context 或经过认证的加密 API，不假设存在通用 `openssl.*` Filter。
```php
// 读取 log.txt 时由内置 Filter 将内容转换为大写。
$input = fopen('php://filter/read=string.toupper/resource=log.txt', 'r');
if ($input === false) {
    throw new RuntimeException('open failed');
}
stream_copy_to_stream($input, STDOUT);
fclose($input);
```
## 自定义 Filter
`php_user_filter::filter()` 接收输入与输出 bucket brigade，并返回：
- **`PSFS_PASS_ON`**：已产生输出，可传给下一个 Filter。
- **`PSFS_FEED_ME`**：暂时没有输出，需要更多输入。
- **`PSFS_ERR_FATAL`**：发生不可恢复错误。
Bucket 边界不等于行边界或业务记录边界。下例通过 `$pending` 保留跨 bucket 的残片，再对完整日志行编号并脱敏手机号。
```php
class LogLineFilter extends php_user_filter
{
    private string $pending = '';
    private int $lineNumber = 1;

    public function filter($in, $out, &$consumed, $closing): int
    {
        $emitted = false;

        while ($bucket = stream_bucket_make_writeable($in)) {
            // $consumed 统计原始输入字节；残片跨 bucket 保存在 $pending。
            $consumed += $bucket->datalen;
            $this->pending .= $bucket->data;

            // 只处理以换行符结束的完整日志行。
            while (($pos = strpos($this->pending, "\n")) !== false) {
                $line = substr($this->pending, 0, $pos + 1);
                $this->pending = substr($this->pending, $pos + 1);
                if (!$this->appendLine($out, $line)) {
                    return PSFS_ERR_FATAL;
                }
                $emitted = true;
            }
        }

        if ($closing && $this->pending !== '') {
            // 流关闭时输出最后一行，即使它没有换行符。
            if (!$this->appendLine($out, $this->pending)) {
                return PSFS_ERR_FATAL;
            }
            $this->pending = '';
            $emitted = true;
        }

        return $emitted ? PSFS_PASS_ON : PSFS_FEED_ME;
    }

    private function appendLine($out, string $line): bool
    {
        // 保留手机号前三位和后四位，中间四位脱敏。
        $masked = preg_replace(
            '/(?<!\d)(1[3-9]\d)\d{4}(\d{4})(?!\d)/',
            '$1****$2',
            $line
        );
        if ($masked === null) {
            return false;
        }

        $data = '[' . $this->lineNumber++ . '] ' . $masked;
        $bucket = stream_bucket_new($this->stream, $data);
        stream_bucket_append($out, $bucket);

        return true;
    }
}

// 注册写过滤器；同一进程中重复注册相同名称会失败。
if (!stream_filter_register('log.line', LogLineFilter::class)) {
    throw new RuntimeException('filter registration failed');
}

$fp = fopen('access_log.txt', 'w'); // w 会覆盖文件；追加写入使用 a
if ($fp === false) {
    throw new RuntimeException('open failed');
}

// 写入数据先经过编号与脱敏，再进入目标文件。
$filter = stream_filter_append($fp, 'log.line', STREAM_FILTER_WRITE);
if ($filter === false) {
    fclose($fp);
    throw new RuntimeException('filter append failed');
}

foreach ([
    "user=1 phone=13812345678\n",
    "user=2 phone=13987654321\n",
] as $line) {
    while ($line !== '') {
        $written = fwrite($fp, $line);
        if ($written === false || $written === 0) {
            fclose($fp);
            throw new RuntimeException('write failed');
        }
        $line = substr($line, $written);
    }
}
fclose($fp);
```
- **`$consumed`**：累计 Filter 已读取的原始输入字节数。
- **`$closing`**：最后一轮处理时为 `true`，用于输出尚未以换行符结尾的残片。
- **执行顺序**：`stream_filter_append()` 追加到链尾；`stream_filter_prepend()` 插入链首。
- **作用方向**：显式传入 `STREAM_FILTER_READ` 或 `STREAM_FILTER_WRITE`，避免依赖打开模式推断。
# 脚本生命周期补充
以下函数不属于 Stream，但常与常驻 Socket 脚本的退出和清理一起出现，因此保留边界说明。
## `register_shutdown_function()`
在当前请求或脚本进入 PHP shutdown 阶段时执行回调。
- **通常执行**：主脚本自然结束、调用 `exit()`，以及进入 PHP shutdown 阶段的多数未捕获异常或致命错误。
- **不保证执行**：进程被 `SIGKILL`、未处理的 `SIGTERM`、段错误或 OOM killer 直接终止。
- **执行顺序**：多个回调按**注册顺序**执行；某个回调调用 `exit()` 后，后续回调不会执行。
- **FPM 语义**：Worker 可处理多个请求；shutdown 回调在每个请求结束阶段执行，不要求 Worker 进程退出。
- **职责边界**：优先用 `try/finally` 完成关键释放；shutdown 回调用于错误记录、临时文件清理和幂等兜底，不能承担必须成功的业务事务。
```php
register_shutdown_function(static function (): void {
    // shutdown 阶段记录最后一个 PHP 错误，便于补充异常日志。
    $error = error_get_last();
    if ($error !== null) {
        error_log(json_encode($error, JSON_UNESCAPED_UNICODE));
    }
});
```
## `set_time_limit()`
`set_time_limit($seconds)` 设置 PHP 自身的执行时间限制，每次调用都会从零重新计时。
- **`0` 的含义**：取消 PHP 自身的执行时间限制，不代表进程可以永久运行。
- **其他限制**：FPM、Web Server、反向代理、容器、进程管理器、内存限制和外部信号仍可终止脚本。
- **平台差异**：Unix-like 系统与 Windows 对外部 I/O 等待时间的计时语义可能不同。
```php
set_time_limit(0);  // 取消 PHP 自身限制，其他基础设施限制仍有效
set_time_limit(20); // 从本次调用起重新计算约 20 秒
```
## `ignore_user_abort()`
`ignore_user_abort(true)` 只表示客户端断开后允许当前脚本继续执行。
- **检测时机**：PHP 通常在尝试向客户端发送数据时才发现断开；输出缓冲和代理会影响检测。
- **不等于后台任务**：它不会取消执行超时，也无法抵抗进程终止和崩溃。
- **适合场景**：短时间、幂等的收尾工作。
- **不适合场景**：常驻循环、计划任务和订单／支付／消息投递等关键作业；这些任务应交给队列 Worker、cron 或进程管理器。
```php
ignore_user_abort(true); // 客户端断开后，当前脚本仍可继续执行
```
# 参考资料
- [PHP：Sockets 扩展](https://www.php.net/manual/zh/book.sockets.php)
- [PHP：Stream 函数](https://www.php.net/manual/zh/ref.stream.php)
- [PHP：stream_select](https://www.php.net/manual/zh/function.stream-select.php)
- [PHP：stream_socket_accept](https://www.php.net/manual/zh/function.stream-socket-accept.php)
- [PHP：php_user_filter::filter](https://www.php.net/manual/zh/php-user-filter.filter.php)
- [PHP：register_shutdown_function](https://www.php.net/manual/zh/function.register-shutdown-function.php)
