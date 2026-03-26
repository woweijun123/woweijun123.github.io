---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Swoole/Hyperf vs PHP-FPM 进程内存消耗计算/","title":"Hyperf vs PHP-FPM 进程内存消耗计算","tags":["flashcards"],"noteIcon":"","created":"2025-04-13T14:56:55.098+08:00","updated":"2026-03-24T17:50:06.492+08:00","dg-note-properties":{"title":"Hyperf vs PHP-FPM 进程内存消耗计算","tags":["flashcards"],"reference linking":null}}
---

# PHP-FPM 进程
## 1. 验证空 PHP 脚本的进程内存
**目标**：测量仅包含 PHP 解释器和基础库的进程内存。
**步骤**：
1. 编写测试脚本`memory_test.php`
```php
$pid = getmypid();
// 模拟一个循环，每次迭代都更新输出
for ($i = 1; $i <= 100; $i++) {
	// 返回当前脚本使用的内存（单位：字节）。
    $memoryUsage     = memory_get_usage() / 1024 / 1024;
	// 返回脚本运行过程中的峰值内存（单位：字节）。
    $memoryPeakUsage = memory_get_peak_usage() / 1024 / 1024;
    echo "\r i: $i, pid: $pid, Memory Usage: {$memoryUsage}MB, Peak Memory Usage: {$memoryPeakUsage}MB ";
    // 暂停 1 秒，模拟耗时操作
    sleep(1);
}
```
运行脚本：
```shell
php memory_test.php
```
预期输出：  
```text
i: 78, pid: 164, Memory Usage: 1.622314453125MB, Peak Memory Usage: 1.799919128418MB
```
（注意：PHP 函数返回的是脚本内部使用的内存，而非整个进程的物理内存。）
2. 通过系统命令验证进程内存：
```bash
# 启动 PHP 脚本并获取 PID
php memory_test.php & echo $!
# 假设 PID 是 164
watch -n 1 awk '/VmRSS/' /proc/164/status
VmRSS:	   25916 kB
```
- **RSS（实际内存）**：约 **25.4MB**（与知识库的 20-30MB 范围一致）。
## 2. 验证 Laravel 路由的进程内存
### 测量 Laravel 简单路由的内存使用
**目标**：测试输出字符到文件的简单路由。
**步骤**：
1. **编写测试路由**：
在 `routes/web.php` 中添加：
```php
Route::get('/test', function () {
    $pid = getmypid();
    // 模拟一个循环，每次迭代都更新输出
    for ($i = 1; $i <= 100; $i++) {
        $memoryUsage = memory_get_usage() / 1024 / 1024;
        $memoryPeakUsage = memory_get_peak_usage() / 1024 / 1024;
        file_put_contents('test.txt', "i: $i, pid: $pid, Memory Usage: {$memoryUsage}MB, Peak Memory Usage: {$memoryPeakUsage}MB ");
        // 暂停 1 秒，模拟耗时操作
        sleep(1);
    }
});
```
2. **访问测试路由**：
打开浏览器或用 `curl` 访问 `http://laravel.prod.com/test`，输出：
>注意：PHP 内置服务器的内存统计可能不完全反映 FPM 的实际消耗。
```shell
i: 41, pid: 7, Memory Usage: 2.8706130981445MB, Peak Memory Usage: 2.8714447021484MB 
```
3. **通过 PHP-FPM 测量真实进程内存**：
访问路由后，通过命令行查看进程内存：
>主要看参数VmRSS「进程当前使用的物理内存大小」
```bash
cat /proc/7/status
```
 **预期输出**
 ```shell
好的，这是 /proc/7/status 文件的中文注释：

Name:        php-fpm             # 进程名称：php-fpm
Umask:       0022                # 文件创建掩码
State:       S (sleeping)        # 进程状态：S (睡眠)
Tgid:        7                   # 线程组ID
Ngid:        0                   # 线程组命名空间ID
Pid:         7                   # 进程ID
PPid:        1                   # 父进程ID
TracerPid:   0                   # 跟踪进程ID（0表示没有被跟踪）
Uid:         1000 1000 1000 1000 # 用户ID：实际UID,有效UID,保存的UID,文件系统UID
Gid:         1000 1000 1000 1000 # 组ID：实际GID,有效GID,保存的GID,文件系统GID
FDSize:      64                  # 文件描述符表大小
Groups:      1000 1000           # 进程所属的组ID列表
NStgid:      7                   # 线程组命名空间线程组ID
NSpid:       7                   # 线程命名空间进程ID
NSpgid:      1                   # 进程组命名空间进程组ID
NSsid:       1                   # 会话命名空间会话ID
VmPeak:      188752 kB           # 进程使用的最大虚拟内存大小
VmSize:      188736 kB           # 进程使用的虚拟内存大小
VmLck:       0 kB                # 进程锁定的内存大小
VmPin:       0 kB                # 进程固定的物理内存大小
VmHWM:       30912 kB            # 进程使用的最大物理内存大小（高水位线）
VmRSS:       30912 kB            # 进程当前使用的物理内存大小（常驻内存集）「约30MB」
RssAnon:     12148 kB            # 进程匿名映射的物理内存大小
RssFile:     6984 kB             # 进程文件映射的物理内存大小
RssShmem:    11780 kB            # 进程共享内存映射的物理内存大小
VmData:      13944 kB            # 进程数据段大小
VmStk:       132 kB              # 进程堆栈段大小
VmExe:       12672 kB            # 进程代码段大小
VmLib:       27332 kB            # 进程使用的共享库大小
VmPTE:       180 kB              # 进程页表项大小
VmSwap:      0 kB                # 进程使用的交换空间大小
HugetlbPages:0 kB                # 进程使用的大页内存大小
CoreDumping: 0                   # 进程是否正在进行核心转储
THP_enabled: 1                   # 透明大页是否启用
Threads:     1                   # 进程中的线程数
SigQ:        0/31376             # 进程等待处理的信号队列大小/最大队列大小
SigPnd:      0000000000000000    # 进程挂起的信号
ShdPnd:      0000000000000000    # 共享挂起的信号
SigBlk:      0000000000000000    # 进程阻塞的信号
SigIgn:      0000000200001000    # 进程忽略的信号
SigCgt:      0000000004004a07    # 进程捕获的信号
CapInh:      0000000000000000    # 进程继承的capabilities
CapPrm:      0000000000000000    # 进程允许的capabilities
CapEff:      0000000000000000    # 进程生效的capabilities
CapBnd:      00000000a80c25fb    # 进程绑定的capabilities
CapAmb:      0000000000000000    # 进程环境capabilities
NoNewPrivs:  0                   # 进程是否禁止获取新的特权
Seccomp:     2                   # 进程使用的Seccomp模式
Seccomp_filters: 1               # 进程使用的Seccomp过滤器数
Speculation_Store_Bypass: vulnerable # 预测存储旁路漏洞状态
SpeculationIndirectBranch: unknown # 预测间接分支漏洞状态
Cpus_allowed: f                   # 进程允许使用的CPU掩码
Cpus_allowed_list: 0-3            # 进程允许使用的CPU列表
Mems_allowed: 1                   # 进程允许使用的内存节点掩码
Mems_allowed_list: 0              # 进程允许使用的内存节点列表
voluntary_ctxt_switches: 570      # 进程主动上下文切换次数
nonvoluntary_ctxt_switches: 171    # 进程被动上下文切换次数
 ```
 **快捷查看命令**
