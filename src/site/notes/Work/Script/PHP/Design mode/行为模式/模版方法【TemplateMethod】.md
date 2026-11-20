---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/模版方法【TemplateMethod】/","title":"模版方法【TemplateMethod】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:10.000+08:00","updated":"2026-03-24T17:30:04.323+08:00","dg-note-properties":{"title":"模版方法【TemplateMethod】","tags":["flashcards"],"reference linking":null}}
---

# 实例
```php
// 对甲乙两名同学所抄试卷,尽量将相同的部分提到父类
// 金庸小说考题试卷
class TestPaper
{
    public function TestQuestion1()
    {
        echo <<<EOF
杨过说过,后来给了郭靖,炼成倚天剑、屠龙刀的玄铁可能是()
    A.球磨铸铁 
    B.马口铁 
    C.高速合金钢 
    D.碳素纤维', PHP_EOL;
EOF
        , PHP_EOL;
        echo '答案 ', $this->answer1(), PHP_EOL;
    }
    public function TestQuestion2()
    {
        echo <<<EOF
杨过、程英、陆无双铲除了情花,造成()
    A.使这种植物不在害人 
    B.使一种珍惜物种灭绝 
    C.破坏了那个生态圈的生态平衡 
    D.造成该地区沙漠化', PHP_EOL;
EOF
        , PHP_EOL;
        echo '答案 ', $this->answer2(), PHP_EOL;
    }
    public function TestQuestion3()
    {
        echo <<<EOF
蓝凤凰致使华山师徒、桃谷六仙呕吐不止,如果你是大夫,会给他们开什么药()
    A.阿司匹林 
    B.牛黄解毒片 
    C.氟哌酸 
    D.让他们喝大量的生牛奶
EOF
        , PHP_EOL;
        echo '答案 ', $this->answer3(), PHP_EOL;
    }
    protected function answer1()
    {
        return '';
    }
    protected function answer2()
    {
        return '';
    }
    protected function answer3()
    {
        return '';
    }
}
// 学生甲抄的试卷
class TestPaperA extends TestPaper
{
    protected function answer1()
    {
        return 'A';
    }
    protected function answer2()
    {
        return 'B';
    }
    protected function answer3()
    {
        return 'C';
    }
}
// 学生乙抄的学生甲抄的试卷
class TestPaperB extends TestPaper
{
    protected function answer1()
    {
        return 'D';
    }
    protected function answer2()
    {
        return 'C';
    }
    protected function answer3()
    {
        return 'A';
    }
}
echo '学生甲抄的试卷: ', PHP_EOL;
$student = new TestPaperA();
$student->TestQuestion1();
$student->TestQuestion2();
$student->TestQuestion3();
echo '====华丽的分隔符====', PHP_EOL;
echo '学生乙抄的试卷: ', PHP_EOL;
$student2 = new TestPaperB();
$student2->TestQuestion1();
$student2->TestQuestion2();
$student2->TestQuestion3();
```
# 总结
模板方法模式通过在**父类中定义固定的算法骨架**，将变化的步骤延迟到**子类中实现**，从而在**不改变算法结构**的前提下实现了代码的复用和灵活扩展。
# 意图
定义一个操作中**算法的骨架**（固定结构），而将其中**可变的步骤延迟到子类**中实现。
# 主要解决
解决多个子类中存在**重复的通用代码**问题，通过**提取公共代码**到父类实现复用。
# 何时使用
有一些通用的方法。
# 如何解决
- 在**抽象父类**中定义**模板方法**（即算法骨架），该方法包含一系列固定步骤的调用。
- 将**不变的行为**在父类中实现，将**可变的行为**定义为抽象方法（或钩子方法），由**子类**重定义实现。
# 关键代码
在抽象类实现，其他步骤在子类实现。
# 优点 
- **代码复用：** 封装了不变部分，消除了子类中的重复代码，便于维护。
- **控制结构：** 父类控制了算法的整体结构，子类仅负责实现特定步骤。
# 缺点
每个不同的实现都需要一个新的子类，可能导致**类的数量增加**。
# 使用场景 
存在**多个子类共有的、逻辑相同**的方法或步骤；复杂的关键方法需要一个**固定的执行流程**。
# 注意事项
为防止恶意操作，一般模板方法都加上 final 关键词。