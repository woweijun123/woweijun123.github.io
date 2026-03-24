---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/PHP-FPM优化策略/","title":"PHP-FPM优化策略","tags":["flashcards"],"noteIcon":"","created":"2025-04-12T20:07:02.869+08:00","updated":"2026-03-24T17:47:44.632+08:00"}
---

# `php-fpm.conf` 参数详解
```ini
; 选择进程管理器如何控制子进程的数量。
; 可选值：
;   static  - 固定数量 (pm.max_children) 的子进程；
;   dynamic - 子进程的数量基于以下指令动态设置。使用此进程管理，始终至少会有一个子进程。
;             pm.max_children      - 同时存活的最大子进程数。
;             pm.start_servers     - 启动时创建的子进程数。
;             pm.min_spare_servers - 处于“空闲”状态（等待处理）的最小子进程数。如果“空闲”进程数小于此数，则会创建一些子进程。
;             pm.max_spare_servers - 处于“空闲”状态（等待处理）的最大子进程数。如果“空闲”进程数大于此数，则会终止一些子进程。
;  ondemand - 启动时不创建子进程。当新请求连接时，才会 fork 子进程。使用的参数如下：
;             pm.max_children           - 同时存活的最大子进程数。
;             pm.process_idle_timeout   - 空闲进程被终止前的秒数。
; 注意：此值是必需的。
pm = dynamic

; 当 pm 设置为“static”时要创建的子进程数，以及当 pm 设置为“dynamic”或“ondemand”时的最大子进程数。
; 此值设置将要服务的并发请求数的限制。相当于 Apache 的 MaxClients 指令（使用 mpm_prefork）。
; 相当于原始 PHP CGI 中的 PHP_FCGI_CHILDREN 环境变量。以下默认值基于资源不足的服务器。
; 请务必调整 pm.* 以满足您的需求。
; 注意：当 pm 设置为“static”、“dynamic”或“ondemand”时使用。
; 注意：此值是必需的。
pm.max_children = 200

; 启动时创建的子进程数。
; 注意：仅当 pm 设置为“dynamic”时使用。
; 默认值：min_spare_servers + (max_spare_servers - min_spare_servers) / 2
pm.start_servers = 100

; 所需的最小空闲服务器进程数。
; 注意：仅当 pm 设置为“dynamic”时使用。
; 注意：当 pm 设置为“dynamic”时是必需的。
pm.min_spare_servers = 100

; 所需的最大空闲服务器进程数。
; 注意：仅当 pm 设置为“dynamic”时使用。
; 注意：当 pm 设置为“dynamic”时是必需的。
pm.max_spare_servers = 150

; 空闲进程被终止前的秒数。
; 注意：仅当 pm 设置为“ondemand”时使用。
; 默认值：10s
;pm.process_idle_timeout = 10s;

; 每个子进程在重生之前应执行的请求数。
; 这对于解决第三方库中的内存泄漏很有用。对于无限请求处理，请指定“0”。相当于 PHP_FCGI_MAX_REQUESTS。
; 默认值：0
pm.max_requests = 1000
```
# 优化实例
针对 **16核16GB内存服务器** 的 PHP-FPM 动态模式（`pm = dynamic`）优化，需结合 CPU 核心数、内存容量及业务负载特点调整参数。以下是详细配置建议及示例：
## 一、核心配置参数解析
### 1. 进程管理参数
| 参数                     | 建议值          | 说明                                             |
| ---------------------- | ------------ | ---------------------------------------------- |
| `pm`                   | `dynamic`    | 动态模式，根据负载自动调整子进程数                              |
| `pm.max_children`      | **200-300**  | 最大子进程数，根据内存和单个进程内存占用计算（见下文）                    |
| `pm.start_servers`     | **32**       | 启动时初始进程数（建议为 `pm.min_spare_servers` 的 1.5-2 倍） |
| `pm.min_spare_servers` | **16**       | 最小空闲进程数（建议为 CPU 核心数 × 1.5，如 16 核 × 1 = 16）     |
| `pm.max_spare_servers` | **64**       | 最大空闲进程数（建议为 CPU 核心数 × 4，如 16 核 × 4 = 64）       |
| `pm.max_requests`      | **500-1000** | 每个子进程处理请求数上限，防止内存泄漏（根据业务稳定性调整）                 |
### 2. 内存分配计算
#### 单进程内存占用估算：
1. **`ps` 筛选并排序**：获取所有 `php-fpm` 进程的内存使用量（RSS），按从高到低排序。
2. **`awk` 计算平均值**：对所有 `php-fpm` 进程的 RSS 求和，计算平均值（单位 MB），并保留两位小数输出。
```bash
# 列出所有 php-fpm 进程，并按 内存使用量（RSS） 排序
ps -ylC php-fpm --sort:rss | awk '{sum+=$8} END {printf "%.2f MB\n", sum/NR/1024}'
```
#### ps 部分 参数详解：
- **`-y`**：显示以 **字节（KB）** 为单位的内存使用量（`VSZ` 和 `RSS`）。
- **`-l`**：以 **长格式** 显示进程信息（包含更多字段，如 PID、PPID、内存等）。
- **`-C php-fpm`**：筛选出进程名为 `php-fpm` 的进程。
- **`--sort:rss`**：按 **RSS（Resident Set Size，常驻内存大小）** 从高到低排序。
#### awk 部分 参数详解：
- **`sum+=$8`**：将 `ps` 输出的 **第 8 列（RSS，单位 KB）** 累加到变量 `sum` 中。
- **`NR`**：`awk` 内置变量，表示总行数（即 `php-fpm` 进程的数量）。
- **`sum/NR`**：计算 **平均每个进程的内存使用量（单位 KB）**。
- **`/1024`**：将 KB 转换为 MB（1 MB = 1024 KB）。
- **`%.2f MB\n`**：格式化输出，保留两位小数，并添加单位 `MB`。

