---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel 中实现 SpringBoot 中的 RequestBody 注解  Laravel China 社区/","title":"Laravel 中实现 SpringBoot 中的 RequestBody 注解  Laravel China 社区","tags":["flashcards"],"noteIcon":"","created":"2025-05-18T10:03:20.444+08:00","updated":"2026-03-24T17:33:57.528+08:00","dg-note-properties":{"title":"Laravel 中实现 SpringBoot 中的 RequestBody 注解  Laravel China 社区","tags":["flashcards"],"reference linking":null}}
---

## Laravel 中实现 SpringBoot 中的 RequestBody 注解
如果你使用过 Java 的 SpringBoot 框架，会不会感觉到当中某些注解很好用呢？比如说 `@GetMapping` 、 `@PostMapping` 之类的路由注解， `@Autowired` 可以通过属性注入依赖。
当然，还有这篇文章的主角 `@RequestBody` 注解，它可以将客户端请求中 `application/json` 格式的参数转换为 Java 中的 DTO（Data Transfer Object）。
## PHP 中的示例
这里就不贴 Java 的代码了，来看 PHP 中的实例:
通过 `#[RequestBody]` 注解，可以自动将前端的请求参数写入到 `RegisterParams` 这个类中，然后在代码中通过这个类暴露出来的 `Getter` 方法来调用。这样的写法有如下这些好处:
1. **改变传统上我们对 PHP 数组的严重依赖** ，虽然 PHP 中的数组非常强大，但是在 IDE 提示是不足的，且单词拼写错误也难以发觉，例如我曾将 `build` 拼写成了 `built` ，这样的代码只有在运行时、测试中，甚至上线后才能发现错误。
2. **让我们写代码更加流程** ，是的，这是我写 Java 代码最大的一个感受，流畅！IDE 对代码的提示的非常精准，基本上是一路回车来完成代码。但是如果你依赖 PHP 中的数组，那么写代码会经常性停顿，去回忆、去猜下一个 `Key` 是什么？

得益于 PHP8 中提供的参数注解，实现 `#[RequestBody]` 有了可能。但要想在 Laravel 中实际实现，还是需要动动脑经的。请看下文分解这个注解的实现过程:
1. 创建路由中间件，拦截请求
2. 在中间件中解析请求的控制器方法中的 `#[RequestBody]` 注解
3. 将 `request()->all()` 中获取到的参数通过反射写入到其修饰的参数的类型中（DTO 对象）
4. 将 DTO 对象注入到 Laravel 的容器中
5. 当执行到控制器的方法时，Laravel 通过从容器中获取我们事先注入的 DTO 对象注入控制器方法的参数中。
## 实现 `#[RequestBody]` 注解
实现 `#[RequestBody]` 注解依赖 Laravel 的路由中间件，中间件的基础代码如下:
```php
class RequestBodyInjector
{
    public function handle(Request $request, Closure $next): mixed
    {
        $route = $request->route();
        if ($route === null) {
            return $next($request);
        }
        $controller = $route->getController();
        $action = $route->getActionMethod();

	    // 通过反射获取控制器方法中的参数
	    $reflectionMethod = new ReflectionMethod($controller, $action);
	    $parameters = $reflectionMethod->getParameters();
	    // 遍历参数
	    foreach ($parameters as $parameter) {
	        // 检查是否存在 #[RequestBody] 的注解
	        $attributes = collect($parameter->getAttributes(RequestBody::class));
	        if ($attributes->isEmpty()) {
	            continue;
	        }
	        // 存在则获取参数写入 DTO 对象
	        $this->addBodyToRequest($request, $parameter);
	    }
	    return $next($request);
    }

	private function addBodyToRequestIfBean(Request $request, ReflectionParameter $parameter): void
	{
	    // 获取请求参数
	    $params = $request->input();
	    // 实例化 DTO 对象，将参数传入
	    $dto = new ($parameter->getType()->getName())($params);
	    // 关键：注入到容器
	    app()->instance($parameter->getType()->getName(), $bean);
	}
}
```

上面实例化 DTO 对象，传入参数这一步我可以给出一个简单的实现:
```php
class DTO
{
    public function __construct(array $data = [])
    {
        foreach ($data as $key => $value) {
            $this->$key = $value;
        }
    }
}
```
其他的对象都继承 `DTO` 就可以了。
## 总结
通过 `#[RequestBody]` 的封装，我们在编写优雅的代码的路上又前进一步。

虽然封装的过程有点小复杂，你需要理解 PHP 中的注解、反射，你还需要理解 Laravel 中的容器、依赖注入、控制反转、中间件等概念。但是使用简单，你只需要在控制器方法中加上一个注解就可以了。这也是封装的意义： **允许封装复杂，但是使用要简单** 。

[Laravel 中实现 SpringBoot 中的 RequestBody 注解 | Laravel China 社区](https://learnku.com/articles/86716)