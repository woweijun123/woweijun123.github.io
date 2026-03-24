---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Guzzle 异步原理/","title":"Guzzle 异步原理","tags":["flashcards"],"noteIcon":"","created":"2025-06-05T13:03:22.682+08:00","updated":"2026-03-24T17:47:34.760+08:00"}
---

# 1. 核心原理
## 1.1 `curl_multi_*` 的并发能力
PHP 的 `curl_multi_*` 函数族允许在 **单线程中同时管理多个 cURL 连接**，通过非阻塞 I/O 和轮询机制实现并发请求。具体步骤：
- **`curl_multi_init()`**：初始化一个 cURL 多连接句柄（`$mh`）。
- **`curl_multi_add_handle($mh, $ch)`**：将多个 cURL 句柄（`$ch`）添加到多连接句柄中。
- **`curl_multi_exec($mh, $active)`**：执行所有已添加的请求，并返回已完成的数量。
- **`curl_multi_select($mh)`**：等待活动句柄的状态变化（如数据可读/写）。
- **`curl_multi_remove_handle($mh, $ch)`**：移除已完成的请求。
- **`curl_multi_close($mh)`**：关闭多连接句柄，释放资源。

## 1.2 模拟异步的事件循环
Guzzle 通过 **事件循环（Event Loop）** 持续监控所有请求的状态，处理已完成的请求并释放资源。其核心流程如下：
1. **批量添加请求**：将多个 HTTP 请求封装为 cURL 句柄（`$ch`），并添加到 `curl_multi` 句柄中。
2. **轮询与监控**：在事件循环中调用 `curl_multi_exec` 和 `curl_multi_select`，检查是否有请求完成。
3. **处理完成请求**：一旦某个请求完成，立即提取响应数据并触发回调。
4. **清理与释放**：移除已完成的请求句柄，避免资源泄露。
5. **重复循环**：直到所有请求完成或超时。
# 2. 实现步骤
以下是 Guzzle 异步请求的简化实现流程（结合代码示例）：
```php
// 初始化 cURL 多连接句柄
$mh = curl_multi_init();

// 添加多个请求到句柄中
$handles = [];
for ($i = 0; $i < 5; $i++) {
    $ch = curl_init("https://example.com");
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_multi_add_handle($mh, $ch);
    $handles[] = $ch;
}

// 注册脚本结束时的回调函数（确保清理资源）
register_shutdown_function(function () use ($mh, $handles) {
    while (curl_multi_exec($mh, $active) === CURLM_CALL_MULTI_PERFORM) {}

    while ($active && curl_multi_exec($mh, $active) >= 0) {
        while ($info = curl_multi_info_read($mh)) {
            // 提取完成的请求句柄
            $ch = $info['handle'];
            $response = curl_multi_getcontent($ch);
            echo "Response: " . substr($response, 0, 100) . "\n";
            curl_multi_remove_handle($mh, $ch);
            curl_close($ch);
        }
        if ($active) {
            curl_multi_select($mh);
        }
    }

    curl_multi_close($mh);
});
```
## 关键点说明
- **`register_shutdown_function`**：确保在脚本结束时处理所有未完成的请求，避免资源泄漏。
- **`curl_multi_exec`**：非阻塞地执行请求，返回已完成的数量。
- **`curl_multi_select`**：等待活动句柄的状态变化，避免 CPU 空转。
- **资源管理**：动态移除已完成的请求句柄，释放内存。
# 3. 与 Promise 的结合
Guzzle 提供了基于 **Promise** 的异步接口（通过 `React\Promise` 库），将每个请求封装为一个 `Promise` 对象。用户可以通过 `then()` 和 `wait()` 方法处理异步结果。例如：

```php
use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();
$promise1 = $client->getAsync('https://example.com');
$promise2 = $client->getAsync('https://google.com');

// 等待所有请求完成
$results = Promise\unwrap([$promise1, $promise2]);

foreach ($results as $response) {
    echo $response->getBody();
}
```
## 底层逻辑
- 每个 `getAsync()` 请求实际上调用的是 `curl_multi` 的并发机制。
- `Promise` 仅作为上层接口，将异步操作的结果包装为可链式调用的对象。
- `unwrap()` 方法会阻塞直到所有 `Promise` 完成，但底层仍通过 `curl_multi` 实现并发。
# 4. 优势与局限性
## 优势
1. **单线程高效并发**：无需多线程/多进程，减少资源消耗。
2. **非阻塞 I/O**：通过 `curl_multi` 的轮询机制，避免主线程阻塞。
3. **兼容性好**：基于 PHP 标准库（cURL），无需额外依赖（如 Swoole）。
## 局限性
1. **并非真正异步**：本质是单线程的轮询，无法利用多核 CPU。
2. **调试复杂**：异步请求的错误处理和调试需要额外技巧。
3. **依赖 cURL**：需确保服务器环境支持 `curl_multi` 扩展。
# 5. 与其他异步方案对比
| **方案**                  | **原理**                    | **是否多线程** | **适用场景**         |
| ----------------------- | ------------------------- | --------- | ---------------- |
| **Guzzle + curl_multi** | 单线程轮询，`curl_multi` 并发     | ❌         | 简单并发请求（如 API 调用） |
| **Swoole 协程**           | 基于协程的异步 I/O，多线程模型         | ✅         | 高并发服务器、长连接       |
| **ReactPHP 事件循环**       | 事件驱动模型，`stream_select` 轮询 | ❌         | 实时应用（如聊天服务器）     |
# 6. 总结
Guzzle 的异步请求实现本质上是 **通过 `curl_multi` 扩展在单线程中模拟并发**，其核心在于：
1. **事件循环**：持续监控请求状态并处理结果。
2. **资源管理**：动态添加/移除请求句柄，避免内存泄漏。
3. **Promise 接口**：提供更友好的异步编程体验。