---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Network/HTTP/","title":"HTTP","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:24:34.000+08:00","updated":"2026-03-24T17:35:37.822+08:00","dg-note-properties":{"title":"HTTP","tags":["flashcards"],"reference linking":null}}
---

# http_response_code
获取或设置HTTP响应代码

#1 如果提供了 response_code，将返回先前的状态码。 如果未提供 response_code，会返回当前的状态码。 在 Web 服务器环境里，这些状态码的默认值都是 200。
#2 如果在非 Web 服务器环境里调用(比如 CLI 应用里)， 不提供 response_code 就会返回FALSE 。 在非Web服务器环境里，提供 response_code 会返回 TRUE(仅仅在先前没有设置过状态码的时候)
```php
// #1 Web 服务器环境内使用 http_response_code()
// 设置新的状态码
var_dump(http_response_code(404));//输出结果: int(404)
// 获取当前状态码
var_dump(http_response_code());//输出结果: int(200)

// #2 在 CLI 环境内使用 http_response_code()
// 获取当前默认的响应状态码 
var_dump(http_response_code());//输出结果: bool(false)
// 设置状态码
var_dump(http_response_code(201));//输出结果: bool(true)
// 获取当前状态码
var_dump(http_response_code());//输出结果: int(201)
```
# http_build_query
生成URL-encode之后的请求字符串
```php
echo http_build_query([
    'foo' => 'bar',
    'baz' => 'boom',
    'cow' => 'milk',
    'php' => 'hypertext processor'
]); // foo=bar&baz=boom&cow=milk&php=hypertext+processor
```

