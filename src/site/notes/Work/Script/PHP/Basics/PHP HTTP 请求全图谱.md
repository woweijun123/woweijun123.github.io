---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Basics/PHP HTTP 请求全图谱/","title":"PHP HTTP 请求全图谱","tags":["flashcards"],"noteIcon":"","created":"2026-03-13T10:17:37.139+08:00","updated":"2026-03-13T10:35:53.871+08:00","dg-note-properties":{"title":"PHP HTTP 请求全图谱","tags":["flashcards"],"reference linking":null}}
---

# PHP 发起 HTTP 请求的四种途径
### 1. 文件流封装器 (Stream Wrappers)
[[Work/Script/PHP/Function/Network/Socket、Stream#Stream\|Socket、Stream#Stream]] 这是 PHP 最内置、最简单的请求方式。它将 HTTP 协议模拟成本地文件操作。
- **常用函数**：`file_get_contents()`, `fopen()`, `readfile()`。
- **底层逻辑**：使用 PHP 内置的 **HTTP Stream Provider**。它是一个非常原始的封装，每次调用都会重新解析 URL、寻找 DNS、建立 TCP 连接。
- **特点**：
    - **无复用**：默认不支持 Keep-Alive（即使 Header 带了，连接也会在函数结束时关闭）。
    - **易用性**：只需配置 `context` 即可处理 Header 和超时。
    - **适用场景**：简单的、低频的内网数据获取。
### 2. cURL 扩展 (Client URL Library)
[[Work/Script/PHP/Basics/Curl\|Curl]] 这是 PHP 生态中最标准、功能最强大的方式，它是基于 C 语言编写的 `libcurl`。
- **常用函数**：`curl_init()`, `curl_exec()`, `curl_multi_init()`。
- **底层逻辑**：PHP 只是包装层，实际逻辑由底层的 `libcurl` 管理。它内部自带了连接管理逻辑。
- **特点**：
    - **连接复用**：在同一个 PHP 请求进程内，多次使用同一个 cURL 句柄（Handle）可以复用 TCP 连接。
    - **协议丰富**：支持 HTTPS、HTTP/2、FTP、SCP 等。
    - **高性能并发**：支持 `curl_multi` 异步并发处理多个请求。
- **适用场景**：几乎所有专业的 API 对接和高性能爬虫。
### 3. 原生 Socket 编程 (Low-Level Sockets)
[[Work/Script/PHP/Function/Network/Socket、Stream#Socket\|Socket、Stream#Socket]] 这是最底层的请求方式，不依赖任何 HTTP 协议库。
- **常用函数**：`fsockopen()`, `pfsockopen()`, `socket_create()`。
- **底层逻辑**：直接通过内核系统调用开启一个 TCP 连接，你需要**手动编写 HTTP 协议报文**（构造 `GET / HTTP/1.1\r\nHost: ...`）。
- **特点**：
    - **极致控制**：你可以精确控制握手过程、字节流读取速度。
    - **持久连接**：`pfsockopen` 可以创建跨请求的**持久化 Socket**，连接不会随脚本结束而关闭。
- **适用场景**：编写自定义网络协议、非常规的 HTTP 交互或学习网络原理。
### 4. 异步与长连接扩展 (Swoole / Workerman)
在常驻内存的 PHP 框架中使用的请求方式。
- **常用方式**：`Swoole\Http\Client`。
- **底层逻辑**：利用 Linux 的 **epoll/kqueue** 事件循环实现非阻塞 IO。
- **特点**：
    - **真正的连接池**：可以实现类似 Go `Transport` 的全局长连接池。
    - **协程支持**：并发请求时不会阻塞当前进程。
- **适用场景**：高性能微服务、实时推送系统。
### 总结对照表
| **请求方式**                       | **抽象层级** | **连接复用能力**  | **配置复杂度**   | **性能** |
| ------------------------------ | -------- | ----------- | ----------- | ------ |
| **Stream (file_get_contents)** | 最高       | 极差 (短连接)    | 低 (一行代码)    | 一般     |
| **cURL**                       | 中        | 优秀 (句柄级)    | 中 (需配置常量)   | 高      |
| **Socket (fsockopen)**         | 最低       | 视代码逻辑定      | 极高 (需手动拼报文) | 理论最高   |
| **Swoole Client**              | 框架级      | 极强 (协程/连接池) | 高 (需常驻内存环境) | 极高     |