| 命令部分                                                  | 作用                  | 关键参数                                        |
| ----------------------------------------------------- | ------------------- | ------------------------------------------- |
| ps -ylC php-fpm --sort:rss                            | 筛选 php-fpm 进程并按内存排序 | -y（内存单位为 KB）、-C（进程名过滤）、--sort:rss（按 RSS 排序） |
| awk '{sum+=$8} END {printf "%.2f MB\n", sum/NR/1024}' | 计算**平均内存**使用量（MB）   | $8（RSS 列）、NR（进程数量）、/1024（KB → MB）           |
#### 最大进程数计算规则
假设单进程平均占用 **60MB**：
```shell
# 最大进程数 = 总内存 × 0.8（保留 20% 给系统）/ 单进程内存
# (16GB × 1024 × 0.8) / 60MB ≈ 218
(16 * 1024 * 0.8)/60 ≈ 218
```
  因此 `pm.max_children` 可设为 **218**。
### 3. 超时与性能参数
| 参数                          | 建议值                 | 说明                         |
| --------------------------- | ------------------- | -------------------------- |
| `request_terminate_timeout` | **30s-60s**         | 请求超时时间，避免长时间阻塞（根据业务接口耗时调整） |
| `request_slowlog_timeout`   | **5s-10s**          | 慢请求日志记录阈值                  |
| `slowlog`                   | `/path/to/slow.log` | 记录慢请求日志路径                  |
## 二、配置优化「8核16GB」
### php-fpm
文件位置 `/usr/local/etc/php-fpm.d/www.conf`
```ini
[www]
; PHP-FPM 进程管理方式，设置为动态。
; 这表示 PHP-FPM 会根据负载自动调整子进程数量。
pm = dynamic

; PHP-FPM 最大子进程数。（根据内存计算，每个 PHP 进程约 88880KB「87MB」）
; 这是可以同时处理的请求的最大数量，应根据服务器内存和预期的并发连接数来设定。
; 计算公式: (16GB * 0.5) / 87MB ≈ 90
pm.max_children = 90

; PHP-FPM 启动时创建的子进程数。
; 这个值应该合理设置，以便在启动时就能处理一部分请求，避免冷启动延迟。
; 启动时的子进程数（建议为 CPU 核数的 1-2 倍）
; 8核 × 1.5 = 12
pm.start_servers = 12

; PHP-FPM 保持的最小空闲子进程数。
; 确保始终有一定数量的空闲进程可以立即处理新请求，提高响应速度。
; 建议 start_servers 的 50%
pm.min_spare_servers = 6

; PHP-FPM 保持的最大空闲子进程数。
; 限制空闲进程的数量，避免浪费过多系统资源。
; 建议 max_children 的 20-30%
; 90 × 0.2 = 18
pm.max_spare_servers = 27

; 每个子进程在重新启动前可以处理的最大请求数。
; 这有助于防止内存泄漏，并确保进程的健康。
pm.max_requests = 1000

; 单个请求的最大执行时间（秒）。
; 如果请求在此时间内未完成，PHP-FPM 将终止该请求。
request_terminate_timeout = 30s

; 记录慢请求的阈值（秒）。
; 如果请求执行时间超过此值，则会被记录到慢日志中。
request_slowlog_timeout = 5s

; 慢请求日志文件的路径。
slowlog = /var/log/php-fpm/slow.log

; 优化参数：设置每个子进程可以打开的最大文件描述符数量。
; 高并发应用可能需要较高的值。
; 不要超过 ulimit -n
rlimit_files = 1024

; 优化参数：设置核心文件大小限制。
; unlimited 表示不限制核心文件的大小，这在调试时很有用。
rlimit_core = unlimited

; 含义：是否清除 PHP-FPM 子进程的环境变量。
; 推荐：生产环境通常推荐设置为 'yes' (开启)。您这里设置为 'no' 是不推荐的。
; 理由：
;   设置为 'yes' (推荐)：会清除不必要的环境变量，只保留 PHP-FPM 配置文件中明确定义的变量。
;                        这有助于提高安全性，防止敏感信息或不必要的变量从父进程泄露到子进程，
;                        减少潜在的安全风险和环境污染。
;   设置为 'no' (不推荐)：所有来自父进程的环境变量都会传递给 PHP-FPM 子进程，
;                         这可能包含一些您不希望 PHP 脚本访问的系统或敏感信息。
clear_env = no

; security.limit_extensions = .php .php3 .php4 .php5 .php7
; 含义：限制 PHP-FPM 仅解析指定扩展名的文件作为 PHP 脚本。
; 推荐：推荐在生产环境开启并设置明确的 .php 扩展名（如只保留 .php）。您这里包含多个版本扩展名。
; 理由：
;   限制扩展名可以有效防止Web服务器配置错误导致非PHP文件被PHP-FPM解析。
;   例如，如果 Web 服务器错误地将 `.txt` 文件发送给 PHP-FPM，并且您的 `security.limit_extensions` 中没有 `.txt`，
;   PHP-FPM 将拒绝处理，从而避免潜在的安全漏洞（如将恶意代码伪装成非 PHP 文件）。
;   通常，只需要 `.php` 扩展名就足够了。如果您仍在使用旧的 `.php3/.php4/.php5/.php7` 扩展名，请保留；
;   否则，为了安全性和简洁性，建议仅保留 `.php`。
security.limit_extensions = .php .php3 .php4 .php5 .php7

; php_flag[display_errors] = off
; 含义：是否在网页上显示 PHP 错误信息。
; 推荐：生产环境强烈推荐设置为 'off' (关闭)。
; 理由：在生产环境中直接显示错误信息可能暴露敏感的系统路径、数据库凭据或代码逻辑，给攻击者提供便利。
;       所有错误都应该记录到日志文件中（通过 log_errors），以便开发者和运维人员私下查看和处理。
php_flag[display_errors] = off

; php_admin_value[error_log] = /var/log/fpm-php.www.log
; 含义：指定 PHP 错误日志文件的路径。
; 推荐：生产环境强烈推荐开启并设置一个绝对路径。
; 理由：将所有 PHP 错误和警告统一记录到日志文件，便于问题排查、监控和审计，而不会暴露给外部用户。
;       确保日志文件路径存在，且 PHP-FPM 运行用户（如 www-data）对该路径有写入权限。
php_admin_value[error_log] = /var/log/fpm-php.www.log

; php_admin_flag[log_errors] = on
; 含义：是否开启错误日志记录。
; 推荐：生产环境强烈推荐设置为 'on' (开启)。
; 理由：这是确保所有 PHP 错误都被记录到指定日志文件的关键设置。与 display_errors=off 配合使用，
;       构成生产环境错误处理的最佳实践。
php_admin_flag[log_errors] = on

; php_admin_value[memory_limit] = 128M
; 含义：限制每个 PHP 脚本可以使用的最大内存量。
; 推荐：推荐根据应用需求设置一个合适的值。128M 是一个常见起点。
; 理由：此设置可以防止有内存泄漏或处理超大数据的脚本耗尽服务器内存，从而影响其他请求或导致系统不稳定。
;       如果您的应用程序处理图片、视频、大量数据或复杂的计算，可能需要更高的内存限制（例如 256M 或 512M）。
;       通过监控应用运行时的内存使用情况（如通过 `php_admin_value[memory_limit]` 导致的错误日志）来调整此值。
php_admin_value[memory_limit] = 128M

; 实时 FPM 状态监控
pm.status_path = /status
```
### OPcache
文件位置 `/usr/local/etc/php/php.ini`
```ini
[opcache]
; 启用 OPcache 扩展 (必须启用) (1 = 启用, 0 = 禁用)
opcache.enable=1
; CLI 模式也启用 Opcache (推荐)
opcache.enable_cli=1
; OPcache 使用的共享内存大小，单位为 MB (根据应用调整，建议 256MB 起步)
opcache.memory_consumption=256
; OPcache 缓存的最大文件数量 (根据应用调整，确保足够大以容纳所有 PHP 文件)
opcache.max_accelerated_files=20000
; 原理：当 validate_timestamps=1 时生效，定义检查文件更新的时间间隔（秒）。
; =0：每个请求都检查文件更新（性能最差，仅适合开发环境）。
; =N（如 60）：每 N 秒检查一次，期间忽略文件修改
opcache.revalidate_freq=2
; 原理：控制 OPcache 是否检查 PHP 文件的修改时间（mtime）。
; =1（启用）：定期检查文件是否更新，若更新则重新编译缓存。
; =0（禁用）：完全跳过文件更新检查，缓存永不自动失效（需手动重置）
opcache.validate_timestamps=0
; 用于存储内部字符串的内存大小「避免存储重复的字符到内存引起的浪费」，单位为 MB。(根据应用调整，如果应用大量重复字符串操作，可以增加此值)
; interned strings 机制会创建一个特殊的共享内存池（Interned Strings Buffer）。当 PHP 遇到一个字符串时，它会首先检查这个字符串是否已经在内存池中。
; 如果存在，它会直接创建一个指向该字符串在内存池中地址的引用。
; 如果不存在，它会将这个字符串添加到内存池中，然后创建一个引用。
opcache.interned_strings_buffer=32
; 优化内存分配 (推荐启用)
opcache.optimization_level=0x7FFFBFFF
; 禁用文件统计，提高性能 (推荐启用)
opcache.use_cwd=1
; opcache.error_log=/var/log/php-fpm-opcache.log
; 含义：指定 Opcache 扩展自身的错误日志文件路径。
; 推荐：生产环境推荐开启并设置一个绝对路径。
; 理由：Opcache 扩展在运行时可能会遇到错误或警告（例如内存不足、文件无法缓存等）。
;       将这些错误记录到单独的日志文件，有助于监控 Opcache 的健康状况，及时发现并解决问题，
;       确保 Opcache 正常工作，从而发挥其性能优化作用。
;       确保日志文件路径存在，且 PHP-FPM 运行用户（如 www-data）对该路径有写入权限。
opcache.error_log=/var/log/php-fpm-opcache.log

; opcache.log_verbosity_level=1
; 含义：设置 Opcache 日志的详细程度。
;       0：不记录任何 Opcache 错误。
;       1：记录错误（默认值）。
;       2：记录警告。
;       3：记录信息（调试信息）。
;       4：记录调试信息。
; 推荐：生产环境推荐设置为 1 或 2。您这里设置为 1 是合适的。
; 理由：设置为 1 (默认) 足够记录 Opcache 的关键错误，有助于发现配置问题或运行时异常。
;       设置为 2 可以记录警告，提供更多信息，但可能增加日志量。
;       不建议在生产环境设置为 3 或 4，因为会产生大量调试信息，迅速填满日志文件，影响性能。
opcache.log_verbosity_level=1

; opcache.max_wasted_percentage=5
; 含义：当 Opcache 共享内存中“浪费”的内存（例如，由于文件更新导致旧缓存失效，但内存未被立即回收）达到此百分比时，
;       Opcache 会尝试重启自身（如果配置允许），以回收这些浪费的内存。
; 推荐：生产环境推荐开启并保持默认值 5% 或根据实际情况微调。您这里设置为 5 是合适的。
; 理由：防止 Opcache 内存碎片化严重，导致有效缓存空间不足。当浪费的内存达到阈值时，
;       Opcache 会尝试清理，确保缓存效率。5% 是一个合理的平衡点，既能避免频繁重启，又能有效管理内存。
opcache.max_wasted_percentage=5

; opcache.validate_permission=1
; 含义：当 Opcache 检查脚本文件的时间戳时，是否也同时检查文件的权限。
; 推荐：生产环境通常推荐设置为 0 (关闭)。您这里设置为 1 是不推荐的。
; 理由：
;   设置为 1 (开启)：Opcache 会在每次文件时间戳验证时，额外检查文件的权限。这会增加文件系统 I/O 开销，
;                    对性能有轻微负面影响。在生产环境中，文件权限通常是固定的，并且由部署流程保证。
;   设置为 0 (关闭，推荐)：Opcache 不会检查文件权限，只检查时间戳。这可以减少不必要的 I/O 操作，
;                    从而提高性能。前提是您已经通过文件系统权限管理确保了 PHP 脚本文件的正确访问权限。
opcache.validate_permission=0

; 启用并设置二级缓存目录。
; 当共享内存已满、服务器重启或共享内存重置时，它应该能提高性能。
; 默认的 "" 会禁用基于文件的缓存。
; 开启文件缓存作为二级缓存可以在某些情况下提供容错和启动加速。
opcache.file_cache=/tmp/opcache_cache

; 启用或禁用共享内存中的操作码缓存。
; 如果启用了 opcache.file_cache，并且你希望仅从文件缓存加载，可以设置为 1。
;opcache.file_cache_only=0
; 当从文件缓存加载脚本时，启用或禁用校验和验证。
; 开启可以确保文件完整性，但会略微增加开销。在生产环境通常保持开启。
opcache.file_cache_consistency_checks=1

;关闭文档注释缓存（节省内存）
opcache.save_comments=0

; 启用大页内存（需OS支持）
opcache.huge_code_pages=1

; +----------------------------------------------------------------------  
; | JIT   
; +----------------------------------------------------------------------
; 保持tracing模式（最佳选择）
opcache.jit = tracing
; 关键！启用JIT（建议100-200MB）
opcache.jit_buffer_size = 160M
; 提升多态调用处理能力（适应依赖注入）
opcache.jit_max_polymorphic_calls = 8
; 增加递归优化深度（适应路由解析）
opcache.jit_max_recursive_calls = 5
; 增加根跟踪数（复杂框架需要）
opcache.jit_max_root_traces = 1024
; 提升热点函数优化阈值
opcache.jit_hot_func = 192
; 其他性能参数
; 保持开启（提升超全局变量性能）
auto_globals_jit = On
; 保持开启（加速正则匹配）
pcre.jit = On
; 提升内联优化能力
; 增加循环展开优化
opcache.jit_hot_loop = 128
; 提升返回点优化
opcache.jit_hot_return = 16
; 增加跟踪能力
; 适应复杂业务逻辑
opcache.jit_max_side_traces = 128
; 允许更长代码路径优化
opcache.jit_max_trace_length = 1024
```
#### JIT 注意事项
1. **内存监控**：  
   每 100MB JIT 缓冲区 ≈ 额外消耗 50MB 内存，确保总内存满足：  
   `(max_children × 单进程内存) + JIT缓冲区 < 总内存`
