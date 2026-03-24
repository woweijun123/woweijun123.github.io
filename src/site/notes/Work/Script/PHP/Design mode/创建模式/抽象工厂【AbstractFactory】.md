---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/创建模式/抽象工厂【AbstractFactory】/","title":"抽象工厂【AbstractFactory】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:23.000+08:00","updated":"2026-03-24T17:26:26.246+08:00"}
---

# 实例
```php
class System{}
class Soft{}

class MacSystem extends System{}
class MacSoft extends Soft{}

class WinSystem extends System{}
class WinSoft extends Soft{}

// 抽象工厂模式
interface AbstractFactory {
    public function CreateSystem();
    public function CreateSoft();
}
// 苹果工厂
class MacFactory implements AbstractFactory {
    public function CreateSystem(){ return new MacSystem(); }
    public function CreateSoft(){ return new MacSoft(); }
}
// windows工厂
class WinFactory implements AbstractFactory {
    public function CreateSystem(){ return new WinSystem(); }
    public function CreateSoft(){ return new WinSoft(); }
}

// 创建MacFactory工厂
$MacFactory = new MacFactory();
// 用MacFactory工厂分别创建不同对象
var_dump($MacFactory->CreateSystem());
var_dump($MacFactory->CreateSoft());

//创建WinFactory
$WinFactory = new WinFactory();
//用WinFactory工厂分别创建不同对象
var_dump($WinFactory->CreateSystem());
var_dump($WinFactory->CreateSoft());

/*
object(MacSystem)#2 (0) { }
object(MacSoft)#2 (0) { }
object(WinSystem)#3 (0) { }
object(WinSoft)#3 (0) { }
*/
```
# 总结
所有在用简单工厂的地方，都可以考虑用反射技术来去除switch或if，解除分支判断代码的耦合。
# 意图
提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
- 主要解决：主要解决接口选择的问题。
- 何时使用：系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。
- 如何解决：在一个产品族里面，定义多个产品。
- 关键代码：在一个工厂里聚合多个同类产品。
# 优点
当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。
# 缺点
产品族扩展非常困难，要增加一个系列的某一产品，既要在抽象的 Creator 里加代码，又要在具体的里面加代码。
# 使用场景
1、QQ 换皮肤，一整套一起换。 
2、生成不同操作系统的程序。
# 注意事项
产品族难扩展，产品等级易扩展。
# 应用实例
抽象工厂模式提供一个**创建一系列相关或相互依赖对象的接口**（如衣柜），而无需指定它们的**具体类**（如商务装或时尚装），保证客户端使用的**一整套产品族**（如上衣、裤子）是协调一致的。