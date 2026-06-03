---
{"dg-publish":true,"permalink":"/Work/踩坑/Guzzle-AOP日志导致协程HTTP响应体读取为空事故复盘/","title":"Guzzle-AOP日志导致协程HTTP响应体读取为空事故复盘","tags":["踩坑","hyperf","guzzle"],"noteIcon":"","created":"2026-06-01T18:31:02.182+08:00","updated":"2026-06-02T15:34:35.616+08:00","dg-note-properties":{"title":"Guzzle-AOP日志导致协程HTTP响应体读取为空事故复盘","tags":["踩坑","hyperf","guzzle"],"reference linking":null}}
---

# 结论
本次事故根因是：全局 Guzzle AOP 日志切面在 `on_stats` 回调中读取了响应体，导致使用 `Hyperf\Guzzle\CoroutineHandler` 的业务客户端后续再通过 `getBody()->getContents()` 读取时拿到空字符串。
默认 Guzzle `CurlHandler` 路径不会暴露该问题，因为 `vendor/guzzlehttp/guzzle/src/Handler/CurlFactory.php` 在 `on_stats` 后、返回 Response 前会执行：
```php
$body = $easy->response->getBody();
if ($body->isSeekable()) {
    $body->rewind();
}
```
但 `RechargeHttpClient` 使用了自定义协程 handler：
```php
$stack = HandlerStack::create(new CoroutineHandler());
```
这条链路不会进入 `CurlFactory::finish()`，所以没有自动 rewind。AOP 先读取 body 后，业务代码再 `getContents()` 就读取不到内容。
# 事故影响
影响路径：`app/Supports/Baokim/HttpClient/RechargeHttpClient.php`
关键代码：
```php
$response = $this->client->request($isPost ? 'POST' : 'GET', $path, $options);
$contents = $response->getBody()->getContents();
$resArray = json_decode($contents, true) ?: [];
```
当全局 AOP 已经读取过 response body 时，`$contents` 为空字符串，导致 `$resArray` 为空数组，进而影响充值/支付网关响应解析。
# 触发条件
同时满足以下条件时触发：
1. 请求经过 `GuzzleHttpHandlerStackAspect` 注入的 `http_log` 中间件。
2. `on_stats` 回调里调用 `parseBody($response)`。
3. `parseBody()` 内部执行 `(string)$message->getBody()` 读取 response body。
4. HTTP 客户端使用 `HandlerStack::create(new CoroutineHandler())`，不走 Guzzle `CurlFactory::finish()`。
5. 后续业务通过 `$response->getBody()->getContents()` 读取响应体。
# 关键代码
## AOP 日志切面
文件：`app/Internal/Aspect/GuzzleHttpHandlerStackAspect.php`
```php
$options['on_stats'] = static function (TransferStats $stats): void {
    $response = $stats->getResponse();
    LogTool::gather('http', [
        'response' => [
            'body' => self::parseBody($response),
        ],
    ]);
};
```
```php
private static function parseBody(?MessageInterface $message): mixed
{
    if ($message === null) {
        return null;
    }
    $body = (string)$message->getBody();
    // 这里读取后，stream 指针会停在末尾。
}
```
## CoroutineHandler 不会自动 rewind
文件：`vendor/hyperf/guzzle/src/CoroutineHandler.php`
```php
if ($callback = $options[RequestOptions::ON_STATS] ?? null) {
    $stats = new TransferStats(
        $request,
        $response,
        $transferTime,
        $raw->statusCode,
        []
    );
    $callback($stats);
}
return $response;
```
这里执行完 `on_stats` 后直接 `return $response`，没有像 `CurlFactory::finish()` 那样执行 `$body->rewind()`。
# 测试用例说明
文件：`test/ExampleTest.php:111`
当前测试用例覆盖了三种路径：
1. 默认 Guzzle Client 请求本地接口 `/public/post`，通过 `getContents()` 读取。
2. `HttpTool::post()` 请求本地接口，通过先 `getContents()` 再 `(string)$body` 验证指针行为。
3. `RechargeHttpClient::send()` 使用 `HandlerStack::create(new CoroutineHandler())`，模拟真实生产链路。
本地测试接口：
文件：`app/Controller/IndexController.php`
```php
public function post(RequestInterface $request): ResponseInterface {
    $inputs = $request->all();
    return $this->respond(["platform" => $inputs ?? []]);
}
```
路由：`config/routes/public/public.php`
```php
Router::post('/post', [IndexController::class, 'post']);
```
# getContents 与 (string) 的差异
## getContents()
```php
$contents = $response->getBody()->getContents();
```
`getContents()` 只从当前 stream 指针位置继续读取。如果指针已经在末尾，则返回空字符串。
## (string)$body
```php
$contents = (string)$response->getBody();
```
Guzzle PSR-7 Stream 的 `__toString()` 会先 `seek(0)`，再 `getContents()`：
```php
public function __toString(): string
{
    if ($this->isSeekable()) {
        $this->seek(0);
    }
    return $this->getContents();
}
```
所以 `(string)$body` 能读到完整内容。但注意：它读完后指针仍然停在末尾，不会恢复原位置。
# 为什么默认 Guzzle 不出问题
默认 Guzzle `CurlHandler` 流程：
1. cURL 完成传输。
2. 执行 `on_stats`，AOP 读取 body。
3. `CurlFactory::finish()` 自动 `$body->rewind()`。
4. Response 返回给调用方。
5. 调用方 `getContents()` 能从头读取。
# 为什么 CoroutineHandler 出问题
`CoroutineHandler` 流程：
1. 协程 HTTP 客户端完成传输。
2. 创建 `Psr7\Response`。
3. 执行 `on_stats`，AOP 读取 body。
4. 直接返回 Response。
5. 没有自动 rewind。
6. 调用方 `getContents()` 从末尾读取，得到空字符串。
# 修复建议
优先修复 AOP 的 `parseBody()`，不要让日志读取改变业务后续读取行为。
推荐恢复原 stream 指针位置：
```php
private static function parseBody(?MessageInterface $message): mixed
{
    if ($message === null) {
        return null;
    }
    $stream = $message->getBody();
    $position = null;
    if ($stream->isSeekable()) {
        $position = $stream->tell();
    }
    $body = (string)$stream;
    if ($position !== null) {
        $stream->seek($position);
    }
    $type = $message->getHeaderLine('Content-Type');
    if (str_contains($type, 'multipart')) {
        return str_replace("\r\n", ' %n% ', substr($body, 0, 256));
    }
    if (str_contains($type, 'json') || str_starts_with($body, '{')) {
        return json_decode($body, true) ?: $body;
    }
    return $body;
}
```
如果只考虑调用方总是从头读取，也可以读完后统一 rewind：
```php
$body = (string)$stream;
if ($stream->isSeekable()) {
    $stream->rewind();
}
```
但恢复原位置更安全，因为它不会改变调用方之前可能已经读取过一部分 body 的场景。
# 防复发建议
1. 给 `GuzzleHttpHandlerStackAspect::parseBody()` 增加单元测试，断言日志读取后 stream 指针位置不变。
2. 给 `RechargeHttpClient::send()` 增加回归测试，覆盖 `HandlerStack::create(new CoroutineHandler()) + getContents()`。
3. 禁止在日志/监控/AOP 中直接消费 PSR-7 Stream 后不恢复指针。
4. 对所有 `getBody()->getContents()` 调用点进行检查，确认上游不会提前消费 body。
5. 日志采集大 body 时需要限制长度，避免 AOP 日志读取大响应造成内存风险。
# 一句话总结
这次事故不是 Guzzle 本身的问题，而是全局 AOP 日志读取 response body 后没有恢复 stream 指针；默认 CurlHandler 帮忙兜底 rewind 了，但 Hyperf `CoroutineHandler` 没有这个兜底，导致 `RechargeHttpClient` 后续 `getContents()` 读到空响应。
