---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/迭代器【Iterator】/","title":"迭代器【Iterator】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:28.000+08:00","updated":"2026-03-24T17:29:56.728+08:00"}
---

# 实例
```php
abstract class IteratorClass
{
    protected $list = [];
    protected $index = 0;
    public function __construct($list)
    {
        $this->list = $list;
    }
    public abstract function next();
    public abstract function isDone();
}

abstract class Aggregate
{
    protected $list = [];
    public function __construct($list)
    {
        $this->list = $list;
    }
    public abstract function createIterator();
}

class ConcreteIterator extends IteratorClass
{
    public function isDone()
    {
        return $this->index < count($this->list);
    }
    public function next()
    {
        return $this->list[$this->index++];
    }
}

class ConcreteAggregate extends Aggregate
{
    public function createIterator()
    {
        return new ConcreteIterator($this->list);
    }
}
$concreteAggregate = new ConcreteAggregate(['A', 'B', 'C']);
$concreteIterator = $concreteAggregate->createIterator();
while ($concreteIterator->isDone()) {
    echo $concreteIterator->next(), PHP_EOL;
}
/*
A
B
C
*/
```
# 总结
迭代器模式通过抽象出**迭代器对象**来接管遍历职责，实现了**聚合对象**（集合）的内部结构与**外部遍历逻辑**的分离，从而提供了统一且灵活的顺序访问方式。
# 意图
**提供一种统一的方法**，顺序访问一个**聚合对象**（集合）中的各个元素，同时**无需暴露**该聚合对象的**内部表示**。
# 主要解决
将遍历聚合对象的**职责从聚合对象本身分离**，并允许以**不同的方式**遍历同一个集合。
# 何时使用
遍历一个聚合对象。
# 如何解决
- 定义**迭代器接口**（如 `hasNext()` 和 `next()`）。
- 将元素游走（遍历）的责任交给**迭代器类**，而不是聚合对象。
# 关键代码
定义接口hasNext, next。
# 优点 
- **分离职责：** 简化了聚合类，实现了数据存储和遍历的分离。
- **多重遍历：** 允许在同一个聚合对象上有**多个独立的遍历**同时进行。
- **高扩展性：** 增加新的聚合类和迭代器类都很方便。
# 缺点
由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性。
# 使用场景 
需要访问聚合对象的内容但**不暴露其内部结构**；需要为同一个聚合对象提供**多种遍历方式**。
# 注意事项
迭代器模式就是分离了集合对象的遍历行为，抽象出一个迭代器类来负责，这样既可以做到不暴露集合的内部结构，又可让外部代码透明地访问集合内部的数据。