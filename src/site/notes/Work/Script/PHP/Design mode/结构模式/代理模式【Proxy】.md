---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/代理模式【Proxy】/","title":"代理模式【Proxy】","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T11:34:34.000+08:00","updated":"2026-03-24T17:26:47.575+08:00","dg-note-properties":{"title":"代理模式【Proxy】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
// 送礼物接口
interface GiveGift
{
    public function GiveDolls();
    public function GiveChocolate();
    public function GiveFlowers();
}

// 学生妹
class SchoolGirl
{
    private $name = '';
    public function __construct($name)
    {
        $this->name = $name;
    }
    public function getName()
    {
        return $this->name;
    }
}

// 追求者实现送礼物接口
class Pursuit implements GiveGift
{
    private $girl = null;
    public function __construct(SchoolGirl $girl)
    {
        $this->girl = $girl;
    }
    public function GiveDolls()
    {
        echo $this->girl->getName(), ': 送你玩偶', PHP_EOL;
    }
    public function GiveChocolate()
    {
        echo $this->girl->getName(), ': 送你巧克力', PHP_EOL;
    }
    public function GiveFlowers()
    {
        echo $this->girl->getName(), ': 送你花花', PHP_EOL;
    }
}

// 代理实现送礼物接口
class Proxy implements GiveGift
{
    private $pursuit = null;
    public function __construct(Pursuit $pursuit)
    {
        $this->pursuit = $pursuit;
    }
    public function GiveDolls()
    {
        $this->pursuit->GiveDolls();
    }
    public function GiveChocolate()
    {
        $this->pursuit->GiveChocolate();
    }
    public function GiveFlowers()
    {
        $this->pursuit->GiveFlowers();
    }
}
$proxy = new Proxy(new Pursuit(new SchoolGirl('韩梅梅')));
$proxy->GiveDolls();
$proxy->GiveChocolate();
$proxy->GiveFlowers();
```
# 总结
代理模式通过创建一个**代理对象**作为目标对象的**中介或替身**，控制客户端对目标对象的访问，并在不修改目标对象的前提下**增加额外的控制和功能**（如安全检查、延迟加载、日志记录等）。
# 意图
为其他对象提供一种代理以控制对这个对象的访问。
# 主要解决
在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。
在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。
# 何时使用
想在访问一个类时做一些控制。
# 如何解决
增加中间层。
# 关键代码
实现与被代理类组合。
# 优点
1. 职责清晰。
2. 高扩展性。
3. 智能化。
# 缺点
1. 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。
2. 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。
# 使用场景
按职责来划分，通常有以下使用场景：
1. 远程代理。
2. 虚拟代理。
3. Copy-on-Write 代理。
4. 保护（Protect or Access）代理。 
5. Cache代理。
6. 防火墙（Firewall）代理。
7. 同步化（Synchronization）代理。
8. 智能引用（Smart Reference）代理。
# 注意事项 
1. 区别**适配器模式**：适配器模式主要**改变所考虑对象的接口**，而代理模式不能改变所代理类的接口。 
2. 区别**装饰器模式**：装饰器模式为了**增强功能**，而代理模式是为了加以控制。 