---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel Aop 观察者 事件/","title":"Laravel Aop 观察者 事件","tags":["flashcards"],"noteIcon":"","created":"2025-05-17T13:41:35.621+08:00","updated":"2026-03-24T17:34:00.734+08:00","dg-note-properties":{"title":"Laravel Aop 观察者 事件","tags":["flashcards"],"reference linking":null}}
---

在 Laravel 中，虽然没有原生的 **AOP（面向切面编程）** 支持，但可以通过 **中间件、事件与监听器、观察者模式** 等机制实现类似 AOP 的功能。以下是详细分析：
### 1. Laravel 中的 AOP 实现
#### 1.1 AOP 的核心概念
AOP（Aspect-Oriented Programming）的核心是将 **横切关注点**（如日志、权限控制、事务管理）从业务逻辑中分离出来，通过统一的“切面”拦截并处理这些关注点。
#### 1.2 Laravel 的 AOP 替代方案
Laravel 通过以下机制实现类似 AOP 的功能：
- **中间件（Middleware）**：拦截 HTTP 请求，处理权限验证、日志记录等。
- **事件与监听器（Events & Listeners）**：解耦业务逻辑，通过事件触发特定操作。
- **模型观察者（Observers）**：监听模型的生命周期事件（如创建、更新、删除）。
- **第三方库（如 `goaop/laravel-bridge`）**：通过注解或代理实现真正的 AOP 功能。
#### 1.3 示例：通过中间件实现类似 AOP
```php
// 定义中间件（权限控制）
public function handle($request, Closure $next)
{
    if (!auth()->check()) {
        return redirect('/login');
    }
    // 记录请求日志
    Log::info("Request to {$request->path()}");
    return $next($request);
}
```
此中间件在请求前验证权限，并在响应后记录日志，功能上类似 AOP 的切面。
### 2. AOP 与观察者模式的对比
#### 2.1 观察者模式（Observer Pattern）
- **定义**：观察者模式通过订阅和通知机制，使对象间保持松耦合。当被观察对象状态变化时，所有依赖它的观察者会自动收到通知。
- **Laravel 实现**：通过 **模型事件**（如 `created`, `updated`）和 **观察者类** 实现。
- **示例**：
```php
// 注册观察者
User::observe(UserObserver::class);

// 观察者类
class UserObserver {
  public function created(User $user) {
	  Log::info("User {$user->id} created.");
  }
}
```
#### 2.2 AOP 与观察者模式的优劣势对比
| **特性**    | **AOP**            | **观察者模式**        |
| --------- | ------------------ | ---------------- |
| **适用场景**  | 横切关注点（如日志、权限、事务）   | 模型生命周期事件（如创建、更新） |
| **灵活性**   | 更灵活，支持任意方法/类拦截     | 仅限模型事件，需绑定模型方法   |
| **侵入性**   | 低（通过注解或代理实现）       | 低（通过模型事件绑定）      |
| **学习成本**  | 高（需引入第三方库，如 goaop） | 低（Laravel 原生支持）  |
| **性能开销**  | 可能较高（动态代理）         | 低（直接调用回调函数）      |
| **代码可读性** | 需额外配置，复杂度较高        | 代码直观，易于调试        |
#### 2.3 选择建议
- **优先使用观察者模式**：若需求集中在模型的生命周期事件（如用户注册后发送邮件），观察者模式更简单直接。
- **使用 AOP**：若需拦截任意类/方法的横切关注点（如全局异常处理、接口调用日志），可引入 `goaop/laravel-bridge` 等库。
### 3. Laravel 中其他监听模型的方式
#### 3.1 通过静态方法监听模型事件
在 `EventServiceProvider` 中直接绑定回调：
```php
// app/Providers/EventServiceProvider.php
public function boot()
{
    User::retrieved(function ($user) {
        Log::info("User {$user->id} retrieved.");
    });
}
```
#### 3.2 通过订阅者（Subscriber）监听模型事件
1. **创建事件和监听器**：
```bash
php artisan event:generate
```
2. **定义事件和监听器逻辑**：
```php
// app/Events/UserCreated.php
class UserCreated {
   public $user;
   public function __construct(User $user) {
	   $this->user = $user;
   }
}

// app/Listeners/NotifyUserCreation.php
class NotifyUserCreation {
   public function handle(UserCreated $event) {
	   Mail::to($event->user->email)->send(new WelcomeEmail());
   }
}
```
3. **注册订阅者**：
```php
protected $listen = [
	'App\Events\UserCreated' => [
	   'App\Listeners\NotifyUserCreation',
	],
];
```
#### 3.3 通过模型事件触发自定义逻辑
直接在模型方法中触发事件：
```php
// 在模型中定义事件
protected $dispatchesEvents = [
    'created' => UserCreated::class,
];
```
### 4. 总结
- **AOP**：适合全局的横切关注点，但需依赖第三方库，适合复杂场景。
- **观察者模式**：Laravel 原生支持，代码简洁，适合模型生命周期事件。
- **其他方式**：静态方法监听、订阅者模式等，提供灵活的扩展性。

根据实际需求选择合适的方式，既能提升开发效率，又能保持代码的可维护性。