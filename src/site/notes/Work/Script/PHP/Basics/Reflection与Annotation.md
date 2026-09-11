---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Basics/Reflection与Annotation/","title":"Reflection与Annotation","tags":["#flashcards"],"noteIcon":"","created":"2026-09-10T09:59:12.000+08:00","updated":"2026-09-10T09:59:12.000+08:00","dg-note-properties":{"title":"Reflection与Annotation","tags":["#flashcards"],"reference linking":null}}
---

# 为什么需要反射
> **普通代码**只能操作编写时**已知的**类和方法；**反射**让程序在**运行时检查、选择、操作**事先**未知的**代码。
> 反射的本质是让程序==1;;观察==程序自身，操作的是类、方法、参数、属性、继承关系和 Attribute 等==1;;元==数据，而非业务数据本身。
## 普通代码的限制
```php
$service = new UserService();
$service->createUser();
```
写这段代码时，程序员已经知道：类叫 `UserService`，方法叫 `createUser`。**程序员知道 → 写死到代码里 → 程序执行**。但有一类程序不能这么干。
## 框架不知道业务代码——DI 容器场景
假设你正在写一个依赖注入容器，拿到了 `OrderController::class`，却要自动创建它未知的依赖：
```php
class OrderController
{
    public function __construct(OrderService $service, Logger $logger) {}
}
```
容器需要自动完成：
```text
给我一个类
      ↓
这个类构造函数是什么？有几个参数？参数分别是什么类型？
      ↓
OrderService → 自动创建
Logger       → 自动创建
      ↓
最终创建 OrderController
```
容器代码的作者不可能提前知道未来所有 Controller 的构造函数需要什么参数，参数随时可能变化。

PHP 实现：
```php
$reflection = new ReflectionClass(OrderController::class);
$constructor = $reflection->getConstructor();

foreach ($constructor->getParameters() as $parameter) {
    $type = $parameter->getType();
    echo $type->getName();
}
```
程序不需要提前知道 `OrderController` 的内部结构，可以在运行时"询问"这个类本身：
- 你是谁？
- 你有哪些方法？
- 你的构造函数需要什么？
- 参数是什么类型？
- 你有什么 Attribute？
- 这个方法是不是 `public`？
- 你继承了谁？

这就是 `Reflection` 的真正含义：**让程序观察程序自身**。
## 为什么不用普通 API
能否让每个类自己告诉框架？容器只需遍历 `OrderController::dependencies()` 即可。**完全可以。**
```php
class OrderController
{
    public static function dependencies()
    {
        return [
            OrderService::class,
            Logger::class,
        ];
    }
}
```
但问题在于，你维护了两份信息：**代码元数据、显式方法申明**，它们表达的是同一件事。修改构造函数却忘了改 `dependencies()`，框架就出问题。

反射直接读取构造函数，因此==1;;代码==本身就是元数据——这是反射的核心价值。
```php
// 构造函数
public function __construct(
    OrderService $service,
    Cache $cache,
    Logger $logger
) {}

// dependencies()
public static function dependencies()
{
    return [
        OrderService::class,
        Logger::class,
    ];
}
```
## 框架如何理解未知业务代码
平时写业务代码是**业务代码 → 调用框架**，但框架还有一种场景：**框架 → 调用你的业务代码**。
Hyperf、Spring、Laravel、PHPUnit 等框架作者不知道你未来会写什么 Controller，但框架依然需要：发现、分析、创建、注入、调用、管理它们。
```text
一个通用框架，怎么理解一个它开发时根本不存在的类？
        ↓
反射就是解决方案之一。
        ↓
越是框架底层，反射越常见。
```
## Attribute 与 ORM——两个典型例子
### 路由 Attribute
```php
#[Route('/user')]
class UserController
{
    #[Get('/info')]
    public function info() {}
}
```
框架启动时需要知道：`/user/info` 对应 `GET`，对应 `UserController::info()`。但框架作者不可能知道你会写 `UserController`。框架可以：
```php
$reflection = new ReflectionClass(UserController::class);
$attributes = $reflection->getAttributes();
$methods    = $reflection->getMethods();

foreach ($methods as $method) {
    $methodAttributes = $method->getAttributes();
}
```
最终自动生成路由表：`GET /user/info → UserController::info()`。你只负责写 `#[Get('/info')]`，框架自动完成。
### ORM
```php
class User
{
    #[Column('id')]
    private int $id;

    #[Column('name')]
    private string $name;
}
```
ORM 通过反射得到：`User` → `id(int, Column('id'))` + `name(string, Column('name'))`，然后自动完成 `SELECT id, name FROM user` 以及数据映射。反射支撑了大量"自动化"。
## 两个世界：数据 vs 元数据
| 世界 | 操作对象 | 举例 | 谁来做 |
| :-- | :-- | :-- | :-- |
| 数据世界 | 业务数据 | `$user->name`、`$order->price`、`$list->count()` | 普通代码 |
| 元数据世界 | 代码的信息 | 这个类叫什么？有哪些方法？构造参数是什么类型？有哪些 Attribute？继承了谁？实现了哪些接口？ | 反射 |

