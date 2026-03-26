---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/备忘录【Memento】/","title":"备忘录【Memento】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:35.000+08:00","updated":"2026-03-24T17:29:53.852+08:00","dg-note-properties":{"title":"备忘录【Memento】","tags":["flashcards"],"reference linking":null}}
---

# 实例
1. 发起人类：创建备忘录，恢复备忘录
2. 备忘录类：设置状态，获取状态
3. 管理者类：设置备忘录，获取备忘录
注: 备忘录模式如果要备份对象,则必须用clone, 否则原始备份对象更改会影响备忘录里保存的对象(因为传对象默认为引用传值)
```php
// 发起人类
class Originator
{
    private $state;
    public function setState($state)
    {
        $this->state = $state;
    }
    public function getState()
    {
        return $this->state;
    }
    // 保存并创建备忘录
    public function setMemento()
    {
        return new Memento($this->getState());
    }
    // 恢复备忘录，将memento导入并将相关数据恢复
    public function getMemento(Memento $memento)
    {
        $this->state = $memento->getState();
    }
    // 显示数据
    public function show()
    {
        echo 'status ', $this->state, PHP_EOL;
    }
}
// 备忘录类
class Memento
{
    private $state;
    function __construct($state)
    {
        $this->state = $state;
    }
    public function getState()
    {
        return $this->state;
    }
}
// 管理者类
class CareTaker
{
    private $memento;
    public function getMemento()
    {
        return $this->memento;
    }
    public function setMemento(Memento $memento)
    {
        $this->memento = $memento;
    }
}
$originator = new Originator(); // Originator初始状态，状态属性on
$originator->setState('On');
$originator->show();
// 保存状态时，由于有了很好的封装，可以隐藏Originator的实现细节
$carataker = new CareTaker();
$carataker->setMemento($originator->setMemento());
// 改变属性
$originator->setState('Off');
$originator->show();
// 恢复属性
$originator->getMemento($carataker->getMemento());
$originator->show();
```
# 总结
备忘录模式在**不暴露对象内部细节**的前提下，通过专门的备忘录对象**存储和管理**历史状态，从而提供**可恢复和撤销**的能力，实现“后悔药”机制。
# 意图
在**不破坏封装性**的前提下，捕获一个对象的**内部状态**，并在该对象**之外**保存，以便将来能够**恢复**到该历史状态。
# 主要解决
提供“**后悔药**”机制，允许用户取消不确定或错误的操作，回到先前状态。
# 何时使用
很多时候我们总是需要记录一个对象的内部状态，这样做的目的就是为了允许用户取消不确定或者错误的操作，能够恢复到他原先的状态，使得他有 "后悔药" 可吃。
# 如何解决
引入一个**备忘录类**专门存储状态，并使用一个**管理类**来存储和管理这些备忘录对象（符合迪米特原则）。
# 关键代码
客户不与备忘录类耦合，与备忘录管理类耦合。
# 优点
- 向用户提供了**状态恢复**机制。
- 实现了信息的**封装**，用户无需关心状态保存的细节。
# 缺点
消耗资源。如果状态信息过多，每次保存都会占用大量的**内存**。
# 使用场景
需要**保存/恢复**数据的相关状态，或提供**可回滚**操作的场景（如游戏存档、Ctrl + Z 撤销、数据库事务）。
# 注意事项
可与**命令模式**结合实现撤销功能；可与**原型模式**结合使用，利用克隆来优化备忘录的创建过程，从而**节约内存**。