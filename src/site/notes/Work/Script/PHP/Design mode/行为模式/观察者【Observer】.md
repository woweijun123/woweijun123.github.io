---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/观察者【Observer】/","title":"观察者【Observer】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:12.000+08:00","updated":"2026-03-24T17:29:59.324+08:00"}
---

# 实例
1. 被观察者：至少要实现实现Observable接口规定的attach()、detach()、notify()三个方法，用以添加、删除和通知观察者。
2. 观察者：每个都必须实现Observer接口规定的update()方法
```php
interface Observable
{
    public function attach(Observer $observer); // 添加
    public function detach(Observer $observer); // 删除
    public function notify(); // 触发通知
    public function getState(); // 获取状态
}
interface Observer
{
    public function update(Observable $observable);
}
/**
 * 被观察者
 * 职责：添加观察者到$observers属性中，
 * 有变动时通过notify()方法运行通知
 */
class Order implements Observable
{
    private $observers = []; // 保存观察者
    private $state = 0; // 订单状态
    // 添加（注册）观察者
    public function attach(Observer $observer)
    {
        $key = array_search($observer, $this->observers);
        if ($key === false) {
            $this->observers[] = $observer;
        }
    }
    // 移除观察者
    public function detach(Observer $observer)
    {
        $key = array_search($observer, $this->observers);
        if ($key !== false) {
            unset($this->observers[$key]);
        }
    }
    // 遍历调用观察者的update()方法进行通知，不关心其具体实现方式
    public function notify()
    {
        foreach ($this->observers as $observer) {
            // 把本类对象传给观察者，以便观察者获取当前类对象的信息
            $observer->update($this);
        }
    }
    // 订单状态有变化时发送通知
    public function addOrder()
    {
        $this->state = 1;
        $this->notify();
    }
    // 获取提供给观察者的状态
    public function getState()
    {
        return $this->state;
    }
}
class Email implements Observer
{
    public function update(Observable $observable)
    {
        $state = $observable->getState();
        if ($state) {
            echo '发送邮件：您已经成功下单。', PHP_EOL;
        } else {
            echo '发送邮件：下单失败，请重试。', PHP_EOL;
        }
    }
}
class Message implements Observer
{
    public function update(Observable $observable)
    {
        $state = $observable->getState();
        if ($state) {
            echo '短信通知：您已下单成功。', PHP_EOL;
        } else {
            echo '短信通知：下单失败，请重试。', PHP_EOL;
        }
    }
}
class Log implements Observer
{
    public function update(Observable $observable)
    {
        echo '记录日志：生成了一个订单记录。', PHP_EOL;
    }
}
$email = new Email(); // 创建Email观察者对象
$message = new Message(); // 创建Message观察者对象
$log = new Log(); // 创建Log观察者对象

$order = new Order(); // 创建订单对象

// 向订单对象中注册3个观察者：发送邮件、短信通知、记录日志
$order->attach($email);
$order->attach($message);
$order->attach($log);
$order->addOrder(); // 添加订单，添加时会自动发送通知给观察者

echo '====华丽的分隔符====', PHP_EOL;

$order->detach($log); // 删除记录日志观察者
$order->addOrder(); // 添加另一个订单，会再次发送通知给观察者
```
# 总结
观察者模式通过目标对象维护并自动通知所有依赖的观察者对象，建立了**一对多、抽象耦合**的订阅/发布机制，从而实现状态变化的**广播通知**。
# 意图
定义对象间 **“一对多”的依赖关系**。当**目标对象**（Subject，又称被观察者）状态改变时，**所有依赖的对象**（Observer，又称观察者）都会被自动通知并更新。
# 主要解决
解决一个对象状态改变后，如何高效、低耦合地**通知所有依赖对象**的问题，建立**广播通知**机制。
# 何时使用
一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。
# 如何解决
- **目标对象**维护一个列表（如 `ArrayList`）存放所有观察者。
- 当状态变化时，目标对象**遍历列表**并调用观察者的更新方法。
- 实现了观察者与目标对象的**抽象耦合**。
# 关键代码
在抽象类里有一个 ArrayList 存放观察者们。
# 优点
实现了对象间的**低耦合**；建立了一套**触发机制**。
# 缺点
- 通知所有观察者可能**耗费时间**；需注意**避免循环依赖**导致的系统崩溃。
- 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行**循环调用**，可能导致**系统崩溃**。
- 没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。
# 使用场景
- 一个对象的改变需要**动态导致**其他一个或多个对象发生改变，但事先**不知道**具体有多少对象。
- 需要创建一个**触发链**（A 影响 B，B 影响 C），并降低对象间耦合度。
# 注意事项
- JAVA 中已经有了对观察者模式的支持类。
- 避免循环引用。
- 如果顺序执行，某一观察者错误会导致系统卡壳，一般采用异步方式。