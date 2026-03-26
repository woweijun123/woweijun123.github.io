---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/桥接模式【Bridge】/","title":"桥接模式【Bridge】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:57.000+08:00","updated":"2026-03-24T17:26:57.306+08:00","dg-note-properties":{"title":"桥接模式【Bridge】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
abstract class Soft
{
    protected $name;
    public function __construct($name)
    {
        $this->name = $name;
    }
    public abstract function run();
}
class GameSoft extends Soft
{
    public function run()
    {
        echo '运行：', $this->name, PHP_EOL;
    }
}
class ChatSoft extends Soft
{
    public function run()
    {
        echo '运行：', $this->name, PHP_EOL;
    }
}
abstract class Handset
{
    protected $name;
    public function __construct($name)
    {
        $this->name = $name;
    }
    public abstract function installSoft(Soft $soft);
    public abstract function run();
}
class HandsetA extends Handset
{
    private $soft;
    public function run()
    {
        echo '手机：', $this->name, PHP_EOL;
        foreach ($this->soft as $soft) {
            $soft->run();
        }
    }
    public function installSoft(Soft $soft)
    {
        $this->soft[] = $soft;
    }
}
class HandsetB extends Handset
{
    private $soft = [];
    public function run()
    {
        echo '手机：', $this->name, PHP_EOL;
        foreach ($this->soft as $soft) {
            $soft->run();
        }
    }
    public function installSoft(Soft $soft)
    {
        $this->soft[] = $soft;
    }
}
$gameSoft = new GameSoft('刺激战场');
$chatSoft = new ChatSoft('微信');

$handsetA = new HandsetA('小米');
$handsetB = new HandsetB('苹果');

$handsetA->installSoft($gameSoft);
$handsetA->installSoft($chatSoft);
$handsetB->installSoft($gameSoft);
$handsetB->installSoft($chatSoft);

$handsetA->run();
$handsetB->run();
/*
手机：小米
运行：刺激战场
运行：微信
手机：苹果
运行：刺激战场
运行：微信
*/
```
# 总结
桥接模式通过**合成/聚合**解耦了事物的**抽象**（做什么）和**实现**（怎么做），适用于存在**两个独立变化维度**的场景，以防止继承导致的类爆炸和僵硬。
# 意图
将一个事物的**抽象部分**与其**实现部分分离**，使二者都能独立地进行变化和扩展。
# 主要解决
解决多角度分类问题，即存在**两个或多个独立变化的维度**时，避免使用多层继承导致的**类爆炸**和不灵活。
# 何时使用
实现系统可能有多个角度分类，每一种角度都可能变化。
# 如何解决
把这种多角度分类分离出来，让它们独立变化，减少它们之间耦合。
# 关键代码
抽象类依赖实现类。
# 优点
- **解耦：** 实现了抽象和实现的彻底分离。
- **高扩展性：** 两个维度（抽象和实现）都可以独立扩展，互不影响。
# 缺点
引入了抽象层和关联关系，增加了系统的**理解和设计难度**。
# 使用场景
1. 如果一个系统需要在构建的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。
2. 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用。
3. 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。
# 注意事项
对于两个独立变化的维度，使用桥接模式再适合不过了。