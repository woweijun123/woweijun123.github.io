---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel 服务提供者检测注解逻辑/","title":"Laravel 服务提供者检测注解逻辑","tags":["flashcards"],"noteIcon":"","created":"2025-05-18T11:57:23.814+08:00","updated":"2026-03-24T17:33:50.687+08:00"}
---

[riven/laravel-amqp - Packagist](https://packagist.org/packages/riven/laravel-amqp)
## 扫描变更加载缓存
```php
namespace App\Providers;

use App\Annotation\Impl;
use Exception;
use FilesystemIterator;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\ServiceProvider;
use Psr\SimpleCache\InvalidArgumentException;
use RecursiveDirectoryIterator;
use RecursiveIteratorIterator;
use ReflectionClass;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;

class ImplServiceProvider extends ServiceProvider
{
    /**
     * @throws InvalidArgumentException
     */
    public function register(): void
    {
        $this->bindImplClasses();
    }

    /**
     * @throws InvalidArgumentException
     */
    protected function bindImplClasses(): void
    {
        $cache = $this->getCache();
        $cacheKey = 'impl_bindings';
        $cacheTtl = 3600; // 缓存 1 小时

        // 尝试从缓存中获取数据
        $cached = $cache->get($cacheKey);
        $bindings = [];
        if ($cached) {
            $cachedData = json_decode($cached, true);
            $lastModifiedFiles = $cachedData['last_modified_files'] ?? [];
            $bindings = $cachedData['bindings'] ?? [];

            // 检查文件修改时间是否一致
            $currentLastModified = $this->getLastModifiedTimes();
            if ($this->isCacheValid($lastModifiedFiles, $currentLastModified)) {
                foreach ($bindings as $interface => $class) {
                    $this->app->singletonIf($interface, $class); // 绑定类「默认单例」
                }
                return;
            }
        }

        // 重新生成绑定数据
        $targetDirs = [app_path('Services')];
        $files = [];

        foreach ($targetDirs as $dir) {
            $files = array_merge($files, $this->getPhpFilesInDir($dir));
        }

        // 处理类
        foreach ($files as $file) {
            $className = $this->getClassFromFilePath($file);
            if (!class_exists($className)) {
                continue;
            }

            $reflection = new ReflectionClass($className);
            $attributes = $reflection->getAttributes(Impl::class);

            foreach ($attributes as $attribute) {
                $impl = $attribute->newInstance();

                // 绑定接口到类
                $bindKey = $impl->key ? "$impl->interface@$impl->key" : $impl->interface;
                if (isset($bindings[$bindKey])) {
                    Log::warning("接口 [{$impl->interface}] 的键 [{$impl->key}] 已绑定到其他类，将被覆盖。类：[{$className}]");
                }

                $bindings[$bindKey] = $className;
            }
        }

        // 获取当前文件修改时间
        $currentLastModified = $this->getLastModifiedTimes();

        // 写入缓存
        $cacheData = [
            'last_modified_files' => $currentLastModified,
            'bindings' => $bindings,
        ];
        $cache->put($cacheKey, json_encode($cacheData), $cacheTtl);

        // 绑定到容器
        foreach ($bindings as $interface => $class) {
            $this->app->singletonIf($interface, $class);
        }
    }

    /**
     * 获取缓存实例（优先 Redis，失败则降级为 file）
     */
    private function getCache(): Repository
    {
        try {
            return Cache::store('redis');
        } catch (Exception $e) {
            Log::warning('Redis 缓存不可用，已降级为文件缓存', ['exception' => $e]);
            return Cache::store('file');
        }
    }

    /**
     * 获取指定目录下的所有 PHP 文件
     */
    protected function getPhpFilesInDir(string $dir): array
    {
        $files = [];
        $iterator = new RecursiveIteratorIterator(
            new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS)
        );

        foreach ($iterator as $file) {
            if ($file->isFile() && $file->getExtension() === 'php') {
                $files[] = $file->getPathname();
            }
        }

        return $files;
    }

    /**
     * 根据文件路径获取类名
     */
    protected function getClassFromFilePath(string $file): string
    {
        $appPath = app_path();
        if (strpos($file, $appPath) !== 0) {
            throw new \InvalidArgumentException("文件 [$file] 不在 app 目录下，无法解析类名。");
        }

        $relativePath = substr($file, strlen($appPath) + 1);
        $relativePath = str_replace('.php', '', $relativePath);
        $className = str_replace('/', '\\', $relativePath);

        return "App\\$className";
    }

    /**
     * 获取所有目标目录下 PHP 文件的最后修改时间
     */
    protected function getLastModifiedTimes(): array
    {
        $targetDirs = [app_path('Services')];
        $lastModified = [];

        foreach ($targetDirs as $dir) {
            $files = $this->getPhpFilesInDir($dir);
            foreach ($files as $file) {
                $lastModified[$file] = filemtime($file);
            }
        }

        return $lastModified;
    }

    /**
     * 检查缓存是否有效（文件修改时间是否一致）
     */
    protected function isCacheValid(array $cachedLastModified, array $currentLastModified): bool
    {
        // 检查所有文件的修改时间是否一致
        foreach ($cachedLastModified as $file => $mtime) {
            if (!isset($currentLastModified[$file]) || $currentLastModified[$file] > $mtime) {
                return false;
            }
        }

        // 检查是否有新文件在缓存中未记录
        foreach ($currentLastModified as $file => $mtime) {
            if (!isset($cachedLastModified[$file])) {
                return false;
            }
        }

        return true;
    }
}

```

## 开发环境直接扫描，生产环境使用缓存
```php
namespace App\Providers;

use App\Annotation\Impl;
use App\Helper\Invoke\Annotation\Callee;
use App\Helper\Invoke\CalleeCollector;
use Exception;
use FilesystemIterator;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\ServiceProvider;
use Psr\SimpleCache\InvalidArgumentException;
use RecursiveDirectoryIterator;
use RecursiveIteratorIterator;
use ReflectionClass;

class ImplServiceProvider extends ServiceProvider
{
    // 接口实现注解
    const string IMPL = 'impl';
    // 回调注解
    const string Callee = 'callee';

    /**
     * 启动服务提供者
     */
    public function boot(): void
    {
        // 调用绑定实现类的方法
        $this->bindImplClasses();
        // 注册 Callee 收集逻辑（注解方法）
        $this->registerCalleeMethods();
    }

    /**
     * 绑定实现类到 Laravel 服务容器
     * @return void
     */
    protected function bindImplClasses(): void
    {
        if (app()->environment('local')) {
            // 开发环境：直接扫描指定目录下的类并绑定，不使用缓存，方便开发调试
            $bindings = $this->discoverImplBindings();
        } else {
            // 生产环境：尝试从缓存中获取绑定信息，如果缓存不存在或读取失败，则扫描并写入缓存
            $bindings = $this->getCachedBindings();
        }
        // 遍历获取到的绑定信息，并将接口绑定到对应的实现类（如果尚未绑定）
        foreach ($bindings as $interface => $class) {
            $this->app->singletonIf($interface, $class); // 使用 singletonIf 确保每个接口只会被实例化一次（单例模式）
        }
    }

    /**
     * 收集 Callee 注解方法到 CalleeCollector
     */
    protected function registerCalleeMethods(): void
    {
        if (app()->environment('local')) {
            // 开发环境：实时扫描
            $calleeMethods = $this->discoverCalleeMethods();
        } else {
            // 生产环境：使用缓存
            $calleeMethods = $this->getCachedBindings(self::Callee);
        }
        // 注册方法到 CalleeCollector
        foreach ($calleeMethods as $callable) {
            CalleeCollector::addCallee(...$callable);
        }
    }

    /**
     * 扫描并发现 Callee 注解方法
     */
    protected function discoverCalleeMethods(): array
    {
        $targetDirs = [app_path('Services')];
        $files      = [];

        foreach ($targetDirs as $dir) {
            $files = array_merge($files, $this->getPhpFilesInDir($dir));
        }

        $calleeMethods = [];
        foreach ($files as $file) {
            $className = $this->getClassFromFilePath($file);
            if (!class_exists($className)) {
                continue;
            }

            $reflection = new ReflectionClass($className);
            $methods    = $reflection->getMethods();

            foreach ($methods as $method) {
                $attributes = $method->getAttributes(Callee::class);
                foreach ($attributes as $attribute) {
                    $callee          = $attribute->newInstance();
                    $event           = $callee->event;
                    $scope           = $callee->scope;
                    $calleeMethods[] = [[$className, $method->getName()], $event, $scope];
                }
            }
        }

        return $calleeMethods;
    }

    /**
     * 扫描指定目录下的类，查找带有 Impl 注解的类，并将其实现的接口绑定到自身
     * @return array 接口到实现类的绑定数组
     */
    private function discoverImplBindings(): array
    {
        $targetDirs = [app_path('Services')]; // 定义需要扫描的目录
        $files      = [];                     // 初始化文件数组

        // 遍历目标目录，获取所有 PHP 文件
        foreach ($targetDirs as $dir) {
            $files = array_merge($files, $this->getPhpFilesInDir($dir));
        }

        $bindings = []; // 初始化绑定数组
        // 遍历所有 PHP 文件
        foreach ($files as $file) {
            $className = $this->getClassFromFilePath($file); // 根据文件路径获取完整的类名
            // 检查类是否存在
            if (!class_exists($className)) {
                continue; // 如果类不存在，则跳过
            }

            $reflection = new ReflectionClass($className);         // 创建反射类
            $attributes = $reflection->getAttributes(Impl::class); // 获取类上所有 Impl 注解
            // 如果类上存在 Impl 注解
            if ($attributes) {
                $interfaceNames = $reflection->getInterfaceNames(); // 获取当前类实现的所有接口名称
                // 遍历实现的接口
                foreach ($interfaceNames as $interfaceName) {
                    $bindings[$interfaceName] = $className; // 将接口名称作为 key，类名作为 value 存储到绑定数组中
                }
            }
        }

        return $bindings; // 返回接口到实现类的绑定数组
    }

    /**
     * 生产环境：从缓存中获取绑定信息（优先使用 Redis 缓存，如果 Redis 不可用则降级为文件缓存）
     * @return array 接口到实现类的绑定数组
     */
    private function getCachedBindings(string $type = self::IMPL): array
    {
        try {
            // 尝试从缓存中获取数据
            $cached = $this->getCache()->get($type);
            if ($cached) {
                // 返回缓存中的绑定信息，如果不存在则返回空数组
                return $cached;
            }
        } catch (InvalidArgumentException $e) {
            // 缓存读取失败（可能是 Redis 连接问题等），记录警告日志并继续尝试扫描
            Log::warning('缓存读取失败，已降级为文件缓存,  key=' . self::IMPL, ['exception' => $e]);
        }
        // 缓存不存在或读取失败时，强制扫描指定目录下的类并生成新地绑定信息
        $bindings = $type === self::IMPL ? $this->discoverImplBindings() : $this->discoverCalleeMethods();
        // 将新地绑定信息永久存储到缓存中
        $this->getCache()->forever($type, $bindings);
        return $bindings; // 返回新生成的绑定数组
    }

    /**
     * 获取缓存实例（优先使用 Redis 缓存，如果 Redis 连接失败则降级为文件缓存）
     * @return Repository 缓存仓库实例
     */
    private function getCache(): Repository
    {
        try {
            // 尝试获取 Redis 缓存存储实例
            return Cache::store('redis');
        } catch (Exception $e) {
            // 如果获取 Redis 缓存实例失败（例如 Redis 服务未运行），记录警告日志并返回文件缓存存储实例
            Log::warning('Redis 缓存不可用，已降级为文件缓存', ['exception' => $e]);

            return Cache::store('file');
        }
    }

    /**
     * 获取指定目录下的所有 PHP 文件
     * @param string $dir 目录路径
     * @return array 包含所有 PHP 文件路径的数组
     */
    protected function getPhpFilesInDir(string $dir): array
    {
        $files = []; // 初始化文件路径数组
        // 递归遍历指定目录，跳过 . 和 .. 目录
        $iterator = new RecursiveIteratorIterator(
            new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS)
        );

        // 遍历迭代器
        foreach ($iterator as $file) {
            // 如果是文件且扩展名为 .php
            if ($file->isFile() && $file->getExtension() === 'php') {
                $files[] = $file->getPathname(); // 将文件完整路径添加到数组
            }
        }

        return $files; // 返回包含所有 PHP 文件路径的数组
    }

    /**
     * 根据文件路径获取完整的类名
     * @param string $file 文件路径
     * @return string 完整的类名
     * @throws \InvalidArgumentException 如果文件不在 app 目录下
     */
    protected function getClassFromFilePath(string $file): string
    {
        $appPath = app_path(); // 获取 app 目录的路径
        // 检查文件路径是否以 app 目录开头
        if (!str_starts_with($file, $appPath)) {
            throw new \InvalidArgumentException("文件 [$file] 不在 app 目录下，无法解析类名。");
        }

        // 获取相对于 app 目录的路径
        $relativePath = substr($file, strlen($appPath) + 1);
        // 移除 .php 扩展名
        $relativePath = str_replace('.php', '', $relativePath);
        // 将路径分隔符 / 替换为命名空间分隔符 \
        $className = str_replace('/', '\\', $relativePath);

        // 返回完整的类名，加上根命名空间 App
        return "App\\$className";
    }
}
```