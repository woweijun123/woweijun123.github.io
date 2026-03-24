---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Hyperf/Hyperf 自定义命令目录/","title":"Hyperf 自定义命令目录","tags":["flashcards"],"noteIcon":"","created":"2025-01-06T16:13:54.000+08:00","updated":"2026-03-24T17:32:03.827+08:00"}
---

自定义命令文件为: `hyperf/my/MyCommand.php`
## 1、composer.json文件中加入"Waoo\\": "waoo/"
编辑文件 `vim composer.php` 添加内容`"My\\": "my/"`如下
```json
...
    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Internal\\": "internal/",
            "My\\": "my/"
        },
        "files": [
        ]
    }
...
```
## 2、文件新增paths
编辑文件 `vim hyperf/config/autoload/annotations.php` 内容如下
```php
declare(strict_types=1);
use Internal\Invoke\CalleeCollector;
return [
    'scan' => [
        'paths' => [
            BASE_PATH . '/app',
            BASE_PATH . '/my', // 自定义命令目录
        ],
        'ignore_annotations' => [
            'mixin',
        ],
        'collectors'         => [
            CalleeCollector::class,
        ],
    ],
];
```