2. **灰度发布**：  
   首次启用 JIT 时：  
```ini
opcache.jit = function ; 先使用低强度模式
opcache.jit_debug=1 ; 记录JIT编译日志
```
3. **性能对比指标**：  
   - 使用 `ab -n 1000 -c 100` 测试路由响应
   - 对比启用前后的 `Requests per second`
   - 监控 `opcache.jit_blacklist_*` 计数（过高需调整参数）
#### 计算 max_accelerated_files 的数量
OPcache 缓存的最大文件数量 (根据应用调整，确保足够大以容纳所有 PHP 文件)
```bash
find . -type f -name "*.php" | wc -l
```
### 内存与执行限制
文件位置 `/usr/local/etc/php/php.ini`
```ini
memory_limit = 256M      ; 根据单请求内存需求调整
max_execution_time = 30  ; 与 request_terminate_timeout 保持一致
```
### Unix Socket 通信：  
替换 TCP 以减少 30% 延迟：  
```ini
listen = /var/run/php/php8.2-fpm.sock  # Nginx 需匹配此路径
listen.owner = www-data
listen.group = www-data
listen.mode = 0666
```  
Nginx 配置示例：  
```nginx
fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
```
### 函数禁用：  
```ini
disable_functions = exec,passthru,shell_exec  ; 生产环境禁用危险函数
open_basedir = /var/www/html:/tmp  ; 限制文件访问范围
```  
### Laravel 优化
#### 1. 缓存与预加载
```bash
# 清除配置缓存文件
php artisan config:clear
# 清除应用缓存文件
php artisan cache:clear
# 清除路由缓存文件
php artisan route:clear
# 清除视图缓存文件
php artisan view:clear
# 清除优化文件（包括缓存、编译文件等）
php artisan optimize:clear

# 生成配置缓存文件，提高配置加载速度（生产环境推荐）
php artisan config:cache
# 生成路由缓存文件，提高路由注册速度（生产环境推荐）
php artisan route:cache
# 生成视图缓存文件，提高视图渲染速度
php artisan view:cache
# 事件缓存（Laravel 5.8+）
php artisan event:cache
# 优化框架加载，包括合并引导文件等（Laravel 9+ 中功能被 optimize:clear 取代或分散到其他缓存命令）
php artisan optimize

composer dump-autoload -o  # 优化类加载映射
```
> 注意：`config:cache` 后 `env()` 失效，需改用 `config()`。
#### 2. 环境与存储优化
- **关闭调试模式**：  
```env
APP_DEBUG=false  # 避免敏感信息泄露及日志膨胀
```
# 监控与维护策略
#### 1. 实时性能监控
- **php-fpm 状态页**：  
```ini
pm.status_path = /status
```
Nginx 代理供 Prometheus 采集指标。  
- **日志聚合**：  
使用 Loki + Grafana 收集 Nginx/PHP-FPM 日志。
#### 2. 性能分析工具
- **Blackfire**：  
深度分析代码调用链，定位 CPU 热点。  
- **Telescope**：  
Laravel 内置调试工具（仅限非生产环境临时启用）。
#### 3. 定期维护命令
```bash
php artisan optimize:clear    # 更新代码后清除缓存
php artisan queue:restart    # 优雅重启队列
```
# 验证与监控
#### 实时状态查看
```bash
# 查看 PHP-FPM 进程池状态
systemctl status php-fpm
# 监控活跃进程数
watch -n 1 "ps -ef | grep php-fpm | grep -v grep | wc -l"
# 监控 php-fpm 进程 使用的内存总量
ps -ylC php-fpm --sort:rss | awk '{sum+=$8} END {print "PHP-FPM 内存总量 (KB): " sum}'
PHP-FPM 内存总量 (KB): 14172432
```
#### 监控 PHP-FPM 状态 (压测时):
- 修改 `php-fpm.conf` 或 `www.conf`: `pm.status_path = /status`
- 配置 Web Server (Nginx) 允许访问 `/status` (注意安全限制，如只允许 localhost 或 IP 白名单)。
- 访问 `http://your-server/status?full`。关键指标：
##### `nginx status` 配置
```nginx
server {
    listen       80;
    server_name  local.php-fpm.com;
    location ~ ^/status {
          # 设置php fastcgi进程监听的IP地址和端口
          fastcgi_pass   php83:9000;
          # 设置首页文件
          fastcgi_index  index.php;
          # 引入fastcgi的配置文件
          include        fastcgi_params;
          # 设置脚本文件请求的路径
          fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }
}
```
##### `status` 参数详解
```ini
;PHP-FPM 进程池的名称
pool:                  www
;进程管理器模式：动态模式，表示 PHP-FPM 会根据负载自动调整子进程数量
process manager:       dynamic
;PHP-FPM 启动时间
start time:            28/Jul/2025:06:24:18 +0000
;PHP-FPM 启动至今的秒数
start since:           433
;接受的连接数
accepted conn:         19434
;监听队列中的连接数，0 表示没有等待的连接(这个值持续 > 0，尤其经常很大，说明 PHP-FPM 进程不够用或进程太忙来不及处理新请求，是导致高延迟的直接原因！)
listen queue:          400
;监听队列的最大长度，即在队列满之前可以排队的最大请求数,历史最大等待队列长度
max listen queue:      716
;监听队列的实际长度, Socket 队列长度 (通常 128)
listen queue len:      4096
;当前空闲的进程数 (理想情况应有一定波动，不会长期为 0 或长期等于 max)
idle processes:        0
;当前活跃的进程数（正在处理请求的进程数）(如果经常接近或等于 `pm.max_children` (90)，说明进程数达到上限，新请求在排队)
active processes:      200
;总进程数（空闲进程 + 活跃进程）
total processes:       200
;达到过的最大活跃进程数,历史最大活跃进程数 (是否达到 max_children？)
max active processes:  200
;达到子进程最大数量的次数，1 表示曾因达到最大子进程数而拒绝过请求(如果压测期间这个数字一直在增长，是核心问题！说明 `pm.max_children` 不够用或进程处理太慢导致它们都被占满)。
max children reached:  2
;慢请求的数量(配置了 `request_slowlog_timeout` 时有用)
slow requests:         0
```
- **结论：** 如果 `active processes` 经常是 90，`idle processes` 经常是 0，`listen queue` 很大且持续增长，`max children reached` 持续增长，那么 PHP-FPM 进程池已经饱和。原因要么是 `max_children` 设置过低，要么是**每个请求处理时间太长**，导致进程无法快速释放来处理新请求。1500ms 的延迟和 100QPS 也印证了这一点 (90个进程 * 1000ms/1500ms ≈ 60 QPS 的理论上限，实际 100QPS 可能因为波动或统计方式略有差异，但基本吻合瓶颈在饱和的进程池)。
#### 压力测试
```bash
# 使用 ab 测试并发能力
ab -n 5000 -c 100 http://your-site.com/
```
#### 性能分析工具
- **Web 工具**：New Relic、Blackfire
- **命令行工具**：`strace`、`perf`