---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/结构模式/合成模式【Composite】/","title":"合成模式【Composite】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:46:29.000+08:00","updated":"2026-03-24T17:26:50.215+08:00"}
---

# 实例
```php
// 规范独立对象和组合对象必须实现的方法，
// 保证它们提供给客户端统一的访问方式
abstract class FileSystem
{
    protected $name;
    public function __construct($name)
    {
        $this->name = $name;
    }
    public abstract function getName();
    public abstract function getSize();
    abstract public function display($depth); // 显示目录结构
}

// 目录类
class Dir extends FileSystem
{
    private $filesystems = [];
    // 组合对象必须实现添加方法
    // 因为传入参数规定为Filesystem类型，所以目录和文件都能添加
    public function add(Filesystem $filesystem)
    {
        $key = array_search($filesystem, $this->filesystems);
        if ($key === false) $this->filesystems[] = $filesystem;
    }
    // 组合对象必须实现移除方法
    public function remove(Filesystem $filesystem)
    {
        $key = array_search($filesystem, $this->filesystems);
        if ($key) unset($this->filesystems[$key]);
    }
    public function getName()
    {
        return '目录：' . $this->name;
    }
    public function getSize()
    {
        $size = 0;
        foreach ($this->filesystems as $filesystem) {
            $size += $filesystem->getSize();
        }
        return $size;
    }
    // 显示其枝节点名称，并对其下级进行遍历
    public function display($depth)
    {
        echo str_repeat('-', $depth), $this->name, PHP_EOL;
        foreach ($this->filesystems as $fileSystems) {
            $fileSystems->display(++$depth);
        }
    }
}
// 文本文件类
class TextFile extends FileSystem
{
    public function getName()
    {
        return '文本：' . $this->name;
    }
    public function getSize()
    {
        return strlen(serialize($this));
    }
    public function display($depth)
    {
        echo str_repeat('-', $depth), $this->getName(), PHP_EOL;
    }
}

// 图片文件类
class ImageFile extends Filesystem
{
    public function getName()
    {
        return '图片：' . $this->name;
    }
    public function getSize()
    {
        return strlen(serialize($this));
    }
    public function display($depth)
    {
        echo str_repeat('-', $depth), $this->name, PHP_EOL;
    }
}
// 视频文件类
class VideoFile extends Filesystem
{
    public function getName()
    {
        return '视频：' . $this->name . PHP_EOL;
    }
    public function getSize()
    {
        return strlen(serialize($this));
    }
    public function display($depth)
    {
        echo str_repeat('-', $depth), $this->name, PHP_EOL;
    }
}

// 创建目录
$root = new Dir('root');
$home = new Dir('home');

// 创建文件
$image = new ImageFile('bg1.png');
$text1 = new TextFile('t1.txt');

// 添加到root目录
$root->add(new TextFile('t2.txt'));
$root->add($image);
$root->add(new VideoFile('film1.mp4'));

// 添加到home目录
$home->add($text1);

// home目录添加到root目录
$root->add($home);

// 打印信息
echo $text1->getName(), '-->', $text1->getSize(), PHP_EOL;
echo $home->getName(), ' --> ', $home->getSize(), PHP_EOL;
echo $root->getName(), ' --> ', $root->getSize(), PHP_EOL;

// 打印目录结构
$root->display(0);
```
# 总结
- 组合模式，将对象组合成树形结构以表示**部分与整体**的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。
- **透明方式：** 所有子类接口**保持一致**（提供最大一致性），但可能导致叶节点实现**无用接口**。
- **安全方式：** 类接口**不一致**（更安全），只实现特定接口，但客户端需要**额外判断**类型，使用起来不便。
需求中是体现部分与整体层次的结构时，或希望用户**可以忽略组合对象与单个对象的不同**，统一地使用组合结构中的所有对象时，**就应该考虑用组合模式了**。
组合模式可以让客户一致地使用组合结构和单个对象。
# 意图
组合模式将对象组织成**树形结构**以表示 **“部分与整体”的层次关系**，其核心目的是让用户可以**一致地对待单个对象和组合对象**。
# 主要解决
它在我们树型结构的问题中，模糊了简单元素和复杂元素的概念，客户程序可以像处理简单元素一样来处理复杂元素，从而使得客户程序与复杂元素的内部结构解耦。
# 何时使用
1. 您想表示对象的部分 - 整体层次结构（树形结构）。
2. 您希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象。
# 如何解决
树枝和叶子实现统一接口，树枝内部组合该接口。
# 优点
1. 高层模块调用简单。
2. 节点自由增加。
# 缺点
在使用组合模式时，其叶子和树枝的声明都是实现类，而不是接口，违反了依赖倒置原则。
# 使用场景
适用于需要表达**树形层次结构**，并希望客户端在处理结构中的**单个对象**和**组合对象**时，能够**统一使用相同接口**的场景。
# 注意事项
定义时为具体类。