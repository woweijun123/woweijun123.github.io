---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel 单元测试/","title":"Laravel 单元测试","tags":["flashcards"],"noteIcon":"","created":"2025-07-11T17:01:16.053+08:00","updated":"2026-03-24T17:33:47.633+08:00","dg-note-properties":{"title":"Laravel 单元测试","tags":["flashcards"],"reference linking":null}}
---

# 单元测试整理

## 一、Mockery 模拟测试
### 1. 验证方法调用与返回值
#### 场景  
验证 Facade 或类的方法是否被调用，且返回值符合预期。
#### 示例代码
```php
use App\Helper\MyFacade;

class MyController extends Controller
{
    public function show()
    {
        $result = MyFacade::myGet();
        return view('result', ['data' => $result]);
    }
}
```
#### 测试代码
```php
use Illuminate\Support\Facades\MyFacade;

public function test_show_method()
{
    // 模拟 myGet 返回指定值
    MyFacade::shouldReceive('myGet')
             ->andReturn('App\Helper\MyManager');

    $response = $this->get('/show');
    $response->assertViewHas('data', 'App\Helper\MyManager');
}
```
### 2. 模拟方法抛出异常
#### 场景  
验证异常处理逻辑是否正常工作。
#### 示例代码
```php
public function show()
{
    try {
        $result = MyFacade::myGet();
    } catch (\Exception $e) {
        return response()->json(['error' => $e->getMessage()], 500);
    }
    return response()->json(['data' => $result]);
}
```
#### 测试代码
```php
use MyNamespace\Exception\CustomException;

public function test_mock_method_throw_exception()
{
    MyFacade::shouldReceive('myMethod')
            ->andThrow(new CustomException('Error message'));

    $this->expectException(CustomException::class);
    MyFacade::myMethod();
}
```
### 3. 链式调用模拟
#### 场景  
验证链式方法调用的正确性。
#### 示例代码
```php
public function show()
{
    $result = MyFacade::myGet()
                      ->process()
                      ->format();
    return $result;
}
```
#### 测试代码
```php
public function test_chain_call()
{
    MyFacade::shouldReceive('myGet')
             ->andReturnSelf()
             ->shouldReceive('process')
             ->andReturnSelf()
             ->shouldReceive('format')
             ->andReturn('Formatted Data');

    $result = MyFacade::myGet()->process()->format();
    $this->assertEquals('Formatted Data', $result);
}
```
## 二、数据库测试
### 1. 数据库隔离与模型工厂
#### 场景  
确保每次测试后数据库状态独立，避免数据污染。
#### 示例代码
```php
class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        return view('users.index', compact('users'));
    }
}
```
#### 测试代码
```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class UserControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_index()
    {
        $user = \App\Models\User::factory()->create();

        $response = $this->get('/users');
        $response->assertStatus(200)
                 ->assertSee($user->name);
    }
}
```
### 2. 验证软删除与数据库状态
#### 场景  
验证软删除逻辑是否生效。
#### 示例代码
```php
class PostController extends Controller
{
    public function destroy($id)
    {
        $post = Post::find($id);
        $post->delete();
        return redirect()->back();
    }
}
```
#### 测试代码
```php
public function test_soft_deleted()
{
    $post = \App\Models\Post::factory()->create();
    $post->delete();

    $this->assertSoftDeleted('posts', ['id' => $post->id]);
}
```
## 三、HTTP 请求测试
### 1. 验证 JSON 响应与结构
#### 场景  
确保 API 返回正确的 JSON 数据和结构。
#### 示例代码
```php
class PostController extends Controller
{
    public function index()
    {
        $posts = Post::all();
        return response()->json(['data' => $posts]);
    }
}
```
#### 测试代码
```php
public function test_json_response()
{
    $response = $this->get('/api/posts');
    $response->assertStatus(200)
             ->assertJson([
                 'data' => [
                     ['id' => 1, 'title' => 'Post 1'],
                     ['id' => 2, 'title' => 'Post 2'],
                 ]
             ]);
}
```
### 2. 验证重定向与会话数据
#### 场景  
验证登录后是否跳转到指定页面，并设置会话数据。
#### 示例代码
```php
public function login(Request $request)
{
    if ($request->input('email') === 'admin@example.com') {
        return redirect('/dashboard')->with('status', 'Logged in successfully');
    }
    return back()->withErrors(['email' => 'Invalid email']);
}
```
#### 测试代码
```php
public function test_login_redirect()
{
    $response = $this->post('/login', ['email' => 'admin@example.com']);
    $response->assertRedirect('/dashboard')
             ->assertSessionHas('status', 'Logged in successfully');
}
```
## 四、事件与队列测试
### 1. 事件触发与监听器绑定
#### 场景  
验证事件是否被触发，以及监听器是否正确绑定。
#### 示例代码
```php
// 触发事件
event(new OrderShipped($order));

// 监听器
class SendShipmentNotification
{
    public function handle(OrderShipped $event)
    {
        // 发送通知
    }
}
```
#### 测试代码
```php
use Illuminate\Support\Facades\Event;

public function test_event_dispatched()
{
    Event::fake();
    event(new OrderShipped());

    Event::assertDispatched(OrderShipped::class);
    Event::assertListening(
        OrderShipped::class,
        \App\Listeners\SendShipmentNotification::class
    );
}
```
### 2. 队列任务推送
#### 场景  
验证队列任务是否被正确推送。
#### 示例代码
```php
ProcessPodcast::dispatch($podcast);
```
#### 测试代码
```php
use Illuminate\Support\Facades\Queue;

public function test_queue_job_dispatch()
{
    Queue::fake();
    ProcessPodcast::dispatch();

    Queue::assertPushed(ProcessPodcast::class);
}
```
## 五、异常与错误处理测试
### 1. 验证异常抛出与消息
#### 场景  
确保异常被正确捕获，并返回预期响应。
#### 示例代码
```php
public function calculate($a, $b)
{
    if ($b === 0) {
        throw new \Exception('Division by zero');
    }
    return $a / $b;
}
```
#### 测试代码
```php
public function test_exception_handling()
{
    $this->expectException(\Exception::class);
    $this->expectExceptionMessage('Division by zero');

    $this->calculate(10, 0);
}
```
## 六、表单验证与中间件测试
### 1. 表单请求验证规则
#### 场景  
验证表单请求的规则是否生效。
#### 示例代码
```php
class StorePostRequest extends FormRequest
{
    public function rules()
    {
        return ['title' => 'required|min:5'];
    }
}
```
#### 测试代码
```php
use App\Http\Requests\StorePostRequest;

public function test_form_validation()
{
    $request = new StorePostRequest();

    $validator = $request->validator(['title' => 'Hi']);
    $this->assertFalse($validator->passes());
}
```
### 2. 中间件行为验证
#### 场景  
验证中间件是否阻止未认证用户访问。
#### 示例代码
```php
class Authenticate
{
    public function handle($request, $next)
    {
        if (!auth()->check()) {
            return redirect('/login');
        }
        return $next($request);
    }
}
```
#### 测试代码
```php
use Illuminate\Http\Request;
use App\Http\Middleware\Authenticate;

public function test_middleware()
{
    $middleware = new Authenticate();
    $request = Request::create('/admin', 'GET');

    $response = $middleware->handle($request, function () {
        return 'continue';
    });

    $this->assertEquals(302, $response->getStatusCode());
}
```
## 七、依赖注入与链式调用测试
### 1. 依赖注入解耦性验证
#### 场景  
间接测试依赖注入的逻辑是否正确。
#### 示例代码
```php
class MyController
{
    protected $myManager;

    public function __construct(MyManager $myManager)
    {
        $this->myManager = $myManager;
    }

    public function show()
    {
        return $this->myManager->myGet();
    }
}
```
#### 测试代码
```php
use Illuminate\Support\Facades\MyFacade;

public function test_dependency_injection()
{
    MyFacade::shouldReceive('myGet')
             ->andReturn('App\Helper\MyManager');

    $controller = new MyController(\App\Helper\MyManager::instance());
    $result = $controller->show();
    $this->assertEquals('App\Helper\MyManager', $result);
}
```
# 常用断言方法