```shell
awk '/VmRSS/' /proc/7/status
# VmRSS:     30980 kB「约30MB」
```
### 测量 Laravel 复杂路由的内存使用
**目标**：测试包含数据库、缓存、第三方库的复杂路由。
**步骤**：
1. **编写测试路由**：
在 `routes/web.php` 中添加：
 ```php
Route::get('/complex', function () {
    // 数据库查询
    $users = DB::table('user')->get();
    echo json_encode($users, 256);
    // Redis 缓存
    Cache::store('file')->set('test', 'value');
    $value = Cache::store('file')->get('test');
    echo json_encode(['test' => $value], 256);
    // 第三方库调用（如 Guzzle HTTP 客户端）
    // basic
    $client = new Client([
        'base_uri' => 'https://reqres.in/api/users',
        'timeout' => 5
    ]);
    $data = json_decode($client->request('GET')->getBody()->getContents(), true);
    echo json_encode($data, 256);
    // 输出内存使用
    echo "Memory Usage: " . (memory_get_usage() / 1024 / 1024) . "MB\n";
});
 ```
2. **访问测试路由**：
打开浏览器或用 `curl` 访问 `http://laravel.prod.com/complex`，输出：
>注意：PHP 内置服务器的内存统计可能不完全反映 FPM 的实际消耗。

 ```bash
 ps -p <PID> -o rss,vsz,comm
 # 或
 htop --sort=MEM
 ```
 3. **通过 PHP-FPM 测量真实进程内存**：
