---
{"dg-publish":true,"permalink":"/Work/Tools/Common/Cloudflare DDNS/","title":"保姆教程 OpenWrt 配置 Cloudflare DDNS","tags":["clippings","ddns","openwrt","cloudflare"],"noteIcon":"","created":"2026-06-02T18:21:37.130+08:00","updated":"2026-06-02T18:27:29.922+08:00","dg-note-properties":{"title":"保姆教程 OpenWrt 配置 Cloudflare DDNS","source":"https://keesenz.com/2020/1067.html","author":["[[keesenz]]"],"tags":["clippings","ddns","openwrt","cloudflare"],"reference linking":null}}
---

# 背景
自从 dnspod 封禁了我的域名，之后注册的域名都由 Cloudflare 解析了，所以 OpenWrt DDNS 也首选 Cloudflare 了，当然前提是自己已经拿到了公网 IP。
# 获取 Cloudflare DDNS 的 OpenWrt 软件包
检查自己的动态 DNS 中 DDNS 的服务商中是否有 Cloudflare.com-v4
![](https://keesenz.com/wp-content/uploads/2020/09/Cloudflare-openwrt.png)
如果有直接进入下一步，如果没有，在软件包中搜索 Cloudflare.com-v4 可添加，如果搜索结果为空，添加软件源：
```text
src-git lienol https://github.com/Lienol/openwrt-package
```
# 配置 Cloudflare
增加一个 A 记录
![](https://keesenz.com/wp-content/uploads/2020/09/Cloudflare-dns.png)
获取 Cloudflare API Key（复制备用）
![](https://keesenz.com/wp-content/uploads/2020/09/Cloudflare-api-key-2048x558.png)
# 配置 OpenWrt 动态 DNS
![](https://keesenz.com/wp-content/uploads/2020/09/Cloudflare-ddns.png)
保存并启动即可，等几秒钟回到 Cloudflare 检查 IP 是否变成了自己公网 IP。
