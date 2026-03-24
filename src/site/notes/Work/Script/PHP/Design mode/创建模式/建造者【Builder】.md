---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/创建模式/建造者【Builder】/","title":"建造者【Builder】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:48.000+08:00","updated":"2026-03-24T17:26:36.872+08:00"}
---

# 实例
```php
// 画小人
abstract class PersonBuilder
{
    abstract public function buildHead();
    abstract public function buildBody();
    abstract public function buildArmLeft();
    abstract public function buildArmRight();
    abstract public function buildLegLeft();
    abstract public function buildLegRight();
}

class PersonThinBuilder extends PersonBuilder
{
    public function buildHead()
    {
        echo '小头', PHP_EOL;
    }
    public function buildBody()
    {
        echo '小身子', PHP_EOL;
    }
    public function buildArmLeft()
    {
        echo '小左臂', PHP_EOL;
    }
    public function buildArmRight()
    {
        echo '小右臂', PHP_EOL;
    }
    public function buildLegLeft()
    {
        echo '小左腿', PHP_EOL;
    }
    public function buildLegRight()
    {
        echo '小右腿', PHP_EOL;
    }
}

class PersonFatBuilder extends PersonBuilder
{
    public function buildHead()
    {
        echo '大头', PHP_EOL;
    }
    public function buildBody()
    {
        echo '大身子', PHP_EOL;
    }
    public function buildArmLeft()
    {
        echo '大左臂', PHP_EOL;
    }
    public function buildArmRight()
    {
        echo '大右臂', PHP_EOL;
    }
    public function buildLegLeft()
    {
        echo '大左腿', PHP_EOL;
    }
    public function buildLegRight()
    {
        echo '大右腿', PHP_EOL;
    }
}

// 导演者
class PersonDirector
{
    private $personBuilder;
    public function __construct(PersonBuilder $personBuilder)
    {
        $this->personBuilder = $personBuilder;
    }
    public function createPerson()
    {
        $this->personBuilder->buildHead();
        $this->personBuilder->buildBody();
        $this->personBuilder->buildArmLeft();
        $this->personBuilder->buildArmRight();
        $this->personBuilder->buildLegLeft();
        $this->personBuilder->buildLegRight();
    }
}
(new PersonDirector(new PersonThinBuilder()))->createPerson();
echo '=====华丽的分隔符=====',PHP_EOL;
(new PersonDirector(new PersonFatBuilder()))->createPerson();
/*
小头
小身子
小左臂
小右臂
小左腿
小右腿
=====华丽的分隔符=====
大头
大身子
大左臂
大右臂
大左腿
大右腿
*/
```
# 总结
建造者模式通过**导演者**固定装配步骤，让**建造者**提供不同零件，灵活地创建具有**复杂且固定结构**的对象。
# 意图
将一个**复杂对象**的**构建步骤**（流程稳定）与其**具体表现**（零件可变）相分离。
# 主要解决
当复杂对象的各个**部件变化剧烈**，但**组装顺序**相对稳定时，用于管理创建过程。
# 何时使用
一些基本部件不会变，而其组合经常变化的时候。
# 如何解决
将变与不变分离开。
# 关键代码
1.  **建造者 (Builder)：** 定义零件创建方法（如 `buildHead()`）。
2.  **导演者 (Director)：** 接收建造者，并按固定顺序调用零件方法，**管理组装流程**。
# 应用实例
1. 去肯德基，汉堡、可乐、薯条、炸鸡翅等是不变的，而其组合是经常变化的，生成出所谓的"套餐"。 
2. JAVA 中的 StringBuilder。
# 优点
* **易于扩展：** 增加新的产品表示（如新体型），只需增加新的建造者子类，无需修改导演者（遵守 OCP）。
* **控制细节：** 便于精细控制复杂对象的创建过程。
# 缺点
1. 产品必须有共同点，范围有限制。 
2. 如内部变化复杂，会有很多的建造类。
# 使用场景
1. 需要生成的对象具有复杂的内部结构。
2. 需要生成的对象内部属性本身相互依赖。
# 注意事项
建造者模式**更关注部件的装配顺序和步骤**，而工厂模式主要关注创建哪个对象。

