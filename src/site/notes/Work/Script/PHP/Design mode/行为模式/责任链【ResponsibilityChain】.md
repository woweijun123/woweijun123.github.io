---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/责任链【ResponsibilityChain】/","title":"责任链【ResponsibilityChain】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:26.000+08:00","updated":"2026-03-24T17:30:06.127+08:00","dg-note-properties":{"title":"责任链【ResponsibilityChain】","tags":["flashcards"],"reference linking":null}}
---

# 实例
## 命令链抽象类
实现设置继承者`setSuccessor()`方法,、保存继承者
## 命令类
每个都**必须实现**命令链抽象类的 **`handleRequest()`抽象方法** , 若不能处理该请求则转移请求
```php
# +----------------------------------------------------------------------
# | 责任链模式「单向」
# +----------------------------------------------------------------------
abstract class CommandChain
{
    protected $successor;

    public function setSuccessor(CommandChain $successor)
    {
        $this->successor = $successor;
    }

    abstract public function handleRequest(int $request);
}

// 如果可以处理请求，就处理之，否者转发给它的后继者
class CommandA extends CommandChain
{
    public function handleRequest(int $request)
    {
        if (0 <= $request && $request < 5) {
            echo 'CommandA handle it', PHP_EOL;
        } elseif ($this->successor != null) {
            $this->successor->handleRequest($request); // 转移
        } else {
            echo 'No processing', PHP_EOL;
        }
    }
}

class CommandB extends CommandChain
{
    public function handleRequest(int $request)
    {
        if (6 <= $request && $request < 10) {
            echo 'CommandB handle it', PHP_EOL;
        } elseif ($this->successor != null) {
            $this->successor->handleRequest($request);
        } else {
            echo 'No processing', PHP_EOL;
        }
    }
}

# +----------------------------------------------------------------------
# | 责任链模式变种(洋葱模型、拦截过滤器模式)「U型环」
# +----------------------------------------------------------------------
class Context {
    public int $index = -1;
    public array $commands;

    public function __construct(array $commands)
    {
        $this->commands = $commands;
    }

    public function next()
    {
        $this->index++;
        if ($this->index < count($this->commands)) {
            $this->commands[$this->index]->handle($this);
        }
    }
}

abstract class Command
{
    public function handle(Context $context)
    {
        echo 'Command', PHP_EOL;
        $context->next();
    }
}

class Command1 extends Command
{
    public function handle(Context $context)
    {
        echo 'Command1 in', PHP_EOL;
        $context->next();
        echo 'Command1 out', PHP_EOL;
    }
}

class Command2 extends Command
{
    public function handle(Context $context)
    {
        echo 'Command2 in', PHP_EOL;
        parent::handle($context);
        echo 'Command2 out', PHP_EOL;
    }
}

// 责任链模式「单向」
$commandA = new CommandA();
$commandA->setSuccessor(new CommandB());
$requests = [1, 6, 10];
foreach ($requests as $value) {
    $commandA->handleRequest($value);
}
/*
CommandA handle it
CommandB handle it
No processing
*/

echo '====华丽的分隔符====', PHP_EOL;

// 责任链模式变种(洋葱模型、拦截过滤器模式)「U型环」
$context = new Context([new Command1(), new Command2()]);
$context->next();
/*
Command1 in
Command2 in
Command2 out
Command1 out
*/
```
# 总结
职责链模式通过将处理对象**串联**起来，使请求沿着**链条依次传递**，从而实现了**请求的发送者与多个潜在处理者**之间的高度解耦，并提供了**灵活可变**的请求处理结构。
# 意图
将多个对象连成一条**处理链**，使它们都有机会处理请求，并沿着链传递请求，直到**有对象处理为止**。
# 主要解决
彻底解除**请求发送者和接收者之间的耦合**。发送者只需将请求发到链上，无需关心谁来处理或链的结构。
# 何时使用
在处理消息的时候以过滤很多道。
# 如何解决
1. 所有处理类实现**统一接口**。 
2. 每个处理类中**聚合自身**（持有指向后继者的引用）。 
3. 处理者在 `HandlerRequest` 方法中判断是否处理；不满足条件则**向下传递**给后继者。
# 关键代码
Handler 里面聚合它自己，在 HandlerRequest 里判断是否合适，如果没达到条件则向下传递，向谁传递之前 set 进去。
# 优点
- **低耦合：** 发送者和接收者完全解耦。
- **高灵活性：** 随时可以**动态增删或修改**链中的处理者及其顺序。
- **简化对象：** 对象只需维护一个后继者的引用。
# 缺点 
- **不能保证处理：** 请求可能传到链末端都得不到处理。
- **性能影响：** 链条过长可能影响系统性能和调试难度，且可能造成循环调用。
# 使用场景 
- **多个对象**可以处理同一个请求，且具体由哪个对象处理在**运行时自动确定**。
- 需要在**不明确指定接收者**的情况下，向多个对象中的一个提交请求。