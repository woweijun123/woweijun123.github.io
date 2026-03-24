---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel notice/","title":"Laravel notice","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:14.000+08:00","updated":"2026-03-24T17:34:05.722+08:00"}
---

# Laravel notice 
 文档
https://github.com/barryvdh/laravel-ide-helper
 安装
```shell
composer require --dev barryvdh/laravel-ide-helper
```
 将以下类添加到config/app.php中的providers数组中:
```php
Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class,
```
 命令
```shell
php artisan ide-helper:generate
```
现在这个命令应该会生成一个叫`_ide_helper.php`的文件，
现在phpstorm已经支持代码提示, 如果没有, 可以重启IDE一下试试
# 自定义公共函数的引入
[Laravel 自定义公共函数的引入\_laravel 公共函数引入-CSDN博客](https://blog.csdn.net/u011415782/article/details/78925048)
## 背景
> - 习惯了 使用 ThinkPHP 框架，有一个公共方法类在代码编写上会快捷很多，所以有必要在此进行配置一番.
> - 测试框架：Laravel 5.5
## 步骤指导
### 1. 创建 `functions.php`
- 在 `app/Helpers/(目录可以自己随便来)`下新建一个文件 `functions.php`,在内部补充如下代码：
```php
<?php
/**
 * Created by PhpStorm.
 * User: moTzxx
 * Date: 2017/12/28
 * Time: 17:47
 */

/**
 * 公用的方法  返回json数据，进行信息的提示
 * @param $status 状态
 * @param string $message 提示信息
 * @param array $data 返回数据
 */
function showMsg($status,$message = '',$data = array()){
    $result = array(
        'status' => $status,
        'message' =>$message,
        'data' =>$data
    );
    exit(json_encode($result));
}
```
### 2. 配置 `composer.json`
- 打开项目根目录下的 `composer.json` 文件，找到`"autoload"` 配置项，补充如下代码：
```json
"autoload": {
	...
	"files":[
		"app/Helper/functions.php"
	]
}
```
### 3. 执行 `composer` 命令
- 打开终端，项目目录执行命令：
```shell
composer dump-auto 
```
### 4\. 测试
- 在控制器的随意一个方法中执行下面代码，有数据输出则配置成功：
```php
showMsg(1,'Hello World!');
```
> ★ 举一反三，以后的公共函数都可写在 `functions.php` 中 …

# laravel-cors跨域解决方案
**1. 安装 `laravel-cors` 包**
使用 Composer 安装 `laravel-cors` 包：
```
composer require fruitcake/laravel-cors
```
**2. 注册中间件**
在 `app/Http/Kernel.php` 文件中，注册 `HandleCors` 中间件。
- 全局注册（推荐）：
```php
protected $middleware = [
	// ...
	\Fruitcake\Cors\HandleCors::class,
];
```
- 路由组注册：
```php
protected $middlewareGroups = [
	'api' => [
		\Fruitcake\Cors\HandleCors::class,
		// ...
	],
];
```
- 路由注册：
```php
protected $routeMiddleware = [
	'cors' => \Fruitcake\Cors\HandleCors::class,
	// ...
];
```

**3. 配置 CORS**
发布配置文件：
```shell
php artisan vendor:publish --provider="Fruitcake\Cors\CorsServiceProvider"
```

编辑 `config/cors.php` 文件，根据您的需求进行配置。以下是一些常用的配置选项：
```php
'paths' => ['api/*', 'sanctum/csrf-cookie'], // 允许跨域的路径
'allowed_methods' => ['*'], // 允许的 HTTP 方法
'allowed_origins' => ['*'], // 允许的来源
'allowed_origins_patterns' => [], // 允许的来源模式
'allowed_headers' => ['*'], // 允许的请求头
'exposed_headers' => [], // 允许暴露的响应头
'max_age' => 0, // 预检请求的缓存时间
'supports_credentials' => false, // 是否允许发送凭证
```
- `paths`：指定哪些路径需要启用 CORS。可以使用通配符 `*`。
- `allowed_methods`：指定允许的 HTTP 方法，如 `['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS']`。
- `allowed_origins`：指定允许的来源，如 `['http://example.com', 'https://example.org']`。使用 `['*']` 允许所有来源（不推荐用于生产环境）。
- `allowed_headers`：指定允许的请求头，如 `['Content-Type', 'Authorization']`。

**4. 路由中使用中间件（可选）**
如果您在 `routeMiddleware` 中注册了中间件，可以在路由中使用它：
```php
Route::middleware('cors')->get('/api/data', function () {
	// ...
});
```
**示例：允许所有来源的跨域请求**
```php
// config/cors.php
'allowed_origins' => ['*'],
```

**示例：允许特定来源的跨域请求**
```php
// config/cors.php
'allowed_origins' => ['http://localhost:3000'],
```

**示例：允许特定 HTTP 方法的跨域请求**
```php
// config/cors.php
'allowed_methods' => ['GET', 'POST', 'OPTIONS'],
```

**示例：允许特定请求头的跨域请求**
```php
// config/cors.php
'allowed_headers' => ['Content-Type', 'Authorization'],
```
**注意事项：**
- 在生产环境中，请务必将 `allowed_origins` 设置为特定的来源，避免安全风险。
- 如果您的 API 需要发送凭证（如 Cookie），请将 `supports_credentials` 设置为 `true`，并在前端请求中设置 `withCredentials: true`。
- 如果遇到 CORS 相关问题，请检查浏览器的开发者工具控制台，查看错误信息。

# save更新失败原因
```php
// $collection = ArticleModel::query()->where('id', 3)->select(['status'])->first(); // 错误写法
$collection = ArticleModel::query()->where('id', 3)->select(['id', 'status'])->first(); // 正确写法, 必须查询出ID否则不能更新status字段
$collection->status = 2;
$collection->save();
```

