---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/命令【Command】/","title":"命令【Command】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:05.000+08:00","updated":"2026-03-24T17:30:02.667+08:00"}
---

# 实例
```php
class Barbecue
{
    public function bakeMutton()
    {
        echo '烤羊肉', PHP_EOL;
    }
    public function bakeChickenWing()
    {
        echo '烤鸡翅', PHP_EOL;
    }
}
abstract class Command
{
    protected $receiver;
    function __construct(Barbecue $receiver)
    {
        $this->receiver = $receiver;
    }
    abstract public function exec();
}
class BakeMuttonCommand extends Command
{
    public function exec()
    {
        $this->receiver->bakeMutton();
    }
}
class BakeChickenWingCommand extends Command
{
    public function exec()
    {
        $this->receiver->bakeChickenWing();
    }
}
class Waiter
{
    private $commands = [];
    public function setOrder(Command $command)
    {
        if ($command instanceof BakeChickenWingCommand) {
            echo '服务员： 鸡翅没有了，请点别的烧烤', PHP_EOL;
        } else {
            echo '增加订单', PHP_EOL;
            array_push($this->commands, $command);
        }
    }
    public function cancelOrder(Command $command)
    {
        $k = array_search($command, $this->commands);
        if ($k !== false) {
            unset($this->commands[$k]);
        }
    }
    public function notify()
    {
        foreach ($this->commands as $value) {
            $value->exec();
        }
    }
}
// 开店前准备
$boy = new Barbecue();
$bakeMutton1 = new BakeMuttonCommand($boy); // +烤羊肉
$bakeMutton2 = new BakeMuttonCommand($boy); // +烤羊肉
$bakeChickenWing1 = new BakeChickenWingCommand($boy); // +烤鸡翅
// 开门营业
$girl = new Waiter(); // 服务员类
$girl->setOrder($bakeMutton1); // 添加烤羊肉订单
$girl->setOrder($bakeMutton2); // 添加烤羊肉订单
$girl->setOrder($bakeChickenWing1); // 添加烤鸡翅订单
$girl->cancelOrder($bakeMutton2); // 取消烤羊肉订单
$girl->notify(); // 通知开始烤
/*
增加订单
增加订单
服务员： 鸡翅没有了，请点别的烧烤
烤羊肉
*/
```
# 总结
命令模式将一个操作请求**封装成独立的对象**，实现了请求者与执行者的彻底解耦，从而方便地支持**日志记录、队列排队**以及**撤销/重做**等复杂功能。
# 意图
将一个**请求**（行为）**封装为一个对象**，从而使客户端可以通过不同的请求对象对操作进行**参数化**。
# 主要解决
将**行为请求者 (Invoker)** 与**行为实现者 (Receiver)** 解耦，解决紧耦合关系不适合处理**记录、撤销、重做、事务**等需求的问题。
# 何时使用
在某些场合，比如要对行为进行 "记录、撤销 / 重做、事务" 等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将 "行为请求者" 与 "行为实现者" 解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。
# 如何解决
1. **接受者 (Receiver)：** 真正的命令执行者。
2. **命令对象 (Command)：** 封装请求。
3. **调用者 (Invoker)：** 使用命令对象的入口。
- **流程**：调用者 $\rightarrow$ 命令 $\rightarrow$ 接受者执行。
# 关键代码
定义三个角色：
1. received 真正的命令执行对象
2. Command
3. invoker 使用命令对象的入口
# 优点
- **低耦合：** 将请求操作的对象与执行操作的对象分离。
- **功能扩展：** 易于实现**命令队列**、**日志记录**以及关键的**撤销 (Undo)** 和**重做 (Redo)** 功能。
- **易扩展：** 增加新的命令类不会影响现有类。
# 缺点
使用命令模式可能会导致某些系统有过多的具体命令类。
# 使用场景
需要对行为进行**记录、撤销、重做**或**事务处理**时；GUI 中按钮点击、宏指令等。
# 注意事项
系统需要支持命令的撤销 (Undo) 操作和恢复 (Redo) 操作，也可以考虑使用命令模式，见命令模式的扩展。