---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/Guzzle/","title":"Guzzle 异步原理","tags":["PHP","Guzzle","异步编程","curl"],"noteIcon":"","created":"2025-06-05T13:03:22.682+08:00","updated":"2026-06-02T15:14:02.033+08:00","dg-note-properties":{"title":"Guzzle 异步原理","tags":["PHP","Guzzle","异步编程","curl"]}}
---

# Guzzle 异步原理
> [!abstract] 核心要点
> Guzzle 的异步请求本质是 **单线程轮询**，通过 `curl_multi` 扩展实现并发，并非真正的异步。
## 1. 基础用法
文档：[Quickstart — Guzzle Documentation](https://docs.guzzlephp.org/en/stable/quickstart.html#making-a-request)
```php
$client = new Client([
    'base_uri' => 'http://www.laravel.com/api/',
    'timeout' => 5
]);

// GET
$options = [
    'headers' => ['Authorization' => 'abc'],
    'query' => ['name' => 'z3']
];
$data = json_decode($client->request('GET', 'test/read', $options)->getBody()->getContents(), true);

// POST
$options = [
    'headers' => ['Authorization' => 'abc'],
    'form_params' => ['name' => '啦啦啦']
];
$data = json_decode($client->request('POST', 'test/add', $options)->getBody()->getContents(), true);

// PUT
$options = [
    'headers' => ['Authorization' => 'abc'],
    'form_params' => ['id' => 1, 'name' => '啦啦啦']
];
$data = json_decode($client->request('PUT', 'test/update', $options)->getBody()->getContents(), true);

// DELETE
$options = [
    'headers' => ['Authorization' => 'abc'],
    'query' => ['id' => 1]
];
$data = json_decode($client->request('DELETE', 'test/delete', $options)->getBody()->getContents(), true);
```
### 并发请求
```php
$route = route('api.test.concurrentOrder');

// 异步请求多条
$promises = [
    $client->postAsync($route, ['form_params' => ['num' => 1]]),
    $client->postAsync($route, ['form_params' => ['num' => 1]]),
    $client->postAsync($route, ['form_params' => ['num' => 1]]),
    $client->postAsync($route, ['form_params' => ['num' => 1]])
];
$results = Utils::unwrap($promises);
$list = [];
foreach ($results as $result) {
    $list[] = json_decode($result->getBody()->getContents(), true);
}
```
### Pool 批量并发
```php
$requests = function ($total) use ($client, $route) {
    for ($i = 0; $i < $total; $i++) {
        yield function () use ($client, $route) {
            return $client->postAsync($route, ['form_params' => ['num' => 1]]);
        };
    }
};

$success = $error = [];
(new Pool($client, $requests(10), [
    'concurrency' => 2,
    'fulfilled' => function (Response $response, $index) use (&$success) {
        $success[] = json_decode($response->getBody()->getContents(), true);
    },
    'rejected' => function (RequestException $reason, $index) use (&$error) {
        $error[] = json_decode($reason->getMessage(), true);
    },
]))->promise()->wait();
```
## 2. 核心原理
### 2.1 `curl_multi_*` 并发能力
PHP 的 `curl_multi_*` 函数族允许在 **单线程**中**同时**管理**多个** cURL **连接**，通过非阻塞 I/O 和轮询机制实现并发请求。

| 函数                                   | 作用               |
| ------------------------------------ | ---------------- |
| `curl_multi_init()`                  | 初始化多连接句柄 `$mh`   |
| `curl_multi_add_handle($mh, $ch)`    | 添加 cURL 句柄到多连接句柄 |
| `curl_multi_exec($mh, $active)`      | 执行请求，返回已完成数量     |
| `curl_multi_select($mh)`             | 等待状态变化（数据可读/写）   |
| `curl_multi_remove_handle($mh, $ch)` | 移除已完成的请求         |
| `curl_multi_close($mh)`              | 关闭句柄，释放资源        |
### 2.2 事件循环模拟异步
Guzzle 通过 **事件循环（Event Loop）** 持续监控所有请求状态：
```
批量添加请求 → 轮询监控 → 处理完成请求 → 清理释放 → 重复循环
```
1. **批量添加请求**：将多个 HTTP 请求封装为 cURL 句柄，添加到 `curl_multi`
2. **轮询与监控**：调用 `curl_multi_exec` 和 `curl_multi_select` 检查完成状态
3. **处理完成请求**：某个请求完成时，提取响应并触发回调
4. **清理与释放**：移除已完成的句柄，避免资源泄露
5. **重复循环**：直到所有请求完成或超时
## 3. 实现示例
```php
// 初始化多连接句柄
$mh = curl_multi_init();

// 添加多个请求
$handles = [];
for ($i = 0; $i < 5; $i++) {
    $ch = curl_init("https://example.com");
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_multi_add_handle($mh, $ch);
    $handles[] = $ch;
}

// 注册结束回调，确保清理资源
register_shutdown_function(function () use ($mh, $handles) {
    while (curl_multi_exec($mh, $active) === CURLM_CALL_MULTI_PERFORM) {}

    while ($active && curl_multi_exec($mh, $active) >= 0) {
        while ($info = curl_multi_info_read($mh)) {
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
> [!warning] 关键点
> - `register_shutdown_function`：确保脚本结束时处理未完成请求
> - `curl_multi_exec`：非阻塞执行，返回已完成数量
> - `curl_multi_select`：避免 CPU 空转
## 4. 与 Promise 的结合
Guzzle 通过 `React\Promise` 库将请求封装为 Promise 对象，提供 `then()` 和 `wait()` 方法。
```php
use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();
$promise1 = $client->getAsync('https://example.com');
$promise2 = $client->getAsync('https://google.com');

$results = Promise\unwrap([$promise1, $promise2]);

foreach ($results as $response) {
    echo $response->getBody();
}
```
> [!info] 底层逻辑
> - `getAsync()` 底层仍调用 `curl_multi` 并发机制
> - `Promise` 是上层接口，将异步操作包装为可链式调用的对象
> - `unwrap()` 会阻塞直到所有 Promise 完成，但底层仍并发执行
## 5. 优劣势分析
### 优势
| 优势      | 说明                     |
| ------- | ---------------------- |
| 单线程高效并发 | 无需多线程/多进程，减少资源消耗       |
| 非阻塞 I/O | 通过轮询避免主线程阻塞            |
| 兼容性好    | 基于 PHP 标准库 cURL，无需额外依赖 |
### 局限性
| 局限      | 说明                      |
| ------- | ----------------------- |
| 并非真正异步  | 本质是单线程轮询，无法利用多核 CPU     |
| 调试复杂    | 错误处理和调试需要额外技巧           |
| 依赖 cURL | 需确保环境支持 `curl_multi` 扩展 |
## 6. 方案对比
| 方案                      | 原理                   | 多线程 | 适用场景           |
| ----------------------- | -------------------- | :-: | -------------- |
| **Guzzle + curl_multi** | 单线程轮询                |  ❌  | 简单并发请求（API 调用） |
| **Swoole 协程**           | 基于协程的异步 I/O          |  ✅  | 高并发服务器、长连接     |
| **ReactPHP 事件循环**       | 事件驱动 + stream_select |  ❌  | 实时应用（聊天服务器）    |
## 7. 总结
> [!tip] Guzzle 异步核心
> 1. **事件循环**：持续监控请求状态并处理结果
> 2. **资源管理**：动态添加/移除请求句柄，避免内存泄漏
> 3. **Promise 接口**：提供友好的异步编程体验