## 一、HTTP 响应断言
### 1. 验证 JSON 响应
```php
public function test_json_response()
{
    $response = $this->get('/api/users');
    $response->assertStatus(200)
             ->assertJson([
                 'data' => [
                     ['id' => 1, 'name' => 'Alice'],
                     ['id' => 2, 'name' => 'Bob'],
                 ]
             ]);
}
```
- `assertJson()`：验证响应是否包含指定的 JSON 结构。
- `assertJsonStructure()`：验证 JSON 结构（不验证值）：
```php
$response->assertJsonStructure([
  'data' => ['*' => ['id', 'name']]
]);
```
### 2. 验证重定向
```php
public function test_redirect()
{
    $response = $this->post('/login', ['email' => 'test@example.com']);
    $response->assertRedirect('/dashboard');
}
```
- `assertRedirect($url)`：验证是否重定向到指定 URL。
- `assertRedirectedToRoute('route.name')`：验证是否重定向到指定路由。
### 3. 验证视图数据
```php
public function test_view_data()
{
    $user = \App\Models\User::factory()->create();
    $response = $this->get('/profile');
    $response->assertViewIs('profile')
             ->assertViewHas('user', $user);
}
```
- `assertViewIs($viewName)`：验证视图名称。
- `assertViewHas($key, $value)`：验证视图中是否存在指定数据。
### 4. 验证会话数据
```php
public function test_session_flash()
{
    $response = $this->post('/submit', []);
    $response->assertSessionHas('status', 'Form submitted successfully');
}
```
- `assertSessionHas($key, $value)`：验证会话中是否存在指定键值。
- `assertSessionMissing($key)`：验证会话中不存在指定键。
## 二、表单验证与错误断言
### 1. 验证表单错误
```php
public function test_form_validation_error()
{
    $response = $this->post('/register', ['email' => 'invalid-email']);
    $response->assertStatus(302)
             ->assertSessionHasErrors(['email' => 'The email must be a valid email address.']);
}
```
- `assertSessionHasErrors($field, $message)`：验证指定字段的错误信息。
### 2. 验证表单无效
```php
public function test_form_invalid()
{
    $request = new \App\Http\Requests\StorePostRequest();
    $validator = $request->validator(['title' => '']);
    $this->assertFalse($validator->passes());
}
```
- `assertInvalid($field)`：验证表单请求是否对指定字段无效（需结合 `assertSessionHasErrors` 使用）。
## 三、数据库断言
### 1. 验证数据库存在
```php
public function test_database_has()
{
    $user = \App\Models\User::factory()->create();
    $this->assertDatabaseHas('users', ['email' => $user->email]);
}
```
- `assertDatabaseHas($table, $data)`：验证数据库中是否存在指定数据。
### 2. 验证数据库缺失
```php
public function test_database_missing()
{
    $this->assertDatabaseMissing('users', ['email' => 'not-exists@example.com']);
}
```
- `assertDatabaseMissing($table, $data)`：验证数据库中不存在指定数据。
### 3. 验证软删除
```php
public function test_soft_deleted()
{
    $user = \App\Models\User::factory()->create();
    $user->delete();
    $this->assertSoftDeleted('users', ['id' => $user->id]);
}
```
- `assertSoftDeleted($table, $data)`：验证记录是否被软删除。
## 四、队列与事件断言
### 1. 验证队列任务
```php
use Illuminate\Support\Facades\Queue;

public function test_queue_job()
{
    Queue::fake();
    \App\Jobs\ProcessPodcast::dispatch();
    Queue::assertPushed(\App\Jobs\ProcessPodcast::class);
}
```
- `assertPushed($jobClass)`：验证队列任务是否被推送。
- `assertNotPushed($jobClass)`：验证队列任务未被推送。
### 2. 验证事件触发
```php
use Illuminate\Support\Facades\Event;

public function test_event_dispatched()
{
    Event::fake();
    event(new \App\Events\OrderShipped());
    Event::assertDispatched(\App\Events\OrderShipped::class);
}
```
- `assertDispatched($eventClass)`：验证事件是否被触发。
- `assertNotDispatched($eventClass)`：验证事件未被触发。
## 五、异常与错误断言
### 1. 验证异常抛出
```php
public function test_exception_thrown()
{
    $this->expectException(\InvalidArgumentException::class);
    throw new \InvalidArgumentException('Invalid input');
}
```
- `expectException($class)`：验证是否抛出指定异常。
### 2. 验证异常消息
```php
public function test_exception_message()
{
    $this->expectExceptionMessage('Invalid email format');
    throw new \Exception('Invalid email format');
}
```
- `expectExceptionMessage($message)`：验证异常消息内容。
## 六、其他实用断言
### 1. 验证响应头
```php
public function test_response_header()
{
    $response = $this->get('/download');
    $response->assertHeader('Content-Type', 'application/pdf');
}
```
- `assertHeader($name, $value)`：验证响应头是否存在或匹配指定值。
### 2. 验证 Cookie
```php
public function test_cookie_set()
{
    $response = $this->get('/login');
    $response->assertCookie('token', 'abc123');
}
```
- `assertCookie($name, $value)`：验证 Cookie 是否被设置。
### 3. 验证文件上传
```php
public function test_file_upload()
{
    $file = UploadedFile::fake()->image('avatar.jpg');
    $response = $this->post('/upload', ['avatar' => $file]);
    $response->assertStatus(200);
}
```
- `UploadedFile::fake()`：模拟文件上传（适用于测试文件处理逻辑）。
## 七、总结：常用断言方法表
| 断言方法                               | 用途           |
| ---------------------------------- | ------------ |
| `assertStatus($code)`              | 验证 HTTP 状态码  |
| `assertJson($data)`                | 验证 JSON 响应内容 |
| `assertJsonStructure($structure)`  | 验证 JSON 结构   |
| `assertRedirect($url)`             | 验证是否重定向      |
| `assertViewIs($name)`              | 验证视图名称       |
| `assertViewHas($key, $value)`      | 验证视图数据       |
| `assertSessionHas($key, $value)`   | 验证会话数据       |
| `assertDatabaseHas($table, $data)` | 验证数据库存在      |
| `assertSoftDeleted($table, $data)` | 验证软删除记录      |
| `assertPushed($jobClass)`          | 验证队列任务推送     |
| `assertDispatched($eventClass)`    | 验证事件触发       |
| `expectException($class)`          | 验证异常抛出       |
| `assertHeader($name, $value)`      | 验证响应头        |
| `assertCookie($name, $value)`      | 验证 Cookie    |