访问路由后，通过命令行查看进程内存：
>主要看参数VmRSS「进程当前使用的物理内存大小」

 **快捷查看命令**
```shell
# PHP-FPM启动后检测到进程的初始内存为
VmRSS:      6908 kB 「约6.7MB」

awk '/VmRSS/' /proc/7/status
# VmRSS:     30980 kB「约30MB」
```
# Hyperf 进程
## 1. 验证空进程内存
**目标**：测量仅启动的进程内存。
**步骤**：
1. 编写测试脚本`memory_total.sh`, 统计hyperf所有进程使用的内存
```shell
#!/bin/bash  
  
# 从命令行获取 PID 列表，如果未提供参数，则显示错误信息并退出  
if [ $# -eq 0 ]; then  
  echo "请提供至少一个 PID 作为参数。"  
  exit 1  
fi  
  
pids="$*" # 获取所有命令行参数  
total_rss=0  
  
for pid in $pids; do  
  # 检查 PID 是否为数字  
  if ! [[ "$pid" =~ ^[0-9]+$ ]]; then  
    echo "警告：'$pid' 不是有效的 PID，已跳过。"  
    continue  
  fi  
  
  rss=$(cat /proc/"$pid"/status | grep VmRSS | awk '{print $2}')  
  
  # 检查是否成功获取到 VmRSS 值  
  if [[ -n "$rss" ]]; then  
    total_rss=$((total_rss + rss))  
  else  
    echo "警告：无法获取 PID '$pid' 的 VmRSS 值，已跳过。"  
  fi  
done  
  
echo "Total RSS: $total_rss KB"
```
运行脚本：
```shell
# 查询hyperf进程ID集合
top
Mem: 19820884K used, 13116732K free, 243380K shrd, 417684K buff, 6325592K cached
CPU:   0% usr   0% sys   0% nic 100% idle   0% io   0% irq   0% sirq
Load average: 0.09 0.22 0.29 2/3472 6202
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
   12     9 root     S     241m   1%   8   0% {php} skeleton.Worker.0
    1     0 root     S     239m   1%  15   0% {php} skeleton.Master
   11     9 root     S     237m   1%   4   0% {php} skeleton.TaskWorker.1
    9     1 root     S     235m   1%   8   0% {php} skeleton.Manager
 4669     0 root     S     1720   0%  15   0% /bin/sh
 6179     0 root     S     1720   0%   2   0% /bin/sh
 4675     0 root     S     1692   0%   2   0% /bin/sh
 6202  4675 root     R     1620   0%   3   0% top

# 传入进程ID集合进行统计
sh memory_total.sh 1 9 11 12
Total RSS: 114312 KB 「111.6MB」
```
## 2.测量简单路由的内存使用
**目标**：测试输出字符到文件的简单路由。
**步骤**：
1. **编写测试路由**：
在 `/hyperf/config/api/routes.php` 中添加：
```php
// 测试简单路由的进程内存
Router::get('/simple', function () {
    $pid = getmypid();
    // 模拟一个循环，每次迭代都更新输出
    for ($i = 1; $i <= 10; $i++) {
        $memoryUsage = memory_get_usage() / 1024 / 1024;
        $memoryPeakUsage = memory_get_peak_usage() / 1024 / 1024;
        file_put_contents('test.txt', "i: $i, pid: $pid, Memory Usage: {$memoryUsage}MB, Peak Memory Usage: {$memoryPeakUsage}MB ");
        // 暂停 1 秒，模拟耗时操作
        sleep(1);
    }
}, [RouterOption::LEVEL => UserLevel::GUEST]);
```
2. **访问测试路由**：
打开浏览器或用 `curl` 访问 `http://127.0.0.1:9501/simple`，输出：
>注意：PHP 内置服务器的内存统计可能不完全反映 FPM 的实际消耗。
```shell
i: 10, pid: 12, Memory Usage: 6.7391128540039MB, Peak Memory Usage: 6.9345321655273MB
```
3. **通过测量真实进程内存**：
访问路由后，通过命令行查看进程内存：
>主要看参数VmRSS「进程当前使用的物理内存大小」
```bash
# hyperf.Worker.0 进程的初始内存为
watch -n 1 sh memory_total.sh 12
Total RSS: 28216 KB「约27.55MB」

# 请求简单路由后
watch -n 1 sh memory_total.sh 12
Total RSS: 29808 KB「约29.11MB」
```
## 3.测量复杂路由的内存使用
**目标**：测试包含数据库、缓存、第三方库的复杂路由。
**步骤**：
1. **编写测试路由**：
在 `/hyperf/config/api/routes.php` 中添加：
 ```php
use GuzzleHttp\Client;
use Hyperf\Redis\Redis;
use Hyperf\Context\ApplicationContext;
// 测试复杂路由的进程内存
Router::get('/complex', function () {
    // 数据库查询
    $users = DB::table('user')->get();
    echo json_encode($users, 256);
    // Redis 缓存
    $redis = ApplicationContext::getContainer()->get(Redis::class);
    $redis->set('test', 'value');
    $value = $redis->get('test');
    echo json_encode(['test' => $value], 256);
    // 第三方库调用（如 Guzzle HTTP 客户端）
    // basic
    $client = new Client([
        'base_uri' => 'https://reqres.in/api/users',
        'timeout' => 5
    ]);
    $data = json_decode($client->request('GET')->getBody()->getContents(), true);
    echo json_encode($data, 256);
    // 输出内存使用
    echo "Memory Usage: " . (memory_get_usage() / 1024 / 1024) . "MB\n";
}, [RouterOption::LEVEL => UserLevel::GUEST]);
 ```
