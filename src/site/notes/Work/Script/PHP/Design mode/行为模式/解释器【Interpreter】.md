---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Design mode/行为模式/解释器【Interpreter】/","title":"解释器【Interpreter】","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:48:14.000+08:00","updated":"2026-03-24T17:30:00.897+08:00"}
---

# 实例
```php
abstract class Expression
{
    public abstract function Interpreter(Content $content);
}
class Content
{
    private $content = '';
    public function getContent(): string
    {
        return $this->content;
    }
    public function setContent(string $content)
    {
        $this->content = $content;
    }
}

class EnglishExpression extends Expression
{
    public function Interpreter(Content $content)
    {
        $translate = [
            '你好' => 'hello'
        ];
        echo $translate[$content->getContent()] 
            ?? 'untranslatable', PHP_EOL;
    }
}
class ChineseExpression extends Expression
{
    public function Interpreter(Content $content)
    {
        $translate = [
            'world' => '世界'
        ];
        echo $translate[$content->getContent()] 
            ?? 'untranslatable', PHP_EOL;
    }
}
$content = new Content();
$content->setContent('world');
$syntax = [];
array_push($syntax, new EnglishExpression());
array_push($syntax, new ChineseExpression());
foreach ($syntax as $v) {
    $v->interpreter($content);
}
/*
untranslatable
世界
*/
```
# 总结
解释器模式通过将语言的文法表示为**抽象语法树**，并定义一个解释器来**递归解析**句子，适用于处理**简单且固定**的语言或重复出现的问题。
# 意图
给定一个**语言**，定义其**文法表示**（即抽象语法树），并定义一个**解释器**来解释该语言的句子。
# 主要解决
为一些**固定文法**（语言类型问题发生频率高）构建解释句子的解释器。
# 何时使用
**构建抽象语法树**，其中包含**终结符**（基本元素）和**非终结符**（组合规则），并通过**递归调用**方法来解释句子。
# 如何解决
构建语法树，定义终结符与非终结符。
# 关键代码
依赖于一个**环境类**（通常是 `HashMap`）来存储解释器所需的全局信息。
# 优点
- **易于扩展：** 易于修改和扩展文法，只需通过继承来改变或扩展文法规则。
- **易于实现：** 适用于简单文法。
# 缺点
- **类膨胀：** 文法中的每条规则至少需要定义一个类，复杂文法会导致**类膨胀，难以维护**。
- **应用场景少：** 适用范围相对有限。
# 使用场景
- 可以将需要解释执行的语言中的句子表示为**抽象语法树**。
- 适合处理**简单语法**需要解释的场景，如运算表达式计算或简化的命令解析。
# 注意事项
可利用场景比较少，JAVA 中如果碰到可以用 expression4J 代替。