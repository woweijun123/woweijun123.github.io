---
{"dg-publish":true,"permalink":"/Work/Tools/Common/HTTP 测试「httpbin-org」/","title":"httpbin.org 测试用例","tags":["httpbin","HTTP","API测试"],"noteIcon":"","created":"2026-06-02T14:29:31.998+08:00","updated":"2026-06-03T11:43:29.013+08:00","dg-note-properties":{"title":"httpbin.org 测试用例","tags":["httpbin","HTTP","API测试"],"reference linking":null}}
---

# 基础信息
httpbin.org 是一个简单但非常实用的 HTTP 请求 & 响应测试服务，由 Kenneth Reitz 开发，提供丰富的 API 端点，可测试认证、请求头、状态码、编码等场景。
- 官方网站: https://httpbin.org
- GitHub: https://github.com/requests/httpbin
- 本地部署: `docker run -p 80:80 kennethreitz/httpbin`
# HTTP 方法测试
## GET
`GET https://httpbin.org/get` — 返回请求的查询参数。
```bash
curl "https://httpbin.org/get?name=test&age=25"
curl -H "X-Custom-Header: test-value" https://httpbin.org/get
```
## POST
`POST https://httpbin.org/post` — 返回请求体内容。
```bash
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"name": "test", "age": 25}'
curl -X POST https://httpbin.org/post \
  -d "username=admin&password=123456"
```
## PUT / PATCH / DELETE
```bash
curl -X PUT https://httpbin.org/put \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "name": "updated"}'
curl -X PATCH https://httpbin.org/patch \
  -H "Content-Type: application/json" \
  -d '{"id": 1}'
curl -X DELETE "https://httpbin.org/delete?id=123"
```
## TRACE
`TRACE https://httpbin.org/anything`
```bash
curl -X TRACE https://httpbin.org/anything
```
# 认证测试
## Basic Auth
`GET https://httpbin.org/basic-auth/{user}/{passwd}`
```bash
curl -u admin:password123 https://httpbin.org/basic-auth/admin/password123
curl -u wrong:credentials https://httpbin.org/basic-auth/admin/password123  # 401
```
## Bearer Token
`GET https://httpbin.org/bearer` — Authorization 头需提供 Bearer token。
```bash
curl -H "Authorization: Bearer my-secret-token" https://httpbin.org/bearer
```
## Digest Auth
`GET https://httpbin.org/digest-auth/{qop}/{user}/{passwd}` — qop 可选 `auth` 或 `auth-int`。
```bash
curl --digest -u admin:password123 \
  https://httpbin.org/digest-auth/auth/admin/password123
```
## Hidden Basic Auth
`GET https://httpbin.org/hidden-basic-auth/{user}/{passwd}` — 认证失败返回 404 而非 401。
```bash
curl -u wrong:credentials https://httpbin.org/hidden-basic-auth/admin/password123
```
# 请求检查
```bash
curl -H "Accept: application/json" \
     -H "X-Request-ID: abc-123" \
     -H "User-Agent: MyApp/1.0" \
     https://httpbin.org/headers
curl https://httpbin.org/ip
curl https://httpbin.org/user-agent
```
# 响应检查
```bash
curl "https://httpbin.org/response-headers?Content-Type=text/plain&X-Custom=test"
curl https://httpbin.org/cache/60
curl -H "If-None-Match: *" https://httpbin.org/cache  # 304 Not Modified
curl https://httpbin.org/etag/abc123
curl -H "If-None-Match: abc123" https://httpbin.org/etag/abc123  # 304
```
# 响应格式
```bash
curl https://httpbin.org/json
curl https://httpbin.org/xml
curl https://httpbin.org/html
curl https://httpbin.org/encoding/utf8
curl https://httpbin.org/image          # 根据 Accept 头返回
curl https://httpbin.org/image/jpeg
curl https://httpbin.org/image/png
curl https://httpbin.org/image/svg
curl https://httpbin.org/image/webp
curl https://httpbin.org/gzip
curl https://httpbin.org/deflate
curl https://httpbin.org/brotli
```
# 状态码测试
`GET https://httpbin.org/status/{codes}` — 返回指定状态码，逗号分隔可随机。
```bash
curl https://httpbin.org/status/200
curl https://httpbin.org/status/201
curl https://httpbin.org/status/204
curl https://httpbin.org/status/301
curl https://httpbin.org/status/302
curl https://httpbin.org/status/304
curl https://httpbin.org/status/400
curl https://httpbin.org/status/401
curl https://httpbin.org/status/403
curl https://httpbin.org/status/404
curl https://httpbin.org/status/422
curl https://httpbin.org/status/429
curl https://httpbin.org/status/500
curl https://httpbin.org/status/502
curl https://httpbin.org/status/503
curl https://httpbin.org/status/200,400,500
```
# 重定向测试
```bash
curl -L https://httpbin.org/redirect/5
curl -L https://httpbin.org/relative-redirect/5
curl -L https://httpbin.org/absolute-redirect/5
curl -L "https://httpbin.org/redirect-to?url=https://httpbin.org/get"
curl -L "https://httpbin.org/redirect-to?url=https://httpbin.org/get&status_code=301"
```
# Cookie 测试
```bash
curl -c cookies.txt "https://httpbin.org/cookies/set/token=abc123/session=true"
curl -b cookies.txt https://httpbin.org/cookies
curl -b cookies.txt -c cookies.txt "https://httpbin.org/cookies/delete?token"
```
# 动态数据
```bash
curl https://httpbin.org/uuid
curl https://httpbin.org/bytes/1024
curl https://httpbin.org/stream/5
curl https://httpbin.org/stream-bytes/1024
curl https://httpbin.org/range/1024
curl https://httpbin.org/base64/aHR0cGJpbiBpcyBhd2Vzb21l  # 解码 "httpbin is awesome"
curl https://httpbin.org/links/10/0
```
# 延迟与流控制
`/delay/{seconds}` 最大 10 秒；`/drip` 参数：`duration`（秒）、`numbytes`、`code`、`delay`。
```bash
curl https://httpbin.org/delay/3
curl --max-time 2 https://httpbin.org/delay/5
curl "https://httpbin.org/drip?duration=2&numbytes=100&delay=1"
```
# Anything 端点
`GET/POST/PUT/PATCH/DELETE/TRACE https://httpbin.org/anything[/{anything}]` — 返回完整请求信息。
```bash
curl -X POST https://httpbin.org/anything \
  -H "Content-Type: application/json" \
  -d '{"test": true}' \
  -H "X-Custom: value"
```
# 其他端点
```bash
curl https://httpbin.org/robots.txt
curl https://httpbin.org/deny
```
# 端点分类速查
| 分类 | 端点 | 说明 |
|------|------|------|
| HTTP 方法 | `/get` `/post` `/put` `/patch` `/delete` | 测试各种 HTTP 方法 |
| 认证 | `/basic-auth/{user}/{passwd}` | Basic 认证 |
| | `/bearer` | Bearer Token |
| | `/digest-auth/{qop}/{user}/{passwd}` | Digest 认证 |
| | `/hidden-basic-auth/{user}/{passwd}` | 失败返回 404 |
| 请求检查 | `/headers` `/ip` `/user-agent` | 请求头 / IP / UA |
| 响应检查 | `/response-headers` `/cache/{s}` `/etag/{etag}` | 响应头 / 缓存 / ETag |
| 响应格式 | `/json` `/xml` `/html` `/encoding/utf8` | 各格式响应 |
| | `/image` `/image/jpeg` 等 | 图片响应 |
| | `/gzip` `/deflate` `/brotli` | 压缩编码 |
| 状态码 | `/status/{codes}` | 指定或随机状态码 |
| 重定向 | `/redirect/{n}` `/relative-redirect/{n}` `/absolute-redirect/{n}` | 链式 / 相对 / 绝对 |
| | `/redirect-to?url=&status_code=` | 重定向到指定 URL |
| Cookie | `/cookies` `/cookies/set` `/cookies/delete` | Cookie 管理 |
| 动态数据 | `/uuid` `/bytes/{n}` `/stream/{n}` `/stream-bytes/{n}` | 动态生成数据 |
| | `/range/{n}` `/base64/{v}` `/links/{n}/{offset}` | Range / Base64 / 链接页 |
| 延迟 | `/delay/{s}` `/drip` | 延迟与流控制 |
| 通用 | `/anything` | 返回完整请求信息 |
# 实际应用场景
## 测试 HTTP 客户端库
```python
import requests

resp = requests.get("https://httpbin.org/get", params={"key": "value"})
assert resp.status_code == 200

resp = requests.post("https://httpbin.org/post", json={"name": "test"})
assert resp.json()["json"]["name"] == "test"

resp = requests.get("https://httpbin.org/basic-auth/user/pass", auth=("user", "pass"))
assert resp.status_code == 200
```
## 测试超时处理
```python
import requests
from requests.exceptions import Timeout

try:
    requests.get("https://httpbin.org/delay/5", timeout=2)
except Timeout:
    print("请求超时")
```
## 测试重定向
```python
import requests

resp = requests.get("https://httpbin.org/redirect/3", allow_redirects=True)
print(f"最终 URL: {resp.url}")

resp = requests.get("https://httpbin.org/redirect/3", allow_redirects=False)
print(f"状态码: {resp.status_code}")
```
## 测试 Cookie
```python
import requests

s = requests.Session()
s.get("https://httpbin.org/cookies/set/token=abc123")
resp = s.get("https://httpbin.org/cookies")
assert resp.json()["cookies"]["token"] == "abc123"
```
## 测试状态码处理
```python
import requests

for code in [200, 201, 204, 400, 401, 403, 404, 500, 502, 503]:
    resp = requests.get(f"https://httpbin.org/status/{code}")
    print(f"{code}: {resp.status_code}")
```
## 测试请求头
```python
import requests

headers = {
    "X-Request-ID": "12345",
    "Accept": "application/json",
    "Authorization": "Bearer test-token"
}
resp = requests.get("https://httpbin.org/headers", headers=headers)
print(resp.json()["headers"])
```
# 注意事项
- **服务稳定性**: httpbin.org 是免费公共服务，请勿进行高并发压测
- **延迟上限**: `/delay` 端点最大延迟为 10 秒
- **数据安全**: 不要在 httpbin.org 上发送敏感数据（密码、token 等）
- **本地部署**: 生产环境建议使用 Docker 本地部署
