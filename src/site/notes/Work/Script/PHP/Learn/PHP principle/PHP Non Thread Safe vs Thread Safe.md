---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/PHP principle/PHP Non Thread Safe vs Thread Safe/","title":"PHP Non Thread Safe vs Thread Safe","tags":["flashcards"],"noteIcon":"","created":"2023-10-14T01:34:18.000+08:00","updated":"2026-03-24T17:46:49.629+08:00"}
---

### 1. 核心定义
* **Thread Safe (TS - 线程安全)**
在多线程环境下运行。它通过 **TSRM (Thread Safe Resource Manager)** 机制对共享资源进行加锁和检查。虽然安全性更高，但由于频繁的检查，会有约 **10% ~ 20%** 的性能损耗。
* **Non Thread Safe (NTS - 非线程安全)**
不包含线程安全检查代码。它假设运行环境本身就是单线程执行（或进程隔离），通过省去锁机制来换取更高的执行效率。
### 2. 运行环境与 SAPI 的匹配关系
选择哪个版本，关键取决于 **PHP 与 Web 服务器的通信方式（SAPI）**：

| SAPI 类型 | 典型应用场景 | 建议版本 | 原因 |
| --- | --- | --- | --- |
| **FastCGI** | Nginx + PHP-FPM / IIS (FastCGI) | **NTS (非线程安全)** | 每个请求由独立的进程处理，不存在线程干扰。 |
| **ISAPI** | 旧版 IIS | **TS (线程安全)** | 以 DLL 形式集成在 Web 服务器进程内，通过多线程处理。 |
| **Apache Module** | Apache (mod_php) | **TS (线程安全)** | 如果 Apache 使用 Worker 或 Event MPM（多线程模式），必须选 TS。 |
| **CLI** | 命令行脚本 | **NTS (非线程安全)** | 脚本通常是单进程执行。 |
### 3. 为什么 FastCGI 推荐 NTS？
1. **进程隔离**：FastCGI 是一种常驻型的 **CGI** 协议，它启动多个独立的进程（如 PHP-FPM 的 worker 进程）来处理请求。
2. **无需检查**：既然每个进程是独立的，就不会出现多个线程竞争同一块内存的情况，线程安全检查就成了多余的“包袱”。
3. **效率优化**：使用 NTS 版本可以最大限度地发挥 PHP 的执行速度。
### 4. 为什么 IIS (ISAPI) 需要 TS？
在旧版 IIS 中，PHP 是作为扩展（ISAPI）加载到服务器进程中的。这意味着一个 IIS 进程里有多个 PHP 线程在跑。由于许多第三方的 PHP 扩展（Extensions）并不是线程安全的，如果不开启 TS 检查，很容易导致内存溢出或服务器崩溃。

> **注意**：现代 Windows 环境下，建议配合 IIS 使用 FastCGI 模式，并同样选择 **NTS** 版本，这已成为官方推荐的最佳实践。
### 总结建议
* **Linux (Nginx/Apache)**：绝大多数场景下直接选 **NTS**。
* **Windows (IIS + FastCGI)**：务必选 **NTS**。
* **Windows (Apache 模块模式)**：通常选 **TS**（x64 版本）。