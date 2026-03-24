---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/状态【State】/","title":"状态【State】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:46.000+08:00","updated":"2026-03-24T17:30:08.691+08:00"}
---

# 实例
```php
abstract class State
{
    abstract public function writeProgram(Work $emergency);
}
class Work
{
    private $current, $hour, $isFinish = false;
    public function __construct(State $current)
    {
        $this->setCurrent($current);
    }
    public function getCurrent()
    {
        return $this->current;
    }
    public function setCurrent($current)
    {
        $this->current = $current;
    }
    public function getHour()
    {
        return $this->hour;
    }
    public function setHour($hour)
    {
        $this->hour = $hour;
    }
    public function isFinish(): bool
    {
        return $this->isFinish;
    }
    public function setIsFinish(bool $isFinish)
    {
        $this->isFinish = $isFinish;
    }
    public function writeProgram()
    {
        $this->current->writeProgram($this);
    }
}

class ForeNoonState extends State
{
    public function writeProgram(Work $emergency)
    {
        if (9 <= $emergency->getHour() && $emergency->getHour() < 12) {
            echo $emergency->getHour(), '点 精神饱满', PHP_EOL;
        } else {
            $emergency->setCurrent(new NoonState());
            $emergency->writeProgram();
        }
    }
}
class NoonState extends State
{
    public function writeProgram(Work $emergency)
    {
        if (12 <= $emergency->getHour() && $emergency->getHour() < 13) {
            echo $emergency->getHour(), '点 饥肠辘辘', PHP_EOL;
        } else {
            $emergency->setCurrent(new AfterNoonState());
            $emergency->writeProgram();
        }
    }
}
class AfterNoonState extends State
{
    public function writeProgram(Work $emergency)
    {
        if (13 <= $emergency->getHour() && $emergency->getHour() < 18) {
            echo $emergency->getHour(), '点 状态不错', PHP_EOL;
        } else {
            $emergency->setCurrent(new EveningState());
            $emergency->writeProgram();
        }
    }
}
class EveningState extends State
{
    public function writeProgram(Work $emergency)
    {
        if ($emergency->isFinish()) {
            $emergency->setCurrent(new RestState());
            $emergency->writeProgram();
        } else {
            if (18 <= $emergency->getHour() && $emergency->getHour() < 21) {
                echo $emergency->getHour(), '点 身心疲惫', PHP_EOL;
            } else {
                $emergency->setCurrent(new SleepState());
                $emergency->writeProgram();
            }
        }
    }
}
class SleepState extends State
{
    public function writeProgram(Work $emergency)
    {
        echo $emergency->getHour(), '点 睡觉', PHP_EOL;
    }
}
class RestState extends State
{
    public function writeProgram(Work $emergency)
    {
        echo $emergency->getHour(), '点 下班休息', PHP_EOL;
    }
}
echo '==加班状态===', PHP_EOL;
$emergency = new Work(new ForeNoonState());
$emergency->setHour(9);
$emergency->writeProgram();

$emergency->setHour(12);
$emergency->writeProgram();

$emergency->setHour(15);
$emergency->writeProgram();

$emergency->setHour(18);
$emergency->writeProgram();

$emergency->setHour(21);
$emergency->writeProgram();
echo '==准点下班状态===', PHP_EOL;
$emergency = new Work(new ForeNoonState());
$emergency->setHour(9);
$emergency->writeProgram();

$emergency->setHour(12);
$emergency->writeProgram();

$emergency->setHour(15);
$emergency->writeProgram();

$emergency->setIsFinish(true);

$emergency->setHour(18);
$emergency->writeProgram();
/*
==加班状态===
9点 精神饱满
12点 饥肠辘辘
15点 状态不错
18点 身心疲惫
21点 睡觉
==准点下班状态===
9点 精神饱满
12点 饥肠辘辘
15点 状态不错
18点 下班休息
*/
```
# 总结
状态模式通过将对象的**每一种状态封装为独立的类**，并在环境对象中持有当前状态的引用，实现了**行为随内部状态动态切换**的目的，从而消除大量的条件判断语句。
# 意图
允许对象在**内部状态发生改变**时，**改变其行为**，使对象看起来好像修改了它的类。
# 主要解决
消除代码中包含**大量与对象状态有关的条件语句**（如 `if...else` 或 `switch`），实现行为随状态改变而改变。
# 何时使用
代码中包含大量与对象状态有关的条件语句。
# 如何解决
将每一种**具体状态**抽象出来成为**独立的状态类**，并在**环境对象（Context）** 中持有当前状态的引用。环境对象将行为委托给当前状态对象执行。
# 关键代码
状态**转换逻辑与状态对象自身合成一体**，而非集中在巨大的条件语句块中。
# 优点
- **消除条件语句：** 将所有与某个状态有关的行为封装到一个类中。
- **封装转换规则：** 允许状态转换逻辑与状态对象合成一体。
- **方便新增：** 增加新状态只需增加新状态类。
# 缺点
- - **类膨胀：** 增加了系统中的类和对象的个数。
- **支持 OCP 不佳：** 增加新的可切换状态时，需要修改负责状态转换的源代码。
# 使用场景
- 对象的行为**随状态改变而改变**的场景。
- 需要作为**条件、分支语句的替代者**时。
# 注意事项
状态数量不宜过多（建议不超过 5 个），否则结构会过于复杂。

