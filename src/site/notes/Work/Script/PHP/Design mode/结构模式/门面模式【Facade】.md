---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/门面模式【Facade】/","title":"门面模式【Facade】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:08.000+08:00","updated":"2026-03-24T17:26:53.906+08:00","dg-note-properties":{"title":"门面模式【Facade】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
// 子系统1
class One
{
    public function methodOne()
    {
        echo '子系统方法1', PHP_EOL;
    }
}

// 子系统2
class Two
{
    public function methodTwo()
    {
        echo '子系统方法2', PHP_EOL;
    }
}

// 子系统3
class Three
{
    public function methodThree()
    {
        echo '子系统方法3', PHP_EOL;
    }
}

// 子系统4
class Four
{
    public function methodFourth()
    {
        echo '子系统方法4', PHP_EOL;
    }
}
// 外观方法
class Facade
{
    private $One;
    private $Two;
    private $Three;
    private $Four;
    function __construct()
    {
        $this->One = new One();
        $this->Two = new Two();
        $this->Three = new Three();
        $this->Four = new Four();
    }
    public function methodA()
    {
        echo '方法A() ---', PHP_EOL;
        $this->One->methodOne();
        $this->Three->methodThree();
    }
    public function methodB()
    {
        echo '方法B() ---', PHP_EOL;
        $this->Two->methodTwo();
        $this->Four->methodFourth();
    }
}

$facade = new Facade();
$facade->methodA();
$facade->methodB();
/*
方法A() ---
子系统方法1
子系统方法3
方法B() ---
子系统方法2
子系统方法4
*/
```
# 总结
外观模式，为子系统中的一组接口提供一个**一致的界面**，此模式定义了一个高层接口，这个接口使得这一子系统更容易使用。
# 设计初期阶段
有意识地在**不同软件层之间**（如三层架构）建立外观，隔离层间依赖。
# 开发阶段
当子系统变得过于复杂、不断重构，或处理难以维护的**遗留系统**时，通过外观提供清晰简单的接口。
# 意图
为**复杂的子系统**提供一个**一致的、高层级的简化接口**（即“接待员”）。
# 主要解决
**降低客户端访问复杂系统的难度**，将子系统内部的复杂联系（调用顺序、依赖关系）封装起来。
# 何时使用
客户端只需要一个**单一入口**访问复杂系统时，或用于**定义系统层次结构的入口**。
# 如何解决
客户端不与系统耦合，外观类与系统耦合。
# 关键代码
引入一个**外观类（Facade）**。客户端只与 Facade 交互（低耦合），而 Facade 负责与复杂子系统交互（高耦合）。
# 优点
**减少系统相互依赖**；提高系统的**灵活性和安全性**。
# 缺点
外观类耦合了所有子系统，如果子系统变化过于频繁，可能需要修改外观类，一定程度上**不符合开闭原则**。
# 使用场景
1. 为复杂的模块或子系统提供外界访问的模块。
2. 子系统相对独立。
3. 预防低水平人员带来的风险。
# 注意事项
在层次化结构中，可以使用外观模式定义系统中每一层的入口。