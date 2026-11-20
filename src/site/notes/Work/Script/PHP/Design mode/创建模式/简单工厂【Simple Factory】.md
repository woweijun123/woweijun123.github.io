---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/创建模式/简单工厂【Simple Factory】/","title":"简单工厂【Simple Factory】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:52.000+08:00","updated":"2026-03-24T17:26:33.039+08:00","dg-note-properties":{"title":"简单工厂【Simple Factory】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
// 操作类
abstract class Operation
{
    protected $a = 0;
    protected $b = 0;

    public function setA($a)
    {
        $this->a = $a;
    }

    public function setB($b)
    {
        $this->b = $b;
    }

    abstract protected function getResult();
}

// 加
class Add extends Operation
{
    public function getResult()
    {
        return $this->a + $this->b;
    }
}

// 减
class Sub extends Operation
{
    public function getResult()
    {
        return $this->a - $this->b;
    }
}

// 乘
class Mul extends Operation
{
    public function getResult()
    {
        return $this->a * $this->b;
    }
}

// 除
class Div extends Operation
{
    public function getResult()
    {
        return $this->a / $this->b;
    }
}

// 操作工厂类
class OperationFactory
{
    public static function createOperation($args)
    {
        switch ($args) {
            case '+':
                $operation = new Add();
                break;
            case '-':
                $operation = new Sub();
                break;
            case '*':
                $operation = new Mul();
                break;
            case '/':
                $operation = new Div();
                break;
        }

        return $operation;
    }
}

$operation = OperationFactory::createOperation('+');
$operation->setA(1);
$operation->setB(2);
echo $operation->getResult() . PHP_EOL; // 3
```

```php
interface ProductInterface
{
    public function showProductInfo();
}

class ProductA implements ProductInterface
{
    public function showProductInfo()
    {
        echo 'This is product A', PHP_EOL;
    }
}

class ProductB implements ProductInterface
{
    public function showProductInfo()
    {
        echo 'This is product B', PHP_EOL;
    }
}

class ProductFactory
{
    public static function factory($ProductType)
    {
        $ProductType = 'Product' . strtoupper($ProductType);
        if (class_exists($ProductType)) {
            return new $ProductType();
        } else {
            throw new Exception("Error Processing Request", 1);
        }
    }
}

// 这里需要一个产品型号为 A 的对象
try {
    $x = ProductFactory::factory('A'); // This is product A
} catch (Exception $e) {
}
$x->showProductInfo(); // This is product A

// 这里需要一个产品型号为 B 的对象
try {
    $o = ProductFactory::factory('B'); // This is product B
} catch (Exception $e) {
}
$o->showProductInfo(); // This is product B
```
# 核心方法论
* **封装 (Encapsulation):** 将数据（属性）和操作数据的方法（行为）捆绑在一起，隐藏内部细节。
    * 体现: `Operation` 基类封装了 `$a` 和 `$b` 属性及其设置方法。
* **继承 (Inheritance):** 子类继承父类的属性和方法。
    * 体现: `Add`, `Sub`, `Mul`, `Div` 继承自抽象类 `Operation`。
* **多态 (Polymorphism):** 不同的子类对象对父类或接口定义的同一方法有不同的实现。
    * 体现: `Add`, `Sub`, `Mul`, `Div` 都实现了 `getResult()`，但返回不同运算结果。
# 设计目标
* **高内聚 (High Cohesion):** 一个模块或类只负责完成一项任务。
    * 体现: 每个操作类（`Add`, `Sub` 等）只专注于实现自己的运算逻辑。
* **低耦合 (Low Coupling):** 模块或类之间的依赖关系降到最低。
    * 体现: **简单工厂模式** (`OperationFactory`) 将客户端代码与具体操作类（`Add`, `Sub` 等）解耦。客户端只需知道工厂和抽象 `Operation` 即可。
# 复用原则
* **复用而非复制：** 强调通过**继承、多态**等 OOD 机制实现代码复用，而不是简单的复制粘贴。
# 示例中的模式
两个示例均体现了**简单工厂模式 (Simple Factory)**：
1.  **第一个示例 (`OperationFactory`):** 通过 `switch` 语句集中创建不同的运算对象，将对象的实例化和客户端使用分离开来。
2.  **第二个示例 (`ProductFactory`):** 利用反射/动态类名（`class_exists` 和 `new $ProductType()`）来创建不同的产品对象，进一步降低了工厂内的逻辑判断复杂度。