---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/创建模式/原型【Prototype】/","title":"原型【Prototype】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:45:28.000+08:00","updated":"2026-03-24T17:26:42.199+08:00","dg-note-properties":{"title":"原型【Prototype】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
// 克隆注入的company类
class Company
{
    private $company;

    public function setName($name)
    {
        $this->company = $name;
    }

    public function getName()
    {
        return $this->company;
    }
}

class Resume
{
    private $name;
    private $sex;
    private $age;
    private $timeArea;
    private $company;

    function __construct($name)
    {
        $this->name = $name;
        $this->company = new Company();
    }

    public function setPersonalInfo($sex, $age)
    {
        $this->sex = $sex;
        $this->age = $age;
        return $this;
    }

    public function setWorkExperience($timeArea, $company)
    {
        $this->timeArea = $timeArea;
        $this->company->setName($company);
        return $this;
    }

    public function display()
    {
        echo $this->name,
        ' ',
        $this->sex,
        ' ',
        $this->age, PHP_EOL;

        echo $this->timeArea,
        ' ',
        $this->company->getName(),
        PHP_EOL;
    }

    // 对引用执行深复制, 克隆后类ID不同, 防止操作同一个company类
    function __clone()
    {
        $this->company = clone $this->company;
    }
}

$resume = new Resume('佩恩');
$resume->setPersonalInfo('男', 18)
    ->setWorkExperience('2012-2018', '晓公司')
    ->display();

$resume2 = clone $resume;
$resume2->setPersonalInfo('男', 20)
    ->setWorkExperience('2018-2019', '火影公司')
    ->display();
/*
佩恩 男 18
2012-2018 晓公司
佩恩 男 20
2018-2019 火影公司
F\project\dnmp\www\localhost\test\php\t1.php77
class Resume#1 (5) {
  private $name =>
  string(6) "佩恩"
  private $sex =>
  string(3) "男"
  private $age =>
  int(18)
  private $timeArea =>
  string(9) "2012-2018"
  private $company =>
  class Company#2 (1) {
    private $company =>
    string(9) "晓公司"
  }
}
F\project\dnmp\www\localhost\test\php\t1.php78
class Resume#3 (5) {
  private $name =>
  string(6) "佩恩"
  private $sex =>
  string(3) "男"
  private $age =>
  int(20)
  private $timeArea =>
  string(9) "2018-2019"
  private $company =>
  class Company#4 (1) {
    private $company =>
    string(12) "火影公司"
  }
}
*/
```
# 总结
原型模式通过**克隆**一个**现有实例**（原型）来创建新的、可定制的对象，它**隐藏了创建细节**，并在对象初始化成本高昂时，能够**极大地提高性能**。
# 意图
**通过拷贝**（克隆）一个现有原型实例来创建新对象，无需了解创建细节。
# 主要解决
快速在**运行期**生成新对象，避免重复执行耗时或复杂的构造过程，从而提高**性能**。
# 何时使用
- 当一个系统应该独立于它的产品创建，构成和表示时。
- 当要实例化的类是在运行时刻指定时，例如，通过动态装载。
- 为了避免创建一个与产品类层次平行的工厂类层次时。
- 当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。
# 如何解决
利用已有的一个原型对象，快速地生成和原型对象一样的实例。
# 关键代码
- 必须实现**克隆操作**（PHP 中是魔术方法 `__clone()`）。
- 实现克隆操作，在 JAVA 继承 Cloneable，重写 clone()，在 .NET 中可以使用 Object 类的 MemberwiseClone() 方法来实现对象的浅拷贝或通过序列化的方式来实现深拷贝。
- 原型模式同样用于隔离类对象的使用者和具体类型（易变类）之间的耦合关系，它同样要求这些"易变类"拥有稳定的接口。
# 应用实例
- 细胞分裂。
- JAVA 中的 Object clone() 方法。
# 优点
- 性能提高。
- 逃避构造函数的约束。
# 缺点
- 配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。
- 必须实现 Cloneable 接口。
# 使用场景
- 资源优化场景。
- 类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。
- 性能和安全要求的场景。
- 通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。
- 一个对象多个修改者的场景。
- 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。
- 在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。原型模式已经与 Java 融为浑然一体，大家可以随手拿来使用。
# 注意事项
* 与传统 `new` 实例化不同，原型模式是**拷贝**现有对象。
* 在处理包含**引用对象**或**循环结构**的类时，必须仔细考虑并实现正确的**深拷贝**逻辑。