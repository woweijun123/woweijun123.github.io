---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Basics/Reflection与Annotation/","title":"Reflection与Annotation","noteIcon":"","created":"2024-06-15T17:27:23.000+08:00","updated":"2026-03-24T17:22:15.306+08:00"}
---


# 预定义常量 [¶](https://www.php.net/manual/zh/class.attribute.php#attribute.constants)
- 类: [Attribute::TARGET_CLASS](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-class)
- 函数: [Attribute::TARGET_FUNCTION](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-function)
- 方法: [Attribute::TARGET_METHOD](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-method)
- 类属性: [Attribute::TARGET_PROPERTY](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-property)
- 类常量: [Attribute::TARGET_CLASS_CONSTANT](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-class-constant)
- 形参: [Attribute::TARGET_PARAMETER](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-parameter)
- 任何: [Attribute::TARGET_ALL](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.target-all)
- [Attribute::IS_REPEATABLE](https://www.php.net/manual/zh/class.attribute.php#attribute.constants.is-repeatable)
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
### 总结对比表
| 常量                                 | 作用范围        | 是否可重复 |
| ---------------------------------- | ----------- | ----- |
| `Attribute::TARGET_CLASS`          | 类           | ❌     |
| `Attribute::TARGET_FUNCTION`       | 函数          | ❌     |
| `Attribute::TARGET_METHOD`         | 方法          | ❌     |
| `Attribute::TARGET_PROPERTY`       | 属性          | ❌     |
| `Attribute::TARGET_CLASS_CONSTANT` | 类常量         | ❌     |
| `Attribute::TARGET_PARAMETER`      | 参数          | ❌     |
| `Attribute::TARGET_ALL`            | 所有目标        | ❌     |
| `Attribute::IS_REPEATABLE`         | 允许重复使用（需组合） | ✅     |
## 实例
运行时检测错误
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
// 获取到所有Attr注解
$attributes = $reflectionClass->getAttributes('Attr');
// 得到Attr实例，这里也可以调用注解类里的方法
$attributeInstance = $attributes[0]->newInstance();
echo $attributeInstance->name; // myAttr
```
# 类注解
## 定义
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
## 使用
```php
#[Attr('myAttr')]
class AttrTest
{
}
```
## 使用反射来获取AttrTest类的注解
```php
$reflectionClass = new ReflectionClass(AttrTest::class);
// 获取到所有Attr注解
$attributes = $reflectionClass->getAttributes('Attr');
// 得到Attr实例，这里也可以调用注解类里的方法
$attributeInstance = $attributes[0]->newInstance();
echo $attributeInstance->name; // myAttr
```
类的注解大致用法就是这样，其他位置的注解同理，都是使用反射来获取。
# 实现路由注解
现在我们来实现一个基础的注解路由配置功能。  
新建 `Group`、`Route`、`Middlewares` 三个注解类，分别为路由组、路由、中间件。
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
三个简单的注解类建好后，再新建一个Student类，给类加上Group注解、方法加上Route、Middlewares注解
## Student
```php
#[Group('student')]
class Student
{
    #[
        Route('/study', 'GET'),
        Middlewares([Auth::class, Log::class])
    ]
    public function study(): string
    {
        return 'Success';
    }
}
```
## 现在来获取注解
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
执行，输出如下
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
到这里我们已经拿到路由的相关配置，以上代码仅为了演示注解的基础使用，请不要在生产环境直接使用。

由于注解路由配置的特性，在真实的业务逻辑中，需要遍历所有可能访问到的方法，获取到所有路由配置并缓存，才能使注解路由生效，这样做是因为我们无法预先知道用户请求什么地址。当然了，路由相关功能不在本文讨论范围内。
# 示例
```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class NameProperty
{
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class AgeProperty
{
}

#[Attribute(Attribute::TARGET_PARAMETER)]
class AgeParameter
{
    /**
     * @var array|string[] 来源 key
     */
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

# +----------------------------------------------------------------------
# | 反射实例化类
# +----------------------------------------------------------------------
$reflectionClass = new ReflectionClass(Riven::class);
# 实例化并调用方法
$reflectionClass->newInstance()->run(); // 执行reflection\role\Riven的程序

# 获取类里的所有常量
var_dump($reflectionClass->getConstants());
/*
array(2) {
  ["HELLO"]=>
  string(5) "hello"
  ["WORLD"]=>
  string(5) "world"
}
*/

# 获取类的所有属性
$reflectionProperties = $reflectionClass->getProperties();
var_dump($reflectionProperties);
/*
array(2) {
  [0]=>
  object(ReflectionProperty)#2 (2) {
    ["name"]=>
    string(4) "name"
    ["class"]=>
    string(5) "Riven"
  }
  [1]=>
  object(ReflectionProperty)#3 (2) {
    ["name"]=>
    string(3) "age"
    ["class"]=>
    string(5) "Riven"
  }
}
*/

# 获取类的默认属性的名称和值
$defaultProperties = $reflectionClass->getDefaultProperties();
foreach ($defaultProperties as $propertyName => $defaultValue) {
    // getDefaultProperties() 返回的是一个数组，键是属性名，值是默认值
    echo "默认属性名称: " . $propertyName . PHP_EOL;
    echo "默认属性值: " . $defaultValue . PHP_EOL;
}
/*
默认属性名称: name
默认属性值: z3
默认属性名称: age
默认属性值: 18
*/

# 获取属性的注解属性
foreach ($reflectionProperties as $property) {
    // 只获取 AgeProperty 的注解属性
    foreach ($property->getAttributes(AgeProperty::class, ReflectionAttribute::IS_INSTANCEOF) as $attribute) {
        var_dump($attribute->getName());
        var_dump($attribute->getArguments());
    }
}
/*
string(11) "AgeProperty"
array(1) {
  ["age"]=>
  int(28)
}
*/

# +----------------------------------------------------------------------
# | 反射实例化方法
# +----------------------------------------------------------------------
$reflectionMethod = new ReflectionMethod(Riven::class, 'run');
# 获取方法的所有参数
$reflectionParameters = $reflectionMethod->getParameters();
# 参数 的注解参数
foreach ($reflectionParameters as $reflectionParameter) {
    // 只获取 AgeParameter 的注解参数
    $reflectionAttributes = $reflectionParameter->getAttributes(AgeParameter::class, ReflectionAttribute::IS_INSTANCEOF);
    foreach ($reflectionAttributes as $reflectionAttribute) {
        echo "---方法参数名称「{$reflectionParameter->getName()}」---" . PHP_EOL;
        echo "---方法注解名称「{$reflectionAttribute->getName()}」---" . PHP_EOL;
        echo "---方法注解实例化---" . PHP_EOL;
        var_dump($reflectionAttribute->newInstance());
        echo "---方法注解参数---" . PHP_EOL;
        var_dump($reflectionAttribute->getArguments());
    }
}
/*
---方法参数名称「age」---
---方法注解名称「AgeParameter」---
---方法注解实例---
object(AgeParameter)#6 (1) {
  ["src"]=>
  array(2) {
    ["age"]=>
    string(2) "20"
    ["age2"]=>
    string(2) "21"
  }
}
---方法注解参数---
array(2) {
  ["age"]=>
  int(20)
  ["age2"]=>
  int(21)
}
---方法参数名称「sex」---
---方法注解名称「AgeParameter」---
---方法注解实例---
object(AgeParameter)#5 (1) {
  ["src"]=>
  array(1) {
    [0]=>
    string(3) "sex"
  }
}
---方法注解参数---
array(1) {
  [0]=>
  string(3) "sex"
}
*/
```
# 反射判断是否实现某个接口
```php
interface ShouldQueue {}  
$class = new class () implements ShouldQueue {};  
var_dump((new ReflectionClass($class))->implementsInterface(ShouldQueue::class)); // true
```