反射操作的是程序的==1;;元==数据（metadata）。普通程序是**代码 → 处理数据**；反射是**代码 → 观察代码**。
## 语言为什么提供反射
不是必须。没有反射的语言，代价是需要：显式注册、代码生成、配置文件、约定、编译器插件。
```php
// 没有反射时，你需要显式告诉框架
$container->bind(
    OrderController::class,
    function () {
        return new OrderController(
            new OrderService(),
            new Logger()
        );
    }
);
```

|      | 没有反射          | 有反射          |
| :--- | :------------ | :----------- |
| 信息来源 | 程序员**显式告诉**框架 | 框架自己**观察**代码 |
## 使用边界——为什么不要滥用
反射在一定程度上绕过正常的程序边界。正常调用是 `$user->getName()`，反射却可能：
```php
$reflection->getProperties();
$reflection->getMethods();
```
甚至操作 `private` 成员。从"按接口使用对象"变成"研究内部结构操作它"，导致：难理解、难静态分析、IDE 追踪困难、重构风险增加、性能下降、破坏封装、错误推迟到运行时。
- **适合反射：** 框架、DI 容器、ORM、路由扫描、序列化、测试工具。
- **谨慎使用：** 普通业务代码优先使用明确接口和依赖注入。
> 写 Hyperf 业务代码时可能很少主动写 `ReflectionClass`，但 Hyperf 底层大量机制都离不开元编程能力。
> **一句话：** 当"代码本身"也需要成为程序处理的数据时，就需要反射。
<!--SR:!2026-10-12,32,270-->
<?e?>
# 预定义常量 [¶](https://www.php.net/manual/zh/class.attribute.php#attribute.constants)
### 预定义常量一览
| 常量                                 | 作用范围        | 是否可重复 | 文档链接 |
| ---------------------------------- | ----------- | ----- | --- |
| `Attribute::TARGET_CLASS`          | 类           | ❌     | [Attribute::TARGET_CLASS](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-class) |
| `Attribute::TARGET_FUNCTION`       | 函数          | ❌     | [Attribute::TARGET_FUNCTION](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-function) |
| `Attribute::TARGET_METHOD`         | 方法          | ❌     | [Attribute::TARGET_METHOD](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-method) |
| `Attribute::TARGET_PROPERTY`       | 属性          | ❌     | [Attribute::TARGET_PROPERTY](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-property) |
| `Attribute::TARGET_CLASS_CONSTANT` | 类常量         | ❌     | [Attribute::TARGET_CLASS_CONSTANT](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-class-constant) |
| `Attribute::TARGET_PARAMETER`      | 参数          | ❌     | [Attribute::TARGET_PARAMETER](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-parameter) |
| `Attribute::TARGET_ALL`            | 所有目标        | ❌     | [Attribute::TARGET_ALL](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-all) |
| `Attribute::IS_REPEATABLE`         | 允许重复使用（需组合） | ✅     | [Attribute::IS_REPEATABLE](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.is-repeatable) |
## 1. `Attribute::TARGET_CLASS`
**作用范围**：仅允许注解修饰 **类**。
**示例**：
```php
#[Attribute(Attribute::TARGET_CLASS)]
class MyAnnotation {
    public function __construct(public string $value) {}
}

// ✅ 正确使用
#[MyAnnotation("This is a class")]
class MyClass {}

// ❌ 错误：不能修饰方法
#[MyAnnotation("Invalid")]
class MyClass {
    #[MyAnnotation("Invalid")] // 编译错误
    public function myMethod() {}
}
```
## 2. `Attribute::TARGET_FUNCTION`
**作用范围**：仅允许注解修饰 **函数**（非类方法）。
**示例**：
```php
#[Attribute(Attribute::TARGET_FUNCTION)]
class MyFunctionAnnotation
{
    public function __construct(public string $description)
    {
    }
}

// ✅ 正确使用
#[MyFunctionAnnotation("This is a function")]
function myFunction()
{
}

// ❌ 错误：不能修饰类或方法
#[MyFunctionAnnotation("Invalid")]
class MyClass
{
    #[MyFunctionAnnotation("Invalid")] // 编译错误
    public function myMethod()
    {
    }
}
```
## 3. `Attribute::TARGET_METHOD`
**作用范围**：仅允许注解修饰 **类方法**。
**示例**：
```php
#[Attribute(Attribute::TARGET_METHOD)]
class MyMethodAnnotation {
    public function __construct(public string $name) {}
}

class MyClass {
    #[MyMethodAnnotation("index")]
    public function myMethod() {}
}

// ❌ 错误：不能修饰类或函数
#[MyMethodAnnotation("Invalid")]
class MyClass {}
```
## 4. `Attribute::TARGET_PROPERTY`
**作用范围**：仅允许注解修饰 **类属性**。
**示例**：
```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class MyPropertyAnnotation {
    public function __construct(public string $type) {}
}

class MyClass {
    #[MyPropertyAnnotation("string")]
    public string $name;
}

// ❌ 错误：不能修饰方法或类
#[MyPropertyAnnotation("Invalid")]
class MyClass {
    #[MyPropertyAnnotation("Invalid")] // 编译错误
    public function myMethod() {}
}
```
## 5. `Attribute::TARGET_CLASS_CONSTANT`
**作用范围**：仅允许注解修饰 **类常量**。
**示例**：
```php
#[Attribute(Attribute::TARGET_CLASS_CONSTANT)]
class MyConstantAnnotation {
    public function __construct(public string $value) {}
}

class MyClass {
    #[MyConstantAnnotation("MAX")]
    public const MAX_VALUE = 100;
}

// ❌ 错误：不能修饰方法或属性
class MyClass {
    #[MyConstantAnnotation("Invalid")] // 编译错误
    public function myMethod() {}
}
```
## 6. `Attribute::TARGET_PARAMETER`
**作用范围**：仅允许注解修饰 **函数/方法参数**。
**示例**：
```php
#[Attribute(Attribute::TARGET_PARAMETER)]
class MyParameterAnnotation {
    public function __construct(public string $description) {}
}

class MyClass {
    public function myMethod(
        #[MyParameterAnnotation("Name")]
        public string $name
    ) {}
}

// ❌ 错误：不能修饰类或属性
#[MyParameterAnnotation("Invalid")]
class MyClass {}
```
## 7. `Attribute::TARGET_ALL`
**作用范围**：允许注解修饰 **所有目标**（类、方法、属性等）。
**示例**：
```php
#[Attribute(Attribute::TARGET_ALL)]
class MyAllAnnotation {
    public function __construct(public string $info) {}
}

// ✅ 可以修饰类、方法、属性等
#[MyAllAnnotation("Class")]
class MyClass {
    #[MyAllAnnotation("Property")]
    public string $name;

    #[MyAllAnnotation("Method")]
    public function myMethod(
        #[MyAllAnnotation("Parameter")]
        public string $param
    ) {}
}
```
## 8. `Attribute::IS_REPEATABLE`
**作用**：允许注解 **在同一元素上重复使用**。
**示例**：
```php
#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class MyRoute {
    public function __construct(public string $path, public string $method) {}
}

class MyController {
    #[MyRoute("/home", "GET")]
    #[MyRoute("/index", "POST")] // ✅ 允许重复
    public function index() {}
}

// ❌ 如果未设置 IS_REPEATABLE，以下会报错
class MyController {
    #[MyRoute("/home", "GET")]
    #[MyRoute("/index", "POST")] // 编译错误（除非设置 IS_REPEATABLE）
    public function index() {}
}
```
实例化执行时报错：
PHP Fatal error:  Uncaught Error: Attribute "MyRoute" must not be repeated in ...
```php
$reflectionClass      = new ReflectionMethod(MyController::class, 'index');
$reflectionAttributes = $reflectionClass->getAttributes();
foreach ($reflectionAttributes as $reflectionAttribute) {
    $newInstance = $reflectionAttribute->newInstance();
    var_dump($newInstance->path, $newInstance->method);
}
```
## 9. 组合使用多个作用范围
**示例**：允许注解同时修饰类和方法：
```php
use Attribute;

#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class MyCombinedAnnotation {
    public function __construct(public string $description) {}
}

// ✅ 正确使用
#[MyCombinedAnnotation("Class")]
class MyClass {
    #[MyCombinedAnnotation("Method")]
    public function myMethod() {}
}
```
## 10. 动态作用范围（未显式指定）
**示例**：未显式指定作用范围时，PHP 会根据注解位置动态确定：
```php
class MyDynamicAnnotation {}

// ✅ 修饰类（自动推断为 TARGET_CLASS）
#[MyDynamicAnnotation]
class MyClass {}

// ✅ 修饰方法（自动推断为 TARGET_METHOD）
class MyClass {
    #[MyDynamicAnnotation]
    public function myMethod() {}
}
```
## 实例
```php
#[Attribute(Attribute::TARGET_METHOD)]
class Attr
{
    public function __construct(
        public string $name,
    ) {
    }
}
#[Attr('myAttr')]
class AttrTest
{
}
$reflectionClass = new ReflectionClass(AttrTest::class);
$attributes = $reflectionClass->getAttributes('Attr');
$attributeInstance = $attributes[0]->newInstance();
echo $attributeInstance->name; // myAttr
```
# 类注解
## 定义与使用
```php
#[Attribute(Attribute::TARGET_CLASS)]
class Attr
{
    public function __construct(
        public string $name,
    ) {
    }
}
```
```php
#[Attr('myAttr')]
class AttrTest
{
}
```
## 反射获取注解
```php
$reflectionClass = new ReflectionClass(AttrTest::class);
$attributes = $reflectionClass->getAttributes('Attr');
$attributeInstance = $attributes[0]->newInstance();
echo $attributeInstance->name; // myAttr
```
其他位置的注解同理，都是使用反射来获取。
# 实现路由注解
实现一个基础的注解路由配置功能，新建 `Group`、`Route`、`Middlewares` 三个注解类。
## Group
```php
/**
 * 路由组
 */
#[Attribute(Attribute::TARGET_CLASS)]
class Group
{
    public function __construct(
        private string $group,
    ) {
    }

    public function getGroup(): string
    {
        return $this->group;
    }
}
```
## Route
```php
/**
 * 路由
 */
#[Attribute(Attribute::TARGET_METHOD)]
class Route
{
    public function __construct(
        private string $pattern,
        private string $requestMethod,
    ) {
    }

    public function getRoute(): array
    {
        return [
            'pattern' => $this->pattern,
            'request_method' => $this->requestMethod,
        ];
    }
}
```
## Middlewares
```php
/**
 * 中间件
 */
#[Attribute(Attribute::TARGET_METHOD)]
class Middlewares
{
    public function __construct(
        private array $middlewares = [],
    ) {
    }

    public function getMiddlewares(): array
    {
        return $this->middlewares;
    }
}
```
注解类建好后，新建 Student 类，给类加上 Group 注解、方法加上 Route、Middlewares 注解：
## Student
```php
#[Group('student')]
class Student
{
    #[Route('/study', 'GET'), Middlewares([Auth::class, Log::class])]
    public function study(): string
    {
        return 'Success';
    }
}
```
## 获取注解
```php
$className = Student::class;
$methodName = 'study';

$data = [];

$refClass = new ReflectionClass($className);
$refMethod = $refClass->getMethod($methodName);

// 获取Group注解
$groupAttributes = $refClass->getAttributes('Group');
$classAttributeInstance = $groupAttributes[0]->newInstance();
$groupName = $classAttributeInstance->getGroup();

$data['group'] = $groupName;

// 获取Route注解
$routeAttributes = $refMethod->getAttributes('Route');
$methodAttributeInstance = $routeAttributes[0]->newInstance();
$route = $methodAttributeInstance->getRoute();

$data['route'] = $route;

// 获取Middlewares注解
$middlewaresAttributes = $refMethod->getAttributes('Middlewares');
$middlewaresInstance = $middlewaresAttributes[0]->newInstance();
$middlewares = $middlewaresInstance->getMiddlewares();

$data['middlewares'] = $middlewares;

print_r($data);
```
执行输出：
```php
Array
(
    [group] => student
    [route] => Array
        (
            [pattern] => /study
            [request_method] => GET
        )
    [middlewares] => Array
        (
            [0] => Auth
            [1] => Log
        )
)
```
以上代码仅演示注解基础使用，生产环境请勿直接使用。真实场景中需要遍历所有方法获取路由配置并缓存。
# 示例
```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class NameProperty {}

#[Attribute(Attribute::TARGET_PROPERTY)]
class AgeProperty {}

#[Attribute(Attribute::TARGET_PARAMETER)]
class AgeParameter
{
    public readonly array $src;

    public function __construct(string ...$src)
    {
        $this->src = $src;
    }
}

class Riven
{
    #[NameProperty(name: 'z4')]
    public string $name = 'z3';
    #[AgeProperty(age: 28)]
    public int    $age  = 18;
    public int    $sex  = 1;
    const HELLO = 'hello';
    const WORLD = 'world';

    public static function run($name = 'l4', #[AgeParameter(age: 20, age2: 21)] $age = 19, #[AgeParameter('sex')] $sex = 2)
    {
        echo '执行' . self::class . '的程序', PHP_EOL;
    }
}

// 反射实例化类
$reflectionClass = new ReflectionClass(Riven::class);
$reflectionClass->newInstance()->run(); // 执行reflection\role\Riven的程序

// 获取类里的所有常量
var_dump($reflectionClass->getConstants());

// 获取类的所有属性
$reflectionProperties = $reflectionClass->getProperties();
var_dump($reflectionProperties);

// 获取类的默认属性的名称和值
$defaultProperties = $reflectionClass->getDefaultProperties();
foreach ($defaultProperties as $propertyName => $defaultValue) {
    echo "默认属性名称: " . $propertyName . PHP_EOL;
    echo "默认属性值: " . $defaultValue . PHP_EOL;
}

// 获取属性的注解
foreach ($reflectionProperties as $property) {
    foreach ($property->getAttributes(AgeProperty::class, ReflectionAttribute::IS_INSTANCEOF) as $attribute) {
        var_dump($attribute->getName());
        var_dump($attribute->getArguments());
    }
}

// 反射实例化方法
$reflectionMethod = new ReflectionMethod(Riven::class, 'run');
$reflectionParameters = $reflectionMethod->getParameters();

// 获取方法参数的注解
foreach ($reflectionParameters as $reflectionParameter) {
    $reflectionAttributes = $reflectionParameter->getAttributes(AgeParameter::class, ReflectionAttribute::IS_INSTANCEOF);
    foreach ($reflectionAttributes as $reflectionAttribute) {
        echo "---方法参数名称「{$reflectionParameter->getName()}」---" . PHP_EOL;
        echo "---方法注解名称「{$reflectionAttribute->getName()}」---" . PHP_EOL;
        var_dump($reflectionAttribute->newInstance());
        var_dump($reflectionAttribute->getArguments());
    }
}
```
# 反射判断是否实现某个接口
```php
interface ShouldQueue {}
$class = new class () implements ShouldQueue {};
var_dump((new ReflectionClass($class))->implementsInterface(ShouldQueue::class)); // true
```
<?e?>