---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/适配器模式【Adapter】/","title":"适配器模式【Adapter】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:01.000+08:00","updated":"2026-03-24T17:29:47.522+08:00","dg-note-properties":{"title":"适配器模式【Adapter】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
/**
 * 适配器接口，所有的支付适配器都需实现这个接口。
 * 不管第三方支付实现方式如何，对于客户端来说，都
 * 用pay()方法完成支付
 */
interface PayAdapter
{
    public static function pay();
}

// 支付宝支付类
class AliPay
{
    public function sendPayment()
    {
        echo '使用支付宝支付', PHP_EOL;
    }
}

// 支付宝适配器
class AliPayAdapter implements PayAdapter
{
    public static function pay()
    {
        // 实例化AliPay类，并用AliPay的方法实现支付
        (new AliPay())->sendPayment();
    }
}

// 微信支付类
class WeChatPay
{
    public function scan()
    {
        echo '扫描二维码后，';
    }

    public function doPay()
    {
        echo '使用微信支付', PHP_EOL;
    }
}

// 微信支付适配器
class WeChatPayAdapter implements PayAdapter
{
    public static function pay()
    {
        // 实例化WeChatPay类，并用WeChatPay的方法实现支付。
        // 注意，微信支付的方式和支付宝的支付方式不一样，但是
        // 适配之后，他们都能用pay()来实现支付功能。
        $weChatPay = (new WeChatPay());
        $weChatPay->scan();
        $weChatPay->doPay();
    }
}

// 客户端代码, 也是用pay()方法实现支付
AliPayAdapter::pay();
WeChatPayAdapter::pay();
```
# 总结
适配器模式，将一个类的接口转化成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

系统的数据和行为都正确，但接口不符时，我们应该考虑用适配器，目的是使控制范围之外的一个原有对象与某个接口匹配。适配器模式主要应用于希望复用一些现存的类。但是接口又与复用环境要求不一致的情况。
1. 两个类所做的事情相同或相似，但是具有不同的接口时要使用它。
2. 在双方都不太容易修改的时候再使用适配器模式适配。
# 意图
将一个**现存类的接口**转换成**客户端期望的另一个接口**。适配器模式的核心作用是使原本由于**接口不兼容**而不能一起工作的类能够协同工作。
# 主要解决
将“**现存对象**”置入“**新环境**”时，现存对象的**接口无法满足**新环境的**要求**。
# 何时使用
1.  当两个类功能相似，但接口不匹配时。
2.  双方都不容易修改，且有**复用现有类**的需求时。
3.  需要通过接口转换，将一个类插入到另一个类体系中。
# 如何解决
通过**继承**或**依赖/合成**（推荐）现有对象，并同时**实现目标接口**。
# 关键代码
**适配器类**继承或依赖“被适配者”（如 110V 插头），并实现“目标接口”（如 220V 插座）。
# 优点
提高了**类的复用性**和**透明度**，能让任何两个没有关联的类一起运行，灵活性好。
# 缺点
**过度使用**适配器会使系统变得**零乱和难以把握**（调用 A 接口，实际执行 B 接口的实现），应在必要时才使用。
# 使用场景
有动机地修改一个正常运行的系统的接口，这时应该考虑使用适配器模式。
# 注意事项
- 适配器模式不是在系统详细设计初期添加的，而是用于**解决正在服役的项目**中接口不匹配的问题。
- 适配器类的名称和客户端引用方式应该是**统一且稳定**的，否则会失去适配的意义。