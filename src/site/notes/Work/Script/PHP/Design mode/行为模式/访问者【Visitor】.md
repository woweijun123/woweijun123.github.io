---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/访问者【Visitor】/","title":"访问者【Visitor】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:21.000+08:00","updated":"2026-03-24T17:29:58.085+08:00","dg-note-properties":{"title":"访问者【Visitor】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
abstract class Visitor
{
    public abstract function getManConclusion(Man $person);
    public abstract function getWomanConclusion(Woman $person);
}
abstract class Person
{
    abstract public function accept(Visitor $visitor);
}
class Success extends Visitor
{
    public function getManConclusion(Man $person)
    {
        echo '背后多半有个伟大的女人', PHP_EOL;
    }
    public function getWomanConclusion(Woman $person)
    {
        echo '背后多半有个弱小的男人', PHP_EOL;
    }
}
class Failing extends Visitor
{
    public function getManConclusion(Man $person)
    {
        echo '闷头喝酒, 谁也劝不动', PHP_EOL;
    }
    public function getWomanConclusion(Woman $person)
    {
        echo '眼泪汪汪, 谁也劝不了', PHP_EOL;
    }
}
class Love extends Visitor
{
    public function getManConclusion(Man $person)
    {
        echo '凡事不懂装懂', PHP_EOL;
    }
    public function getWomanConclusion(Woman $person)
    {
        echo '凡事懂装不懂', PHP_EOL;
    }
}
class Man extends Person
{
    public function accept(Visitor $visitor)
    {
        $visitor->getManConclusion($this);
    }
}
class Woman extends Person
{
    public function accept(Visitor $visitor)
    {
        $visitor->getWomanConclusion($this);
    }
}
class ObjectStructure
{
    private $person = [];
    public function attach(Person $person)
    {
        array_push($this->person, $person);
    }
    public function display(Visitor $visitor)
    {
        foreach ($this->person as $person) {
            $person->accept($visitor);
        }
    }
}
$object = new ObjectStructure();
$object->attach(new Man());
$object->attach(new Woman());
$object->display(new Success());
$object->display(new Failing());
$object->display(new Love());
/*
背后多半有一个伟大的女人
背后多半有一个不成功的男人
男人失败时, 闷头喝酒, 谁也劝不动
女人失败时, 眼泪汪汪, 谁也劝不了
男人恋爱时, 凡事不懂也要装懂
女人恋爱时, 凡事都懂也要装傻
*/
```
# 总结
访问者模式将**作用于稳定数据结构上的易变操作**封装到独立的**访问者类**中，通过元素类提供的“接待”接口，实现了**不修改数据结构即可灵活扩展新功能**的目的。
# 意图
表示一个作用于**对象结构中各个元素**的**操作**，它允许你在**不改变元素类**的前提下，**定义新的操作**。
# 主要解决
**数据结构稳定**（对象元素类少变）和**数据操作易变**（算法经常变化）之间的耦合问题。
# 何时使用
需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作 "污染" 这些对象的类，使用访问者模式将这些封装到类中。
# 如何解决
- **被访问元素 (Element)：** 提供一个**接受访问者**的接口，将自身的引用传入访问者。
- **访问者 (Visitor)：** 将所有相关的**行为（操作）集中**在一个对象中，实现针对不同元素的不同操作。
# 关键代码
在数据基础类里面有一个方法接受访问者，将自身引用传入访问者。
# 优点
- **优秀的扩展性：** **增加新操作**（新访问者）非常容易，符合**单一职责原则**。
- **解耦：** 有效地将数据结构和操作分离。
# 缺点
- **元素难变：** 增加新的**数据结构**（元素类）非常困难。
- **公布细节：** 元素类必须对外公布细节来接受访问者，可能违反**迪米特原则**和**依赖倒置原则**。
# 使用场景
对象结构中的**元素类很少改变**，但需要经常在这些元素上**定义新的、不相关的操作**，且不希望“污染”元素类本身。
# 注意事项
访问者可以对功能进行统一，可以做报表、UI、拦截器与过滤器。