2. **访问测试路由**：
打开浏览器或用 `curl` 访问 `http://10.0.0.6:9501/complex`，输出：
>注意：PHP 内置服务器的内存统计可能不完全反映 FPM 的实际消耗。

 3. **测量真实进程内存**：
访问路由后，通过命令行查看进程内存：
>主要看参数VmRSS「进程当前使用的物理内存大小」
 **快捷查看命令**
```shell
# hyperf.Worker.0 进程的初始内存为
watch -n 1 sh memory_total.sh 12
Total RSS: 27816 KB「约27.2MB」

# 请求复杂路由后
watch -n 1 sh memory_total.sh 12
# VmRSS:     38044 kB「约37.15MB」
```
# Hyperf 协程内存
**目标**：测试10万次并发，协程使用的内存情况。
**步骤**：
1. **编写测试路由**：
在 `/hyperf/config/api/routes.php` 中添加：
```php
// 测试协程路由的内存使用
Router::get('/coroutine', function () {
    $pid = getmypid();
    echo $pid;
    // 模拟10万并发，检测使用内存
    for ($i = 0; $i < 100000; $i++) {
        Coroutine::create(function () {
            sleep(5);
        });
    }
    sleep(10);
    echo 'success';
}, [RouterOption::LEVEL => UserLevel::GUEST]);
```
2. **访问测试路由**：
打开浏览器或用 `curl` 访问 `http://10.0.0.9501/coroutine`
 3. **测量真实进程内存**：
访问路由后，通过命令行查看进程内存：
```shell
# hyperf.Worker.0 进程的初始内存为
watch -n 1 sh memory_total.sh 12
Total RSS: 29252 KB

# 请求测试协程路由的内存使用接口后
watch -n 1 sh memory_total.sh 12
Total RSS: 1710692 KB

# 结果每个协程耗费约16.81KB
(1710692-29252)/100000≈16.81KB
```
# 总结
## PHP-FPM进程
| **场景**       | **PHP 内置函数结果** | **系统命令（RSS）** |
| ------------ | -------------- | ------------- |
| FPM命令行空脚本    | ~1.62MB        | ~25.4MB       |
| Laravel 简单路由 | ~2.9MB         | ~27.5MB       |
| Laravel复杂路由  | ~3.2MB         | ~31.9MB       |
## Hyperf worker进程
| **场景** | **PHP 内置函数结果** | **系统命令（RSS）** |
| ------ | -------------- | ------------- |
| 简单路由   | ~6.74MB        | ~29.11MB      |
| 复杂路由   | ~7.29MB        | ~37.15MB      |
# 注意事项
1. **单位差异**：PHP 函数返回的内存是脚本内部使用的内存，而 `ps` 显示的是整个进程的物理内存（包含 PHP 解释器、扩展等）。
2. **环境影响**：开启的 PHP 扩展（如 OPcache、Xdebug）会显著增加内存消耗。
3. **FPM 池配置**：PHP-FPM 的 `pm.max_children` 和 `pm.start_servers` 设置会影响总内存占用（单进程 × 子进程数）。