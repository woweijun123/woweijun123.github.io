---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel 异步队列事件/","title":"Laravel 异步队列事件","tags":["flashcards"],"noteIcon":"","created":"2025-07-21T20:11:35.690+08:00","updated":"2026-03-24T17:33:56.104+08:00","dg-note-properties":{"title":"Laravel 异步队列事件","tags":["flashcards"],"reference linking":null}}
---

下面是一个完整的Laravel示例，展示如何使用事件和Redis队列异步执行耗时任务：
### 1. 创建事件类
```bash
php artisan make:event OrderImEvent
```
**app/Events/OrderImEvent.php**
```php
namespace App\Events;

use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderImEvent
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $immMsgType;
    public $playersId;
    public $userId;
    public $actionRole;
    public $order;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct($immMsgType, $playersId, $userId, $actionRole, $order)
    {
        $this->immMsgType = $immMsgType;
        $this->playersId = $playersId;
        $this->userId = $userId;
        $this->actionRole = $actionRole;
        $this->order = $order;
    }
}
```
### 2. 创建监听器类（实现ShouldQueue接口）
```bash
php artisan make:listener ProcessOrderImNotification --event=OrderImEvent
```
**app/Listeners/ProcessOrderImNotification.php**
```php
<?php

namespace App\Listeners;

use App\Events\OrderImEvent;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class ProcessOrderImNotification implements ShouldQueue
{
    use InteractsWithQueue;

    // 自定义队列连接
    public $connection = 'redis';

    // 自定义队列名称
    public $queue = 'im_notifications';

    /**
     * 处理事件
     */
    public function handle(OrderImEvent $event)
    {
        // 获取事件数据
        $immMsgType = $event->immMsgType;
        $playersId = $event->playersId;
        $userId = $event->userId;
        $actionRole = $event->actionRole;
        $order = $event->order;

        // 这里是你的耗时任务逻辑
        $this->sendImNotification(
            $immMsgType, 
            $playersId, 
            $userId, 
            $actionRole, 
            $order
        );
    }

    /**
     * 发送IM通知（模拟耗时任务）
     */
    protected function sendImNotification($immMsgType, $playersId, $userId, $actionRole, $order)
    {
        // 实际业务逻辑 - 这可能是耗时操作
        // 例如：调用第三方IM服务API、处理大数据等
        
        // 模拟耗时操作（实际使用时删除）
        sleep(120);
        
        logger()->info('IM通知已发送', [
            'msg_type' => $immMsgType,
            'players' => $playersId,
            'user_id' => $userId,
            'role' => $actionRole,
            'order_id' => $order->id
        ]);
    }

    /**
     * 处理失败任务
     */
    public function failed(OrderImEvent $event, $exception)
    {
        // 失败处理逻辑
        logger()->error('IM通知发送失败: ' . $exception->getMessage(), [
            'order_id' => $event->order->id,
            'user_id' => $event->userId
        ]);
    }
}
```
### 3. 注册事件监听器
**app/Providers/EventServiceProvider.php**
```php
<?php

namespace App\Providers;

use App\Events\OrderImEvent;
use App\Listeners\ProcessOrderImNotification;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        OrderImEvent::class => [
            ProcessOrderImNotification::class,
        ],
    ];

    // ...
}
```
### 4. 触发事件（在控制器中）
```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderImEvent;
use App\Models\Order;
use App\Enums\IMMsgType;
use App\Enums\ActionRole;

class OrderController extends Controller
{
    public function placeOrder()
    {
        // 创建订单逻辑...
        $order = Order::create([...]);
        
        // 获取玩家ID列表
        $playersId = [/* 玩家ID数组 */];
        
        // 触发事件（异步执行）
        event(new OrderImEvent(
            IMMsgType::AfterPlaceOrder,
            $playersId,
            $order->user_id,
            ActionRole::User,
            $order
        ));

        return response()->json(['success' => true]);
    }
}
```
### 5. 配置Redis队列
**.env 文件**
```env
QUEUE_CONNECTION=redis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
REDIS_QUEUE=default
```
### 6. 启动队列处理器
```bash
# 处理指定队列
php artisan queue:work redis --queue=im_notifications

# 或者在后台运行（使用Supervisor管理）
# 更多选项：--tries=3 --timeout=300
```
### 7. 监控队列（可选）
使用Laravel Horizon进行高级监控：
```bash
composer require laravel/horizon

# 发布配置文件
php artisan horizon:install
```
**config/horizon.php**
```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['im_notifications'], // 监控我们的队列
            'processes' => 5, // 进程数
            'tries' => 3,
            'timeout' => 300,
        ],
    ],
],
```
常用命令
```bash
horizon:clear : 删除指定队列中的所有任务。
horizon:clear-metrics : 删除所有任务和队列的指标数据。
horizon:continue : 指示主监控器（master supervisor）继续处理任务。
horizon:continue-supervisor : 指示指定的监控器继续处理任务。
horizon:forget : 删除一个失败的队列任务。
horizon:install : 安装所有 Horizon 资源。
horizon:list : 列出所有已部署的机器。
horizon:pause : 暂停主监控器。
horizon:pause-supervisor : 暂停一个指定的监控器。
horizon:publish : 发布所有 Horizon 资源。
horizon:purge : 终止所有不正常的 Horizon 进程。
horizon:snapshot : 存储队列指标的快照。
horizon:status : 获取 Horizon 的当前状态。
horizon:supervisor-status : 显示指定监控器的状态。
horizon:supervisors : 列出所有监控器。
horizon:terminate : 终止主监控器，以便可以重新启动它。
```
### 关键点说明：
1. **异步处理**：
   - 事件触发后立即返回响应
   - 耗时任务在队列中异步执行
   - 不会阻塞主请求流程

2. **队列配置**：
   - 使用Redis作为队列驱动
   - 自定义队列名称`im_notifications`
   - 支持失败任务重试和日志记录

3. **优势**：
   - 解耦业务逻辑
   - 提高系统响应速度
   - 支持任务重试和错误处理
   - 可水平扩展（增加更多队列worker）

4. **处理长时间任务**：
   - 在队列worker中设置`--timeout`参数
   - 在`config/queue.php`中配置重试次数：
```php
	'redis' => [
	 'driver' => 'redis',
	 'connection' => 'default',
	 'queue' => env('REDIS_QUEUE', 'default'),
	 'retry_after' => 300, // 秒
	 'block_for' => null,
	],
```