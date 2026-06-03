---
{"dg-publish":true,"permalink":"/Work/Tools/Composer/Composer autoload vs provider/","title":"Composer autoload vs provider","tags":["flashcards","#review"],"noteIcon":"","created":"2026-04-12T22:49:17.000+08:00","updated":"2026-04-12T22:49:17.000+08:00","dg-note-properties":{"title":"Composer autoload vs provider","tags":["flashcards","#review"],"reference linking":null,"sr-due":"2026-04-13","sr-interval":1,"sr-ease":230}}
---

# 1. 第三方包的基本结构
以自定义的[GitHub - woweijun123/laravel-amqp: 适用于laravel的AMQP组件](https://github.com/woweijun123/laravel-amqp)包为例，其结构如下：
```shell
laravel-amqp/annotation-package/
├── src/
│   ├── Amqp/ # 自定义AMQP类
│   │   └── ...php
│   └── Providers/ # 服务提供者
│       └── ....php
├── composer.json
└── README.md
```
# 2. 配置 Composer 自动加载
在 `composer.json` 中配置自动加载规则，确保注解类可以被加载：
```json
{
    "name": "riven/laravel-amqp",
    "type": "library",
    "license": "MIT",
    "keywords": [
        "php",
        "laravel",
        "amqp"
    ],
    "description": "Laravel amqp",
    "autoload": {
        "psr-4": {
            "Riven\\": "src/" // 💡定义类路径映射，确保文件能被自动加载
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Test\\": "tests" // 💡定义类路径映射，确保文件能被自动加载「仅开发包」
        }
    },
    "require": {
        "php": ">=8.3",
        "laravel/framework": "^12.0",
        "php-amqplib/php-amqplib": "^3.7",
        "predis/predis": "^2.0",
        "ext-redis": "*"
    },
    "require-dev": {
        "phpunit/phpunit": "*",
        "friendsofphp/php-cs-fixer": "^3.0",
        "phpstan/phpstan": "^1.0"
    },
    "config": {
        "sort-packages": true
    },
    "scripts": {
        "test": "co-phpunit --prepend test/bootstrap.php -c phpunit.xml --colors=always",
        "analyze": "phpstan analyse --memory-limit 300M -l 0 ./src",
        "cs-fix": "php-cs-fixer fix $1"
    },
    "extra": {
        "laravel": {
	        // 💡告诉 `Laravel` 自动注册服务提供者，无需手动添加到 `config/app.php`。
	        // 运行 `composer dump-autoload` 生成自动加载文件。
            "providers": [
                "Riven\\Providers\\AmqpProvider"
            ]
        }
    }
}
```
# 3. 编写服务提供者
创建 `AmqpProvider.php`，用于扫描注解并处理逻辑
# 4. 在项目中中使用
在项目中安装包后，可以直接在控制器中使用注解：
```php
namespace App\Http\Controllers;

use Riven\Amqp\AmqpManager;

class UserController {
    public function store() {
        // 实例化 AmqpManager
        app(AmqpManager::class);
    }
}
```
# 5. 服务提供者自动注册的原理
1. 运行 `composer require riven/laravel-amqp` 时，`Laravel` 会自动读取 `composer.json` 中的 `extra.laravel.providers` 字段。
2. 将 `AmqpProvider` 注册到 `config/app.php` 的 `providers` 数组中，无需手动操作。
## bootstrap/cache 目录
在 Laravel 项目中主要用于存放框架生成的缓存文件，这些文件用于加速应用的启动和运行。
以下是该目录中常见文件的作用说明：
### packages.php
这个文件由 composer 生成，包括了所有已安装包的**服务提供者**和**服务别名**。
它允许 Laravel 快速加载服务提供者而无需每次解析 `composer.json`。
### services.php
这个文件是 Laravel 的服务容器缓存文件。
它包括所有注册的**服务提供者**及其**绑定**关系，有助于加快服务容器的解析速度。
### .gitignore
此文件确保 bootstrap/cache 中的所有内容都被 Git 忽略。
通常**不应该**将缓存文件提交到版本控制系统中，因为**它们是自动生成**的，并且可能**因环境不同而有所差异**。
## Laravel 源码中provider的加载机制
### 入口文件
`public/index.php:18`
```php
$app = require_once __DIR__.'/../bootstrap/app.php'; // 💡配置并引入新的Laravel应用实例
$app->handleRequest(Request::capture());
```
### App文件
`bootstrap/app.php:18`
```php
return Application::configure(basePath: dirname(__DIR__))
                  ->withRouting(...)
                  ->withMiddleware(...)
                  ->withExceptions(...)->create();
```
### `Application的__construct`装填`Provider`
`Application.php:223`
```php
public function __construct($basePath = null)
{
	if ($basePath) {
		$this->setBasePath($basePath);
	}
	$this->registerBaseBindings(); // 将基本绑定注册到容器中
	$this->registerBaseServiceProviders(); // 💡注册基础服务提供者
	$this->registerCoreContainerAliases();
	$this->registerLaravelCloudServices();
}
```
### 注册所有基础服务提供者
`Application.php:299`
```php
protected function registerBaseServiceProviders()
{
	$this->register(new EventServiceProvider($this));
	$this->register(new LogServiceProvider($this));
	$this->register(new ContextServiceProvider($this));
	$this->register(new RoutingServiceProvider($this));
}
```
### 将给定的提供者标记为已注册
`Application.php:974`
```php
protected function markAsRegistered($provider)
{
	$class = get_class($provider);
	$this->serviceProviders[$class] = $provider; // 💡所有提供者的容器
	$this->loadedProviders[$class] = true;
}
```
### `Application::configure`包括事件
`Application.php:243`
```php
public static function configure(?string $basePath = null)
{
	$basePath = match (true) {
		is_string($basePath) => $basePath,
		default => static::inferBasePath(),
	};

	return (new Configuration\ApplicationBuilder(new static($basePath)))
		->withKernels() // 💡包括Kernel
		->withEvents() // 💡包括事件
		->withCommands()
		->withProviders();
}
```
### 加载`http`的`kernel`并调用`handel`方法
`Application.php:1218`
```php
public function handleRequest(Request $request)
{
	$kernel = $this->make(HttpKernelContract::class); // 💡加载 HttpKetnel
	$response = $kernel->handle($request)->send();
	$kernel->terminate($request, $response);
}
```
### 发送请求到路由
`Kernel.php:144`
```php
public function handle($request)
{
	$this->requestStartedAt = Carbon::now();
	try {
		$request->enableHttpMethodParameterOverride();
		$response = $this->sendRequestThroughRouter($request); // 💡发送请求到路由
	} catch (Throwable $e) {
		$this->reportException($e);
		$response = $this->renderException($request, $e);
	}
	$this->app['events']->dispatch(
		new RequestHandled($request, $response)
	);
	return $response;
}
```
### 执行`bootstrap`方法
`Kernel.php:170`
```php
protected function sendRequestThroughRouter($request)
{
	$this->app->instance('request', $request);
	Facade::clearResolvedInstance('request');
	$this->bootstrap();
	return (new Pipeline($this->app))
		->send($request)
		->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
		->then($this->dispatchToRouter());
}

public function bootstrap()
{
	if (! $this->app->hasBeenBootstrapped()) {
		$this->app->bootstrapWith($this->bootstrappers()); // 💡 包括循环启动项
	}
}
```
### 读取框架启动项数组
`Kernel.php:43`
```php
protected $bootstrappers = [
	\Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
	\Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
	\Illuminate\Foundation\Bootstrap\HandleExceptions::class,
	\Illuminate\Foundation\Bootstrap\RegisterFacades::class,
	\Illuminate\Foundation\Bootstrap\RegisterProviders::class,
	\Illuminate\Foundation\Bootstrap\BootProviders::class,
];
```
### 循环框架启动项数组并调用
`Application.php:342`
```php
public function bootstrapWith(array $bootstrappers)
{
	$this->hasBeenBootstrapped = true;
	foreach ($bootstrappers as $bootstrapper) {
		$this['events']->dispatch('bootstrapping: '.$bootstrapper, [$this]);
		$this->make($bootstrapper)->bootstrap($this); // 💡解析类并执行
		$this['events']->dispatch('bootstrapped: '.$bootstrapper, [$this]);
	}
}
```
### 解析`RegisterFacades`类并执行
>debug条件：`$bootstrapper == 'Illuminate\Foundation\Bootstrap\RegisterFacades'`
#### 反射创建 `RegisterFacades` 类
`Container.php:1039`
```php
try {
	$reflected = new ReflectionClass($abstract);
} catch (ReflectionException) {
	return $abstract;
}
```
#### 开始执行`RegisterFacades`类的`bootstrap`方法
`RegisterFacades.php:18`
```php
public function bootstrap(Application $app)
{
	Facade::clearResolvedInstances();
	Facade::setFacadeApplication($app);
	AliasLoader::getInstance(array_merge(
		$app->make('config')->get('app.aliases', []),
		$app->make(PackageManifest::class)->aliases()
	))->register(); // 💡获取所有的别名类并自动加载
}
```
#### 获取当前的包清单：若缓存文件「packages.php、services.php」不存在则生成缓存文件
`PackageManifest.php:107`
```php
// $this->manifestPath = root/bootstrap/cache/packages.php
if (!is_file($this->manifestPath)) {
	$this->build();
}
```
#### 读取composer中定义的提供者
扫描文件`/composer/installed.json`，读取到包`riven/laravel-amqp`，并解析字段`extra`的`provider`

`PackageManifest.php:124`
```php
public function build()
{
	$packages = [];
	// 💡扫描/composer/installed.json文件
	if ($this->files->exists($path = $this->vendorPath.'/composer/installed.json')) {
		$installed = json_decode($this->files->get($path), true);

		$packages = $installed['packages'] ?? $installed;
	}

	$ignoreAll = in_array('*', $ignore = $this->packagesToIgnore());

	$this->write((new Collection($packages))->mapWithKeys(function ($package) {
		return [$this->format($package['name']) => $package['extra']['laravel'] ?? []];
	})->each(function ($configuration) use (&$ignore) {
		$ignore = array_merge($ignore, $configuration['dont-discover'] ?? []);
	})->reject(function ($configuration, $package) use ($ignore, $ignoreAll) {
		return $ignoreAll || in_array($package, $ignore);
	})->filter()->all());
}
```

`installed.json`
```json
{
	...
	"extra": {
		"laravel": {
			"providers": [
				"Riven\\Providers\\AmqpProvider"
			]
		}
	}
	...
}
```
#### 生成文件缓存
`/www/xwl/coplay/bootstrap/cache/packages.php`
`/www/xwl/coplay/bootstrap/cache/services.php`

`Filesystem.php:231`
```php
public function replace($path, $content, $mode = null)
{
	// If the path already exists and is a symlink, get the real path...
	clearstatcache(true, $path);
	$path = realpath($path) ?: $path;
	$tempPath = tempnam(dirname($path), basename($path));
	// Fix permissions of tempPath because `tempnam()` creates it with permissions set to 0600...
	if (! is_null($mode)) {
		chmod($tempPath, $mode);
	} else {
		chmod($tempPath, 0777 - umask());
	}
	file_put_contents($tempPath, $content); // 💡写入缓存文件
	rename($tempPath, $path);
}
```
#### 加载provider，合并`app.providers`和缓存文件`cache/services.php`里的provider
##### 获取`cache/services.php`文件
`Application.php:1284`
```php
public function getCachedServicesPath()
{
	return $this->normalizeCachePath('APP_SERVICES_CACHE', 'cache/services.php');
}
```
##### 包括`cache/services.php`文件
`ProviderRepository.php:91`
```php
public function loadManifest()
{
	//服务清单是一个包括每个服务的JSON表示的文件
	//应用程序提供的服务，以及它的提供者是否正在使用
	//延迟加载或应该在每个请求中加载。
	if ($this->files->exists($this->manifestPath)) {
		$manifest = $this->files->getRequire($this->manifestPath);

		if ($manifest) {
			return array_merge(['when' => []], $manifest);
		}
	}
}
```
##### 注入到容器
>debug条件：`$provider == 'Riven\Providers\AmqpProvider'`
- 解析类：`Application.php:893`
- 注册类：`Application.php:896`
```php
//如果给定的“provider”是一个字符串，我们将解析它，并传入
//应用程序实例自动为开发人员。这很简单
//一种更方便的方式来指定你的服务提供商类。
if (is_string($provider)) {  
    $provider = $this->resolveProvider($provider); // 💡解析类
}  
  
$provider->register(); // 💡注册类
```
### 解析`BootProviders`类并执行
`$this->make($bootstrapper)->bootstrap($this);`
>debug条件：`$bootstrapper == 'Illuminate\Foundation\Bootstrap\BootProviders'`
#### 反射创建 `BootProviders` 类
`Container.php:1039`
```php
try {
	$reflected = new ReflectionClass($abstract);
} catch (ReflectionException) {
	return $abstract;
}
```
#### 开始执行`BootProviders`类的`bootstrap`方法
`BootProviders.php:17`
```php
public function bootstrap(Application $app)
{
	$app->boot();
}
```
#### 启动前回调执行：在应用正式启动前，先执行 bootingCallbacks 中注册的所有回调函数
`Application.php:1129`
```php
$this->fireAppCallbacks($this->bootingCallbacks);
```
#### 服务提供者启动：遍历并启动所有注册的服务提供者
`Application.php:1132`
```php
array_walk($this->serviceProviders, function ($p) {
	$this->bootProvider($p);
});
```
##### 启动给定的服务提供者
`Application.php:1151`
>debug条件：`get_class($provider) == 'Riven\Providers\AmqpProvider'`
`BoundMethod.php:28`
```php
protected function bootProvider(ServiceProvider $provider)
{
	$provider->callBootingCallbacks();

	if (method_exists($provider, 'boot')) {
		$this->call([$provider, 'boot']); // 💡调用服务提供者boot方法「若存在」
	}

	$provider->callBootedCallbacks();
}
```
##### 调用给定的闭包/类
`BoundMethod.php:25`
```php
/**
 * 调用回调函数或方法
 * @param mixed $container 容器实例
 * @param mixed $callback 要调用的回调
 * @param array $parameters 传递给回调的参数
 * @param string|null $defaultMethod 默认方法名
 * @return mixed 回调执行结果
 */
public static function call($container, $callback, array $parameters = [], $defaultMethod = null)
{
    // 检查回调是否是字符串且存在 __invoke 方法
    // 如果是，则将 __invoke 设为默认方法
    if (is_string($callback) && ! $defaultMethod && method_exists($callback, '__invoke')) {
        $defaultMethod = '__invoke';
    }

    // 检查回调是否使用 @ 符号格式（如 "Class@method"）或指定了默认方法
    // 如果是，则调用 callClass 方法处理类方法调用
    if (static::isCallableWithAtSign($callback) || $defaultMethod) {
        return static::callClass($container, $callback, $parameters, $defaultMethod);
    }

    // 对于普通回调，调用 callBoundMethod 方法处理
    // 闭包中执行回调并注入依赖
    return static::callBoundMethod($container, $callback, function () use ($container, $callback, $parameters) {
        // 解析回调方法的依赖项，并执行回调
        return $callback(...array_values(static::getMethodDependencies($container, $callback, $parameters)));
    });
}
```
# 6. 对比 Laravel 内置注解扫描
| **特性**  | **第三方包实现**                        | **Laravel 内置机制**         |
| ------- | --------------------------------- | ------------------------ |
| 注解类自动加载 | 通过 Composer 的 `psr-4` 配置          | 通过 Composer 的 `psr-4` 配置 |
| 服务提供者注册 | 通过 `extra.laravel.providers` 自动注册 | 需手动添加到 `config/app.php`  |
| 注解扫描逻辑  | 由服务提供者通过反射实现                      | 无内置支持，需自行实现              |
| 处理逻辑封装  | 第三方包封装了注解的扫描和处理逻辑                 | 用户需自行实现                  |
# Hyperf 配置自动加载`Provider`
## composer.json
```php
{
    "name": "riven/hyperf-zego",
    "type": "library",
    "description": "Zego Server Assistant",
    "keywords": [
        "zego",
        "Server Assistant"
    ],
    "homepage": "https://www.zego.im",
    "license": "MIT",
    "require": {
        "php": ">=8.3",
        "hyperf/guzzle": "^3.1",
        "hyperf/config": "~3.1.0",
        "ext-openssl": "*"
    },
    "require-dev": {
        "hyperf/devtool": "~3.1.0",
        "hyperf/testing": "^3.1"
    },
    "autoload": {
        "psr-4": {
            "Zego\\": "src/",
            "Test\\": "test/"
        }
    },
    "extra": { // 💡 配置自动加载的路径
        "hyperf": {
            "config": "Zego\\ConfigProvider"
        }
    }
}

```
## 加载并合并所有组件的 Provider 配置
`ProviderConfig.php:37`
```php
/**
 * 加载并合并所有组件的 Provider 配置。
 * 注意：该方法会将配置结果缓存到静态属性中，
 * 如果想要重置该静态属性，请调用 ProviderConfig::clear() 方法。
 */
public static function load(): array
{
	if (! static::$providerConfigs) {
		// 💡从 composer.json 的 extra.hyperf.config 中获取所有的 Provider
		$providers = Composer::getMergedExtra('hyperf')['config'] ?? [];
		static::$providerConfigs = static::loadProviders($providers);
	}
	return static::$providerConfigs;
}
```
## 加载并合并所有 Provider 的配置
```php
/**
 * 加载并合并所有 Provider 的配置
 * @param array $providers Provider 类名列表
 * @return array 合并后的配置数组
 */
protected static function loadProviders(array $providers): array
{
	$providerConfigs = [];
	foreach ($providers as $provider) {
		// 💡确保 provider 是字符串、类存在且定义了 __invoke 方法
		if (is_string($provider) && class_exists($provider) && method_exists($provider, '__invoke')) {
			// 实例化 Provider 并调用其 __invoke 方法获取配置
			$providerConfigs[] = (new $provider())();
		}
	}

	// 使用 static::merge 方法将所有获取到的配置递归合并并返回
	return static::merge(...$providerConfigs);
}
```
## 将工厂定义（factory definition）解析为具体的实例或值。
`FactoryResolver.php:46`
```php
/**
 * 将工厂定义（factory definition）解析为具体的实例或值。
 * @param FactoryDefinition $definition 定义了如何获取该值的对象
 * @param array $parameters 用于构建该条目的可选参数
 * @return mixed 从定义中获取到的值
 * @throws InvalidDefinitionException 如果定义无法被解析，则抛出此异常
 */
public function resolve(DefinitionInterface $definition, array $parameters = [])
{
    $callable = null;
    try {
        // 获取定义的工厂回调（callable）
        $callable = $definition->getFactory();

        // 💡检查该对象或类是否包含 __invoke 方法（即可调用性检查）
        if (! method_exists($callable, '__invoke')) {
            throw new NotCallableException();
        }

        // 如果工厂是一个字符串（通常是容器中的类名或服务 ID），
        // 则从容器中获取该实例
        if (is_string($callable)) {
            $callable = $this->container->get($callable);
        }

        // 执行该工厂回调，并传入容器实例及参数
        return $callable($this->container, $parameters);

    } catch (NotCallableException $e) {
        // 提供自定义错误信息以辅助调试
        
        // 特殊情况处理：如果是字符串、类存在且有 __invoke 方法，但仍抛出异常
        // 说明可能是因为禁用了自动注入（autowiring），导致无法自动实例化该类
        if (is_string($callable) && class_exists($callable) && method_exists($callable, '__invoke')) {
            throw new InvalidDefinitionException(sprintf(
                '条目 "%s" 无法解析：工厂 %s。如果容器禁用了自动注入，则无法自动解析可调用的类。' . 
                '您需要启用自动注入或手动定义该条目。',
                $definition->getName(),
                $e->getMessage()
            ));
        }

        // 通用的无法解析异常
        throw new InvalidDefinitionException(sprintf(
            '条目 "%s" 无法解析：工厂 %s',
            $definition->getName(),
            $e->getMessage()
        ));
    }
}
```