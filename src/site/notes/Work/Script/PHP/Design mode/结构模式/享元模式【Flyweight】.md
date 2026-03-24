---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/享元模式【Flyweight】/","title":"享元模式【Flyweight】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:44:52.000+08:00","updated":"2026-03-24T17:29:49.717+08:00"}
---

# 实例
```php
abstract class WebSite
{
    protected $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public abstract function use(User $user);
}

class ConcreteWebSite extends WebSite
{
    public function use(User $user)
    {
        echo '分类:', $this->name, PHP_EOL;
        echo '用户:', $user->getName(), PHP_EOL;
    }
}

class WebSiteFactory
{
    private $flyweight = [];

    public function getWebSiteCategory($key)
    {
        if (empty($this->flyweight[$key])) {
            $this->flyweight[$key] = new ConcreteWebSite($key);
        }

        return $this->flyweight[$key];
    }

    public function getWebSiteCount()
    {
        echo '分类总数:', count($this->flyweight), PHP_EOL;
    }
}

class User
{
    private $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }
}

$webSiteFactory = new WebSiteFactory();

$videoSite = $webSiteFactory->getWebSiteCategory('video');
$videoSite->use(new User('小红'));

$liveSite = $webSiteFactory->getWebSiteCategory('live');
$liveSite->use(new User('小明'));

$liveSite = $webSiteFactory->getWebSiteCategory('video');
$liveSite->use(new User('小牛'));

$liveSite = $webSiteFactory->getWebSiteCategory('live');
$liveSite->use(new User('小黄'));

$webSiteFactory->getWebSiteCount();
/*
分类:video
用户:小红
分类:live
用户:小明
分类:video
用户:小牛
分类:live
用户:小黄
分类总数:2
*/
```
# 总结
享元模式通过**工厂集中控制**，将对象的**共享部分（内部状态）抽象出来**并复用，将**变化部分（外部状态）作为参数传入**，以极少的对象实例高效地管理海量细粒度对象。
# 意图
运用**共享技术**，高效支持**大量细粒度的对象**。
# 主要解决
避免创建大量相似对象导致的**内存溢出**和存储开销。
# 何时使用
1. 系统中有大量对象。
2. 这些对象消耗大量内存。
3. 这些对象的状态大部分可以外部化。
4. 这些对象可以按照内蕴状态分为很多组，当把外蕴对象从对象中剔除出来时，每一组对象都可以用一个对象来代替。
5. 系统不依赖于这些对象身份，这些对象是不可分辨的。
# 如何解决
- 将对象状态划分为**内部状态**（可共享，存储在享元对象内）和**外部状态**（不可共享，作为方法参数传入）。
- 使用**工厂对象**（如 `HashMap` 存储）来控制创建，通过**唯一标识码**判断并返回内存中已有的共享对象。
# 关键代码
用 HashMap 存储这些对象。
# 优点
大幅**减少对象创建**，显著降低系统内存占用，提高效率。
# 缺点
提高了系统**复杂性**，必须准确划分内部与外部状态，且需警惕**线程安全**问题。
# 使用场景
- 系统存在**大量相似对象**且**消耗内存巨大**。
- 对象的大多数状态可以被**外部化**（外部化后，少数共享对象即可取代多数原始对象）。
- 适用于**缓冲池**场景（如 Java 字符串常量池、数据库连接池）。
# 注意事项
1. 注意划分外部状态和内部状态，否则可能会引起线程安全问题。
2. 这些类必须有一个工厂对象加以控制。 