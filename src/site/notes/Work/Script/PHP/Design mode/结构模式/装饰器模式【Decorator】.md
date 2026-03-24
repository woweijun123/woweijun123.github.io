---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/装饰器模式【Decorator】/","title":"装饰器模式【Decorator】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:39.000+08:00","updated":"2026-03-24T17:29:51.763+08:00"}
---

# 实例
```php
// 行为接口
interface Behavior
{
    public function show(); // 展示动作
}
// 人类
class Person implements Behavior
{
    private $name;

    function __construct($name)
    {
        $this->name = $name;
    }

    public function show()
    {
        echo '打扮者:', $this->name, PHP_EOL;
    }
}
// 服饰类
class Finery implements Behavior
{
    protected $person;

    public function __construct(Behavior $person)
    {
        $this->person = $person;
    }

    public function show()
    {
        if ($this->person != null) $this->person->show();
    }
}

// 具体服饰类
class Cap extends Finery
{
    public function show()
    {
        echo '红帽', PHP_EOL;
        parent::show();
    }
}

class Suit extends Finery
{
    public function show()
    {
        echo '西装', PHP_EOL;
        parent::show();
    }
}
class Tie extends Finery
{
    public function show()
    {
        echo '领带', PHP_EOL;
        parent::show();
    }
}

(new Tie(new Suit(new Cap(new Person('riven')))))->show();
/*
领带
西装
红帽
打扮者:riven
*/
```
# 总结
装饰模式通过创建**包装类**，在**运行时动态且灵活地**为对象添加功能和职责，从而避免了使用继承所带来的类膨胀和静态耦合问题。
# 意图
**动态地**为一个对象添加**额外的职责或功能**，比使用继承（生成子类）更为灵活。
# 主要解决
避免使用继承导致的**类爆炸和静态特征**，解决了为满足特殊行为而向核心类中添加冗余代码的问题。
# 何时使用
在不想增加很多子类的情况下扩展类。
# 如何解决
1. 将每个要添加的**装饰功能**放置在独立的**装饰类**中。
2. 装饰类**引用并继承/实现**被装饰对象（抽象）的接口。
3. 在**运行时**，客户端可以根据需要**选择性地、按顺序地包装**对象，动态赋予新功能。
# 关键代码 
1. Component 类充当抽象角色，不应该具体实现。 
2. 修饰类引用和继承 Component 类，具体扩展类重写父类方法。
# 优点
- **简化原类：** 将装饰功能从核心类中搬离，简化了核心类职责。
- **动态灵活：** 可以动态增加或撤销功能。
- **替代继承：** 装饰模式是继承的一种灵活替代方案，解决了继承的**静态特性**限制。
# 缺点
多层包装和装饰会使系统的**复杂度提高**。
# 使用场景 
1. 扩展一个类的功能。 
2. 动态增加功能，动态撤销。
# 注意事项
可代替继承。