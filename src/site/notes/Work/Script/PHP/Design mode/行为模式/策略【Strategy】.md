---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/策略【Strategy】/","title":"策略【Strategy】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:04.000+08:00","updated":"2026-03-24T17:29:55.236+08:00","dg-note-properties":{"title":"策略【Strategy】","tags":["flashcards"],"reference linking":null}}
---

# 实例
## 策略模式
```php
// 策略抽象类
abstract class Strategy
{
    abstract public function algorithm(); // 算法抽象方法
}
// 算法A
class StrategyA extends Strategy
{
    public function algorithm()
    {
        echo '算法A实现', PHP_EOL;
    }
}
// 算法B
class StrategyB extends Strategy
{
    public function algorithm()
    {
        echo '算法B实现', PHP_EOL;
    }
}
// 算法C
class StrategyC extends Strategy
{
    public function algorithm()
    {
        echo '算法C实现', PHP_EOL;
    }
}
// 内容类
class Context
{
    private $strategy;

    function __construct(Strategy $strategy)
    {
        $this->strategy = $strategy;
    }

    public function show()
    {
        $this->strategy->algorithm();
    }
}
(new Context(new StrategyA()))->show(); // 算法A实现
(new Context(new StrategyB()))->show(); // 算法B实现
(new Context(new StrategyC()))->show(); // 算法C实现
```
## 策略模式和简单工厂结合
```php
// 只需要修改上方的Context类
class Context
{
    private $strategy;

    function __construct($operation)
    {
        switch ($operation) {
            case 'A':
                $this->strategy = new StrategyA();
                break;
            case 'B':
                $this->strategy = new StrategyB();
                break;
            case 'C':
                $this->strategy = new StrategyC();
                break;
            default:
                throw new Exception('Not Found' . $operation);
        }
    }
    public function show()
    {
        $this->strategy->algorithm();
    }
}
try {
    (new Context('A'))->show(); // 算法A实现
} catch (Exception $e) {
    echo $e->getMessage();
}
```
# 总结
策略模式将不同的算法或行为**封装在独立的类中**，通过实现**统一接口**，允许系统在运行时**动态、灵活地选择和替换**执行的算法，从而避免复杂的条件判断。
# 意图
定义一系列**算法**（或行为），将它们**逐个封装**成独立的类，并使它们之间可以**相互替换**。
# 主要解决
消除使用 `if...else` 或 `switch` 等**多重条件判断**带来的复杂性和难以维护性。
# 何时使用
一个系统有许多许多类，而区分它们的只是他们直接的行为。
# 如何解决
- 定义一个**统一的接口**（策略接口）。
- 将每种算法或行为封装成一个**具体策略类**，实现该接口。
- **上下文**（Context）对象持有策略引用，并负责执行选定的策略。
# 关键代码
实现同一个接口。
# 优点 
- **自由切换：** 算法（行为）可以动态替换。
- **消除判断：** 避免使用多重条件判断语句，简化了客户端逻辑。
- **高扩展性：** 增加新的算法只需添加新的策略类，符合开闭原则。
# 缺点 
- **类增多：** 每增加一种算法就需要增加一个类，可能导致策略类膨胀。
- **暴露策略：** 所有策略类都需要对客户端暴露。
# 使用场景 
1. 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 
2. 一个系统需要动态地在几种算法中选择一种。 
3. 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。
# 注意事项
如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。

