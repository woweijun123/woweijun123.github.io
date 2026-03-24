---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/创建模式/工厂【Factory】/","title":"工厂【Factory】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:18.000+08:00","updated":"2026-03-24T17:26:29.457+08:00"}
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

// 工厂接口
interface IFactory
{
    public function createOperation();
}

class AddFactory implements IFactory
{
    public function createOperation()
    {
        return new Add();
    }
}

class SubFactory implements IFactory
{
    public function createOperation()
    {
        return new Sub();
    }
}

class MulFactory implements IFactory
{
    public function createOperation()
    {
        return new Mul();
    }
}

class DivFactory implements IFactory
{
    public function createOperation()
    {
        return new Div();
    }
}

$operationFactory = new AddFactory();
$operation = $operationFactory->createOperation();
$operation->setA(10);
$operation->setB(10);
echo $operation->getResult(), PHP_EOL; // 20
```
# 总结
### 简单工厂模式 (Simple Factory)
* **核心：** 单一工厂集中所有对象的创建逻辑和判断。
* **优点：** 客户端无需依赖具体产品，使用方便。
* **缺点：** 扩展性差，**每增加一个产品就必须修改工厂类**，**违反开闭原则 (OCP)**。
### 工厂方法模式 (Factory Method)
* **核心：** 定义一个创建对象的接口，让**子类工厂**决定实例化哪个类，将创建职责分散。
* **优点：** 扩展性好，增加新产品只需新增子类工厂，**遵守开闭原则 (OCP)**。
* **缺点：** 客户端需要自行判断和选择实例化哪一个具体工厂类，**选择的判断逻辑被转移到了客户端**。

**一句话总结：**
简单工厂是**集中式创建**，易用但难扩展；工厂方法是**分散式创建**，易扩展但将**工厂选择权**留给了客户端。
# 意图
定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
- 主要解决：主要解决接口选择的问题。
- 何时使用：我们明确地计划不同条件下创建不同实例时。
- 如何解决：让其子类实现工厂接口，返回的也是一个抽象的产品。
- 关键代码：创建过程在其子类执行。
# 优点
- 一个调用者想创建一个对象，只要知道其名称就可以了。 
- 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。 
- 屏蔽产品的具体实现，调用者只关心产品的接口。
# 缺点
每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。
# 使用场景
- 日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方。 
- 数据库访问，当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时。 
- 设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。

注意事项：作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。