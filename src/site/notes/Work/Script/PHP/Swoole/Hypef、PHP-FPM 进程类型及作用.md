---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Swoole/Hypef、PHP-FPM 进程类型及作用/","title":"Hypef、PHP-FPM 进程类型及作用","tags":["flashcards"],"noteIcon":"","created":"2025-04-09T09:50:21.902+08:00","updated":"2026-03-24T17:50:05.037+08:00","dg-note-properties":{"title":"Hypef、PHP-FPM 进程类型及作用","tags":["flashcards"],"reference linking":null}}
---

# 1.Hyperf 启动的进程类型及作用
Hyperf 基于 **Swoole 扩展**，其进程模型默认包含以下类型：
## (1) 主进程（Master Process）
- **作用**：  
  - 管理所有子进程（Worker、Manager、Task 等）。
  - 监控子进程状态，当子进程崩溃时自动重启。
  - 处理系统信号（如 `SIGTERM`、`SIGINT` 等），优雅地关闭服务。
- **进程状态**：
  - 通常以 `swoole_http_server` 或 `hyperf` 的主进程名显示。
## (2) 管理进程（Manager Process，可选）
- **作用**：  
  - 负责执行一些管理任务（如日志清理、定时任务、配置热更新）。
  - 在某些配置下，Manager 进程会接管部分 Master 的职责，但并非所有场景都需要。
- **是否启用**：  
  - 默认不启用，可通过 `enable_manager` 配置开启。
## (3) 工作进程（Worker Process）
- **作用**：  
  - **直接处理客户端请求**（如 HTTP、WebSocket 请求）。
  - 每个工作进程内部通过 **协程** 实现 **单进程多任务**（即一个进程内同时处理数千个请求）。
- **数量配置**：  
  - 默认为 CPU 核心数，可通过 `worker_num` 参数调整。
- **进程状态**：  
  - 通常以 `swoole_http_server` 的 Worker 进程名显示。
## (4) 任务进程（Task Worker Process）
- **作用**：  
  - 处理 **异步任务**（如耗时计算、数据库批量操作、消息队列任务）。
  - 与工作进程分离，避免阻塞请求处理。
- **数量配置**：  
  - 默认为 `worker_num` 的一半，可通过 `task_worker_num` 参数调整。
- **进程状态**：  
  - 通常以 `swoole_http_server` 的 Task Worker 进程名显示。
## (5) 其他进程（如 Reactor 线程）
- **作用**：  
  - **Reactor 线程**：负责监听网络请求（如 TCP/HTTP 连接），将请求分发到 Worker 进程。
  - **User Thread**：可自定义的线程，用于执行特定任务（如日志处理）。
- **线程数量**：  
  - 默认为 CPU 核心数，可通过 `reactor_num` 参数调整。
# 2.PHP-FPM 启动的进程类型及作用
PHP-FPM（PHP FastCGI Process Manager）采用 **Master/Worker 进程模型**，通过不同类型的进程协作实现对PHP请求的高效处理。以下是其启动的进程类型及其作用：
### 1. Master 进程
**作用**：
- **进程管理**：负责创建、监控和管理所有 Worker 进程。
- **监听端口或 Socket**：通过配置的监听地址（如 `127.0.0.1:9000` 或 Unix Socket）接收来自 Web 服务器（如 Nginx）的请求，将请求分发给空闲的 Worker 进程。
- **动态调整进程数量**：根据配置的 `pm`（进程管理）模式（静态、动态、按需）动态增减 Worker 进程数量。
- **重载配置**：在不中断服务的情况下重新加载 PHP 配置（如 `php-fpm reload`）。
- **监控与重启**：监控 Worker 进程的健康状态，若 Worker 异常终止，Master 会自动重启新的 Worker 进程。
### 2. Worker 进程
**作用**：
- **处理 PHP 请求**：每个 Worker 进程独立处理一个 PHP 请求，执行 PHP 脚本并返回结果给 Web 服务器。
- **单请求处理**：每个 Worker 进程同一时间只能处理一个请求，处理完成后会释放资源，等待下一个请求。
- **资源隔离**：每个 Worker 进程拥有独立的内存空间，避免因某个脚本错误影响其他进程。
### 进程管理模式（pm 配置）
PHP-FPM 支持三种进程管理模式，影响 Worker 进程的数量和行为：
#### (1) 静态模式（Static）
**配置参数**：`pm = static`
**行为**：
- 启动时创建固定数量的 Worker 进程（由 `pm.max_children` 指定）。
- 进程数量不随请求量变化，适合请求量稳定的环境。
**优缺点**：
- **优点**：资源占用稳定，系统负载波动小。
- **缺点**：请求量激增时可能导致排队，内存占用固定。
#### (2) 动态模式（Dynamic）
**配置参数**：`pm = dynamic`
**行为**：
- 启动时创建一定数量的 Worker 进程（`pm.start_servers`）。
- 根据请求量动态调整进程数，在 `pm.min_spare_servers`（最小空闲进程数）和 `pm.max_children`（最大进程数）之间浮动。
- 请求高峰时创建新进程，空闲时回收多余进程。
**优缺点**：
- **优点**：资源利用率高，适应请求波动。
- **缺点**：进程频繁增减可能增加系统开销，需合理配置参数。
#### (3) 按需模式（Ondemand）
**配置参数**：`pm = ondemand`
**行为**：
- 初始不启动 Worker 进程，请求到达时动态创建。
- 根据 `pm.start_servers`、`pm.max_children` 和空闲超时时间（`pm.process_idle_timeout`）调整进程数。
**优缺点**：
- **优点**：低负载时几乎不占用资源。
- **缺点**：高并发时 Master 进程可能因频繁创建/销毁进程而占用 CPU，**不适合高流量场景**。
### 进程间协作流程
1. **请求到达**：Web 服务器（如 Nginx）将 PHP 请求发送到 PHP-FPM 的监听端口/Socket。
2. **Master 分配**：Master 进程将请求分发给空闲的 Worker 进程。
3. **Worker 处理**：Worker 进程执行 PHP 脚本，生成响应结果。
4. **结果返回**：Worker 将响应返回给 Web 服务器，最终发送给客户端。
5. **资源回收**：Worker 进程释放资源，等待下一个请求。
### 关键配置参数
以下参数在 `php-fpm.conf` 或池配置文件中控制进程行为：
- `pm`：进程管理模式（static/dynamic/ondemand）。
- `pm.max_children`：最大 Worker 进程数（静态模式下固定进程数）。
- `pm.start_servers`：动态模式下初始 Worker 进程数。
- `pm.min_spare_servers` 和 `pm.max_spare_servers`：动态模式下空闲进程数的上下限。
- `pm.process_idle_timeout`：按需模式下 Worker 空闲超时后被销毁的时间。
### 总结
- **Master 进程**：负责进程管理、监听请求和动态调整进程数量。
- **Worker 进程**：实际处理 PHP 请求，每个进程单线程单请求。
- **进程模式选择**：根据业务场景选择静态（稳定负载）、动态（波动负载）或按需（低负载）模式。