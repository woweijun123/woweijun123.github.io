---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/中介者【Mediator】/","title":"中介者【Mediator】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:45:12.000+08:00","updated":"2026-03-24T17:30:07.555+08:00"}
---

# 实例
```php
// 中介者
abstract class Mediator
{
    abstract public function send($message, Colleague $colleague);
}
// 同事
abstract class Colleague
{
    protected $mediator;

    public function __construct(Mediator $mediator)
    {
        $this->mediator = $mediator;
    }

    // 通知
    abstract public function notify($message);
}
// 具体中介者
class ConcreteMediator extends Mediator
{
    private $colleague1, $colleague2;

    public function setColleague1(Colleague $colleague)
    {
        $this->colleague1 = $colleague;
    }

    public function setColleague2(Colleague $colleague)
    {
        $this->colleague2 = $colleague;
    }

    public function send($message, Colleague $colleague)
    {
        if ($this->colleague1 == $colleague) {
            $this->colleague1->notify($message);
        } else {
            $this->colleague2->notify($message);
        }
    }
}
// 具体同事1
class ConcreteColleague1 extends Colleague
{
    public function send($message)
    {
        $this->mediator->send($message, $this);
    }

    public function notify($message)
    {
        echo self::class . ':', $message, PHP_EOL;
    }
}
// 具体同事2
class ConcreteColleague2 extends Colleague
{
    public function send($message)
    {
        $this->mediator->send($message, $this);
    }

    public function notify($message)
    {
        echo self::class . ':', $message, PHP_EOL;
    }
}
// 具体中介者
$mediator = new ConcreteMediator();
$c1 = new ConcreteColleague1($mediator);
$c2 = new ConcreteColleague2($mediator);
$mediator->setColleague1($c1);
$mediator->setColleague2($c2);

$c1->send('do you eat?');
$c2->send('no, do you want to invite me to dinner?');
```
# 总结
中介者模式通过引入一个**中介对象**来集中封装和管理**复杂的对象间交互**，将网状耦合转化为简单的星型结构，从而降低了系统复杂度和对象间的依赖关系。
# 意图
引入一个**中介对象**来**封装一系列的对象交互**，使各对象无需显式地相互引用，从而实现**松散耦合**。
# 主要解决
解决对象之间存在**大量复杂的网状关联关系**时，导致的系统结构复杂、难以复用和维护的问题。
# 何时使用
多个类相互耦合，形成了网状结构。
# 如何解决
将复杂的**多对多**交互结构分离为简单的**星型结构**。所有对象（同事类）不再直接通信，而是只与**中介者**通信，由中介者集中处理协作逻辑。
# 关键代码
对象 Colleague 之间的通信封装到一个类中单独处理。
# 优点 
- **降低耦合：** 将网状依赖转化为一对一（对象到中介者）依赖，高度解耦。
- **降低复杂度：** 符合迪米特原则，将协作行为抽象出来，简化了同事类。
- **宏观视角：** 方便站在宏观角度定制和改变对象间的交互。
# 缺点
中介者类会承载所有交互逻辑，**自身可能变得过于庞大和复杂**，难以维护。
# 使用场景
- 系统中对象间存在**复杂的引用关系**和混乱的依赖结构。
- 想通过一个**中间类**来封装多个类中的行为，同时**避免生成大量子类**。
# 注意事项
不要在系统设计不合理、职责混乱时急于使用，应首先反思系统设计。