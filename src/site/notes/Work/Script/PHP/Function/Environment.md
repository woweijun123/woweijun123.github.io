---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Environment/","title":"Environment","tags":["flashcards"],"noteIcon":"","created":"2026-03-04T11:12:29.000+08:00","updated":"2026-03-24T17:38:08.157+08:00","dg-note-properties":{"title":"Environment","tags":["flashcards"],"reference linking":null}}
---

# getenv()
取得系统的环境变量(不支持IIS的isapi方式运行的PHP)
```php
getenv('REMOTE_ADDR'); // 172.19.0.1

// 同样可以获取到客户端的IP地址, 支持IIS的isapi方式运行的PHP
$_SERVER['REMOTE_ADDR']; // 172.19.0.1
```

