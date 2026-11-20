---
{"dg-publish":true,"permalink":"/Work/Frontend/JS/JS basic/","title":"JS basic","noteIcon":"","created":"2026-04-23T23:31:44.000+08:00","updated":"2026-05-25T16:47:03.045+08:00","dg-note-properties":{"title":"JS basic","tags":null,"reference linking":null}}
---

# 入门
## 导论
[导论 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/basic/introduction)
## 历史
[JavaScript 语言的历史 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/basic/history)
## 基本语法
[JavaScript 的基本语法 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/basic/grammar)
### 语句
JavaScript 程序的执行单位为行（line），也就是一行一行地执行。一般情况下，每一行就是一个语句。

语句（statement）是为了完成某种任务而进行的操作，比如下面就是一行赋值语句。
```js
var a = 1 + 3;
```

**表达式不需要分号结尾**。一旦在表达式后面添加分号，则 JavaScript 引擎就将表达式视为语句，这样会产生一些没有任何意义的语句。
```js
1 + 3;
'abc';
```
#### 语句和表达式的区别
- **语句**: 主要为了进行某种操作，一般情况下不需要返回值；
- **表达式**: 则是为了得到返回值，一定会返回一个值
#### 书写位置
script标签、外部文件、伪链接、事件
```html
<button onclick="alert('warning')">warning</button>
<a href="javascript:alert('warning')">warning</a>
<script type="text/javascript" src="t2.js"></script>
```
### 变量
#### 概念
变量是对“值”的具名引用。变量就是为“值”起名，然后引用这个名字，就等同于引用这个值。变量的名字就是变量名。
```js
var a = 1;
```
**注意**: JavaScript 的变量名**区分大小写**，`A`和`a`是两个不同的变量。

变量的声明和赋值可以是分开的
```js
var a;
a = 1;
```

若只声明变量没有赋值，则该变量的值是`undefined`。`undefined`是一个特殊的值，表示“无定义”。
```js
var a;
a // undefined
```

若变量赋值没写`var`命令也是有效的。**但是不推荐不写`var`的做法**，不利于表达意图，而且容易不知不觉地创建全局变量，所以建议总是使用`var`命令声明变量。
```js
var a = 1;
// 基本等同
a = 1;
```

如果一个变量没有声明就直接使用，JavaScript 会报错，告诉你变量未定义。
```js
x
// ReferenceError: x is not defined
```

可以在同一条`var`命令中声明多个变量。
```js
var a, b;
```

JavaScript 是一种动态类型语言，也就是说，变量的类型没有限制，变量可以随时更改类型。
```js
var a = 1;
a = 'hello';
```

若使用`var`重新声明一个已经存在的变量，后面申明的是无效的。
```js
var x = 1;
var x;
x // 1
```
**但是**，如果第二次声明的时候还进行了赋值，则会**覆盖掉前面的值**。
```js
var x = 1;
var x = 2;
// 等同于
var x = 1;
var x;
x = 2;
```
#### 变量提升
JavaScript 引擎的工作方式是:
**先解析代码，获取所有被声明的变量**，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做**变量提升（hoisting）**。
```js
console.log(a);
var a = 1;
```
上面代码首先使用`console.log`方法，在控制台（console）显示变量`a`的值。
这时变量`a`还没有声明和赋值，所以这是一种错误的做法，但是实际上不会报错。
因为存在变量提升，真正运行的是下面的代码。
```js
var a;
console.log(a);
a = 1;
```
最后的结果是显示`undefined`，表示变量`a`已声明，但还未赋值。
#### 变量修饰符对比表
| 特性        | var                                              | let                                       | const                              |
| --------- | ------------------------------------------------ | ----------------------------------------- | ---------------------------------- |
| 作用域       | 函数级作用域（函数内部声明的变量仅在函数内有效，全局声明的变量属于全局作用域）          | 块级作用域（在 {} 内有效，如 if、for、while 等代码块）       | 块级作用域（与 let 相同）                    |
| 变量提升      | 存在变量提升（变量声明被提升到作用域顶部，但赋值留在原处，导致访问时可能为 undefined） | 不存在变量提升，但存在暂时性死区（Temporal Dead Zone, TDZ） | 不存在变量提升，但存在暂时性死区（TDZ）              |
| 重复声明      | 允许重复声明同一变量（但可能引发逻辑错误）                            | 不允许重复声明（同一作用域内声明同名变量会报错）                  | 不允许重复声明（与 let 相同）                  |
| 初始化要求     | 允许声明时不初始化（初始化后可修改）                               | 允许声明时不初始化，但必须在**声明后赋值（否则报错）**             | **必须在声明时初始化**（否则报错）                |
| 值的可变性     | 可以重新赋值                                           | 可以重新赋值                                    | **不可重新赋值**（但对象/数组的属性或元素可修改）        |
| 全局变量挂载    | 全局作用域的变量会挂载到全局对象（浏览器中是 window）                   | 全局作用域的变量不会挂载到全局对象                         | 全局作用域的变量不会挂载到全局对象                  |
| 临时死区（TDZ） | 不存在                                              | 存在（在声明前访问变量会报错 ReferenceError）            | 存在（与 let 相同）                       |
| 声明前访问     | 可以访问（但值为 undefined）                              | 无法访问（在声明前访问会报错 ReferenceError）            | 无法访问（与 let 相同）                     |
| 适用数据类型    | 所有数据类型（基本类型和引用类型）                                | 所有数据类型（基本类型和引用类型）                         | 所有数据类型（基本类型和引用类型）                  |
| 引用类型特性    | 对象/数组的属性或元素可修改                                   | 对象/数组的属性或元素可修改                            | 对象/数组的引用地址不可修改，但对象属性或数组元素可修改       |
| 生命周期      | 函数执行完毕后变量被销毁（局部变量），全局变量始终存在                      | 块级作用域执行完毕后变量被销毁                           | 块级作用域执行完毕后变量被销毁（与 let 相同）          |
| 历史与兼容性    | ES3 引入，兼容性最好（支持所有旧浏览器）                           | ES6 引入，需 polyfill 或 Babel 转译支持旧浏览器        | ES6 引入，需 polyfill 或 Babel 转译支持旧浏览器 |
#### 使用场景建议
| 修饰符   | 推荐场景                               | 避免场景                   |
| ----- | ---------------------------------- | ---------------------- |
| var   | 需要兼容旧浏览器、全局变量、函数作用域内的临时变量。         | 需要块级作用域、避免变量提升导致的意外行为。 |
| let   | 需要块级作用域、可变的变量（如循环计数器）。             | 需要常量（不可变值）、需要严格初始化的场景。 |
| const | 声明常量（不可变引用地址）、对象/数组的不可变引用、模块导出的常量。 | 需要频繁修改值或引用地址的变量。       |
#### var和let的区别
作用域：
- var **函数内**，**存在变量提升**
- let **代码块内**，**不存在变量提升**
- let不允许在相同作用域内，重复声明同一个变量。
##### 基本语法
使用let声明的变量报错，var声明的返回正确的值，let声明的变量只在它所在的代码块有效。
```javascript
{
    let a = 125;
    var b = 521;
}
a // Uncaught ReferenceError: a is not defined
b // 521
```
let配合for循环的应用，计数器i只在for循坏体内有效，再循环体外引用就会报错。
```javascript
for (let i = 0; i < 5; i++) {
    console.log(i); // 0 1 2 3 4 
}
console.log(i); // ReferenceError: i is not defined
```
常见的**面试题目**:
```javascript
for (var i = 0; i < 3; i++) {
    // 同步注册回调函数到 异步的 宏任务队列。
    setTimeout(function () {
        console.log(i); // 执行此代码时，同步代码for循环已经执行完成
    }, 0);
}
// 输出结果：3 3 3
```
如果把 var改成 let声明：
```javascript
// i虽然在全局作用域声明，但是在for循环体局部作用域中使用的时候，变量会被固定，不受外界干扰。
for (let i = 0; i < 10; i++) {
    setTimeout(function () {
        console.log(i); // i 是循环体内局部作用域，不受外界影响。
    }, 0);
}
// 输出结果：0 1 2 3 4 5 6 7 8 9
```
另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
```javascript
for (let i = 0; i < 3; i++) {
  let i = 'love';
  console.log(i);
}
// 输出结果：love love love
```
##### 不存在变量提升
var命令会发生”变量提升“现象，即变量可以在声明之前使用，值为undefined。
let命令则不同，它所声明的变量一定要在声明后使用，否则报错。
```javascript
// var 的情况
console.log(ar); // 输出undefined
var ar = 512;

// let 的情况
console.log(et); // 报错ReferenceError
let et = 512;
```
##### 暂时性死区（temporal dead zone，简称 TDZ）
当存在全局变量tmp，并且块级作用域内let又声明了一个局部变量tmp，会导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。
```javascript
var tmp = 521;
if (true) { 
    tmp = 'abc'; // ReferenceError: tmp is not defined
    let tmp;
}
```
ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。
(使用const声明的是常量，在后面出现的代码中不能再修改该常量的值。)
##### 不允许重复声明
let不允许在相同作用域内，重复声明同一个变量。
```javascript
// 报错
function func() {
    let a = 10;
    var a = 1;
}
// 报错
function func() {
    let a = 10;
    let a = 1;
}
// 因此，不能在函数内部重新声明参数。
function func(arg) {
    let arg; // 报错
}
function func(arg) {
    {
        let arg; // 不报错
    }
}
```
### 标识符
标识符（identifier）指的是用来识别各种值的合法名称。最常见的标识符就是变量名，以及后面要提到的函数名。JavaScript 语言的标识符对大小写敏感，所以`a`和`A`是两个不同的标识符。

标识符命名规则如下。
- 第一个字符，可以是任意 Unicode 字母（包括英文字母和其他语言的字母），以及美元符号（`$`）和下划线（`_`）。
- 第二个字符及后面的字符，除了 Unicode 字母、美元符号和下划线，还可以用数字`0-9`。
下面这些都是合法的标识符。
```js
arg0
_tmp
$elem
π
```
下面这些则是**不合法的标识符**。
```js
1a  // 第一个字符不能是数字
23  // 同上
***  // 标识符不能包含星号
a+b  // 标识符不能包含加号
-d  // 标识符不能包含减号或连词线
```
中文是合法的标识符，可以用作变量名。
```js
var 临时变量 = 1;
```
> JavaScript 有一些保留字，不能用作标识符：arguments、break、case、catch、class、const、continue、debugger、default、delete、do、else、enum、eval、export、extends、false、finally、for、function、if、implements、import、in、instanceof、interface、let、new、null、package、private、protected、public、return、static、super、switch、this、throw、true、try、typeof、var、void、while、with、yield。
### 注释
源码中被 JavaScript 引擎忽略的部分就叫做注释，它的作用是对代码进行解释。JavaScript 提供两种注释的写法：一种是单行注释，用`//`起头；另一种是多行注释，放在`/*`和`*/`之间。
```js
// 单行注释

/*
 多行
 注释
*/
```
此外，由于历史上 JavaScript 可以兼容 HTML 代码的注释，所以`<!--`和`-->`也被视为合法的单行注释。
```js
x = 1; <!-- x = 2;
--> x = 3;
```
上面代码中，只有`x = 1`会执行，其他的部分都被注释掉了。

需要注意的是，`-->`只有在行首，才会被当成单行注释，否则会当作正常的运算。
```js
function countdown(n) {
  while (n --> 0) console.log(n);
}
countdown(3)
// 2
// 1
// 0
```
上面代码中，`n --> 0`实际上会当作`n-- > 0`，因此输出2、1、0。
### 区块
JavaScript 使用大括号，将多个相关的语句组合在一起，称为“区块”（block）。

对于`var`命令来说，JavaScript 的区块不构成单独的作用域（scope）。
```js
{
  var a = 1;
}

a // 1
```
上面代码在区块内部，使用`var`命令声明并赋值了变量`a`，然后在区块外部，变量`a`依然有效，区块对于`var`命令不构成单独的作用域，与不使用区块的情况没有任何区别。在 JavaScript 语言中，单独使用区块并不常见，区块往往用来构成其他更复杂的语法结构，比如`for`、`if`、`while`、`function`等。
### 条件语句
JavaScript 提供`if`结构和`switch`结构，完成条件判断，即只有满足预设的条件，才会执行相应的语句。
#### if 结构
`if`结构先判断一个表达式的布尔值，然后根据布尔值的真伪，执行不同的语句。
```js
if (布尔值)
  语句;

// 或者
if (布尔值) 语句;
```
注意，`if`后面的表达式之中，不要混淆赋值表达式（`=`）、严格相等运算符（`===`）和相等运算符（`==`）。尤其是赋值表达式不具有比较作用。
```js
var x = 1;
var y = 2;
if (x = y) {
  console.log(x);
}
// "2"
```
上面代码的原意是，当`x`等于`y`的时候，才执行相关语句。但是，不小心将严格相等运算符写成赋值表达式，结果变成了将`y`赋值给变量`x`，再判断变量`x`的值（等于2）的布尔值（结果为`true`）。
#### if...else 结构
`if`代码块后面，还可以跟一个`else`代码块，表示不满足条件时，所要执行的代码。
```js
if (m === 3) {
  // 满足条件时，执行的语句
} else {
  // 不满足条件时，执行的语句
}
```
#### switch 结构
多个`if...else`连在一起使用的时候，可以转为使用更方便的`switch`结构。
```js
switch (fruit) {
  case "banana":
    // ...
    break;
  case "apple":
    // ...
    break;
  default:
    // ...
}
```

需要注意的是，`switch`语句后面的表达式，与`case`语句后面的表示式比较运行结果时，采用的是**严格相等运算符（`===`）**，而不是相等运算符（`==`），这意味着比较时不会发生类型转换。
```js
var x = 1;

switch (x) {
  case true:
    console.log('x 发生类型转换');
    break;
  default:
    console.log('x 没有发生类型转换');
}
// x 没有发生类型转换
```
#### 三元运算符 ?:
JavaScript 还有一个三元运算符（即该运算符需要三个运算子）`?:`，也可以用于逻辑判断。
```js
(条件) ? 表达式1 : 表达式2
```

```javascript
var a = 1, b = 2, c = 3
c = a < b ? a : b;
console.log(c); // 1

// 错误示范
function xx() {
    return a < b ? return false : '' ;
}
// 错误原因如下
if (expression) {
    return (return false);
}
```
### 循环语句
循环语句用于重复执行某个操作，它有多种形式。
#### while 循环
`while`语句包括一个循环条件和一段代码块，只要条件为真，就不断循环执行代码块。代码块部分，如果**只有一条语句，可以省略大括号**，否则就必须加上大括号。
```js
while (条件)
  语句;

// 或者
while (条件) 语句;

// 或者
while (条件) {
  语句;
}
```
例子
```js
var i = 0;

while (i < 100) {
  console.log('i 当前为：' + i);
  i = i + 1;
}
```
#### for 循环
`for`语句是循环命令的另一种形式，可以指定循环的起点、终点和终止条件。它的格式如下。
```js
for (初始化表达式; 条件; 递增表达式)
  语句

// 或者

for (初始化表达式; 条件; 递增表达式) {
  语句
}
```
例子
```js
var x = 3;
for (var i = 0; i < x; i++) {
  console.log(i);
}
// 0
// 1
// 2
```
#### do...while 循环
`do...while`循环与`while`循环类似，唯一的区别就是**先运行一次循环体**，然后判断循环条件。
```js
do
  语句
while (条件);

// 或者
do {
  语句
} while (条件);
```
**不管条件是否为真**，`do...while`循环**至少运行一次**，这是这种结构最大的特点。另外，`while`语句后面的**分号注意不要省略**。

例子
```js
var x = 3;
var i = 0;

do {
  console.log(i);
  i++;
} while(i < x);
```
#### break 语句和 continue 语句
`break`语句和`continue`语句都具有跳转作用，可以让代码不按既有的顺序执行。

`break`语句用于跳出代码块或循环。
```js
var i = 0;

while(i < 100) {
  console.log('i 当前为：' + i);
  i++;
  if (i === 10) break;
}
```
上面代码只会执行10次循环，一旦`i`等于10，就会跳出循环。

`continue`语句用于立即终止本轮循环，返回循环结构的头部，开始下一轮循环。
```js
var i = 0;

while (i < 100){
  i++;
  if (i % 2 === 0) continue;
  console.log('i 当前为：' + i);
}
```
上面代码只有在`i`为奇数时，才会输出`i`的值。如果`i`为偶数，则直接进入下一轮循环。
##### 多重循环
不带参数的`break`语句和`continue`语句都只针对最内层循环。
```js
let i = 0;
let j = 0;

while (i < 3) {
    console.log('外 当前为：' + i);
    i++;
    while (j < 3) {
        console.log('内 当前为：' + j);
        j++;
        if (j === 2) break;
    }
}
/*
外 当前为：0
内 当前为：0
内 当前为：1
外 当前为：1
内 当前为：2
外 当前为：2
*/
```

从内部结束外层循环
```js
let i = 0;
let j = 0;

outerLoop: while (i < 3) {
    console.log('外 当前为：' + i);
    i++;
    while (j < 3) {
        console.log('内 当前为：' + j);
        j++;
        if (j === 2) break outerLoop; // 跳出外层循环
    }
}
/*
外 当前为：0
内 当前为：0
内 当前为：1
*/
```

三层循环，在第三层判断是否结束第二层的循环
```js
let i = 0;
let j = 0;
let k = 0;

while (i < 3) {
    console.log('外 当前为：' + i);
    i++;
    outerLoop2: while (j < 3) {
        console.log('内1 当前为：' + j);
        j++;
        while (k < 3) {
            console.log('内2 当前为：' + k);
            k++;
            if (k === 2) break outerLoop2; // 结束第二层循环
        }
    }
}
```
#### 标签（label）
JavaScript 语言允许，语句的前面有标签（label），相当于定位符，用于跳转到程序的任意位置，标签的格式如下。
```js
label:
  语句
```

标签通常与`break`语句和`continue`语句配合使用，跳出特定的循环。
```js
top:for (var i = 0; i < 3; i++) {
    for (var j = 0; j < 3; j++) {
        if (i === 1 && j === 1) break top;
        console.log('i=' + i + ', j=' + j);
    }
}
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
```

标签也可以用于跳出代码块。
```js
foo: {
  console.log(1);
  break foo;
  console.log('本行不会输出');
}
console.log(2);
// 1
// 2
```
#### `for`
```javascript
var arr = [1, 2, 3];
arr.test = 'test';
for (var i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}
// 1 2 3
```
#### `for...in`
**ES5的标准**，该方法**遍历**的是对象的属性名称 **(key：键名)**。
- 一般用于遍历对象自身的和继承的可枚举属性。以及对象从构造函数原型中继承的属性。对于每个不同的属性，语句都会被执行。
- 不建议使用for in 遍历数组，因为输出的顺序是不固定的。
- 如果迭代的对象的变量值是null或者undefined, for in不执行循环体，建议在使用for in循环之前，先检查该对象的值是不是null或者undefined
- for…in 语句以原始插入顺序迭代对象的可枚举属性
(只能迭代出可枚举的属性，可枚举属性【js自定义属性】/不可枚举属性【对象的内置属性，如数组的length 就是一个内置属性，所以for…in遍历不出来】)。
- for…in的原理是: Object.keys()：返回给定对象所有可枚举属性的字符串数组
#### `for...of `
**ES6的标准**，该方法**遍历**的是对象的属性所对应的值 **(value：键值)**。
- for…of 语句在可迭代对象（包括 Array，Map，Set，String，TypedArray，arguments 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句
- for…of 语句遍历可迭代对象定义要迭代的数据（非自定义属性）
- for…of循环可以使用的范围包括数组、Set 和 Map 结构、某些类似数组的对象、Generator 对象，以及字符串。
- for…of循环不能循环普通的对象，对普通对象的属性遍历推荐使用for…in
```javascript
// +----------------------------------------------------------------------
// | 遍历数组
// +----------------------------------------------------------------------
let array = ['A', 'B', 'C'];
for (let arrayV of array) {
    console.log(arrayV); // 输出：A　B　C
}
for (let arrayV in array) {
    console.log(arrayV); // 输出：0 1 2
}
array.forEach((v, k) => {
    console.log(k, '=>', v); // 输出：0 => A 1 => B 2 => C
})

// +----------------------------------------------------------------------
// | 遍历Set集合
// +----------------------------------------------------------------------
let set = new Set(['A', 'B', 'C']);
for (let setV of set) {
    console.log(setV); // 输出：A　B　C
}
// 不能使用for...in循环遍历Set集合
for (let setV in set) {
    console.log(setV);
}
set.forEach((v, k) => {
    console.log(k, '=>', v); // 输出：A => A B => B C => C
})

// +----------------------------------------------------------------------
// | 遍历Map集合
// +----------------------------------------------------------------------
let map = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
for (let mapV of map) {
    console.log(mapV[0] + "=" + mapV[1]); // 既可以拿到键名，也可以拿到键值，输出：1=x 2=y 3=z
}
// for...in循环不能用于遍历Map
for (let mapV in map) {
    console.log(mapV[0] + "=" + mapV[1]);
}
map.forEach((v, k) => {
    console.log(k, '=>', v); // 输出：1 => x 2 => y 3 => z
})

//+----------------------------------------------------------------------
//| 遍历对象
//+----------------------------------------------------------------------
let object = {a: 1, b: 2, c: 3};
// 报错：TypeError: object is not iterable
// for (let objectV of object) {
//     console.log(objectV);
// }
// 使用Object.keys()方法获取对象key的数组
for (let key of Object.keys(object)) {
    console.log(key, '=>', object[key]); // 输出：a => 1 b => 2 c => 3
}
for (let key in object) {
    console.log(key, '=>', object[key]); // 输出：a => 1 b => 2 c => 3
}
// 报错：TypeError: object.forEach is not a function
// object.forEach((v, k) => {
//     console.log(k, '=>', v); // 输出：1 => x 2 => y 3 => z
// })
for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```
# 数据类型
## 六种数据类型
- 数值（number）：整数和小数（比如`1`和`3.14`）。
- 字符串（string）：文本（比如`Hello World`）。
- 布尔值（boolean）：表示真伪的两个特殊值，即`true`（真）和`false`（假）。
- `undefined`：表示“未定义”或不存在，即由于目前没有定义，所以此处暂时没有任何值。
- `null`：表示空值，即此处的值为空。
- 对象（object）：各种值组成的集合。
>ES6 又新增了 Symbol 和 BigInt 数据类型，本教程不涉及。
### 对象是最复杂的数据类型，又可以分成三个子类型。
- 狭义的对象（object）
- 数组（array）
- 函数（function）
### typeof 运算符
JavaScript 有三种方法，可以确定一个值到底是什么类型。
- `typeof`运算符「返回一个字符串，运算符可以返回一个值的数据类型」
- `instanceof`运算符「是否是它定义出来的」
- `Object.prototype.toString`方法
#### 示例「Object、String、Boolean、Function」
```javascript
// Numbers
typeof 37 === 'number';
typeof 3.14 === 'number';
typeof Math.LN2 === 'number';
typeof Infinity === 'number';
typeof NaN === 'number'; // 尽管NaN是"Not-A-Number"的缩写
typeof Number(1) === 'number'; // 但不要使用这种形式!

// Strings
typeof "" === 'string';
typeof "bla" === 'string';
typeof (typeof 1) === 'string'; // typeof总是返回一个字符串
typeof String("abc") === 'string'; // 但不要使用这种形式!

// Booleans
typeof true === 'boolean';
typeof false === 'boolean';
typeof Boolean(true) === 'boolean'; // 但不要使用这种形式!

// Symbols
typeof Symbol() === 'symbol';
typeof Symbol('foo') === 'symbol';
typeof Symbol.iterator === 'symbol';

// Undefined
typeof v === 'undefined'; // 用来检查一个没有声明的变量，而不报错。
typeof undefined === 'undefined';
typeof declaredButUndefinedVariable === 'undefined';
typeof undeclaredVariable === 'undefined'; 

// Objects
typeof {a:1} === 'object';


typeof [1, 2, 4] === 'object';
typeof new Date() === 'object';

// 区分数组和对象
Array.isArray({}); // false
Array.isArray([]); // true

Object.prototype.toString.call({}); // [object Object]  
Object.prototype.toString.call([]); // [object Array]

{} instanceof Array // false
[] instanceof Array // true

// 下面的容易令人迷惑，不要使用！
typeof new Boolean(true) === 'object';
typeof new Number(1) === 'object';
typeof new String("abc") === 'object';

// 函数
typeof function(){} === 'function';
typeof class C{} === 'function'
typeof Math.sin === 'function';
typeof new Function() === 'function';
```
#### 对象的简单类型区别
```javascript
let a = "abc";
let b = new String("abc");
console.log(typeof a, typeof b); // 输出：string object
a = new String("a"); // 将简单类型字符串转换成对象(只转换一次，优化性能)
console.log(a.charAt(0)); // 输出：a「自动装箱(自动把简单类型转换成String类型), 」
```
#### 区分数组和对象
```js
var o = {};
var a = [];

o instanceof Array // false
a instanceof Array // true
```
## null, undefined 和布尔值
[null, undefined 和布尔值 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/types/null-undefined-boolean)
null与undefined都可以表示“没有”，含义非常相似。将一个变量赋值为undefined或null，老实说，语法效果几乎没区别。
### `undefined`的典型场景
```js
// 变量声明了，但没有赋值
var i;
i // undefined

// 调用函数时，应该提供的参数没有提供，该参数等于 undefined
function f(x) {
  return x;
}
f() // undefined

// 对象没有赋值的属性
var  o = new Object();
o.p // undefined

// 函数没有返回值时，默认返回 undefined
function f() {}
f() // undefined
```
### 布尔值
#### 除了下面六个值被转为`false`，其他值都视为`true`。
- `undefined`
- `null`
- `false`
- `0`
- `NaN`
- `""`或`''`（空字符串）
#### 注意
空数组（`[]`）和空对象（`{}`）对应的布尔值，**都是true**。
```js
if ([]) {
  console.log('true'); // true
}
if ({}) {
  console.log('true'); // true
}
```
## 空值处理
判断
```js
0、””、null、undefined
typeof(var)==‘undefined’
```
处理
```js
var v = nullValue || defaultValue…
```
循环
```js
for(var r, i=0; r=arr[ i++ ];)、for...in、for...of
```
空值处理
```javascript
// b不存在就默认为1
function f(a, b) {
    console.log(a + (b || 1))
}
f(1); // 2
```
### 判断 undefined
```javascript
// 以下是不正确的用法：
// exp 为 null 时，也会得到与 undefined 相同的结果，虽然 null 和 undefined 不一样。
// 注意：要同时判断 undefined 和 null 时可使用本法。
var exp = undefined;
if (exp == undefined) {
    alert("undefined");
}
var exp = undefined;
if (typeof (exp) == undefined) {
    alert("undefined");
}
// 以下是正确的用法：
var exp = undefined;
if (typeof (exp) == "undefined") {
    alert("undefined");
}
```
### JS 中如何判断 null
```javascript
// 以下是不正确的用法：
// #1 exp 为 undefined 时，也会得到与 null 相同的结果，虽然 null 和 undefined 不一样。
// 注意：要同时判断 null 和 undefined 时可使用本法。
var exp = null;
if (exp == null) {
    alert("is null");
}
var exp = null;
if (!exp) {
    alert("is null");
}

// #2 如果 exp 为 undefined 或者数字零，也会得到与 null 相同的结果，虽然 null 和二者不一样。
// 注意：要同时判断 null、undefined 和数字零时可使用本法。
var exp = null;
if (typeof (exp) == "null") {
    alert("is null");
}

// 为了向下兼容，exp 为 null 时，typeof 总返回 object。
// #3 JavaScript 中没有 isNull 这个函数。
var exp = null;
if (isNull(exp)) {
    alert("is null");
}

// 以下是正确的用法：
var exp = null;
if (!exp && typeof (exp) != "undefined" && exp != 0) {
    alert("is null");
}
```
尽管如此，我们在 DOM 应用中，一般只需要用 (!exp) 来判断就可以了，因为 DOM 应用中，可能返回 null，可能返回 undefined，如果具体判断 null 还是 undefined 会使程序过于复杂。
### 数组空值判断
1. 长度属性检查 **「推荐」**
```javascript
if (array.length === 0) {
console.log('Array is empty');
}
```
2. **严格相等比较空数组**：
```javascript
if (JSON.stringify(array) === JSON.stringify([])) {
console.log('Array is empty');
}
```
注意：这种方法虽然有效，但对于大型数组性能较差。
3. **使用Array.isArray()结合length属性**：
```javascript
if (Array.isArray(array) && array.length === 0) {
console.log('Array is empty');
}
```
### 对象空值判断
1. Object.keys()方法 **「推荐」**
```javascript
if (Object.keys(obj).length === 0) {
	console.log('Object is empty');
}
```
2. for...in循环检查属性
```javascript
let isEmpty = true;
for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
        isEmpty = false;
        break;
    }
}
if (isEmpty) {
    console.log('Object is empty');
}
```
3. ES6扩展运算符与严格相等比较空对象
>注: 这种方法对于大型对象性能较差。
```javascript
if (JSON.stringify({ ...obj }) === '{}') {
	console.log('Object is empty');
}
```
4. lodash库的方法「如果项目中已经引入了lodash」
```javascript
if (_.isEmpty(obj)) {
	console.log('Object is empty');
}
```
## 数值
[数值 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/types/number)
### 整数和浮点数
JavaScript 内部，所有数字都是以**64位浮点数**形式储存，即使整数也是如此。所以，`1`与`1.0`是相同的，是同一个数。
```js
1 === 1.0 // true
```
这就是说，JavaScript 语言的**底层根本没有整数**，所有数字都是小数（64位浮点数）。
#### 现代发展：`BigInt`
| **类型**       | **内部存储** | **精度限制**    | **用法示例**                                 |
| ------------ | -------- | ----------- | ---------------------------------------- |
| **`number`** | 64 位浮点数  | 限于 $2^{53}$ | `123` 或 `123.45`                         |
| **`BigInt`** | 任意精度整数   | 仅受内存限制      | `123n` 或 `BigInt("1000000000000000000")` |
在 **ES2020** 引入了新的原始数据类型：**`BigInt`** 使得 JavaScript **现在有了** 专门用于表示**任意精度整数**的机制。

容易造成混淆的是，**某些运算只有整数才能完成，此时 JavaScript 会自动把64位浮点数，转成32位整数**，然后再进行运算，参见《运算符》一章的“位运算”部分。
由于浮点数不是精确的值，所以涉及小数的比较和运算要特别小心。
```js
0.1 + 0.2 === 0.3 // false
0.3 / 0.1 // 2.9999999999999996
(0.3 - 0.2) === (0.2 - 0.1) // false
```
### 数值精度
64 位双精度浮点数（IEEE 754 Double Precision）的标准公式如下：
$$\text{Value} = (-1)^S \times (1.M) \times 2^{e}$$
#### 公式组成部分详解
| **符号**      | **描述**                     | **含义**        | **存储位数**      | **值域或计算方式**                                                                                                       | **范围**           |
| ----------- | -------------------------- | ------------- | ------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------- |
| **$S$**     | **符号位 (Sign)**             | 表示数值的**正负**   | 1 位           | 决定正负：$S=0$ 为正， $S=1$ 为负。                                                                                          | $0$ 或 $1$        |
| **$1.M$**   | **有效数字 (Significand)**<br> | 表示数值的**精度**   | 52 位 (存储 $M$) | $1$ 是：**隐含位(Hidden Bit)**。<br>$M$ 是：存储在**尾数(Mantissa)** 域的 52 位小数。<br>$\text{Value}_{\text{Significand}} = 1 + M$ | $0$ 到 $2^{52}-1$ |
| **$2^{e}$** | **指数 (Exponent)**          | 表示数值的**大小**范围 | 11 位 (存储 $E$) | $e$ 是：**实际指数**，通过**移码**计算得出：$e = E - 1023$<br>$E$ 是：存储的 11 位无符号指数值。$1023$ 是双精度的**偏差值 (Bias)**。                    | $0$ 到 $2047$     |
##### 注意：
- **规范化数 (Normalized Numbers):** 当 $0 < E < 2047$ 时使用此公式。
- **特殊情况：** 当 $E=0$ 或 $E=2047$ 时，用于表示 $\pm 0$、**非规范化数**、$\pm \infty$ 和 **NaN** (Not a Number)。
#### 有效数字的定义和规则
1. **非零数字**（1, 2, 3, ..., 9）都是有效数字
    - 例子：
        - `5` → 1 位有效数字
        - `234` → 3 位有效数字（2, 3, 4）
        - `0.00078` → 2 位有效数字（7, 8）
2. **夹心零**（夹在两个非零数字之间的零）是有效数字
    - 例子：
        - `105` → 3 位有效数字（1, 0, 5）
        - `1005` → 4 位有效数字（1, 0, 0, 5）
        - `0.00405` → 3 位有效数字（4, 0, 5）
3. **末尾零**（在小数点后或明确标注的零）是有效数字
    - 例子：
        - `0.500` → 3 位有效数字（5, 0, 0）
        - `1200.` → 4 位有效数字（1, 2, 0, 0，注意末尾的点表示精确到个位）
        - `1.230 × 10³` → 4 位有效数字（1, 2, 3, 0）
4. **前导零**（在第一个非零数字前的零）**不是有效数字**
	- 例子：
		- `0.0045` → 2 位有效数字（4, 5）
		- `000123` → 3 位有效数字（1, 2, 3）
		- `0.000000789` → 3 位有效数字（7, 8, 9）
#### 指数部分
一共有11个二进制位，对应的十进制是**0~2047**($2^{11}-1$)。
IEEE 754 规定，如果**指数部分**的值在**0~2047**之间（**不含两个端点**），那么**有效数字**的**第一位**默认**总是1**，**不保存**在**64位浮点数之中**。
也就是说，有效数字这时总是`1.xx...xx`的形式，其中`xx..xx`的部分保存在64位浮点数之中，最长为**52位**。
因此，JavaScript 提供的**有效数字最长为53个二进制位**。
```js
(-1)^符号位 * 1.xx...xx * 2^指数部分
```
上面公式是正常情况下（指数部分在0到2047之间），一个数在 JavaScript 内部实际的表示形式。
#### 科学计数法理解
- **十进制科学计数法**：`±M × 10^E`
	- `±` 是符号位。
	- `M` 是有效数字（例如 `1.23`）。
	- `E` 是指数部分。
	- **有效数字 `M` 的精度由其小数位数决定**，而 `±` 和 `E` 不影响 `M` 的精度。
- **二进制科学计数法**：`±1.xx...xx × 2^E`
	- `±` 是符号位。
	- `1.xx...xx` 是有效数字（隐含的 1 + 小数部分）。
	- `E` 是指数部分。
	- **有效数字的精度由小数部分决定**，**符号位**和**指数**部分仅调整**正负**和**量级**。
安全精度为**53位**，这意味着，$|N| \le 2^{53}$ ($-2^{53}+1$ ~ $2^{53}-1$ )的整数都可以精确表示
```js
console.log('2^53-1：'  + (Math.pow(2, 53) - 1).toExponential());   // 2^53-1： 9.007199254740991e+15 (安全: 16位内十进制)
console.log('2^53：'    + (Math.pow(2, 53)).toExponential());       // 2^53：   9.007199254740992e+15 (看似安全)
console.log('2^53+1：'  + (Math.pow(2, 53) + 1).toExponential());   // 2^53+1： 9.007199254740992e+15
console.log('2^53+2：'  + (Math.pow(2, 53) + 2).toExponential());   // 2^53+2： 9.007199254740994e+15 精确
console.log('2^53+3：'  + (Math.pow(2, 53) + 3).toExponential());   // 2^53+3： 9.007199254740996e+15
console.log('2^53+4：'  + (Math.pow(2, 53) + 4).toExponential());   // 2^53+4： 9.007199254740996e+15 精确
console.log('2^53+5：'  + (Math.pow(2, 53) + 5).toExponential());   // 2^53+5： 9.007199254740996e+15
console.log('2^53+6：'  + (Math.pow(2, 53) + 6).toExponential());   // 2^53+6： 9.007199254740998e+15 精确
console.log('2^53+7：'  + (Math.pow(2, 53) + 7).toExponential());   // 2^53+7： 9.007199254741000e+15
console.log('2^53+8：'  + (Math.pow(2, 53) + 8).toExponential());   // 2^53+8： 9.007199254741000e+15 精确
console.log('2^53+9：'  + (Math.pow(2, 53) + 9).toExponential());   // 2^53+9： 9.007199254741000e+15
console.log('2^53+10：' + (Math.pow(2, 53) + 10).toExponential());  // 2^53+10：9.007199254741002e+15 精确
console.log('2^53+11：' + (Math.pow(2, 53) + 11).toExponential());  // 2^53+11：9.007199254741004e+15
console.log('2^53+12：' + (Math.pow(2, 53) + 12).toExponential());  // 2^53+12：9.007199254741004e+15 精确
console.log('--------------------------');
console.log('2^54：'    + (Math.pow(2, 54)).toExponential(16));       // 2^54：   1.8014398509481984e+16 精确
console.log('2^54+1：'  + (Math.pow(2, 54) + 1).toExponential(16));   // 2^54+1： 1.8014398509481984e+16
console.log('2^54+2：'  + (Math.pow(2, 54) + 2).toExponential(16));   // 2^54+2： 1.8014398509481984e+16
console.log('2^54+3：'  + (Math.pow(2, 54) + 3).toExponential(16));   // 2^54+3： 1.8014398509481988e+16
console.log('2^54+4：'  + (Math.pow(2, 54) + 4).toExponential(16));   // 2^54+4： 1.8014398509481988e+16 精确
console.log('2^54+5：'  + (Math.pow(2, 54) + 5).toExponential(16));   // 2^54+5： 1.8014398509481988e+16
console.log('2^54+6：'  + (Math.pow(2, 54) + 6).toExponential(16));   // 2^54+6： 1.8014398509481992e+16
console.log('2^54+7：'  + (Math.pow(2, 54) + 7).toExponential(16));   // 2^54+7： 1.8014398509481992e+16
console.log('2^54+8：'  + (Math.pow(2, 54) + 8).toExponential(16));   // 2^54+8： 1.8014398509481992e+16 精确
console.log('2^54+9：'  + (Math.pow(2, 54) + 9).toExponential(16));   // 2^54+9： 1.8014398509481992e+16
console.log('2^54+10：' + (Math.pow(2, 54) + 10).toExponential(16));  // 2^54+10：1.8014398509481992e+16
console.log('2^54+11：' + (Math.pow(2, 54) + 11).toExponential(16));  // 2^54+11：1.8014398509481996e+16
console.log('2^54+12：' + (Math.pow(2, 54) + 12).toExponential(16));  // 2^54+12：1.8014398509481996e+16 精确
```
当$>2^{53}$以后，整数运算的结果开始出现错误。**故：$>2^{53}$的数值无法保持精度**。
由于 **$2^{53}$对应$10^{16}$** ，所以简单的法则就是，JavaScript **对$10^{15}$数都可以精确处理**。
##### 二进制转换十进制：
将 $2^{53}$ 转换为 $10$ 的幂次，即求 $x$ 使得 $2^{53} = 10^x$：$$x = \log_{10}(2^{53}) = 53 \times \log_{10}(2)$$使用 $\log_{10}(2) \approx 0.3010299957$:$$x \approx 53 \times 0.3010299957 \approx 15.9545897721$$
$2^{53}$ 大约等于 $10^{15.9546}$。
这意味着 $2^{53}$ 是一个具**有 $16$ 位**数字的整数（$10^{15}$ 到 $10^{16}$ 之间），它等于 $9,007,199,254,740,992$。
$$2^{53} \approx 10^{15.9546} \approx 9.0e15$$
```js
Math.pow(2, 53)
9007199254740992
9007199254740992111 // 多出的三个有效数字，将无法保存
9007199254740992000
```
上面示例表明，$>2^{53}$以后，多出来的有效数字（最后三位的`111`）都会无法保存，变成0。
#### $2^{53}$ 在 64 位浮点数中的存储分析
##### 1. 将 $2^{53}$ 转换为规范化二进制形式
首先，我们将 $2^{53}$ 视为一个纯二进制整数，然后转换为规范化的科学计数法形式 $\mathbf{1.} M \times 2^e$。
* **整数表示**： $2^{53}$ 在二进制中是一个 $1$ 后面跟着 $53$ 个 $0$。
$$\underbrace{1}_{1} \underbrace{000\dots000}_{53 \text{个 } 0}$$
* **规范化形式 $(\mathbf{1.} M \times 2^e)$**： 为了将其写成 $\mathbf{1.} M$ 的形式，我们需要将小数点向左移动 53 位。
$$\mathbf{1.} \underbrace{000\dots000}_{53 \text{个 } 0} \times 2^{53}$$
##### 2. 确定浮点数三个字段的值
根据规范化形式 $\mathbf{1.} \underbrace{000\dots000}_{53 \text{个 } 0} \times 2^{53}$，我们确定 $S, M, E$ 的值。
###### A. 符号位 ($S$)
* $2^{53}$ 是**正数**。
* **$S = 0$**
###### B. 尾数位 ($M$)
* 尾数 $M$ 存储的是规范化形式中**小数点右边**的部分。
* 在 $\mathbf{1.} \underbrace{000\dots000}_{53 \text{个 } 0}$ 中，小数点右边有 $53$ 个 $0$。
* 但**尾数只有 52 位**存储空间。
* 因此，我们取小数点右边的前 52 个 $0$ 存储起来。
* **$M = \underbrace{000\dots000}_{52 \text{个 } 0}$**
###### C. 指数位 ($E$)
* 规范化形式中的**实际指数** $e = 53$。
* 我们需要计算**存储指数** $E$：$E = e + \text{Bias} = 53 + 1023 = \mathbf{1076}$
* 我们将 $1076$ 转换为 11 位的二进制数：$1076_{10} = 10000110100$
* **$E = 10000110100$**
##### 3. 最终存储结构
将计算出的 $S, E, M$ 拼接起来，就是 $2^{53}$ 在 64 位内存中的存储结构：

| 字段 | 符号位 ($S$) | 指数位 ($E$) (11 bits) | 尾数位 ($M$) (52 bits) |
| :--- | :--- | :--- | :--- |
| **值** | $\mathbf{0}$ | $\mathbf{10000110100}$ | $\underbrace{000\dots000}_{52 \text{个 } 0}$ |
##### 结论
$2^{53}$ 可以被**精确表示**，因为它：
1.  **符号位** $S=0$。
2.  **指数位** $E=1076$ 是一个有效值。
3.  **尾数** $M$（小数点右边的 52 位）全部是 $0$，完美地**填满了** 52 位存储空间，且**没有发生舍入**。
**存储顺序（64位）：**$\underbrace{0}_{S} \underbrace{10000110100}_{E} \underbrace{0000000000000000000000000000000000000000000000000000}_{M}$
总共 **64 位**，可以**精确**表示 $2^{53}$。
#### 详细解析 $2^{53}$ 的存储和截断
我们再次使用 $2^{53}$ 的规范化形式：$V = \mathbf{1.} \underbrace{M_1 M_2 \dots M_{52}}_{\text{52 个尾数位}} \underbrace{M_{53}}_{\text{第 53 个尾数位}} \underbrace{M_{54} \dots}_{\text{更低位}} \times 2^{53}$
##### 1. 识别“位”的编号
在浮点数中，有两种“位”的编号需要区分：
* **原始有效位总数：** $2^{53}$ 的有效位是 $1$ 后面跟着 $53$ 个 $0$，总共 $54$ 位。
* **尾数位编号（小数点后的位）：** 从 $M_1$ 到 $M_{53}$，以此类推。
对于 $2^{53}$：$\mathbf{1.} \underbrace{0}_{M_1} \underbrace{0}_{M_2} \dots \underbrace{0}_{M_{52}} \underbrace{0}_{M_{53}} \underbrace{0}_{M_{54}} \dots \times 2^{53}$

| 名称 | 位在原数中的位置 | 数值 | 是否被存储？ |
| :--- | :--- | :--- | :--- |
| **隐含位** | 小数点左边 | $\mathbf{1}$ | 否 (规则决定) |
| **尾数 $M_1 \dots M_{52}$** | 小数点后第 $1 \dots 52$ 位 | $\mathbf{0}$ | **是** (存储在 52 位 $M$ 域中) |
| **尾数 $M_{53}$** | 小数点后第 $53$ 位 | $\mathbf{0}$ | **否** (超出存储空间) |
| **尾数 $M_{54} \dots$** | 小数点后第 $54$ 位及以后 | $\mathbf{0}$ | **否** (超出存储空间) |
##### 2. 存储限制与截断 (Truncation)
64 位浮点数只能存储 $\mathbf{1}.M_1 M_2 \dots M_{52}$。
当我们把 $2^{53}$ 存入浮点数时，存储器只能容纳到 $M_{52}$ 为止。后面的 $M_{53}, M_{54}, \dots$ 必须被处理。
* **截断点：** 在 $M_{52}$ 之后。
* **需要被处理的位：** $G = M_{53}$ (Guard bit), $R = M_{54}$ (Round bit), $S$ (Sticky bit, $M_{55}$ 及以后所有位的 OR 结果)。
##### 3. “安全丢弃”的原理
在 IEEE 754 舍入规则中，只有当被丢弃的位**不全为 $0$** 时，才需要进行舍入操作（例如向上取整或向下截断）。
对于 $2^{53}$ 的情况：
$$\mathbf{1.} M_1 M_2 \dots M_{52} \,|\, M_{53} M_{54} \dots \times 2^{53}$$
$$\mathbf{1.} \underbrace{00\dots0}_{52 \text{个 } 0} \,|\, \underbrace{000\dots}_{\text{被丢弃的位}} \times 2^{53}$$
1.  **被丢弃的部分**（$M_{53}$ 及之后）**全部是 $0$**。
2.  丢弃一串 $0$ 相当于从小数 $\mathbf{1}.00\dots0000$ 中移除尾随的 $0$。
3.  **数值上：** $\mathbf{1}.00\dots00 \times 2^{53}$ **精确等于** $\mathbf{1}.00\dots0000 \times 2^{53}$。
**结论：**
* $M_{53}$ (第 53 个 $0$) 确实超出了 52 位的存储限制而被**丢弃**（截断）。
* 但因为它是一个 $0$，丢弃它**没有改变**原始数字的数学值，因此**没有引起舍入**，也没有损失精度。
这正是 $2^{53}$ 能够被**精确**存储的原因：尽管它的有效数字串有 54 位，但超出存储限制的位刚好都是 $0$。
#### 64 位双精度浮点数精度规律
双精度浮点数（$64\text{-bit}$）使用 $53$ 位有效数字（$1$ 位隐含位 $+ 52$ 位尾数 $M$）来表示数值。
当数值增大时，指数 $e$ 随之增大，导致可精确表示的数字之间的**最小间隔 (Ulp)** 增大。
##### 最小精确整数范围（安全区）
* **范围:** $|N| \le 2^{53}$
    * $2^{53} = 9,007,199,254,740,992$
* **最小间隔 (Ulp):** $\mathbf{1}$
* **规律:** 在此范围内的**所有整数**都能被**精确**表示。这是浮点数计算中保证整数精度的**安全范围**。
##### 精度阶梯（危险区）
当数值超过 $2^{53}$ 后，浮点数开始出现间隔，**只有间隔的倍数**才能被精确表示。
最小间隔 $Ulp$ 的计算公式为：$\text{Ulp} = 2^{e - 52}$
其中 $e$ 是该数值范围的指数。

| 数值范围 $V$           | 对应指数 $e$ | 最小间隔 (Ulp) $2^{e-52}$     | 精度规律（可精确表示的整数） |
| :----------------- | :------- | :------------------------ | :------------- |
| $[2^{53}, 2^{54})$ | $e = 53$ | $2^{53-52} = \mathbf{2}$  | **偶数** (2 的倍数) |
| $[2^{54}, 2^{55})$ | $e = 54$ | $2^{54-52} = \mathbf{4}$  | **4 的倍数**      |
| $[2^{55}, 2^{56})$ | $e = 55$ | $2^{55-52} = \mathbf{8}$  | **8 的倍数**      |
| $[2^{56}, 2^{57})$ | $e = 56$ | $2^{56-52} = \mathbf{16}$ | **16 的倍数**     |
| $\dots$            | $\dots$  | $\dots$                   | $\dots$        |
##### 舍入规则
###### **关键解释：ULP 和“舍入到最近，遇半进一到偶数”**
在 JavaScript 中，`number` 类型是 64 位双精度浮点数，它遵循 IEEE 754 标准，使用的默认舍入模式是 **"Round Half to Even"**（舍入到最近，如果恰好位于中间，则舍入到偶数）。
- 小于中点 2，向下舍入
- 大于中点 2，向上舍入
- 当遇到 $2^{53} + 1$ 或 $2^{54} + 1$ 等**刚好**位于 $2^N$ 起点的中点时，**向下舍入**到 $2^N$ (尾数全零)。
- 在其他中点（如 $2^{53} + 3$），则按常规舍入到 **最近的偶数**（通常是向上舍入）。
###### ULP 确定
当数值超过 $2^{53}$ 时，ULP（最小间隔）为 $2^{53 - 52} = 2^1 = 2$。
* **可精确表示的数**必须是 **2 的倍数**。
###### **可精确表示的相邻数值**
我们来看 $2^{53}$ 周围的可精确表示的数值（即 2 的倍数）：

| 可精确表示的值 | 相当于 $2^{53}$ 加上... |
| :--- | :--- |
| $9007199254740992$ | $2^{53} + 0$ |
| $9007199254740994$ | $2^{53} + 2$ |
| $9007199254740996$ | $2^{53} + 4$ |
###### **分析 $2^{53} + 3$**
输入 $X = 2^{53} + 3$ (即 $9007199254740995$) 位于两个精确值之间：
* $9007199254740994$
* $9007199254740996$
$X$ 恰好是这两个可表示数值的**中间值**！
$$\frac{9007199254740994 + 9007199254740996}{2} = 9007199254740995$$
###### **应用“遇半进一到偶数”规则**
当一个数字恰好位于两个可表示数值的中间时，IEEE 754 规则要求舍入到**最近的偶数**（可表示数）：

| 中间值 $X$ | 舍入到... | 结果 |
| :--- | :--- | :--- |
| $9007199254740995$ | 向上舍入到偶数 $9007199254740996$ | **9007199254740996** |
这就是为什么 $2^{53} + 3$ 会舍弃 3，直接跳到 6 的位置。它并没有舍入到 $2^{53} + 2$（9007199254740994），而是向上舍入到了下一个偶数 $2^{53} + 4$ (9007199254740996)。
###### **总结**
您观察到的现象**不是错误**，而是 64 位浮点数在处理位于最小间隔中点的数字时，**严格遵循 IEEE 754 舍入规则**（Round Half to Even）的结果。
##### 64 位浮点数边界效应示例
在下面的例子中，**加粗下划线的 53 位** ($\mathbf{\underline{1}} + 52$ 位) 是浮点数**能够精确存储**的有效数字部分。
###### **1. $2^{53}$ 边界（Ulp = 2，只能精确表示偶数）**

| 数值 (十进制) | 原始二进制 (54 位) | 规范化有效位 ($\times 2^{53}$) | 存储结果/精度 |
| :--- | :--- | :--- | :--- |
| **$2^{53}$** | $1 \text{ 后面 } 53 \text{ 个 } 0$ | $\mathbf{\underline{1.} 00\dots00} \underline{\mathbf{0}} \times 2^{53}$ | **精确表示** |
| | | 52 个 $0$ | 被截断的 $0$ |
| **$2^{53} + 1$** | $1 \dots 001$ | $\mathbf{\underline{1.} 00\dots00} \underline{\mathbf{1}} \times 2^{53}$ | **精度丢失** |
| | | $M = 52$ 个 $0$ | 被截断的 $1 \to$ **向下舍入**到 $2^{53}$ |
| **$2^{53} + 2$** | $1 \dots 010$ | $\mathbf{\underline{1.} 00\dots01} \underline{\mathbf{0}} \times 2^{53}$ | **精确表示** |
| | | 50 个 $0$, 然后是 $10$ | 被截断的 $0$ |
###### **2. $2^{54}$ 边界（Ulp = 4，只能精确表示 4 的倍数）**

| 数值 (十进制)         | 原始二进制 (55 位)                    | 规范化有效位 ($\times 2^{54}$)                                                  | 存储结果/精度                          |
| :--------------- | :------------------------------ | :------------------------------------------------------------------------ | :------------------------------- |
| **$2^{54}$**     | $1 \text{ 后面 } 54 \text{ 个 } 0$ | $\mathbf{\underline{1.} 00\dots00} \underline{\mathbf{00}} \times 2^{54}$ | **精确表示**                         |
|                  |                                 | 52 个 $0$                                                                  | 被截断的 $00$                        |
| **$2^{54} + 1$** | $1 \dots 001$                   | $\mathbf{\underline{1.} 00\dots00} \underline{\mathbf{01}} \times 2^{54}$ | **精度丢失**                         |
|                  |                                 | 52 个 $0$                                                                  | 被截断的 $01 \to$ **向下舍入**到 $2^{54}$ |
| **$2^{54} + 4$** | $1 \dots 100$                   | $\mathbf{\underline{1.} 00\dots01} \underline{\mathbf{00}} \times 2^{54}$ | **精确表示**                         |
|                  |                                 | 50 个 $0$, 然后 $100$                                                        | 被截断的 $00$                        |
##### 总结
1.  **精度上限：** $2^{53}$ 是**连续整数**能够被精确表示的上限。
2.  **误差源头：** 在危险区内，如果一个整数不是 $\text{Ulp}$ 的倍数，它就会被舍入到最近的可表示数，导致**精度丢失**。
3.  **计算安全：** 对于需要精确整数运算的场景（如金额或 ID），应确保数值始终保持在 $|N| \le 2^{53}$ 的**安全范围**内。
#### 避免精度丢失
- **使用 `BigInt` 处理大整数**：
```js
const bigIntValue = 9007199254740992n + 1n; // 9007199254740993n
```
- **高精度库**：  
    使用 `decimal.js` 或 `BigNumber.js` 处理浮点数运算。
- **限制范围**：  
    在需要精确计算的场景中（如金融、科学计算），避免使用超出 `2^53` 的整数。
### 数值范围
根据国际标准 IEEE 754，64 位浮点数使用**11 位**来表示**指数**，最大值是2047($2^{11}-1$)，这使其**数值范围为：** **$2^{-1023}$ 到 $2^{1024}$** 。
如果$>=2^{1024}$，那么就会发生“**正向溢出**”，即 JavaScript 无法表示这么大的数，这时就会返回`Infinity`。
```js
console.log(Math.pow(2, 1024)); // Infinity
```
如果$<=2^{-1075}$（指数部分最小值-1023，再加上小数部分的52位），那么就会发生为“**负向溢出**”，即 JavaScript 无法表示这么小的数，这时会直接返回0。
```js
console.log(Math.pow(2, -1075)); // 0
```
下面是一个实际的例子。
```js
var x = 0.5;
for(var i = 0; i < 25; i++) {
  x = x * x;
}
x // 0
```
上面代码中，对`0.5`连续做25次平方，由于最后结果太接近0，超出了可表示的范围，JavaScript 就直接将其转为0。

JavaScript 提供`Number`对象的 **`MAX_VALUE`** 和 **`MIN_VALUE`** 属性，返回可以表示的具体的**最大值**和**最小值**。
```js
Number.MAX_VALUE // 1.7976931348623157e+308
Number.MIN_VALUE // 5e-324
```
### 数值的表示法
JavaScript 的数值有多种表示方法，可以用字面形式直接表示，比如`35`（十进制）和`0xFF`（十六进制）。
数值也可以采用科学计数法表示，下面是几个科学计数法的例子。
```js
123e3 // 123000
123e-3 // 0.123
-3.1E+12
.1e-23
```
科学计数法允许**字母`e`或`E`的后面**，跟着一个**整数**，表示这个数值的**指数部分**。
以下两种情况，JavaScript 会**自动**将数值**转为科学计数法**表示，**其他情况**都采用**字面形式直接表示**。
#### 小数点前的数字多于21位
```js
console.log(123456789012345678901); // 123456789012345680000  
console.log(1234567890123456789012); // 1.2345678901234568e+21
```
#### 小数点后的零多于5个
```js
console.log(0.0000003); // 3e-7
console.log(0.000003); // 0.000003
```
### 数值的进制
使用字面量（literal）直接表示一个数值时，JavaScript 对整数提供四种进制的表示方法：十进制、十六进制、八进制、二进制。
- 十进制：没有前导0的数值。
- 八进制：有前缀`0o`或`0O`的数值，或者有前导0、且只用到0-7的八个阿拉伯数字的数值。
- 十六进制：有前缀`0x`或`0X`的数值。
- 二进制：有前缀`0b`或`0B`的数值。
默认情况下，JavaScript 内部会**自动**将八进制、十六进制、二进制**转为十进制**。下面是一些例子。
```js
0xff // 255
0o377 // 255
0b11 // 3
```
如果八进制、十六进制、二进制的数值里面，出现不属于该进制的数字，就会报错。
```js
0xzz // 报错
0o88 // 报错
0b22 // 报错
```
上面代码中，十六进制出现了字母`z`、八进制出现数字`8`、二进制出现数字`2`，因此报错。
通常来说，有前导0的数值会被视为八进制，但是如果**前导0后面有数字`8`和`9`**，则该数值被**视为十进制**。
```js
0888 // 888
0777 // 511
```
前导0表示八进制，处理时很容易造成混乱。ES5 的严格模式和 ES6，已经废除了这种表示法，但是浏览器为了兼容以前的代码，目前还继续支持这种表示法。
### 特殊数值
JavaScript 提供了几个特殊的数值。
#### 正零和负零
JavaScript 内部实际上存在2个`0`：一个是`+0`，一个是`-0`，区别就是64位浮点数表示法的符号位不同。它们是等价的。
```js
-0 === +0 // true
0 === -0 // true
0 === +0 // true
```
几乎所有场合，正零和负零都会被当作正常的`0`。
```js
+0 // 0
-0 // 0
(-0).toString() // '0'
(+0).toString() // '0'
```
唯一有区别的场合是，**`+0`或`-0`当作分母，返回的值是不相等的**。
```js
(1 / +0) === (1 / -0) // false
```
上面的代码之所以出现这样结果，是因为除以正零得到`+Infinity`，除以负零得到`-Infinity`，这两者是不相等的（关于`Infinity`详见下文）。
#### NaN
##### 含义
`NaN`是 JavaScript 的特殊值，表示“**非数字**”（**Not a Number**），主要出现在将字符串解析成数字出错的场合。
```js
5 - 'x' // NaN
```
上面代码运行时，会自动将字符串`x`转为数值，但是由于`x`不是数值，所以最后得到结果为`NaN`，表示它是“非数字”（`NaN`）。
另外，一些数学函数的运算结果会出现`NaN`。
```js
Math.acos(2) // NaN
Math.log(-1) // NaN
Math.sqrt(-1) // NaN
```
`0`除以`0`也会得到`NaN`。
```js
0 / 0 // NaN
```
需要注意的是，`NaN`**不是独立的数据类型**，而是一个**特殊数值**，它的数据类型依然属于`Number`，使用`typeof`运算符可以看得很清楚。
```js
typeof NaN // 'number'
```
##### 运算规则
`NaN`**不等于任何值**，包括它本身。
```js
NaN === NaN // false
```
数组的`indexOf`方法内部使用的是严格相等运算符，所以该方法对`NaN`不成立。
```js
[NaN].indexOf(NaN) // -1
```
`NaN`在布尔运算时被当作`false`。
```js
Boolean(NaN) // false
```
`NaN`与任何数（包括它自己）的运算，得到的都是`NaN`。
```js
NaN + 32 // NaN
NaN - 32 // NaN
NaN * 32 // NaN
NaN / 32 // NaN
```
但是，ES6 引入指数运算符（`**`）后，出现了一个例外。
```js
NaN ** 0 // 1
```
#### Infinity
##### 含义
`Infinity`表示“**无穷**”，用来表示两种场景。一种是一个正的数值**太大**，或一个负的数值**太小**，无法表示；
另一种是非0数值除以0，得到`Infinity`。
```js
// 场景一
Math.pow(2, 1024)
// Infinity

// 场景二
0 / 0 // NaN
1 / 0 // Infinity
```
上面代码中，第一个场景是一个表达式的计算结果太大，超出了能够表示的范围，因此返回`Infinity`。
第二个场景是`0`除以`0`会得到`NaN`，而非0数值除以`0`，会返回`Infinity`。

`Infinity`有正负之分，`Infinity`表示正的无穷，`-Infinity`表示负的无穷。
```js
Infinity === -Infinity // false

1 / -0 // -Infinity
-1 / -0 // Infinity
```
上面代码中，非零正数除以`-0`，会得到`-Infinity`，负数除以`-0`，会得到`Infinity`。

由于数值正向溢出（overflow）、负向溢出（underflow）和被`0`除，**JavaScript** 都不报错，所以单纯的**数学运算几乎没有可能抛出错误**。

**`Infinity`大于一切数值（除了`NaN`），`-Infinity`小于一切数值（除了`NaN`）。**
```js
Infinity > 1000 // true
-Infinity < -1000 // true
```
`Infinity`与`NaN`比较，总是返回`false`。
```js
Infinity > NaN // false
-Infinity > NaN // false

Infinity < NaN // false
-Infinity < NaN // false
```
##### 运算规则
`Infinity`的四则运算，符合无穷的数学计算规则。
```js
5 * Infinity // Infinity
5 - Infinity // -Infinity
Infinity / 5 // Infinity
5 / Infinity // 0
```

0乘以`Infinity`，返回`NaN`；0除以`Infinity`，返回`0`；`Infinity`除以0，返回`Infinity`。
```js
0 * Infinity // NaN
0 / Infinity // 0
Infinity / 0 // Infinity
```

`Infinity`加上或乘以`Infinity`，返回的还是`Infinity`。
```js
Infinity + Infinity // Infinity
Infinity * Infinity // Infinity
```

`Infinity`减去或除以`Infinity`，得到`NaN`。
```js
Infinity - Infinity // NaN
Infinity / Infinity // NaN
```

`Infinity`与`null`计算时，`null`会转成0，等同于与`0`的计算。
```js
null * Infinity // NaN
null / Infinity // 0
Infinity / null // Infinity
```

`Infinity`与`undefined`计算，返回的都是`NaN`。
```js
undefined + Infinity // NaN
undefined - Infinity // NaN
undefined * Infinity // NaN
undefined / Infinity // NaN
Infinity / undefined // NaN
```
### 与数值相关的全局方法
#### parseInt()
**（1）基本用法**
`parseInt`方法用于将字符串转为整数。
```js
parseInt('123') // 123
```
如果字符串头部有空格，空格会被自动去除。
```js
parseInt('   81') // 81
```
如果`parseInt`的参数不是字符串，则**会先转为字符串再转换**。
```js
parseInt(1.23) // 1
// 等同于
parseInt('1.23') // 1
```
字符串转为整数的时候，是一个个字符依次转换，如果遇到不能转为数字的字符，就不再进行下去，返回已经转好的部分。
```js
parseInt('8a') // 8
parseInt('12**') // 12
parseInt('12.34') // 12
parseInt('15e2') // 15
parseInt('15px') // 15
```
上面代码中，`parseInt`的参数都是字符串，结果只返回字符串头部可以转为数字的部分。
如果字符串的第一个字符不能转化为数字（后面跟着数字的正负号除外），返回`NaN`。
```js
parseInt('abc') // NaN
parseInt('.3') // NaN
parseInt('') // NaN
parseInt('+') // NaN
parseInt('+1') // 1
```
所以，`parseInt`的返回值只有两种可能，要么是一个十进制整数，要么是`NaN`。
如果字符串以`0x`或`0X`开头，`parseInt`会将其按照十六进制数解析。
```js
parseInt('0x10') // 16
```
如果字符串以`0`开头，将其按照10进制解析。
```js
parseInt('011') // 11
```
对于那些会自动转为科学计数法的数字，`parseInt`会将科学计数法的表示方法视为字符串，因此导致一些奇怪的结果。
```js
parseInt(1000000000000000000000.5) // 1
// 等同于
parseInt('1e+21') // 1

parseInt(0.0000008) // 8
// 等同于
parseInt('8e-7') // 8
```

**（2）进制转换**
`parseInt`方法还可以接受第二个参数（**2到36之间**），表示被解析的值的进制，返回该值对应的十进制数。默认情况下，`parseInt`的**第二个参数为10，即默认是十进制**转十进制。
```js
parseInt('1000') // 1000
// 等同于
parseInt('1000', 10) // 1000
```
下面是转换指定进制的数的例子。
```js
parseInt('1000', 2) // 8 「二进制」
parseInt('1000', 6) // 216 「六进制」
parseInt('1000', 8) // 512 「八进制」
```
这意味着，可以用`parseInt`方法进行**进制的转换**。

如果第二个参数不是数值，会被自动转为一个整数。这个整数只有在2到36之间，才能得到有意义的结果，超出这个范围，则返回`NaN`。
如果第二个参数是`0`、`undefined`和`null`，则直接忽略。
```js
parseInt('10', 37) // NaN
parseInt('10', 1) // NaN
parseInt('10', 0) // 10
parseInt('10', null) // 10
parseInt('10', undefined) // 10
```

如果字符串包含对于指定进制无意义的字符，则从最高位开始，只返回可以转换的数值。如果最高位无法转换，则直接返回`NaN`。
```js
parseInt('1546', 2) // 1
parseInt('546', 2) // NaN
```
上面代码中，对于二进制来说，`1`是有意义的字符，`5`、`4`、`6`都是无意义的字符，所以第一行返回1，第二行返回`NaN`。

前面说过，如果`parseInt`的**第一个参数不是字符串，会被先转为字符串**。这会导致一些令人意外的结果。
```js
parseInt(0x11, 36) // 43
parseInt(0x11, 2) // 1

// 等同于
parseInt(String(0x11), 36)
parseInt(String(0x11), 2)

// 等同于
parseInt('17', 36)
parseInt('17', 2)
```

上面代码中，十六进制的`0x11`会被先转为十进制的17，再转为字符串。然后，再用36进制或二进制解读字符串`17`，最后返回结果`43`和`1`。

这种处理方式，对于八进制的前缀0，尤其需要注意。
```js
parseInt(011, 2) // NaN
// 等同于
parseInt(String(011), 2)
// 等同于
parseInt(String(9), 2)
```

上面代码中，第一行的`011`会被先转为字符串`9`，因为`9`不是二进制的有效字符，所以返回`NaN`。如果直接计算`parseInt('011', 2)`，`011`则是会被当作二进制处理，返回3。

JavaScript 不再允许将带有前缀0的数字视为八进制数，而是要求忽略这个`0`。但是，为了保证兼容性，大部分浏览器并没有部署这一条规定。

#### parseFloat()
`parseFloat`方法用于将一个字符串转为浮点数。
```js
parseFloat('3.14') // 3.14
```
如果字符串符合科学计数法，则会进行相应的转换。
```js
parseFloat('314e-2') // 3.14
parseFloat('0.0314E+2') // 3.14
```
如果字符串包含不能转为浮点数的字符，则不再进行往后转换，返回已经转好的部分。
```js
parseFloat('3.14more non-digit characters') // 3.14
```
`parseFloat`方法会自动过滤字符串前导的空格。
```js
parseFloat('\t\v\r12.34\n ') // 12.34
```
如果参数不是字符串，则会先转为字符串再转换。
```js
parseFloat([1.23]) // 1.23
// 等同于
parseFloat(String([1.23])) // 1.23
```
如果字符串的第一个字符不能转化为浮点数，则返回`NaN`。
```js
parseFloat([]) // NaN
parseFloat('FF2') // NaN
parseFloat('') // NaN
```
上面代码中，尤其值得注意，`parseFloat`会将空字符串转为`NaN`。

这些特点使得`parseFloat`的转换结果不同于`Number`函数。
```js
parseFloat(true)  // NaN
Number(true) // 1

parseFloat(null) // NaN
Number(null) // 0

parseFloat('') // NaN
Number('') // 0

parseFloat('123.45#') // 123.45
Number('123.45#') // NaN
```
#### isNaN()
`isNaN`方法可以用来判断一个值是否为`NaN`。
```js
isNaN(NaN) // true
isNaN(123) // false
```
但是，`isNaN`只对数值有效，如果传入其他值，**会被先转成数值**。
比如，传入字符串的时候，字符串会被先转成`NaN`，所以最后返回`true`，这一点要特别引起注意。
**也就是说，`isNaN`为`true`的值，有可能不是`NaN`，而是一个字符串。**
```js
isNaN('Hello') // true
// 相当于
isNaN(Number('Hello')) // true
```
出于同样的原因，对于对象和数组，`isNaN`也返回`true`。
```js
isNaN({}) // true
// 等同于
isNaN(Number({})) // true

isNaN(['xzy']) // true
// 等同于
isNaN(Number(['xzy'])) // true
```
但是，对于空数组和只有一个数值成员的数组，`isNaN`返回`false`。
```js
isNaN([]) // false
isNaN([123]) // false
isNaN(['123']) // false
```
上面代码之所以返回`false`，原因是这些数组能被`Number`函数转成数值，请参见《数据类型转换》一章。

因此，使用`isNaN`之前，最好判断一下数据类型。
```js
function myIsNaN(value) {
  return typeof value === 'number' && isNaN(value);
}
```
判断`NaN`更可靠的方法是，**利用`NaN`为唯一不等于自身的值的这个特点**，进行判断。
```js
function myIsNaN(value) {
  return value !== value;
}
```
#### isFinite()
`isFinite`方法返回一个布尔值，表示某个值是否为正常的数值。
```js
isFinite(Infinity) // false
isFinite(-Infinity) // false
isFinite(NaN) // false
isFinite(undefined) // false
isFinite(null) // true
isFinite(-1) // true
```
除了`Infinity`、`-Infinity`、`NaN`和`undefined`这几个值会返回`false`，`isFinite`对于其他的数值都会返回`true`。
## 字符串
[字符串 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/types/string)
### 定义
字符串就是零个或多个排在一起的字符，放在单引号或双引号之中。
### 连接运算符（`+`）
可以连接多个单行字符串，将长字符串拆成多行书写，输出的时候也是单行。
```js
var longString = 'Long '
  + 'long '
  + 'long '
  + 'string';
```
### 转义
反斜杠（`\`）在字符串内有特殊含义，用来表示一些特殊字符，所以又称为转义符。
需要用反斜杠转义的特殊字符，主要有下面这些。
- `\0` ：null（`\u0000`）
- `\b` ：后退键（`\u0008`）
- `\f` ：换页符（`\u000C`）
- `\n` ：换行符（`\u000A`）
- `\r` ：回车键（`\u000D`）
- `\t` ：制表符（`\u0009`）
- `\v` ：垂直制表符（`\u000B`）
- `\'` ：单引号（`\u0027`）
- `\"` ：双引号（`\u0022`）
- `\\` ：反斜杠（`\u005C`）
### 字符串与数组
字符串可以被视为字符数组，因此可以使用数组的方括号运算符，用来返回某个位置的字符（位置编号从0开始）。
```js
var s = 'hello';
s[0] // "h"
s[1] // "e"
s[4] // "o"

// 直接对字符串使用方括号运算符
'hello'[1] // "e"
```
如果方括号中的数字超过字符串的长度，或者方括号中根本不是数字，则返回`undefined`。
```js
'abc'[3] // undefined
'abc'[-1] // undefined
'abc'['x'] // undefined
```
但是，字符串与数组的**相似性仅此而已**。实际上，无法改变字符串之中的单个字符。
```js
var s = 'hello';

delete s[0];
s // "hello"

s[1] = 'a';
s // "hello"

s[5] = '!';
s // "hello"
```
上面代码表示，字符串内部的单个字符无法改变和增删，这些操作会默默地失败。
### length 属性
`length`属性返回字符串的长度，该属性也是无法改变的。
```js
var s = 'hello';
s.length // 5

s.length = 3;
s.length // 5

s.length = 7;
s.length // 5
```
上面代码表示字符串的`length`属性无法改变，但是不会报错。
### 字符集
JavaScript 使用 Unicode 字符集。JavaScript 引擎内部，所有字符都用 Unicode 表示。
### Base64 转码
JavaScript 原生提供两个 Base64 相关的方法。
- `btoa()`：任意值转为 Base64 编码
- `atob()`：Base64 编码转为原来的值
```js
var string = 'Hello World!';
btoa(string) // "SGVsbG8gV29ybGQh"
atob('SGVsbG8gV29ybGQh') // "Hello World!"
```
注意，这两个方法不适合非 ASCII 码的字符，会报错。
```js
btoa('你好') // 报错
```
要将非 ASCII 码字符转为 Base64 编码，必须中间插入一个转码环节，再使用这两个方法。
```js
function b64Encode(str) {
  return btoa(encodeURIComponent(str));
}

function b64Decode(str) {
  return decodeURIComponent(atob(str));
}

b64Encode('你好') // "JUU0JUJEJUEwJUU1JUE1JUJE"
b64Decode('JUU0JUJEJUEwJUU1JUE1JUJE') // "你好"
```
## 对象
[对象 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/types/object)
### 概述
#### 生成方法
对象（object）是 JavaScript 语言的核心概念，也是最重要的数据类型。
什么是对象？简单说，对象就是一组“键值对”（key-value）的集合，是一种无序的复合数据集合。
```js
var obj = {
  foo: 'Hello',
  bar: 'World'
};
```
#### 键名
对象的所有键名都是字符串（ES6 又引入了 Symbol 值也可以作为键名），所以加不加引号都可以。上面的代码也可以写成下面这样。
```js
var obj = {
  'foo': 'Hello',
  'bar': 'World'
};
```
如果键名是数值，会被自动转为字符串。
```js
var obj = {
  1: 'a',
  3.2: 'b',
  1e2: true,
  1e-2: true,
  .234: true,
  0xFF: true
};

obj
// Object {
//   1: "a",
//   3.2: "b",
//   100: true,
//   0.01: true,
//   0.234: true,
//   255: true
// }

obj['100'] // true
```
上面代码中，对象`obj`的所有键名虽然看上去像数值，**实际上都被自动转成了字符串。**

如果键名不符合标识名的条件（比如第一个字符为数字，或者含有空格或运算符），且也不是数字，则必须加上引号，否则会报错。
```js
// 报错
var obj = {
  1p: 'Hello World'
};

// 不报错
var obj = {
  '1p': 'Hello World',
  'h w': 'Hello World',
  'p+q': 'Hello World'
};
```
上面对象的三个键名，都不符合标识名的条件，所以必须加上引号。

对象的每一个键名又称为“属性”（property），它的 **“键值”可以是任何数据类型**。
如果一个**属性的值为函数**，通常把这个**属性称为“方法”**，它可以**像函数那样调用**。
```js
var obj = {
  p: function (x) {
    return 2 * x;
  }
};

obj.p(1) // 2
```
上面代码中，对象`obj`的属性`p`，就指向一个函数。

如果属性的值还是一个对象，就形成了链式引用。
```js
var o1 = {};
var o2 = { bar: 'hello' };

o1.foo = o2;
o1.foo.bar // "hello"
```
上面代码中，对象`o1`的属性`foo`指向对象`o2`，就可以链式引用`o2`的属性。

对象的属性之间用逗号分隔，最后一个属性后面可以加逗号（trailing comma），也可以不加。
```js
var obj = {
  p: 123,
  m: function () { ... },
}
```
上面的代码中，`m`属性后面的那个逗号，有没有都可以。

属性可以动态创建，不必在对象声明时就指定。
```js
var obj = {};
obj.foo = 123;
obj.foo // 123
```
上面代码中，直接对`obj`对象的`foo`属性赋值，结果就在运行时创建了`foo`属性。
#### 对象的引用
如果不同的变量名指向同一个对象，那么它们都是这个对象的引用，也就是说指向同一个内存地址。修改其中一个变量，**会影响到其他所有变量**。
```js
var o1 = {};
var o2 = o1;

o1.a = 1;
o2.a // 1

o2.b = 2;
o1.b // 2
```
上面代码中，`o1`和`o2`指向同一个对象，因此为其中任何一个变量添加属性，另一个变量都可以读写该属性。

此时，如果取消某一个变量对于原对象的引用，不会影响到另一个变量。
```js
var o1 = {};
var o2 = o1;

o1 = 1;
o2 // {}
```
上面代码中，`o1`和`o2`指向同一个对象，然后`o1`的值变为1，这时不会对`o2`产生影响，`o2`还是指向原来的那个对象。

但是，这种引用只局限于对象，如果两个变量指向同一个**原始类型的值。那么，变量这时都是值的拷贝。**
```js
var x = 1;
var y = x;

x = 2;
y // 1
```
上面的代码中，当`x`的值发生变化后，`y`的值并不变，这就表示`y`和`x`并不是指向同一个内存地址。
#### 表达式还是语句？
对象采用大括号表示，这导致了一个问题：如果行首是一个大括号，它到底是表达式还是语句？
```js
{ foo: 123 }
```
JavaScript 引擎读到上面这行代码，会发现可能有两种含义。第一种可能是，这是一个表达式，表示一个包含`foo`属性的对象；第二种可能是，这是一个语句，表示一个代码区块，里面有一个标签`foo`，指向表达式`123`。

为了避免这种歧义，JavaScript 引擎的做法是，如果遇到这种情况，无法确定是对象还是代码块，**一律解释为代码块**。
```js
{ console.log(123) } // 123
```
上面的语句是一个代码块，而且只有解释为代码块，才能执行。

如果要解释为对象，最好在大括号前加上圆括号。因为**圆括号的里面，只能是表达式**，所以确保大括号只能解释为对象。
```js
({ foo: 123 }) // 正确
({ console.log(123) }) // 报错
```

这种差异在`eval`语句（作用是对字符串求值）中反映得最明显。
```js
eval('{foo: 123}') // 123
eval('({foo: 123})') // {foo: 123}
```
上面代码中，如果没有圆括号，`eval`将其理解为一个代码块；加上圆括号以后，就理解成一个对象。
### 属性的操作
#### 属性的读取
读取对象的属性，有两种方法。
- 一种是使用**点运算符**。
- 一种是使用**方括号运算符**。
```js
var obj = {
  p: 'Hello World'
};

obj.p // "Hello World"
obj['p'] // "Hello World"
```
上面代码分别采用点运算符和方括号运算符，读取属性`p`。

请注意，如果使用方括号运算符，键名必须放在引号里面，否则会被当作变量处理。
```js
var foo = 'bar';

var obj = {
  foo: 1,
  bar: 2
};

obj.foo  // 1
obj[foo]  // 2
```
上面代码中，引用对象`obj`的`foo`属性时：
- 如果使用点运算符，`foo`就是字符串
- 如果使用方括号运算符，但是不使用引号，那么`foo`就是一个变量，指向字符串`bar`

方括号运算符内部还可以使用表达式。
```js
obj['hello' + ' world']
obj[3 + 3]
```

数字键可以不加引号，因为会自动转成字符串。
```js
var obj = {
  0.7: 'Hello World'
};

obj['0.7'] // "Hello World"
obj[0.7] // "Hello World"
```
上面代码中，对象`obj`的数字键`0.7`，加不加引号都可以，因为会被自动转为字符串。

注意，**数值键名不能使用点运算符**（因为会被当成小数点），只能使用方括号运算符。
```js
var obj = {
  123: 'hello world'
};

obj.123 // 报错
obj[123] // "hello world"
```
上面代码的第一个表达式，对数值键名`123`使用点运算符，结果报错。第二个表达式使用方括号运算符，结果就是正确的。
#### 属性的赋值
点运算符和方括号运算符，不仅可以用来读取值，还可以用来赋值。
```js
var obj = {};

obj.foo = 'Hello';
obj['bar'] = 'World';
```
上面代码中，分别使用点运算符和方括号运算符，对属性赋值。

JavaScript 允许属性的“后绑定”，也就是说，你可以在任意时刻新增属性，没必要在定义对象的时候，就定义好属性。
```js
var obj = { p: 1 };

// 等价于

var obj = {};
obj.p = 1;
```
#### 属性的查看
查看一个对象本身的所有属性，可以使用`Object.keys`方法。
```js
var obj = {
  key1: 1,
  key2: 2
};

Object.keys(obj);
// ['key1', 'key2']
```
#### 属性的删除：delete 命令
`delete`命令用于删除对象的属性，删除成功后返回`true`。
```js
var obj = { p: 1 };
Object.keys(obj) // ["p"]

delete obj.p // true
obj.p // undefined
Object.keys(obj) // []
```
上面代码中，`delete`命令删除对象`obj`的`p`属性。删除后，再读取`p`属性就会返回`undefined`，而且`Object.keys`方法的返回值也不再包括该属性。

**注意**：删除一个不存在的属性，`delete`不报错，而且返回`true`。
```js
var obj = {};
delete obj.p // true
```
上面代码中，对象`obj`并没有`p`属性，但是`delete`命令照样返回`true`。因此，不能根据`delete`命令的结果，认定某个属性是存在的。

只有一种情况，`delete`命令会返回`false`，那就是该属性存在，且不得删除。
```js
var obj = Object.defineProperty({}, 'p', {
  value: 123,
  configurable: false
});

obj.p // 123
delete obj.p // false
```
上面代码之中，对象`obj`的`p`属性是不能删除的，所以`delete`命令返回`false`（关于`Object.defineProperty`方法的介绍，请看《标准库》的 Object 对象一章）。

另外，需要注意的是，`delete`命令只能删除对象本身的属性，**无法删除继承的属性**（关于继承参见《面向对象编程》章节）。
```js
var obj = {};
delete obj.toString // true
obj.toString // function toString() { [native code] }
```
上面代码中，`toString`是对象`obj`继承的属性，虽然`delete`命令返回`true`，但该属性并没有被删除，依然存在。这个例子还说明，即使`delete`返回`true`，该属性依然可能读取到值。
#### 属性是否存在：in 运算符
`in`运算符用于检查对象是否包含某个属性（**注意，检查的是键名，不是键值**），如果包含就返回`true`，否则返回`false`。它的左边是一个字符串，表示属性名，右边是一个对象。
```js
var obj = { p: 1 };
'p' in obj // true
'toString' in obj // true
```
`in`运算符的一个问题是，它**不能识别哪些属性是对象自身的，哪些属性是继承的**。就像上面代码中，对象`obj`本身并没有`toString`属性，但是`in`运算符会返回`true`，因为这个属性是继承的。

这时，可以使用对象的`hasOwnProperty`方法判断一下，是否为对象自身的属性。
```js
var obj = {};
if ('toString' in obj) {
  console.log(obj.hasOwnProperty('toString')) // false
}
```
#### 属性的遍历：for...in 循环
`for...in`循环用来遍历一个对象的全部属性。
```js
var obj = {a: 1, b: 2, c: 3};

for (var i in obj) {
  console.log('键名：', i);
  console.log('键值：', obj[i]);
}
// 键名： a
// 键值： 1
// 键名： b
// 键值： 2
// 键名： c
// 键值： 3
```
**`for...in`循环有两个使用注意点。**
- 它遍历的是对象所有可遍历（enumerable）的属性，**会跳过不可遍历的属性**。
- 它不仅遍历对象自身的属性，**还遍历继承的属性**。

举例来说，对象都继承了`toString`属性，但是`for...in`循环不会遍历到这个属性。
```js
var obj = {};

// toString 属性是存在的
obj.toString // toString() { [native code] }

for (var p in obj) {
  console.log(p);
} // 没有任何输出
```
上面代码中，对象`obj`继承了`toString`属性，该属性不会被`for...in`循环遍历到，因为它**默认是“不可遍历”的**。关于对象属性的可遍历性，参见《标准库》章节中 Object 一章的介绍。

如果继承的属性是可遍历的，那么就会被`for...in`循环遍历到。但是，一般情况下，都是只想遍历对象自身的属性，所以使用`for...in`的时候，应该结合使用`hasOwnProperty`方法，在循环内部判断一下，某个属性是否为对象自身的属性。
```js
var person = { name: '老张' };

for (var key in person) {
  if (person.hasOwnProperty(key)) {
    console.log(key);
  }
}
// name
```
### with 语句 **「不建议使用」**
`with`语句的格式如下：
```js
with (对象) {
  语句;
}
```
它的作用是操作同一个对象的多个属性时，提供一些书写的方便。
```js
// 例一
var obj = {
  p1: 1,
  p2: 2,
};
with (obj) {
  p1 = 4;
  p2 = 5;
}
// 等同于
obj.p1 = 4;
obj.p2 = 5;

// 例二
with (document.links[0]){
  console.log(href);
  console.log(title);
  console.log(style);
}
// 等同于
console.log(document.links[0].href);
console.log(document.links[0].title);
console.log(document.links[0].style);
```

**注意**：如果`with`区块内部有变量的赋值操作，必须是当前对象已经存在的属性，否则会创造一个当前作用域的全局变量。
```js
var obj = {};
with (obj) {
  p1 = 4;
  p2 = 5;
}

obj.p1 // undefined
p1 // 4
```
上面代码中，对象`obj`并没有`p1`属性，对`p1`赋值等于创造了一个全局变量`p1`。正确的写法应该是，先定义对象`obj`的属性`p1`，然后在`with`区块内操作它。

这是因为`with`区块没有改变作用域，它的内部依然是当前作用域。这造成了`with`语句的一个很大的弊病，就是绑定对象不明确。
```js
with (obj) {
  console.log(x);
}
```

单纯从上面的代码块，根本无法判断`x`到底是全局变量，还是对象`obj`的一个属性。这非常不利于代码的除错和模块化，编译器也无法对这段代码进行优化，只能留到运行时判断，这就拖慢了运行速度。
**因此，建议不要使用`with`语句，可以考虑用一个临时变量代替`with`。**
```js
with(obj1.obj2.obj3) {
  console.log(p1 + p2);
}

// 可以写成
var temp = obj1.obj2.obj3;
console.log(temp.p1 + temp.p2);
```
## 函数
函数是一段可以反复调用的代码块。函数还能接受输入的参数，不同的参数有唯一对应的返回值。
### 概述
#### 函数的声明
##### 1. function 命令
`function`命令后面是函数名，函数名后面是一对圆括号，里面是传入函数的参数。
>注：语句的结尾**不用**加上`;`分号
```js
function print(s) {
  console.log(s);
}
```
##### 2. 函数表达式
这种写法将一个匿名函数赋值给变量。这时，这个**匿名函数**又称函数表达式（Function Expression），因为赋值语句的等号右侧只能放表达式。
>注：语句的结尾**须**加上`;`分号，表示语句结束
```js
var print = function(s) {
  console.log(s);
};
```
一般采用函数表达式声明函数时，`function`命令后面不带有函数名。
**如果加上函数名**，该函数名只在函数体内部有效，在函数体外部无效，作用如下
- 一是**可以在函数体内部调用自身**
- 二是**方便除错**（除错工具显示函数调用栈时，将显示函数名，而不再显示这里是一个匿名函数）
```js
var print = function x() {
    console.log(typeof x);
};

x
// ReferenceError: x is not defined

print()
// function
```
##### 3. Function 构造函数
第三种声明函数的方式是`Function`构造函数。
```js
var add = new Function(
  'x',
  'y',
  'return x + y'
);

// 等同于
function add(x, y) {
  return x + y;
}
```
上面代码中，`Function`构造函数接受三个参数，除了最后一个参数是`add`函数的“函数体”，其他参数都是`add`函数的参数。

你可以传递任意数量的参数给`Function`构造函数，只有**最后一个参数**会**被当做函数体**，如果只有一个参数，该参数就是函数体。
`Function`构造函数**可以不使用`new`命令**，返回结果完全一样。
```js
var foo = Function(
  'return "hello world";'
);

// 等同于
function foo() {
  return 'hello world';
}
```
总的来说，这种声明函数的方式非常不直观，**几乎无人使用**。
#### 函数的重复声明
如果同一个函数被多次声明，后面的声明就会覆盖前面的声明。
```js
function f() {
  console.log(1);
}
f() // 2

function f() {
  console.log(2);
}
f() // 2
```
#### 第一等公民
JavaScript 语言将函数看作一种值，**与其它值（数值、字符串、布尔值等等）地位相同**。凡是可以使用值的地方，就能使用函数。
比如，可以把函数赋值给变量和对象的属性，也可以当作参数传入其他函数，或者作为函数的结果返回。
函数只是一个可以执行的值，此外并无特殊之处。

由于函数与其他数据类型地位平等，所以在 JavaScript 语言中又称函数为第一等公民。
```js
function add(x, y) {
  return x + y;
}

// 将函数赋值给一个变量
var operator = add;

// 将函数作为参数和返回值
function a(op){
  return op;
}
a(add)(1, 1)
// 2
```
#### 函数名的提升
JavaScript 引擎将函数名视同变量名，所以**采用`function`命令声明函数时**，整个函数会像变量声明一样，**被提升到代码头部**。所以，下面的代码不会报错。
```js
f();

function f() {}
```

**注意**：如果像下面例子那样，采用`function`命令和`var`赋值语句声明同一个函数，**由于存在函数提升**，最后会采用`var`赋值语句的定义。
```js
var f = function () {
  console.log('1');
}

function f() {
  console.log('2');
}

f() // 1
```
上面例子中，表面上后面声明的函数`f`，应该覆盖前面的`var`赋值语句，但是由于存在函数提升，实际上正好反过来。
### 函数的属性和方法
#### name 属性
函数的`name`属性返回函数的名字。
```js
function f1() {}
f1.name // "f1"
```

如果是通过变量赋值定义的函数，那么`name`属性返回变量名。
```js
var f2 = function () {};
f2.name // "f2"
```
但是，上面这种情况，只有在变量的值是一个匿名函数时才是如此。

如果变量的值是一个具名函数，那么`name`属性返回`function`关键字之后的那个函数名。
```js
var f3 = function myName() {};
f3.name // 'myName'
```
上面代码中，`f3.name`返回函数表达式的名字。注意，真正的函数名还是`f3`，而`myName`这个名字只在函数体内部可用。

`name`属性的一个用处，就是获取参数函数的名字。
```js
var myFunc = function () {};

function test(f) {
  console.log(f.name);
}

test(myFunc) // myFunc
```
上面代码中，函数`test`内部通过`name`属性，就可以知道传入的参数是什么函数。
#### length 属性
函数的`length`属性返回函数预期传入的参数个数，即函数定义之中的参数个数。
```js
function f(a, b) {}
f.length // 2
```
上面代码定义了空函数`f`，它的`length`属性就是定义时的参数个数。不管调用时输入了多少个参数，`length`属性始终等于2。
`length`属性提供了一种机制，判断定义时和调用时参数的差异，以便实现面向对象编程的“方法重载”（overload）。
#### toString()
函数的`toString()`方法返回一个字符串，内容是函数的源码。
```js
function f() {
  a();
  b();
  c();
}

f.toString()
// function f() {
//  a();
//  b();
//  c();
// }
```
上面示例中，函数`f`的`toString()`方法返回了`f`的源码，包含换行符在内。

对于那些原生的函数，`toString()`方法返回`function (){[native code]}`。
```js
Math.sqrt.toString()
// "function sqrt() { [native code] }"
```
上面代码中，`Math.sqrt()`是 JavaScript 引擎提供的原生函数，`toString()`方法就返回原生代码的提示。

函数内部的注释也可以返回。
```js
function f() {/*
  这是一个
  多行注释
*/}

f.toString()
// "function f(){/*
//   这是一个
//   多行注释
// */}"
```
利用这一点，可以变相实现多行字符串。
```js
var multiline = function (fn) {
  var arr = fn.toString().split('\n');
  return arr.slice(1, arr.length - 1).join('\n');
};

function f() {/*
  这是一个
  多行注释
*/}

multiline(f);
// " 这是一个
//   多行注释"
```
上面示例中，函数`f`内部有一个多行注释，`toString()`方法拿到`f`的源码后，去掉首尾两行，就得到了一个多行字符串。
### 函数作用域
#### 定义
作用域（scope）指的是变量存在的范围。
1. 全局作用域，变量在整个程序中一直存在，所有地方都可以读取
2. 函数作用域，变量只在函数内部存在
3. 块级作用域「ES6 新增」
##### 全局作用域
对于顶层函数来说，**函数外部**声明的变量称为 **全局变量（global variable）**，它**可以在函数内部读取**。
```js
var v = 1;

function f() {
  console.log(v);
}

f()
// 1
```

**隐式全局变量**
原因
- **未声明变量**：`i = 5` 没有使用 `var`/`let`/`const`。
- **隐式全局变量**：在非严格模式下，未声明的变量会自动绑定到 **全局作用域**（浏览器中是 `window.i`）。
- **作用域链**：函数内部找不到 `i` 时，会从全局作用域中查找，最终访问到全局变量。
潜在问题
虽然这段代码能正常运行，但存在以下问题：
1. **隐式全局变量的副作用**
    - 隐式声明的全局变量可能导致命名冲突，污染全局命名空间。
    - 在严格模式（`"use strict"`）下，未声明的变量赋值会抛出 `ReferenceError` 错误。
2. **代码可维护性**  
    隐式全局变量的写法不符合现代 JavaScript 最佳实践（推荐使用 `let` 或 `const` 显式声明变量）。
```js
(function () {
    i = 5; // 未声明变量
})();
console.log(i); // 输出: 5
```
##### 函数作用域
在**函数内部**定义的变量，称为 **局部变量（local variable）** 外部无法读取。
```js
function f(){
  var v = 1;
}

v // ReferenceError: v is not defined
```

**函数作用域覆盖全局作用域**
```js
var v = 1;

function f(){
  var v = 2;
  console.log(v);
}

f() // 2
v // 1
```

**打印最里层的变量**
```js
var param = "global";
(function () {
    var param = "big";
    (function () {
        console.log(param);
        var param = "small"
    })();
})();
// small
```

**Function 内的变量也是局部变量**
```js
var code = "var i=3000;i++;console.log(i)";
var i = 500;
new Function(code)(); // 用局部变量执行更安全
console.log("外面的", i);
/*
3001
外面的 500
*/
```
##### 注意事项
对于`var`命令来说，局部变量**只能在函数内部**声明，**在其他区块中声明，一律都是全局变量**。

**实例**
```js
if (true) {
  var x = 5; // 产生变量提升「申明会贝自动放在第一行」
}
console.log(x);  // 5

if (false) {
    var o = 5; // 产生变量提升「申明会贝自动放在第一行」
}
console.log(o); // undefined
```
#### 函数内部的变量提升
与全局作用域一样，函数作用域内部也会产生“**变量提升**”现象。`var`命令声明的变量，不管在什么位置，变量声明都会被提升到**函数体的头部**。
```js
function foo(x) {
  if (x > 100) {
    var tmp = x - 100;
  }
}

// 等同于
function foo(x) {
  var tmp;
  if (x > 100) {
    tmp = x - 100;
  };
}
```
#### 函数本身的作用域
函数本身也是一个值，也有自己的作用域。它的作用域与变量一样，**就是其声明时所在的作用域**，与其运行时所在的作用域无关。
```js
var a = 1;
var x = function () {
  console.log(a);
};

function f() {
  var a = 2;
  x();
}

f() // 1
```
上面代码中，函数`x`是在函数`f`的外部声明的，所以**它的作用域绑定外层**，内部变量`a`不会到函数`f`体内取值，所以输出`1`，而不是`2`。

同样的，函数体内部声明的函数，作用域绑定函数体内部。
```js
function foo() {
  var x = 1;
  function bar() {
    console.log(x);
  }
  return bar;
}

var x = 2;
var f = foo();
f() // 1
```
上面代码中，函数`foo`内部声明了一个函数`bar`，`bar`的作用域绑定`foo`。当我们在`foo`外部取出`bar`执行时，变量`x`指向的是`foo`内部的`x`，而不是`foo`外部的`x`。正是这种机制，构成了下文要讲解的 **“闭包”** 现象。
### 参数
#### 概述
函数运行的时候，有时需要提供外部数据，不同的外部数据会得到不同的结果，这种外部数据就叫参数。
```js
function square(x) {
  return x * x;
}

square(2) // 4
square(3) // 9
```
#### 参数的省略
函数`f`定义了两个参数，但是运行时无论提供多少个参数（或者不提供参数），JavaScript 都不会报错。省略的参数的值就变为`undefined`。
```js
function f(a, b) {
  return a;
}

f(1, 2, 3) // 1
f(1) // 1
f() // undefined

f.length // 2
```

没有办法只省略靠前的参数，而保留靠后的参数。如果一定要省略靠前的参数，只有显式传入`undefined`。
```js
function f(a, b) {
  return a;
}

f( , 1) // SyntaxError: Unexpected token ,(…)
f(undefined, 1) // undefined
```
#### 传递方式
##### 传值传递
函数参数如果是**原始类型**的值（数值、字符串、布尔值），传递方式是**传值传递**（passes by value）。这意味着，在函数体内修改参数值，不会影响到函数外部。
```js
var p = 2;

function f(p) {
  p = 3;
}
f(p);

p // 2
```
##### 传址传递
但是，如果函数参数是**复合类型**的值（数组、对象、其他函数），传递方式是**传址传递**（pass by reference）。也就是说，传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。
```js
var obj = { p: 1 };

function f(o) {
  o.p = 2;
}
f(obj);

obj.p // 2
```
**注意事项**
如果函数内部修改的，不是参数对象的某个属性，而是替换掉整个参数，这时**不会影响到原始值**。
```js
var obj = [1, 2, 3];

function f(o) {
  o = [2, 3, 4];
}
f(obj);

obj // [1, 2, 3]
```
上面代码中，在函数f()内部，参数对象obj被整个替换成另一个值。这时不会影响到原始值。
这是因为，形式参数（o）的值实际是参数obj的地址，**重新对o赋值导致o指向另一个地址**，保存在原地址上的值当然不受影响。
#### 同名参数
如果有同名的参数，则**取最后出现的那个值**。
```js
function f(a, a) {
  console.log(a);
}

f(1, 2) // 2
```
上面代码中，函数f()有两个参数，且参数名都是a。取值的时候，以后面的a为准，即使后面的a没有值或被省略，也是以其为准。
```js
function f(a, a) {
  console.log(a);
}

f(1) // undefined
```
调用函数f()的时候，没有提供第二个参数，a的取值就变成了undefined。这时，如果要获得第一个a的值，可以使用arguments对象。
```js
function f(a, a) {
  console.log(arguments[0]);
}

f(1) // 1
```
#### arguments 对象
##### 定义
由于 JavaScript 允许函数有不定数目的参数，所以需要一种机制，可以在函数体内部读取所有参数。这就是`arguments`对象的由来。

`arguments`对象包含了函数运行时的所有参数，`arguments[0]`就是第一个参数，`arguments[1]`就是第二个参数，以此类推。这个对象**只有在函数体内部，才可以使用**。
```js
var f = function (one) {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
}

f(1, 2, 3)
// 1
// 2
// 3
```
正常模式下，arguments对象可以在运行时修改。
```js
var f = function(a, b) {
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 5
```
**严格模式下**，`arguments`对象与函数参数不具有联动关系。也就是说，**修改`arguments`对象不会影响到实际的函数参数**。
```js
var f = function(a, b) {
  'use strict'; // 开启严格模式
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 2
```
通过`arguments`对象的`length`属性，可以判断函数调用时到底带几个参数。
```js
function f() {
  return arguments.length;
}

f(1, 2, 3) // 3
f(1) // 1
f() // 0
```
##### 与数组的关系
需要注意的是，虽然`arguments`很像数组，但它是一个**对象**。数组专有的方法（比如`slice`和`forEach`），**不能在`arguments`对象上直接使用**。

如果要让`arguments`对象使用数组方法，真正的解决方法是将`arguments`转为真正的数组。下面是两种常用的转换方法：`slice`方法和逐一填入新数组。
```js
var args = Array.prototype.slice.call(arguments);

// 或者
var args = [];
for (var i = 0; i < arguments.length; i++) {
  args.push(arguments[i]);
}
```
##### callee 属性
`arguments`对象带有一个`callee`属性，返回它所对应的原函数。
```js
var f = function () {
  console.log(arguments.callee === f);
}

f() // true
```
可以通过`arguments.callee`，达到**调用函数自身的目的**。这个属性在严格模式里面是禁用的，因此**不建议使用**。
#### ...「允许传入多个参数」
```js
// ...允许传入多个参数
function f(...$data)
{
    console.log($data);
}
f(1, 3.14, 'a', true); // [ 1, 3.14, 'a', true ]
```
### 函数的其他知识点
#### 闭包
闭包（closure）是 JavaScript 语言的一个难点，也是它的特色，很多高级应用都要依靠闭包实现。
##### 读取外层函数内部的变量
正常情况下，函数外部无法读取函数内部声明的变量。
```js
function f1() {
  var n = 999;
}

console.log(n)
// Uncaught ReferenceError: n is not defined(
```
但是通过在函数的内部，再定义一个函数，可以获取到函数内部申明的变量 **「链式作用域：父对象的所有变量，对子对象都是可见的，反之则不成立。」**
**闭包就是函数`f2`**，即能够读取其他函数内部变量的函数。
```js
function f1() {
  var n = 999;
  function f2() {
    console.log(n);
  }
  return f2;
}

var result = f1();
result(); // 999
```
##### 让变量始终保持在内存中
即闭包可以使得它诞生环境一直存在
```js
function createIncrementor(start) {
  return function () {
    return start++;
  };
}

var inc = createIncrementor(5);

inc() // 5
inc() // 6
inc() // 7
```
为什么闭包能够返回外层函数的内部变量？原因是闭包（上例的`inc`）用到了外层变量（`start`），导致外层函数（`createIncrementor`）不能从内存释放。只要闭包没有被垃圾回收机制清除，外层函数提供的运行环境也不会被清除，它的内部变量就始终保存着当前值，供闭包读取。
##### 封装对象的私有属性、方法
```js
function Person(name) {
  var _age;
  function setAge(n) {
    _age = n;
  }
  function getAge() {
    return _age;
  }

  return {
    name: name,
    getAge: getAge,
    setAge: setAge
  };
}

var p1 = Person('张三');
p1.setAge(25);
p1.getAge() // 25
```
上面代码中，函数`Person`的内部变量`_age`，通过闭包`getAge`和`setAge`，变成了返回对象`p1`的私有变量。
**注意事项**
外层函数每次运行，都会生成一个新的闭包，而这个闭包又会保留外层函数的内部变量，所以内存消耗很大。**因此不能滥用闭包，否则会造成网页的性能问题**。
#### 立即调用的函数表达式（IIFE: Immediately-Invoked Function Expression）
根据 JavaScript 的语法，圆括号`()`跟在函数名之后，表示调用该函数。比如，`print()`就表示调用`print`函数。

有时，我们需要在定义函数之后，立即调用该函数。这时，你不能在函数的定义之后加上圆括号，这会产生语法错误。
```js
function(){ /* code */ }();
// SyntaxError: Unexpected token (
```

产生这个错误的原因是，`function`这个关键字既可以当作**语句**，也可以当作**表达式**。
```js
// 语句
function f() {}

// 表达式
var f = function f() {}
```

当作表达式时，函数可以定义后直接加圆括号调用。
```js
var f = function f(){ return 1}();
f // 1
```
上面的代码中，函数定义后直接加圆括号调用，没有报错。原因就是`function`作为表达式，引擎就把函数定义当作一个值。这种情况下，就不会报错。

为了避免解析的歧义，JavaScript 规定，**如果`function`关键字出现在行首，一律解释成语句**。因此，引擎看到行首是`function`关键字之后，认为这一段都是函数的定义，不应该以圆括号结尾，所以就报错了。

**函数定义后立即调用的解决方法**，就是不要让`function`出现在行首，让引擎将其理解成一个表达式。最简单的处理，就是将其放在一个**圆括号里面**。
```js
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```
上面两种写法都是以圆括号开头，引擎就会认为后面跟的是一个表达式，而不是函数定义语句，所以就避免了错误。
这就叫做**立即调用的函数表达式(Immediately-Invoked Function Expression)**，简称 **IIFE**。

注意，上面两种写法最后的**分号都是必须的**。如果省略分号，遇到连着两个 IIFE，可能就会报错。
```js
// 报错
(function(){ /* code */ }())
(function(){ /* code */ }())
```
上面代码的两行之间**没有分号**，JavaScript 会将它们**连在一起解释**，将第二行解释为第一行的参数。

推而广之，任何让解释器以表达式来处理函数定义的方法，都能产生同样的效果，比如下面三种写法。
```js
var i = function(){ return 10; }();
true && function(){ /* code */ }();
0, function(){ /* code */ }();
```

甚至像下面这样写，也是可以的。
```js
!function () { /* code */ }();
~function () { /* code */ }();
-function () { /* code */ }();
+function () { /* code */ }();
```

通常情况下，只对匿名函数使用这种“立即执行的函数表达式”。
它的目的有两个：
1. 不必为函数命名，避免了污染全局变量；
2. IIFE 内部形成了一个单独的作用域，可以封装一些外部无法读取的私有变量。
```js
// 写法一
var tmp = newData;
processData(tmp);
storeData(tmp);

// 写法二「推荐」
(function () {
  var tmp = newData;
  processData(tmp);
  storeData(tmp);
}());
```
上面代码中，写法二比写法一更好，因为完全避免了污染全局变量。
#### 函数调用的方式
##### 1. 普通调用
- **特点**：`this` 指向全局对象（非严格模式）或 `undefined`（严格模式）。
- **示例**：
```javascript
function f() { console.log(this); }
f(); // 全局对象（如 window）
```
##### 2. 方法调用
- **特点**：`this` 指向调用对象。
- **示例**：
```javascript
const obj = {
    name: "A", greet() {
        console.log(this.name);
    }
};
obj.greet(); // 输出 A
```
##### 3. 构造函数调用
- **特点**：`new` 调用，`this` 指向新对象。
- **示例**：
```javascript
function Person(name) { this.name = name; }
const p = new Person("B");
console.log(p.name); // B
```
##### 4. 显式绑定调用
- **方法**：`call` / `apply` / `bind`，强制绑定 `this`。
- **示例**：
通过 **`call/apply`**，可以**动态修改函数执行时的 this**，从而复用函数逻辑或操作不同对象的属性（如示例中访问 o.name）。
```javascript
const o = {name: "C"};

function f(a, b) {
    console.log(this.name, a, b);
}

// 直接传参
f.call(o, 1, 2);     // C 1 2
// 数组传参(不确定参数数量时使用)
f.apply(o, [1, 2]);   // C 1 2
const bound = f.bind(o, 1, 2);
bound();              // C 1 2
```
##### 5. 立即执行函数（IIFE）
- **特点**：定义后立即执行，创建私有作用域。
- **示例**：
```javascript
(function () { console.log("IIFE"); })(); // IIFE
```
##### 6. 箭头函数调用
- **特点**：无独立 `this`，继承外层作用
- **示例**：
```javascript
const obj = { name: "D", greet: () => console.log(this.name) };
obj.greet(); // 输出 undefined（外层 this.name 未定义）
```
##### 7. 回调函数调用
- **特点**：`this` 由调用方决定。
- **示例**：
```javascript
setTimeout(() => console.log("Callback"), 1000); // 1秒后输出 Callback
```
##### 核心结论
| 调用方式   | `this` 指向        | 典型场景          |
| ------ | ---------------- | ------------- |
| 普通调用   | 全局对象/`undefined` | 独立函数执行        |
| 方法调用   | 调用对象             | 对象方法调用        |
| 构造函数调用 | 新对象              | 创建实例          |
| 显式绑定   | 指定对象             | 动态上下文绑定       |
| IIFE   | 定义时上下文           | 私有作用域         |
| 箭头函数   | 外层作用域            | 保持 `this` 一致性 |
| 回调函数   | 由调用方决定           | 异步/事件处理       |
### eval命令
#### 基本用法
`eval`命令接受一个字符串作为参数，并将这个字符串当作语句执行。
```js
eval('var a = 1;');
a // 1
```
上面代码将字符串当作语句运行，生成了变量`a`。

如果参数字符串无法当作语句运行，那么就会报错。
```js
eval('3x') // Uncaught SyntaxError: Invalid or unexpected token
```

放在`eval`中的字符串，应该有独自存在的意义，不能用来与`eval`以外的命令配合使用。举例来说，下面的代码将会报错。
```js
eval('return;'); // Uncaught SyntaxError: Illegal return statement
```
上面代码会报错，因为`return`不能单独使用，必须在函数中使用。

如果`eval`的参数不是字符串，那么会原样返回。
```js
eval(123) // 123
```

`eval`**没有自己的作用域，都在当前作用域内执行**，因此可能会修改当前作用域的变量的值，造成安全问题。
```js
var a = 1;
eval('a = 2');

a // 2
```
上面代码中，`eval`命令修改了外部变量`a`的值。由于这个原因，`eval`有安全风险。

为了防止这种风险，JavaScript 规定，如果使用严格模式，`eval`内部声明的变量，不会影响到外部作用域。
```js
(function f() {
  'use strict';
  eval('var foo = 123');
  console.log(foo);  // ReferenceError: foo is not defined
})()
```
上面代码中，函数`f`内部是严格模式，这时`eval`内部声明的`foo`变量，就不会影响到外部。

不过，即使在严格模式下，**`eval`依然可以读写当前作用域的变量。**
```js
(function f() {
  'use strict';
  var foo = 1;
  eval('foo = 2');
  console.log(foo);  // 2
})()
```
上面代码中，严格模式下，`eval`内部还是改写了外部变量，可见安全风险依然存在。

总之，`eval`的本质是在当前作用域之中，注入代码。由于安全风险和不利于 JavaScript 引擎优化执行速度，**一般不推荐使用**。通常情况下，`eval`最常见的场合是解析 JSON 数据的字符串，不过正确的做法应该是使用原生的`JSON.parse`方法。
#### eval 的别名调用
前面说过`eval`不利于引擎优化执行速度。更麻烦的是，还有下面这种情况，引擎在静态代码分析的阶段，**根本无法分辨执行的是`eval`**。
```js
var m = eval;
m('var x = 1');
x // 1
```
上面代码中，变量`m`是`eval`的别名。静态代码分析阶段，引擎分辨不出`m('var x = 1')`执行的是`eval`命令。

为了保证`eval`的别名不影响代码优化，JavaScript 的标准规定，**凡是使用别名执行`eval`，`eval`内部一律是全局作用域**。
```js
var a = 1;

function f() {
  var a = 2;
  var e = eval;
  e('console.log(a)');
}

f() // 1
```
上面代码中，`eval`是别名调用，所以即使它是在函数中，它的作用域还是全局作用域，因此输出的`a`为全局变量。这样的话，引擎就能确认`e()`不会对当前的函数作用域产生影响，优化的时候就可以把这一行排除掉。

`eval`的别名调用的形式五花八门，只要不是直接调用，都属于别名调用，因为引擎只能分辨`eval()`这一种形式是直接调用。
```js
eval.call(null, '...')
window.eval('...')
(1, eval)('...')
(eval, eval)('...')
```
上面这些形式都是`eval`的别名调用，作用域都是全局作用域。
## 数组
### 定义
数组（array）是按次序排列的一组值。每个值的位置都有编号（从0开始），整个数组用方括号表示。

任何类型的数据，都可以放入数组。
```js
var arr = [
  {a: 1},
  [1, 2, 3],
  function() {return true;}
];

arr[0] // Object {a: 1}
arr[1] // [1, 2, 3]
arr[2] // function (){return true;}
```
### 数组的本质
本质上，数组属于一种特殊的对象。typeof运算符会返回数组的类型是object。
```js
typeof [1, 2, 3] // "object"
```

Object.keys方法返回数组的所有键名。可以看到数组的键名就是整数0、1、2。
```js
var arr = ['a', 'b', 'c'];

Object.keys(arr)
// ["0", "1", "2"]
```

JavaScript 语言规定，**对象的键名一律为字符串，所以，数组的键名其实也是字符串**。
之所以可以用数值读取，是因为**非字符串的键名**会被**转为字符串**。
```js
var arr = ['a', 'b', 'c'];

arr['0'] // 'a'
arr[0] // 'a'
```

注意，这点在赋值时也成立。一个值总是先转成字符串，再作为键名进行赋值。
由于`1.00`转成字符串是`1`，所以通过数字键`1`可以读取值。
```js
var a = [];

a[1.00] = 6;
a[1] // 6
```

读取成员的方法：只能用方括号读取
```js
var arr = [1, 2, 3];
arr.0 // SyntaxError
```
上面代码中，`arr.0`的写法不合法，因为单独的数值不能作为标识符（identifier）。所以，数组成员只能用方括号`arr[0]`表示（方括号是运算符，可以接受数值）。
### length 属性
数组的`length`属性，返回数组的成员数量。
```js
['a', 'b', 'c'].length // 3
```

JavaScript 使用一个32位整数，保存数组的元素个数。这意味着，`length`属性的最大值就是 **4294967295** 个（2^32 - 1）个。
只要是数组，就一定有`length`属性。**该属性是一个动态的值，等于键名中的最大整数加上`1`。**
```js
var arr = ['a', 'b'];
arr.length // 2

arr[2] = 'c';
arr.length // 3
```

`length`属性是可写的。如果人为设置一个小于当前成员个数的值，该数组的成员数量会自动减少到`length`设置的值。
```js
var arr = [ 'a', 'b', 'c' ];
arr.length // 3

arr.length = 2;
arr // ["a", "b"]
```

清空数组的一个有效方法，就是将length属性设为0。
```js
var arr = [ 'a', 'b', 'c' ];

arr.length = 0;
arr // []
```

如果人为设置length大于当前元素个数，则数组的成员数量会增加到这个值，新增的位置都是空位。
```js
var a = ['a'];

a.length = 3;
a[1] // undefined
```

如果人为设置length为不合法的值，JavaScript 会报错。
```js
// 设置负值
[].length = -1
// RangeError: Invalid array length

// 数组元素个数大于等于2的32次方
[].length = Math.pow(2, 32)
// RangeError: Invalid array length

// 设置字符串
[].length = 'abc'
// RangeError: Invalid array length
```

值得注意的是，由于数组本质上是一种对象，所以可以为数组添加属性，**但是这不影响`length`属性的值。**
```js
var a = [];

a['p'] = 'abc';
a.length // 0

a[2.1] = 'abc';
a.length // 0
```
上面代码将数组的键分别设为字符串和小数，结果都不影响`length`属性。因为，`length`属性的值就是等于最大的数字键加1，而这个数组没有整数键，所以`length`属性保持为`0`。

如果数组的键名是添加超出范围的数值，该键名会自动转为字符串。
```js
var arr = [];
arr[-1] = 'a';
arr[Math.pow(2, 32)] = 'b';

arr.length // 0
arr[-1] // "a"
arr[4294967296] // "b"
```
上面代码中，我们为数组`arr`添加了两个不合法的数字键，结果`length`属性没有发生变化。这些**数字键都变成了字符串键名**。最后两行之所以会取到值，是因为取键值时，数字键名会默认转为字符串。
### in 运算符
检查某个键名是否存在的运算符`in`，适用于对象，也适用于数组。
```js
var arr = [ 'a', 'b', 'c' ];
2 in arr  // true
'2' in arr // true
4 in arr // false
```
上面代码表明，数组存在键名为`2`的键。由于键名都是字符串，所以数值`2`会自动转成字符串。

注意，如果数组的某个位置是空位，`in`运算符返回`false`。
```js
var arr = [];
arr[100] = 'a';

100 in arr // true
1 in arr // false
```
上面代码中，数组`arr`只有一个成员`arr[100]`，其他位置的键名都会返回`false`。
### for...in 循环和数组的遍历
`for...in`循环不仅可以遍历对象，也可以遍历数组，毕竟数组只是一种特殊对象。
```js
var a = [1, 2, 3];

for (var i in a) {
  console.log(a[i]);
}
// 1
// 2
// 3
```
但是，`for...in`不仅会遍历数组所有的数字键，还会遍历非数字键。
```js
var a = [1, 2, 3];
a.foo = true;

for (var key in a) {
  console.log(key);
}
// 0
// 1
// 2
// foo
```
上面代码在遍历数组时，也遍历到了非整数键`foo`。所以，**不推荐使用`for...in`遍历数组。**

数组的遍历可以考虑使用`for`循环或`while`循环。
```js
var a = [1, 2, 3];

// for循环
for(var i = 0; i < a.length; i++) {
  console.log(a[i]);
}

// while循环
var i = 0;
while (i < a.length) {
  console.log(a[i]);
  i++;
}

var l = a.length;
while (l--) {
  console.log(a[l]);
}
```
上面代码是三种遍历数组的写法。最后一种写法是逆向遍历，即从最后一个元素向第一个元素遍历。

数组的`forEach`方法，也可以用来遍历数组，详见《标准库》的 Array 对象一章。
```js
var colors = ['red', 'green', 'blue'];
colors.forEach(function (color) {
  console.log(color);
});
// red
// green
// blue
```
### 数组的空位
当数组的某个位置是空元素，即两个逗号之间没有任何值，我们称该数组存在空位（hole）。
```js
var a = [1, , 1];
a.length // 3
```
上面代码表明，数组的空位不影响`length`属性。虽然这个位置没有值，引擎依然认为这个位置是有效的。

需要注意的是，如果最后一个元素后面有逗号，并不会产生空位。也就是说，有没有这个逗号，结果都是一样的。
```js
var a = [1, 2, 3,];

a.length // 3
a // [1, 2, 3]
```

数组的空位是可以读取的，返回`undefined`。
```js
var a = [, , ,];
a[1] // undefined
```

使用`delete`命令删除一个数组成员，会形成空位，并且不会影响`length`属性。
```js
var a = [1, 2, 3];
delete a[1];

a[1] // undefined
a.length // 3
```
`length`属性不过滤空位。所以，使用`length`属性进行数组遍历，一定要非常小心。

数组的某个位置是空位，与某个位置是`undefined`，是不一样的。
如果是空位，使用数组的`forEach`方法、`for...in`结构、以及`Object.keys`方法进行遍历，**空位都会被跳过**。
```js
var a = [, , ,];

a.forEach(function (x, i) {
  console.log(i + '. ' + x);
})
// 不产生任何输出

for (var i in a) {
  console.log(i);
}
// 不产生任何输出

Object.keys(a)
// []
```

如果某个位置是`undefined`，遍历的时候 **`undefined`不会被跳过**。
```js
var a = [undefined, undefined, undefined];

a.forEach(function (x, i) {
  console.log(i + '. ' + x);
});
// 0. undefined
// 1. undefined
// 2. undefined

for (var i in a) {
  console.log(i);
}
// 0
// 1
// 2

Object.keys(a)
// ['0', '1', '2']
```
### 类似数组的对象
如果一个对象的所有键名都是正整数或零，**并且有`length`属性**，那么这个对象就很像数组，语法上称为“**类似数组的对象**”（array-like object）。
```js
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};

obj[0] // 'a'
obj[1] // 'b'
obj.length // 3
obj.push('d') // TypeError: obj.push is not a function
```
上面代码中，对象`obj`就是一个类似数组的对象。但是，**“类似数组的对象”并不是数组**，因为它们不具备数组特有的方法。
**对象`obj`没有数组的`push`方法，使用该方法就会报错。**

“类似数组的对象”的根本特征，**就是具有`length`属性**。只要有`length`属性，就可以认为这个对象类似于数组。
但是有一个问题，**这种`length`属性不是动态值**，不会随着成员的变化而变化。
```js
var obj = {
  length: 0
};
obj[3] = 'd';
obj.length // 0
```
上面代码为对象`obj`添加了一个数字键，**但是`length`属性没变。这就说明了`obj`不是数组。**

典型的“类似数组的对象”是函数的`arguments`对象，以及大多数 DOM 元素集，还有字符串。
```js
// arguments对象
function args() { return arguments }
var arrayLike = args('a', 'b');

arrayLike[0] // 'a'
arrayLike.length // 2
arrayLike instanceof Array // false

// DOM元素集
var elts = document.getElementsByTagName('h3');
elts.length // 3
elts instanceof Array // false

// 字符串
'abc'[1] // 'b'
'abc'.length // 3
'abc' instanceof Array // false
```
上面代码包含三个例子，它们都不是数组（`instanceof`运算符返回`false`），但是看上去都非常像数组。

**数组的`slice`方法可以将“类似数组的对象”变成真正的数组。**
```js
var arr = Array.prototype.slice.call(arrayLike);
```

除了转为真正的数组，“类似数组的对象”还有一个办法可以使用数组的方法，**就是通过`call()`把数组的方法放到对象上面。**
```js
function print(value, index) {
  console.log(index + ' : ' + value);
}

Array.prototype.forEach.call(arrayLike, print);
```
上面代码中，`arrayLike`代表一个类似数组的对象，本来是不可以使用数组的`forEach()`方法的，但是通过`call()`，可以把`forEach()`嫁接到`arrayLike`上面调用。

下面的例子就是通过这种方法，在`arguments`对象上面调用`forEach`方法。
```js
// forEach 方法
function logArgs() {
  Array.prototype.forEach.call(arguments, function (elem, i) {
    console.log(i + '. ' + elem);
  });
}

// 等同于 for 循环
function logArgs() {
  for (var i = 0; i < arguments.length; i++) {
    console.log(i + '. ' + arguments[i]);
  }
}
```

字符串也是类似数组的对象，所以也可以用`Array.prototype.forEach.call`遍历。
```js
Array.prototype.forEach.call('abc', function (chr) {
  console.log(chr);
});
// a
// b
// c
```

注意，这种方法**比直接使用数组原生的`forEach`要慢**，所以**最好还是先将“类似数组的对象”转为真正的数组**，然后再直接调用数组的`forEach`方法。
```js
var arr = Array.prototype.slice.call('abc');
arr.forEach(function (chr) {
  console.log(chr);
});
// a
// b
// c
```
# 运算符
## 算数运算符
### 概述
JavaScript 共提供10个算术运算符，用来完成基本的算术运算。
- **加法运算符**：`x + y`
- **减法运算符**： `x - y`
- **乘法运算符**： `x * y`
- **除法运算符**：`x / y`
- **指数运算符**：`x ** y`
- **余数运算符**：`x % y`
- **自增运算符**：`++x` 或者 `x++`
- **自减运算符**：`--x` 或者 `x--`
- **数值运算符**： `+x`
- **负数值运算符**：`-x`
### 加法运算符
#### 基本规则
```js
// 两个数值的和
1 + 1 // 2

// 非数值的相加
true + true // 2
1 + true // 2

// 字符串相加
'a' + 'bc' // "abc"

// 数值和字符串相加
1 + 'a' // "1a"
false + 'a' // "falsea"

// 多个数值和多个字符串相加「由于从左到右的运算次序, 运算子的不同，导致了不同的语法行为，这种现象称为“重载”（overload），」
'3' + 4 + 5 // "345"
3 + 4 + '5' // "75"
```
除了加法运算符，其他算术运算符（比如减法、除法和乘法）都**不会发生重载**。它们的规则是：所有运算子**一律转为数值**，再进行相应的数学运算。
```js
1 - '2' // -1
1 * '2' // 2
1 / '2' // 0.5
```
#### 对象的相加
如果运算子是对象，必须先转成原始类型的值，然后再相加。
```js
var obj = { p: 1 };
obj + 2 // "[object Object]2"
```
上面代码中，对象`obj`转成原始类型的值是`[object Object]`，再加`2`就得到了上面的结果。

对象转成原始类型的值，规则如下。
首先，自动调用对象的`valueOf`方法。
```js
var obj = { p: 1 };
obj.valueOf() // { p: 1 }
```

一般来说，对象的`valueOf`方法总是返回对象自身，这时再自动调用对象的`toString`方法，将其转为字符串。
```js
var obj = { p: 1 };
obj.valueOf().toString() // "[object Object]"
```
对象的`toString`方法默认返回`[object Object]`，所以就得到了最前面那个例子的结果。

知道了这个规则以后，就可以自己定义`valueOf`方法或`toString`方法，得到想要的结果。
```js
var obj = {
  valueOf: function () {
    return 1;
  }
};

obj + 2 // 3
```
上面代码中，我们定义`obj`对象的`valueOf`方法返回`1`，于是`obj + 2`就得到了`3`。这个例子中，由于`valueOf`方法直接返回一个原始类型的值，所以不再调用`toString`方法。

下面是自定义`toString`方法的例子。
```js
var obj = {
  toString: function () {
    return 'hello';
  }
};

obj + 2 // "hello2"
```
上面代码中，对象`obj`的`toString`方法返回字符串`hello`。前面说过，只要有一个运算子是字符串，加法运算符就变成连接运算符，返回连接后的字符串。

**这里有一个特例，如果运算子是一个`Date`对象的实例**，那么会优先执行`toString`方法。
```js
var obj = new Date();
obj.valueOf = function () { return 1 };
obj.toString = function () { return 'hello' };

obj + 2 // "hello2"
```
上面代码中，对象`obj`是一个`Date`对象的实例，并且自定义了`valueOf`方法和`toString`方法，结果`toString`方法优先执行。
### 余数运算符
余数运算符（`%`）返回前一个运算子被后一个运算子除，所得的余数。
```js
12 % 5 // 2
```

需要注意的是，运算结果的正负号由第一个运算子的正负号决定。
```js
-1 % 2 // -1
1 % -2 // 1
```

所以，为了得到负数的正确余数值，可以先使用绝对值函数。
```js
// 错误的写法
function isOdd(n) {
  return n % 2 === 1;
}
isOdd(-5) // false
isOdd(-4) // false

// 正确的写法
function isOdd(n) {
  return Math.abs(n % 2) === 1;
}
isOdd(-5) // true
isOdd(-4) // false
```

余数运算符还可以用于浮点数的运算。但是，由于浮点数不是精确的值，无法得到完全准确的结果。
```js
6.5 % 2.1
// 0.19999999999999973
```
### 自增和自减运算符
自增和自减运算符，是一元运算符，只需要一个运算子。它们的作用是将运算子首先转为数值，然后加上1或者减去1。它们会修改原始变量。
```js
var x = 1;
++x // 2
x // 2

--x // 1
x // 1
```

运算符放在变量之后，会先返回变量操作前的值，再进行自增/自减操作
运算符放在变量之前，会先进行自增/自减操作，再返回变量操作后的值
```js
var x = 1;
var y = 1;

x++ // 1
++y // 2
```
### 数值运算符，负数值运算符
数值运算符（`+`）同样使用加号，但它是一元运算符（只需要一个操作数），而加法运算符是二元运算符（需要两个操作数）。
数值运算符的作用在于可以将**任何值转为数值**（与`Number`函数的作用相同）。
```js
+true // 1
+[] // 0
+{} // NaN
```

负数值运算符（`-`），也同样具有将一个值转为数值的功能，只不过得到的值正负相反。连用两个负数值运算符，等同于数值运算符。
```js
var x = 1;
-x // -1
-(-x) // 1
```
上面代码最后一行的**圆括号不可少**，否则会变成自减运算符。
数值运算符号和负数值运算符，**都会返回一个新的值，而不会改变原始变量的值。**
### 指数运算符
指数运算符（`**`）完成指数运算，前一个运算子是底数，后一个运算子是指数。
```js
2 ** 4 // 16
```
注意，指数运算符是右结合，而不是左结合。即多个指数运算符连用时，**先进行最右边的计算**。
```js
// 相当于 2 ** (3 ** 2)
2 ** 3 ** 2
// 512
```
### 赋值运算符
赋值运算符（Assignment Operators）用于给变量赋值。

最常见的赋值运算符，当然就是等号（`=`）。
```js
// 将 1 赋值给变量 x
var x = 1;

// 将变量 y 的值赋值给变量 x
var x = y;
```

赋值运算符还可以与其他运算符结合，形成变体。下面是与算术运算符的结合。
```js
// 等同于 x = x + y
x += y

// 等同于 x = x - y
x -= y

// 等同于 x = x * y
x *= y

// 等同于 x = x / y
x /= y

// 等同于 x = x % y
x %= y

// 等同于 x = x ** y
x **= y
```

下面是与位运算符的结合（关于位运算符，请见后文的介绍）。
```js
// 等同于 x = x >> y
x >>= y

// 等同于 x = x << y
x <<= y

// 等同于 x = x >>> y
x >>>= y

// 等同于 x = x & y
x &= y

// 等同于 x = x | y
x |= y

// 等同于 x = x ^ y
x ^= y
```
这些复合的赋值运算符，都是先进行指定运算，然后将得到值返回给左边的变量。
## 比较运算符
### 概述
比较运算符用于比较两个值的大小，然后返回一个布尔值，表示是否满足指定的条件。
```js
2 > 1 // true
```
JavaScript 一共提供了8个比较运算符。
- `>` 大于运算符
- `<` 小于运算符
- `<=` 小于或等于运算符
- `>=` 大于或等于运算符
- `==` 相等运算符
- `===` 严格相等运算符
- `!=` 不相等运算符
- `!==` 严格不相等运算符
### 非相等运算符：字符串的比较
字符串按照字典顺序进行比较。
```js
'cat' > 'dog' // false
'cat' > 'catalog' // false
```

JavaScript 引擎内部首先比较首字符的 Unicode 码点。如果相等，再比较第二个字符的 Unicode 码点，以此类推。
```js
'cat' > 'Cat' // true'
```
上面代码中，小写的`c`的 Unicode 码点（`99`）大于大写的`C`的 Unicode 码点（`67`），所以返回`true`。

由于所有字符都有 Unicode 码点，因此汉字也可以比较。
```js
'大' > '小' // false
```

上面代码中，“大”的 Unicode 码点是22823，“小”是23567，因此返回`false`。
### 非相等运算符：非字符串的比较
如果两个运算子之中，至少有一个不是字符串，需要分成以下两种情况。
**（1）原始类型值**
如果两个运算子都是原始类型的值，则是先转成数值再比较。
```js
5 > '4' // true
// 等同于 5 > Number('4')
// 即 5 > 4

true > false // true
// 等同于 Number(true) > Number(false)
// 即 1 > 0

2 > true // true
// 等同于 2 > Number(true)
// 即 2 > 1
```
上面代码中，字符串和布尔值都会先转成数值，再进行比较。

这里需要注意与`NaN`的比较。任何值（包括`NaN`本身）与`NaN`使用非相等运算符进行比较，返回的都是`false`。
```js
1 > NaN // false
1 <= NaN // false
'1' > NaN // false
'1' <= NaN // false
NaN > NaN // false
NaN <= NaN // false
```

**（2）对象**
如果运算子是对象，会转为原始类型的值，再进行比较。
对象转换成原始类型的值，算法是先调用`valueOf`方法；如果返回的还是对象，再接着调用`toString`方法，详细解释参见《数据类型的转换》一章。
```js
var x = [2];
x > '11' // true
// 等同于 [2].valueOf().toString() > '11'
// 即 '2' > '11'

x.valueOf = function () { return '1' };
x > '11' // false
// 等同于 (function () { return '1' })() > '11'
// 即 '1' > '11'
```

两个对象之间的比较也是如此。
```js
[2] > [1] // true
// 等同于 [2].valueOf().toString() > [1].valueOf().toString()
// 即 '2' > '1'

[2] > [11] // true
// 等同于 [2].valueOf().toString() > [11].valueOf().toString()
// 即 '2' > '11'

({ x: 2 }) >= ({ x: 1 }) // true
// 等同于 ({ x: 2 }).valueOf().toString() >= ({ x: 1 }).valueOf().toString()
// 即 '[object Object]' >= '[object Object]'
```
### 严格相等运算符
#### 不同类型的值
如果两个值的类型不同，直接返回`false`。
```js
1 === "1" // false
true === "true" // false
```
#### 同一类的原始类型值
比较十进制的`1`与十六进制的`1`，因为类型和值都相同，返回`true`。
```js
1 === 0x1 // true
```
需要注意的是，`NaN`与任何值都不相等（包括自身）。另外，正`0`等于负`0`。
```js
NaN === NaN  // false
+0 === -0 // true
```
#### 复合类型值
两个复合类型（对象、数组、函数）的数据比较时，**不是比较它们的值是否相等，而是比较它们是否指向同一个地址**。
```js
{} === {} // false
[] === [] // false
(function () {} === function () {}) // false
```
如果两个变量引用同一个对象，则它们相等。
```js
var v1 = {};
var v2 = v1;
v1 === v2 // true
```
**注意**: 对于两个对象的比较，严格相等运算符比较的是地址，**而大于或小于运算符比较的是值**。
```js
var obj1 = {};
var obj2 = {};

obj1 > obj2 // false
obj1 < obj2 // false
obj1 === obj2 // false
```
#### undefined 和 null
`undefined`和`null`与自身严格相等。
```js
undefined === undefined // true
null === null // true
```
由于变量声明后默认值是`undefined`，因此两个只声明未赋值的变量是相等的。
```js
var v1;
var v2;
v1 === v2 // true
```
### 严格不相等运算符
严格相等运算符有一个对应的“严格不相等运算符”（`!==`），它的算法就是先求严格相等运算符的结果，然后返回相反值。
```js
1 !== '1' // true
// 等同于
!(1 === '1')
```
### 相等运算符
相等运算符用来比较相同类型的数据时，与严格相等运算符完全一样。
```js
1 == 1.0
// 等同于
1 === 1.0
```
比较不同类型的数据时，相等运算符会先将**数据进行类型转换**，然后再用严格相等运算符比较。下面分成几种情况，讨论不同类型的值互相比较的规则。
#### 原始类型值
原始类型的值会转换成数值再进行比较。
```js
1 == true // true
// 等同于 1 === Number(true)

0 == false // true
// 等同于 0 === Number(false)

2 == true // false
// 等同于 2 === Number(true)

2 == false // false
// 等同于 2 === Number(false)

'true' == true // false
// 等同于 Number('true') === Number(true)
// 等同于 NaN === 1

'' == 0 // true
// 等同于 Number('') === 0
// 等同于 0 === 0

'' == false  // true
// 等同于 Number('') === Number(false)
// 等同于 0 === 0

'1' == true  // true
// 等同于 Number('1') === Number(true)
// 等同于 1 === 1

'\n  123  \t' == 123 // true
// 因为字符串转为数字时，省略前置和后置的空格
```
上面代码将字符串和布尔值都转为数值，然后再进行比较。具体的字符串与布尔值的类型转换规则，参见《数据类型转换》一章。
#### 对象与原始类型值比较
对象（这里指广义的对象，包括数组和函数）与原始类型的值比较时，对象转换成原始类型的值，再进行比较。

具体来说，先调用对象的`valueOf()`方法，如果得到原始类型的值，就按照上一小节的规则，互相比较；如果得到的还是对象，则再调用`toString()`方法，得到字符串形式，再进行比较。

下面是数组与原始类型值比较的例子。
```js
// 数组与数值的比较
[1] == 1 // true

// 数组与字符串的比较
[1] == '1' // true
[1, 2] == '1,2' // true

// 对象与布尔值的比较
[1] == true // true
[2] == true // false
```

下面是一个更直接的例子。
```js
const obj = {
  valueOf: function () {
    console.log('执行 valueOf()');
    return obj;
  },
  toString: function () {
    console.log('执行 toString()');
    return 'foo';
  }
};

obj == 'foo'
// 执行 valueOf()
// 执行 toString()
// true
```
#### undefined 和 null
`undefined`和`null`只有与自身比较，或者互相比较时，才会返回`true`；与其他类型的值比较时，结果都为`false`。
```js
undefined == undefined // true
null == null // true
undefined == null // true

false == null // false
false == undefined // false

0 == null // false
0 == undefined // false
```
#### 相等运算符的缺点
相等运算符隐藏的类型转换，会带来一些违反直觉的结果。
```js
0 == ''             // true
0 == '0'            // true

2 == true           // false
2 == false          // false

false == 'false'    // false
false == '0'        // true

false == undefined  // false
false == null       // false
null == undefined   // true

' \t\r\n ' == 0     // true
```
上面这些表达式都不同于直觉，很容易出错。因此建议不要使用相等运算符（`==`），**最好只使用严格相等运算符（`===`）**。
### 不相等运算符
相等运算符有一个对应的“不相等运算符”（`!=`），它的算法就是先求相等运算符的结果，然后返回相反值。
```js
1 != '1' // false

// 等同于
!(1 == '1')
```
## 布尔运算符
布尔运算符用于将表达式转为布尔值，一共包含四个运算符。
- 取反运算符：`!`
- 且运算符：`&&`
- 或运算符：`||`
- 三元运算符：`?:`
## 二进制运算符
[二进制位运算符 - JavaScript 教程 - 网道](https://wangdoc.com/javascript/operators/bit)
### 概述
二进制位运算符用于直接对二进制位进行计算，一共有7个。
- **二进制或运算符**（or）：符号为`|`，表示若两个二进制位都为`0`，则结果为`0`，否则为`1`。
- **二进制与运算符**（and）：符号为`&`，表示若两个二进制位都为1，则结果为1，否则为0。
- **二进制否运算符**（not）：符号为`~`，表示对一个二进制位取反。
- **异或运算符**（xor）：符号为`^`，表示若两个二进制位不相同，则结果为1，否则为0。
- **左移运算符**（left shift）：符号为`<<`，详见下文解释。
- **右移运算符**（right shift）：符号为`>>`，详见下文解释。
- **头部补零的右移运算符**（zero filled right shift）：符号为`>>>`，详见下文解释。
## 其他运算符，运算顺序
### void 运算符
`void`运算符的作用是执行一个表达式，然后不返回任何值，或者说返回`undefined`。
```js
void 0 // undefined
void(0) // undefined
```
上面是`void`运算符的两种写法，都正确。
**建议采用后一种形式**，即总是使用圆括号。
因为`void`运算符的优先性很高，如果不使用括号，容易造成错误的结果。
比如，`void 4 + 7`实际上等同于`(void 4) + 7`。

下面是`void`运算符的一个例子。
```js
var x = 3;
void (x = 5) //undefined
x // 5
```
这个运算符的主要用途是**浏览器的书签工具（Bookmarklet）**，以及在超级链接中插入代码防止网页跳转。

请看下面的代码。
```js
<script>
function f() {
  console.log('Hello World');
}
</script>
<a href="http://example.com" onclick="f(); return false;">点击</a>
```
上面代码中，点击链接后，会先执行`onclick`的代码，由于`onclick`返回`false`，所以浏览器不会跳转到 example.com。
`void`运算符可以取代上面的写法。
```js
<a href="javascript: void(f())">文字</a>
```

下面是一个更实际的例子，用户点击链接提交表单，但是不产生页面跳转。
```js
<a href="javascript: void(document.form.submit())">
  提交
</a>
```
### 逗号运算符
逗号运算符用于对两个表达式求值，并**返回后一个表达式的值**。
```js
'a', 'b' // "b"

var x = 0;
var y = (x++, 10);
x // 1
y // 10
```
上面代码中，逗号运算符返回后一个表达式的值。
逗号运算符的一个用途是，在返回一个值之前，进行一些辅助操作。
```js
var value = (console.log('Hi!'), true);
// Hi!

value // true
```
上面代码中，先执行逗号之前的操作，然后返回逗号后面的值。
### 运算顺序
#### 优先级
JavaScript 各种运算符的优先级别（Operator Precedence）是不一样的。优先级高的运算符先执行，优先级低的运算符后执行。
```js
4 + 5 * 6 // 34
```
上面的代码中，乘法运算符（`*`）的优先性高于加法运算符（`+`），所以先执行乘法，再执行加法，相当于下面这样。
```js
4 + (5 * 6) // 34
```
如果多个运算符混写在一起，常常会导致令人困惑的代码。
```js
var x = 1;
var arr = [];

var y = arr.length <= 0 || arr[0] === undefined ? x : arr[0];
```
上面代码中，变量`y`的值就很难看出来，因为这个表达式涉及5个运算符，到底谁的优先级最高，实在不容易记住。
根据语言规格，这五个运算符的优先级从高到低依次为：小于等于（`<=`)、严格相等（`===`）、或（`||`）、三元（`?:`）、等号（`=`）。因此上面的表达式，实际的运算顺序如下。
```js
var y = ((arr.length <= 0) || (arr[0] === undefined)) ? x : arr[0];
```
**记住所有运算符的优先级，是非常难的，也是没有必要的。**

#### 圆括号的作用
圆括号（`()`）可以用**来提高运算的优先级**，因为它的优先级是最高的，即圆括号中的表达式会第一个运算。
```js
(4 + 5) * 6 // 54
```
上面代码中，由于使用了圆括号，加法会先于乘法执行。
运算符的优先级别十分繁杂，且都是硬性规定，因此建议总是使用圆括号，保证运算顺序清晰可读，这对代码的维护和除错至关重要。

**圆括号不是运算符，而是一种语法结构**。
它一共有两种用法：
1. 一种是把表达式放在圆括号之中，提升运算的优先级；
2. 一种是跟在函数的后面，作用是调用函数。
注意，因为圆括号不是运算符，所以不具有求值作用，只改变运算的优先级。
```js
var x = 1;
(x) = 2;
```
上面代码的第二行，如果圆括号具有求值作用，那么就会变成`1 = 2`，这是会报错了。但是，上面的代码可以运行，这验证了圆括号只改变优先级，不会求值。
这也意味着，如果整个表达式都放在圆括号之中，那么不会有任何效果。
```js
(expression)
// 等同于
expression
```

函数放在圆括号中，会返回函数本身。如果圆括号紧跟在函数的后面，就表示调用函数。
```js
function f() {
  return 1;
}

(f) // function f(){return 1;}
f() // 1
```

圆括号之中，只能放置表达式，如果将语句放在圆括号之中，就会报错。
```js
(var a = 1)
// SyntaxError: Unexpected token var
```
#### 左结合与右结合
对于优先级别相同的运算符，同时出现的时候，就会有计算顺序的问题。
```js
a OP b OP c
```
上面代码中，`OP`表示运算符。它可以有两种解释方式。
```js
// 方式一
(a OP b) OP c

// 方式二
a OP (b OP c)
```
上面的两种方式，得到的计算结果往往是不一样的。
方式一是: 将左侧两个运算数结合在一起，采用这种解释方式的运算符，称为“左结合”（left-to-right associativity）运算符；
方式二是: 将右侧两个运算数结合在一起，这样的运算符称为“右结合”运算符（right-to-left associativity）。

JavaScript 语言的大多数运算符是“左结合”，请看下面加法运算符的例子。
```js
x + y + z

// 引擎解释如下
(x + y) + z
```
上面代码中，`x`与`y`结合在一起，它们的预算结果再与`z`进行运算。

少数运算符是“右结合”，其中最主要的是赋值运算符（`=`）和三元条件运算符（`?:`）。
```js
w = x = y = z;
q = a ? b : c ? d : e ? f : g;
```
上面代码的解释方式如下。
```js
w = (x = (y = z));
q = a ? b : (c ? d : (e ? f : g));
```
上面的两行代码，都是右侧的运算数结合在一起。

另外，指数运算符（`**`）也是右结合。
```js
2 ** 3 ** 2
// 相当于 2 ** (3 ** 2)
// 512
```
# 语法专题
## 数据类型的转换
### 概述
JavaScript 是一种动态类型语言，变量没有类型限制，可以随时赋予任意值。
```js
var x = y ? 1 : 'a';
```

虽然是两个字符串相减，但是依然会得到结果数值`1`，原因就在于 JavaScript 将运算子自动转为了数值。
```js
'4' - '3' // 1
```
### 强制转换
强制转换主要指使用`Number()`、`String()`和`Boolean()`三个函数，手动将各种类型的值，分别转换成数字、字符串或者布尔值。
#### Number()
使用`Number`函数，可以将任意类型的值转化成数值。

下面分成两种情况讨论，一种是参数是原始类型的值，另一种是参数是对象。
##### 原始类型值
原始类型值的转换规则如下。
```js
// 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('324') // 324

// 字符串：如果不可以被解析为数值，返回 NaN
Number('324abc') // NaN

// 空字符串转为0
Number('') // 0

// 布尔值：true 转成 1，false 转成 0
Number(true) // 1
Number(false) // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成0
Number(null) // 0
```

`Number`函数将字符串转为数值，要比`parseInt`函数严格很多。基本上，**只要有一个字符无法转成数值，整个字符串就会被转为`NaN`。**
```js
parseInt('42 cats') // 42
Number('42 cats') // NaN
```
上面代码中，`parseInt`逐个解析字符，而`Number`函数整体转换字符串的类型。

另外，`parseInt`和`Number`函数**都会自动过滤一个字符串前导和后缀的空格**。
```js
parseInt('\t\v\r12.34\n') // 12
Number('\t\v\r12.34\n') // 12.34
```
##### 对象
简单的规则是，`Number`方法的参数是对象时，将返回`NaN`，除非是包含单个数值的数组。
```js
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([5]) // 5
```

之所以会这样，是因为`Number`背后的转换规则比较复杂。
1. 调用对象自身的`valueOf`方法。如果返回原始类型的值，则直接对该值使用`Number`函数，不再进行后续步骤。
2. 如果`valueOf`方法返回的还是对象，则改为调用对象自身的`toString`方法。如果`toString`方法返回原始类型的值，则对该值使用`Number`函数，不再进行后续步骤。
3. 如果`toString`方法返回的是对象，就报错。

请看下面的例子。
```js
var obj = {x: 1};
Number(obj) // NaN

// 等同于
if (typeof obj.valueOf() === 'object') {
  Number(obj.toString());
} else {
  Number(obj.valueOf());
}
```
上面代码中，`Number`函数将`obj`对象转为数值。背后发生了一连串的操作，首先调用`obj.valueOf`方法, 结果返回对象本身；于是，继续调用`obj.toString`方法，这时返回字符串`[object Object]`，对这个字符串使用`Number`函数，得到`NaN`。

默认情况下，对象的`valueOf`方法返回对象本身，所以一般总是会调用`toString`方法，而`toString`方法返回对象的类型字符串（比如`[object Object]`）。所以，会有下面的结果。
```js
Number({}) // NaN
```

如果`toString`方法返回的不是原始类型的值，结果就会报错。
```js
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

Number(obj)
// TypeError: Cannot convert object to primitive value
```
上面代码的`valueOf`和`toString`方法，返回的都是对象，所以转成数值时会报错。

从上例还可以看到，`valueOf`和`toString`方法，都是可以自定义的。
```js
Number({
  valueOf: function () {
    return 2;
  }
})
// 2

Number({
  toString: function () {
    return 3;
  }
})
// 3

Number({
  valueOf: function () {
    return 2;
  },
  toString: function () {
    return 3;
  }
})
// 2
```
上面代码对三个对象使用`Number`函数。第一个对象返回`valueOf`方法的值，第二个对象返回`toString`方法的值，第三个对象表示`valueOf`方法先于`toString`方法执行。
#### String()
`String`函数可以将任意类型的值转化成字符串，转换规则如下。
##### 原始类型值
- **数值**：转为相应的字符串。
- **字符串**：转换后还是原来的值。
- **布尔值**：`true`转为字符串`"true"`，`false`转为字符串`"false"`。
- **undefined**：转为字符串`"undefined"`。
- **null**：转为字符串`"null"`。
```js
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
```
##### 对象
`String`方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。
```js
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```
`String`方法背后的转换规则，与`Number`方法基本相同，只是**互换了`valueOf`方法和`toString`方法的执行顺序**。
1. 先调用对象自身的`toString`方法。如果返回原始类型的值，则对该值使用`String`函数，不再进行以下步骤。
2. 如果`toString`方法返回的是对象，再调用原对象的`valueOf`方法。如果`valueOf`方法返回原始类型的值，则对该值使用`String`函数，不再进行以下步骤。
3. 如果`valueOf`方法返回的是对象，就报错。

下面是一个例子。
```js
String({a: 1})
// "[object Object]"

// 等同于
String({a: 1}.toString())
// "[object Object]"
```

上面代码先调用对象的`toString`方法，发现返回的是字符串`[object Object]`，就不再调用`valueOf`方法了。

如果`toString`法和`valueOf`方法，返回的都是对象，就会报错。
```js
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

String(obj)
// TypeError: Cannot convert object to primitive value
```

下面是通过自定义`toString`方法，改变返回值的例子。
```js
String({
  toString: function () {
    return 3;
  }
})
// "3"

String({
  valueOf: function () {
    return 2;
  }
})
// "[object Object]"

String({
  valueOf: function () {
    return 2;
  },
  toString: function () {
    return 3;
  }
})
// "3"
```
上面代码对三个对象使用`String`函数。第一个对象返回`toString`方法的值（数值3），第二个对象返回的还是`toString`方法的值（`[object Object]`），第三个对象表示`toString`方法先于`valueOf`方法执行。
#### Boolean()
`Boolean()`函数可以将任意类型的值转为布尔值。

它的转换规则相对简单：除了以下五个值的转换结果为`false`，其他的值全部为`true`。
- `undefined`
- `null`
- `0`（包含`-0`和`+0`）
- `NaN`
- `''`（空字符串）
```js
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false
```

当然，`true`和`false`这两个布尔值不会发生变化。
```js
Boolean(true) // true
Boolean(false) // false
```

注意，所有对象（包括空对象）的转换结果都是`true`，甚至连`false`对应的布尔对象`new Boolean(false)`也是`true`（详见《原始类型值的包装对象》一章）。
```js
Boolean({}) // true
Boolean([]) // true
Boolean(new Boolean(false)) // true
```
所有对象的布尔值都是`true`，这是因为 JavaScript 语言设计的时候，出于性能的考虑，如果对象需要计算才能得到布尔值，对于`obj1 && obj2`这样的场景，可能会需要较多的计算。为了保证性能，就统一规定，对象的布尔值为`true`。
### 自动转换
下面介绍自动转换，它是以强制转换为基础的。

遇到以下三种情况时，JavaScript 会自动转换数据类型，即转换是自动完成的，用户不可见。

第一种情况，不同类型的数据互相运算。
```js
123 + 'abc' // "123abc"
```

第二种情况，对非布尔值类型的数据求布尔值。
```js
if ('abc') {
  console.log('hello')
}  // "hello"
```

第三种情况，对非数值类型的值使用一元运算符（即`+`和`-`）。
```js
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```
自动转换的规则是这样的：
预期什么类型的值，就调用该类型的转换函数。
比如，某个位置预期为字符串，就调用`String()`函数进行转换。如果该位置既可以是字符串，也可能是数值，那么默认转为数值。

由于自动转换具有不确定性，而且不易除错，建议在预期为布尔值、数值、字符串的地方，全部使用`Boolean()`、`Number()`和`String()`函数进行显式转换。
#### 自动转换为布尔值
JavaScript 遇到预期为布尔值的地方（比如`if`语句的条件部分），就会将非布尔值的参数自动转换为布尔值。系统内部会自动调用`Boolean()`函数。

因此除了以下五个值，其他都是自动转为`true`。
- `undefined`
- `null`
- `+0`或`-0`
- `NaN`
- `''`（空字符串）

下面这个例子中，条件部分的每个值都相当于`false`，使用否定运算符后，就变成了`true`。
```js
if ( !undefined
  && !null
  && !0
  && !NaN
  && !''
) {
  console.log('true');
} // true
```

下面两种写法，有时也用于将一个表达式转为布尔值。它们内部调用的也是`Boolean()`函数。
```js
// 写法一
expression ? true : false

// 写法二
!! expression
```
#### 自动转换为字符串
JavaScript 遇到预期为字符串的地方，就会将非字符串的值自动转为字符串。具体规则是，先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串。

字符串的自动转换，主要发生在字符串的加法运算时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。
```js
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

这种自动转换很容易出错。
```js
var obj = {
  width: '100'
};

obj.width + 20 // "10020"
```
上面代码中，开发者可能期望返回`120`，但是由于自动转换，实际上返回了一个字符`10020`。
#### 自动转换为数值
JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值。系统内部会自动调用`Number()`函数。

除了加法运算符（`+`）有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。
```js
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1
undefined + 1 // NaN
```
上面代码中，运算符两侧的运算子，都被转成了数值。
> 注意：`null`转为数值时为`0`，而`undefined`转为数值时为`NaN`。

一元运算符也会把运算子转成数值。
```js
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```
## 错误处理的机制
### Error 实例对象
JavaScript 解析或运行时，一旦发生错误，引擎就会抛出一个错误对象。JavaScript 原生提供`Error`构造函数，**所有抛出的错误都是这个构造函数的实例**。
```js
var err = new Error('出错了');
err.message // "出错了"
```

上面代码中，我们调用`Error()`构造函数，生成一个实例对象`err`。`Error()`构造函数接受一个参数，表示错误提示，可以从实例的`message`属性读到这个参数。抛出`Error`实例对象以后，整个程序就中断在发生错误的地方，不再往下执行。

JavaScript 语言标准只提到，`Error`实例对象必须有`message`属性，表示出错时的提示信息，没有提到其他属性。大多数 JavaScript 引擎，对`Error`实例还提供`name`和`stack`属性，分别表示错误的名称和错误的堆栈，但它们是非标准的，不是每种实现都有。
- **message**：错误提示信息
- **name**：错误名称（非标准属性）
- **stack**：错误的堆栈（非标准属性）

使用`name`和`message`这两个属性，可以对发生什么错误有一个大概的了解。
```js
if (error.name) {
  console.log(error.name + ': ' + error.message);
}
```

`stack`属性用来查看错误发生时的堆栈。
```js
function throwit() {
  throw new Error('');
}

function catchit() {
  try {
    throwit();
  } catch(e) {
    console.log(e.stack); // print stack trace
  }
}

catchit()
// Error
//    at throwit (~/examples/throwcatch.js:9:11)
//    at catchit (~/examples/throwcatch.js:3:9)
//    at repl:1:5
```

上面代码中，错误堆栈的最内层是`throwit`函数，然后是`catchit`函数，最后是函数的运行环境。
### 原生错误类型
`Error`实例对象是最一般的错误类型，在它的基础上，JavaScript 还定义了其他6种错误对象。也就是说，存在`Error`的6个派生对象。
#### SyntaxError 对象
`SyntaxError`对象是解析代码时发生的语法错误。
```js
// 变量名错误
var 1a;
// Uncaught SyntaxError: Invalid or unexpected token

// 缺少括号
console.log 'hello');
// Uncaught SyntaxError: Unexpected string
```
上面代码的错误，都是在语法解析阶段就可以发现，所以会抛出`SyntaxError`。第一个错误提示是“token 非法”，第二个错误提示是“字符串不符合要求”。
#### ReferenceError 对象
`ReferenceError`对象是引用一个不存在的变量时发生的错误。
```js
// 使用一个不存在的变量
unknownVariable
// Uncaught ReferenceError: unknownVariable is not defined
```

另一种触发场景是，将一个值分配给无法分配的对象，比如对函数的运行结果赋值。
```js
// 等号左侧不是变量
console.log() = 1
// Uncaught ReferenceError: Invalid left-hand side in assignment
```
上面代码对函数`console.log`的运行结果赋值，结果引发了`ReferenceError`错误。
#### RangeError 对象
`RangeError`对象是一个值超出有效范围时发生的错误。主要有几种情况，一是数组长度为负数，二是`Number`对象的方法参数超出范围，以及函数堆栈超过最大值。
```js
// 数组长度不得为负数
new Array(-1)
// Uncaught RangeError: Invalid array length
```
#### TypeError 对象
`TypeError`对象是变量或参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用`new`命令，就会抛出这种错误，因为`new`命令的参数应该是一个构造函数。
```js
new 123
// Uncaught TypeError: 123 is not a constructor

var obj = {};
obj.unknownMethod()
// Uncaught TypeError: obj.unknownMethod is not a function
```
上面代码的第二种情况，调用对象不存在的方法，也会抛出`TypeError`错误，因为`obj.unknownMethod`的值是`undefined`，而不是一个函数。
#### URIError 对象
`URIError`对象是 URI 相关函数的参数不正确时抛出的错误，主要涉及`encodeURI()`、`decodeURI()`、`encodeURIComponent()`、`decodeURIComponent()`、`escape()`和`unescape()`这六个函数。
```js
decodeURI('%2')
// URIError: URI malformed
```
#### EvalError 对象
`eval`函数没有被正确执行时，会抛出`EvalError`错误。该错误类型已经不再使用了，只是为了保证与以前代码兼容，才继续保留。
#### 总结
以上这6种派生错误，连同原始的`Error`对象，都是构造函数。开发者可以使用它们，手动生成错误对象的实例。这些构造函数都接受一个参数，代表错误提示信息（message）。
```js
var err1 = new Error('出错了！');
var err2 = new RangeError('出错了，变量超出有效范围！');
var err3 = new TypeError('出错了，变量类型无效！');

err1.message // "出错了！"
err2.message // "出错了，变量超出有效范围！"
err3.message // "出错了，变量类型无效！"
```
### 自定义错误
除了 JavaScript 原生提供的七种错误对象，还可以定义自己的错误对象。
```js
function UserError(message) {
  this.message = message || '默认信息';
  this.name = 'UserError';
}

UserError.prototype = new Error();
UserError.prototype.constructor = UserError;
```

上面代码自定义一个错误对象`UserError`，让它继承`Error`对象。然后，就可以生成这种自定义类型的错误了。
```js
new UserError('这是自定义的错误！');
```
### throw 语句
`throw`语句的作用是手动中断程序执行，抛出一个错误。
```js
var x = -1;

if (x <= 0) {
  throw new Error('x 必须为正数');
}
// Uncaught Error: x 必须为正数
```
上面代码中，如果变量`x`小于等于`0`，就手动抛出一个错误，告诉用户`x`的值不正确，整个程序就会在这里中断执行。可以看到，`throw`抛出的错误就是它的参数，这里是一个`Error`对象的实例。

`throw`也可以抛出自定义错误。
```js
function UserError(message) {
  this.message = message || '默认信息';
  this.name = 'UserError';
}

throw new UserError('出错了！');
// Uncaught UserError {message: "出错了！", name: "UserError"}
```
上面代码中，`throw`抛出的是一个`UserError`实例。

实际上，`throw`可以抛出任何类型的值。也就是说，它的参数可以是任何值。
```js
// 抛出一个字符串
throw 'Error！';
// Uncaught Error！

// 抛出一个数值
throw 42;
// Uncaught 42

// 抛出一个布尔值
throw true;
// Uncaught true

// 抛出一个对象
throw {
  toString: function () {
    return 'Error!';
  }
};
// Uncaught {toString: ƒ}
```
对于 JavaScript 引擎来说，遇到`throw`语句，程序就中止了。引擎会接收到`throw`抛出的信息，可能是一个错误实例，也可能是其他类型的值。
### try...catch 结构
一旦发生错误，程序就中止执行了。JavaScript 提供了`try...catch`结构，允许对错误进行处理，选择是否往下执行。
```js
try {
  throw new Error('出错了!');
} catch (e) {
  console.log(e.name + ": " + e.message);
  console.log(e.stack);
}
// Error: 出错了!
//   at <anonymous>:3:9
//   ...
```
上面代码中，`try`代码块抛出错误（上例用的是`throw`语句），JavaScript 引擎就立即把代码的执行，转到`catch`代码块，或者说错误被`catch`代码块捕获了。`catch`接受一个参数，表示`try`代码块抛出的值。

如果你不确定某些代码是否会报错，就可以把它们放在`try...catch`代码块之中，便于进一步对错误进行处理。
```js
try {
  f();
} catch(e) {
  // 处理错误
}
```
上面代码中，如果函数`f`执行报错，就会进行`catch`代码块，接着对错误进行处理。

`catch`代码块捕获错误之后，程序不会中断，会按照正常流程继续执行下去。
```js
try {
  throw "出错了";
} catch (e) {
  console.log(111);
}
console.log(222);
// 111
// 222
```
上面代码中，`try`代码块抛出的错误，被`catch`代码块捕获后，程序会继续向下执行。

`catch`代码块之中，还可以再抛出错误，甚至使用嵌套的`try...catch`结构。
```js
var n = 100;

try {
  throw n;
} catch (e) {
  if (e <= 50) {
    // ...
  } else {
    throw e;
  }
}
// Uncaught 100
```
上面代码中，`catch`代码之中又抛出了一个错误。

为了捕捉不同类型的错误，`catch`代码块之中可以加入判断语句。
```js
try {
  foo.bar();
} catch (e) {
  if (e instanceof EvalError) {
    console.log(e.name + ": " + e.message);
  } else if (e instanceof RangeError) {
    console.log(e.name + ": " + e.message);
  }
  // ...
}
```
上面代码中，`catch`捕获错误之后，会判断错误类型（`EvalError`还是`RangeError`），进行不同的处理。
### finally 代码块
`try...catch`结构允许在最后添加一个`finally`代码块，**表示不管是否出现错误，都必需在最后运行的语句。**
```js
function cleansUp() {
  try {
    throw new Error('出错了……');
    console.log('此行不会执行');
  } finally {
    console.log('完成清理工作');
  }
}

cleansUp()
// 完成清理工作
// Uncaught Error: 出错了……
//    at cleansUp (<anonymous>:3:11)
//    at <anonymous>:10:1
```
上面代码中，由于没有`catch`语句块，一旦发生错误，代码就会中断执行。中断执行之前，会先执行`finally`代码块，然后再向用户提示报错信息。

```js
function idle(x) {
  try {
    console.log(x);
    return 'result';
  } finally {
    console.log('FINALLY');
  }
}

idle('hello')
// hello
// FINALLY
```
上面代码中，`try`代码块没有发生错误，而且里面还包括`return`语句，但是`finally`代码块依然会执行。而且，这个函数的返回值还是`result`。

下面的例子说明，`return`语句的执行是排在`finally`代码之前，只是等`finally`代码执行完毕后才返回。
```js
var count = 0;
function countUp() {
  try {
    return count;
  } finally {
    count++;
  }
}

countUp()
// 0
count
// 1
```
上面代码说明，`return`语句里面的`count`的值，是在`finally`代码块运行之前就获取了。

下面是`finally`代码块用法的典型场景。
```js
openFile();

try {
  writeFile(Data);
} catch(e) {
  handleError(e);
} finally {
  closeFile();
}
```
上面代码首先打开一个文件，然后在`try`代码块中写入文件，如果没有发生错误，则运行`finally`代码块关闭文件；一旦发生错误，则先使用`catch`代码块处理错误，再使用`finally`代码块关闭文件。

下面的例子充分反映了`try...catch...finally`这三者之间的执行顺序。
```js
function f() {
  try {
    console.log(0);
    throw 'bug';
  } catch(e) {
    console.log(1);
    return true; // 这句原本会延迟到 finally 代码块结束再执行
    console.log(2); // 不会运行
  } finally {
    console.log(3);
    return false; // 这句会覆盖掉前面那句 return
    console.log(4); // 不会运行
  }

  console.log(5); // 不会运行
}

var result = f();
// 0
// 1
// 3

result
// false
```
上面代码中，`catch`代码块结束执行之前，会先执行`finally`代码块。

`catch`代码块之中，触发转入`finally`代码块的标志，不仅有`return`语句，还有`throw`语句。
```js
function f() {
  try {
    throw '出错了！';
  } catch(e) {
    console.log('捕捉到内部错误');
    throw e; // 这句原本会等到finally结束再执行
  } finally {
    return false; // 直接返回
  }
}

try {
  f();
} catch(e) {
  // 此处不会执行
  console.log('caught outer "bogus"');
}

//  捕捉到内部错误
```
上面代码中，进入`catch`代码块之后，一遇到`throw`语句，就会去执行`finally`代码块，其中有`return false`语句，因此就直接返回了，不再会回去执行`catch`代码块剩下的部分了。

`try`代码块内部，还可以再使用`try`代码块。
```js
try {
  try {
    consle.log('Hello world!'); // 报错
  }
  finally {
    console.log('Finally');
  }
  console.log('Will I run?');
} catch(error) {
  console.error(error.message);
}
// Finally
// consle is not defined
```
上面代码中，`try`里面还有一个`try`。内层的`try`报错（`console`拼错了），这时会执行内层的`finally`代码块，然后抛出错误，被外层的`catch`捕获。
## console 对象与控制台
### console 对象
`console`对象是 JavaScript 的原生对象，它有点像 Unix 系统的标准输出`stdout`和标准错误`stderr`，可以输出各种信息到控制台，并且还提供了很多有用的辅助方法。
### console 对象的静态方法
`console`对象提供的各种静态方法，用来与控制台窗口互动。
#### console.log()**「常用」**，console.info()，console.debug()
##### console.log
方法用于在控制台输出信息。它可以接受一个或多个参数，将它们连接起来输出。
```js
console.log('Hello World')
// Hello World
console.log('a', 'b', 'c')
// a b c
```
如果第一个参数是格式字符串（使用了格式占位符），`console.log`方法将依次用后面的参数替换占位符，然后再进行输出。
```js
console.log(' %s + %s = %s', 1, 1, 2)
//  1 + 1 = 2
```
`console.log`方法支持以下占位符，不同类型的数据必须使用对应的占位符。
- `%s` 字符串
- `%d` 整数
- `%i` 整数
- `%f` 浮点数
- `%o` 对象的链接
- `%c` CSS 格式字符串
```js
var number = 11 * 9;
var color = 'red';

console.log('%d %s balloons', number, color);
// 99 red balloons
```
##### console.info
是`console.log`方法的别名，用法完全一样。只不过`console.info`方法会在输出信息的前面，加上一个**蓝色图标**。
##### console.debug
方法与`console.log`方法类似，会在控制台输出调试信息。但是，默认情况下，`console.debug`输出的信息不会显示，只有在打开显示级别在`verbose`的情况下，才会显示。
`console`对象的所有方法，都可以被覆盖。因此，可以按照自己的需要，定义`console.log`方法。
```js
['log', 'info', 'warn', 'error'].forEach(function(method) {
  console[method] = console[method].bind(
    console,
    new Date().toISOString()
  );
});

console.log("出错了！");
// 2014-05-18T09:00.000Z 出错了！
```

上面代码表示，使用自定义的`console.log`方法，可以在显示结果添加当前时间。
#### console.warn()，console.error()
`warn`方法和`error`方法也是在控制台输出信息，它们与`log`方法的不同之处在于，`warn`方法输出信息时，在最前面加一个**黄色三角**，表示警告；
`error`方法输出信息时，在最前面加一个**红色的叉**，表示出错。同时，还会高亮显示输出文字和错误发生的堆栈。其他方面都一样。
```js
console.error('Error: %s (%i)', 'Server is not responding', 500)
// Error: Server is not responding (500)
console.warn('Warning! Too few nodes (%d)', document.childNodes.length)
// Warning! Too few nodes (1)
```
可以这样理解，`log`方法是写入标准输出（`stdout`），`warn`方法和`error`方法是写入标准错误（`stderr`）。
#### console.table()
对于某些复合类型的数据，`console.table`方法可以将其转为表格显示。
```js
var languages = [
  { name: "JavaScript", fileExtension: ".js" },
  { name: "TypeScript", fileExtension: ".ts" },
  { name: "CoffeeScript", fileExtension: ".coffee" }
];

console.table(languages);
```
上面代码的`language`变量，转为表格显示如下。

|(index)|name|fileExtension|
|---|---|---|
|0|"JavaScript"|".js"|
|1|"TypeScript"|".ts"|
|2|"CoffeeScript"|".coffee"|

下面是显示表格内容的例子。
```js
var languages = {
  csharp: { name: "C#", paradigm: "object-oriented" },
  fsharp: { name: "F#", paradigm: "functional" }
};

console.table(languages);
```
上面代码的`language`，转为表格显示如下。

| (index) | name | paradigm          |
| ------- | ---- | ----------------- |
| csharp  | "C#" | "object-oriented" |
| fsharp  | "F#" | "functional"      |
#### console.count()
`count`方法用于计数，输出它被调用了多少次。
```js
function greet(user) {
  console.count();
  return 'hi ' + user;
}

greet('bob')
//  : 1
// "hi bob"

greet('alice')
//  : 2
// "hi alice"

greet('bob')
//  : 3
// "hi bob"
```
上面代码每次调用`greet`函数，内部的`console.count`方法就输出执行次数。

该方法可以接受一个字符串作为参数，作为标签，对执行次数进行分类。
```js
function greet(user) {
  console.count(user);
  return "hi " + user;
}

greet('bob')
// bob: 1
// "hi bob"

greet('alice')
// alice: 1
// "hi alice"

greet('bob')
// bob: 2
// "hi bob"
```

上面代码根据参数的不同，显示`bob`执行了两次，`alice`执行了一次。
#### console.dir()**「常用」**，console.dirxml()
`dir`方法用来对一个**对象进行检查**（inspect），并以易于阅读和打印的格式显示。
```js
console.log({f1: 'foo', f2: 'bar'})
// Object {f1: "foo", f2: "bar"}

console.dir({f1: 'foo', f2: 'bar'})
// Object
//   f1: "foo"
//   f2: "bar"
//   __proto__: Object
```
上面代码显示`dir`方法的输出结果，比`log`方法更易读，信息也更丰富。

该方法对于输出 DOM 对象非常有用，因为会显示 DOM 对象的所有属性。
```js
console.dir(document.body)
```

Node 环境之中，还可以指定以代码高亮的形式输出。
```js
console.dir(obj, {colors: true})
```

`dirxml`方法主要用于以目录树的形式，显示 DOM 节点。
```js
console.dirxml(document.body)
```

如果参数不是 DOM 节点，而是普通的 JavaScript 对象，`console.dirxml`等同于`console.dir`。
```js
console.dirxml([1, 2, 3])
// 等同于
console.dir([1, 2, 3])
```
#### console.assert()
`console.assert`方法主要用于程序运行过程中，进行条件判断，如果不满足条件，就**显示一个错误**，但不会中断程序执行。这样就相当于提示用户，内部状态不正确。

它接受两个参数，第一个参数是表达式，第二个参数是字符串。只有当第一个参数为`false`，才会提示有错误，在控制台输出第二个参数，否则不会有任何结果。
```js
console.assert(false, '判断条件不成立')
// Assertion failed: 判断条件不成立

// 相当于
try {
  if (!false) {
    throw new Error('判断条件不成立');
  }
} catch(e) {
  console.error(e);
}
```
下面是一个例子，判断子节点的个数是否大于等于500。
```js
console.assert(list.childNodes.length < 500, '节点个数大于等于500')
```
上面代码中，如果符合条件的节点小于500个，不会有任何输出；只有大于等于500时，才会在控制台提示错误，并且显示指定文本。
#### console.time()，console.timeEnd()
这两个方法用于计时，可以算出一个操作所花费的准确时间。
```js
console.time('Array initialize');

var array= new Array(1000000);
for (var i = array.length - 1; i >= 0; i--) {
  array[i] = new Object();
};

console.timeEnd('Array initialize');
// Array initialize: 49.752ms
```
`time`方法表示计时开始，`timeEnd`方法表示计时结束。它们的参数是计时器的名称。调用`timeEnd`方法之后，控制台会显示“计时器名称: 所耗费的时间”。
#### console.group()，console.groupEnd()，console.groupCollapsed()
`console.group`和`console.groupEnd`这两个方法用于将显示的信息分组。它只在输出大量信息时有用，分在一组的信息，可以用鼠标折叠/展开。
```js
console.group('一级分组');
console.log('一级分组的内容');

console.group('二级分组');
console.log('二级分组的内容');

console.groupEnd(); // 二级分组结束
console.groupEnd(); // 一级分组结束
```
上面代码会将“二级分组”显示在“一级分组”内部，并且“一级分组”和“二级分组”前面都有一个折叠符号，可以用来折叠本级的内容。

`console.groupCollapsed`方法与`console.group`方法很类似，唯一的区别是该组的内容，在第一次显示时是收起的（collapsed），而不是展开的。
```js
console.groupCollapsed('Fetching Data');

console.log('Request Sent');
console.error('Error: Server not responding (500)');

console.groupEnd();
```
上面代码只显示一行”Fetching Data“，点击后才会展开，显示其中包含的两行。
#### console.trace()，console.clear()
`console.trace`方法显示当前执行的代码在堆栈中的调用路径。
```js
console.trace()
// console.trace()
//   (anonymous function)
//   InjectedScript._evaluateOn
//   InjectedScript._evaluateAndWrap
//   InjectedScript.evaluate
```
`console.clear`方法用于清除当前控制台的所有输出，将光标回置到第一行。如果用户选中了控制台的“Preserve log”选项，`console.clear`方法将不起作用。
### 控制台命令行 API
浏览器控制台中，除了使用`console`对象，还可以使用一些控制台自带的命令行方法。
#### （1）`$_`
`$_`属性返回上一个表达式的值。
```js
2 + 2
// 4
$_
// 4
```
#### （2）`$0` - `$4`
控制台保存了最近5个在 Elements 面板选中的 DOM 元素，`$0`代表倒数第一个（最近一个），`$1`代表倒数第二个，以此类推直到`$4`。
#### （3）`$(selector)`
`$(selector)`返回第一个匹配的元素，等同于`document.querySelector()`。注意，如果页面脚本对`$`有定义，则会覆盖原始的定义。比如，页面里面有 jQuery，控制台执行`$(selector)`就会采用 jQuery 的实现，返回一个数组。
#### （4）`$$(selector)`
`$$(selector)`返回选中的 DOM 对象，等同于`document.querySelectorAll`。
#### （5）`$x(path)`
`$x(path)`方法返回一个数组，包含匹配特定 XPath 表达式的所有 DOM 元素。
```js
$x("//p[a]")
```
上面代码返回所有包含`a`元素的`p`元素。
#### （6）`inspect(object)`
`inspect(object)`方法打开相关面板，并选中相应的元素，显示它的细节。DOM 元素在`Elements`面板中显示，比如`inspect(document)`会在 Elements 面板显示`document`元素。JavaScript 对象在控制台面板`Profiles`面板中显示，比如`inspect(window)`。
#### （7）`getEventListeners(object)`
`getEventListeners(object)`方法返回一个对象，该对象的成员为`object`登记了回调函数的各种事件（比如`click`或`keydown`），每个事件对应一个数组，数组的成员为该事件的回调函数。
#### （8）`keys(object)`，`values(object)`
`keys(object)`方法返回一个数组，包含`object`的所有键名。
`values(object)`方法返回一个数组，包含`object`的所有键值。
```js
var o = {'p1': 'a', 'p2': 'b'};

keys(o)
// ["p1", "p2"]
values(o)
// ["a", "b"]
```
#### （9）`monitorEvents(object[, events]) ，unmonitorEvents(object[, events])`
`monitorEvents(object[, events])`方法监听特定对象上发生的特定事件。事件发生时，会返回一个`Event`对象，包含该事件的相关信息。`unmonitorEvents`方法用于停止监听。
```js
monitorEvents(window, "resize");
monitorEvents(window, ["resize", "scroll"])
```
上面代码分别表示单个事件和多个事件的监听方法。
```js
monitorEvents($0, 'mouse');
unmonitorEvents($0, 'mousemove');
```
上面代码表示如何停止监听。

`monitorEvents`允许监听同一大类的事件。所有事件可以分成四个大类。
- mouse："mousedown", "mouseup", "click", "dblclick", "mousemove", "mouseover", "mouseout", "mousewheel"
- key："keydown", "keyup", "keypress", "textInput"
- touch："touchstart", "touchmove", "touchend", "touchcancel"
- control："resize", "scroll", "zoom", "focus", "blur", "select", "change", "submit", "reset"
```js
monitorEvents($("#msg"), "key");
```
上面代码表示监听所有`key`大类的事件。
#### （10）其他方法
命令行 API 还提供以下方法。
- `clear()`：清除控制台的历史。
- `copy(object)`：复制特定 DOM 元素到剪贴板。
- `dir(object)`：显示特定对象的所有属性，是`console.dir`方法的别名。
- `dirxml(object)`：显示特定对象的 XML 形式，是`console.dirxml`方法的别名。
### debugger 语句 **「常用」**
`debugger`语句主要用于除错，作用是设置断点。如果有正在运行的除错工具，程序运行到`debugger`语句时会自动停下。如果没有除错工具，`debugger`语句不会产生任何结果，JavaScript 引擎自动跳过这一句。

Chrome 浏览器中，当代码运行到`debugger`语句时，就会暂停运行，自动打开脚本源码界面。
```js
for(var i = 0; i < 5; i++){
  console.log(i);
  if (i === 2) debugger;
}
```

上面代码打印出0，1，2以后，就会暂停，自动打开源码界面，等待进一步处理。
# 标准库
## Object 对象
### 概述
JavaScript 原生提供`Object`对象 **（注意起首的`O`是大写）**，本章介绍该对象原生的各种方法。
JavaScript 的所有其他对象**都继承自`Object`对象**，即那些对象都是`Object`的实例。
`Object`对象的原生方法分成两类：`Object`本身的方法与`Object`的实例方法。
#### （1）`Object`对象本身的方法
所谓“本身的方法”就是直接定义在`Object`对象的方法。
```js
Object.print = function (o) { console.log(o) };
```
上面代码中，`print`方法就是直接定义在`Object`对象上。
#### （2）`Object`的实例方法
所谓实例方法就是定义在`Object`原型对象`Object.prototype`上的方法。它可以被`Object`实例直接使用。
```js
Object.prototype.print = function () {
  console.log(this);
};

var obj = new Object();
obj.print() // Object
```
上面代码中，`Object.prototype`定义了一个`print`方法，然后生成一个`Object`的实例`obj`。
`obj`直接继承了`Object.prototype`的属性和方法，可以直接使用`obj.print`调用`print`方法。也就是说，`obj`对象的`print`方法实质上就是调用`Object.prototype.print`方法。

关于原型对象`object.prototype`的详细解释，参见《面向对象编程》章节。这里只要知道，**凡是定义在`Object.prototype`对象上面的属性和方法，将被所有实例对象共享就可以了**。

以下先介绍`Object`作为函数的用法，然后再介绍`Object`对象的原生方法，分成对象自身的方法（又称为“静态方法”）和实例方法两部分。
### Object()
**`Object`本身是一个函数**，可以当作工具方法使用，**将任意值转为对象**。这个方法常用于保证某个值一定是对象。

如果参数为空（或者为`undefined`和`null`），`Object()`返回一个空对象。
```js
var obj = Object();
// 等同于
var obj = Object(undefined);
var obj = Object(null);

obj instanceof Object // true
```
上面代码的含义，是将`undefined`和`null`转为对象，结果得到了一个空对象`obj`。

**`instanceof`运算符用来验证，一个对象是否为指定的构造函数的实例**。
`obj instanceof Object`返回`true`，就表示`obj`对象是`Object`的实例。

如果参数是原始类型的值，`Object`方法将其转为对应的包装对象的实例（参见《原始类型的包装对象》一章）。
```js
var obj = Object(1);
obj instanceof Object // true
obj instanceof Number // true

var obj = Object('foo');
obj instanceof Object // true
obj instanceof String // true

var obj = Object(true);
obj instanceof Object // true
obj instanceof Boolean // true
```
上面代码中，`Object`函数的参数是各种原始类型的值，转换成对象就是原始类型值对应的包装对象。

如果`Object`方法的参数是一个对象，它总是返回该对象，即不用转换。
```js
var arr = [];
var obj = Object(arr); // 返回原数组
obj === arr // true

var value = {};
var obj = Object(value) // 返回原对象
obj === value // true

var fn = function () {};
var obj = Object(fn); // 返回原函数
obj === fn // true
```

利用这一点，可以写一个判断变量是否为对象的函数。
```js
function isObject(value) {
  return value === Object(value);
}

isObject([]) // true
isObject(true) // false
```
### Object 构造函数
`Object`不仅可以当作工具函数使用，还可以当作构造函数使用，即前面可以使用`new`命令。

`Object`构造函数的首要用途，是直接通过它来生成新对象。
```js
var obj = new Object();
```
> 注意，通过`var obj = new Object()`的写法生成新对象，与字面量的写法`var obj = {}`是等价的。或者说，后者只是前者的一种简便写法。

`Object`构造函数的用法与工具方法很相似，几乎一模一样。使用时，可以接受一个参数，如果该参数是一个对象，则直接返回这个对象；如果是一个原始类型的值，则返回该值对应的包装对象（详见《包装对象》一章）。
```js
var o1 = {a: 1};
var o2 = new Object(o1);
o1 === o2 // true

var obj = new Object(123);
obj instanceof Number // true
```
虽然用法相似，但是`Object(value)`与`new Object(value)`两者的语义是不同的：
- `Object(value)`: 表示将`value`转成一个对象。
- `new Object(value)`: 表示新生成一个对象，它的值是`value`。
### Object 的静态方法
所谓“静态方法”，是指部署在`Object`对象自身的方法。
#### Object.keys()，Object.values()，Object.getOwnPropertyNames()
`Object.keys`方法和`Object.getOwnPropertyNames`方法都用来遍历对象的属性。
`Object.keys`方法的参数是一个对象，返回一个数组。该数组的成员都是该对象自身的（**而不是继承的**）所有属性名。
```js
var obj = {p1: 123, p2: 456};
Object.keys(obj) // ["p1", "p2"]
Object.values(obj) // [ 123, 456 ]
```

`Object.getOwnPropertyNames`方法与`Object.keys`类似，也是接受一个对象作为参数，返回一个数组，包含了该对象自身的所有属性名。
```js
var obj = {p1: 123, p2: 456};
Object.getOwnPropertyNames(obj) // ["p1", "p2"]
```

对于一般的对象来说，`Object.keys()`和`Object.getOwnPropertyNames()`返回的结果是一样的。只有涉及不可枚举属性时，才会有不一样的结果。`Object.keys`方法只返回可枚举的属性（详见《对象属性的描述对象》一章），**`Object.getOwnPropertyNames`方法还返回不可枚举的属性名**。
```js
var a = ['Hello', 'World'];

Object.keys(a) // ["0", "1"]
Object.getOwnPropertyNames(a) // ["0", "1", "length"]
```
上面代码中，数组的`length`属性是不可枚举的属性，所以只出现在`Object.getOwnPropertyNames`方法的返回结果中。

由于 JavaScript 没有提供计算对象属性个数的方法，所以可以用这两个方法代替。
```js
var obj = {p1: 123, p2: 456};
Object.keys(obj).length // 2
Object.getOwnPropertyNames(obj).length // 2
```
一般情况下，几乎总是使用`Object.keys`方法，遍历对象的属性。
#### 其他方法
除了上面提到的两个方法，`Object`还有不少其他静态方法，将在后文逐一详细介绍。
##### （1）对象属性模型的相关方法
- `Object.getOwnPropertyDescriptor()`：获取某个属性的描述对象。
- `Object.defineProperty()`：通过描述对象，定义某个属性。
- `Object.defineProperties()`：通过描述对象，定义多个属性。
##### （2）控制对象状态的方法
- `Object.preventExtensions()`：防止对象扩展。
- `Object.isExtensible()`：判断对象是否可扩展。
- `Object.seal()`：禁止对象配置。
- `Object.isSealed()`：判断一个对象是否可配置。
- `Object.freeze()`：冻结一个对象。
- `Object.isFrozen()`：判断一个对象是否被冻结。
##### （3）原型链相关方法
- `Object.create()`：该方法可以指定原型对象和属性，返回一个新的对象。
- `Object.getPrototypeOf()`：获取对象的`Prototype`对象。
### Object 的实例方法
除了静态方法，还有不少方法定义在`Object.prototype`对象。它们称为实例方法，所有 **`Object`的实例对象都继承了这些方法**。

`Object`实例对象的方法，主要有以下六个。
- `Object.prototype.valueOf()`：返回当前对象对应的值。
- `Object.prototype.toString()`：返回当前对象对应的字符串形式。
- `Object.prototype.toLocaleString()`：返回当前对象对应的本地字符串形式。
- `Object.prototype.hasOwnProperty()`：判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性。
- `Object.prototype.isPrototypeOf()`：判断当前对象是否为另一个对象的原型。
- `Object.prototype.propertyIsEnumerable()`：判断某个属性是否可枚举。

本节介绍前四个方法，另外两个方法将在后文相关章节介绍。
#### Object.prototype.valueOf()
`valueOf`方法的作用是返回一个对象的“值”，默认情况下返回对象本身。
```js
var obj = new Object();
obj.valueOf() === obj // true
```
上面代码比较`obj.valueOf()`与`obj`本身，两者是一样的。

`valueOf`方法的主要用途是，JavaScript 自动类型转换时会默认调用这个方法（详见《数据类型转换》一章）。
```js
var obj = new Object();
1 + obj // "1[object Object]"
```
上面代码将对象`obj`与数字`1`相加，这时 JavaScript 就会默认调用`valueOf()`方法，求出`obj`的值再与`1`相加。所以，如果自定义`valueOf`方法，就可以得到想要的结果。

```js
var obj = new Object();
obj.valueOf = function () {
  return 2;
};

1 + obj // 3
```
上面代码自定义了`obj`对象的`valueOf`方法，于是`1 + obj`就得到了`3`。这种方法就相当于用自定义的`obj.valueOf`，覆盖`Object.prototype.valueOf`。
#### Object.prototype.toString()
`toString`方法的作用是返回一个对象的字符串形式，默认情况下返回类型字符串。
```js
var o1 = new Object();
o1.toString() // "[object Object]"

var o2 = {a:1};
o2.toString() // "[object Object]"
```
上面代码表示，对于一个对象调用`toString`方法，会返回字符串`[object Object]`，该字符串说明对象的类型。

字符串`[object Object]`本身没有太大的用处，但是通过自定义`toString`方法，可以让对象在自动类型转换时，得到想要的字符串形式。
```js
var obj = new Object();

obj.toString = function () {
  return 'hello';
};

obj + ' ' + 'world' // "hello world"
```
上面代码表示，当对象用于字符串加法时，会自动调用`toString`方法。由于自定义了`toString`方法，所以返回字符串`hello world`。

数组、字符串、函数、Date 对象都分别部署了自定义的`toString`方法，覆盖了`Object.prototype.toString`方法。
```js
[1, 2, 3].toString() // "1,2,3"

'123'.toString() // "123"

(function () {
  return 123;
}).toString()
// "function () {
//   return 123;
// }"

(new Date()).toString()
// "Tue May 10 2016 09:11:31 GMT+0800 (CST)"
```
上面代码中，**数组、字符串、函数、Date 对象调用`toString`方法，并不会返回`[object Object]`**，因为它们都自定义了`toString`方法，覆盖原始方法。
#### toString() 的应用：判断数据类型
`Object.prototype.toString`方法返回对象的类型字符串，因此可以用来判断一个值的类型。
```js
var obj = {};
obj.toString() // "[object Object]"
```
上面代码调用空对象的`toString`方法，结果返回一个字符串`[object Object]`，其中第二个`Object`表示该值的构造函数。这是一个十分有用的判断数据类型的方法。

由于实例对象可能会自定义`toString`方法，覆盖掉`Object.prototype.toString`方法，所以为了得到类型字符串，最好直接使用`Object.prototype.toString`方法。通过函数的`call`方法，可以在任意值上调用这个方法，帮助我们判断这个值的类型。
```js
Object.prototype.toString.call(value)
```
上面代码表示对`value`这个值调用`Object.prototype.toString`方法。

不同数据类型的`Object.prototype.toString`方法返回值如下。
- 数值：返回`[object Number]`。
- 字符串：返回`[object String]`。
- 布尔值：返回`[object Boolean]`。
- undefined：返回`[object Undefined]`。
- null：返回`[object Null]`。
- 数组：返回`[object Array]`。
- arguments 对象：返回`[object Arguments]`。
- 函数：返回`[object Function]`。
- Error 对象：返回`[object Error]`。
- Date 对象：返回`[object Date]`。
- RegExp 对象：返回`[object RegExp]`。
- 其他对象：返回`[object Object]`。

这就是说，`Object.prototype.toString`可以看出一个值到底是什么类型。
```js
Object.prototype.toString.call(2) // "[object Number]"
Object.prototype.toString.call('') // "[object String]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(Math) // "[object Math]"
Object.prototype.toString.call({}) // "[object Object]"
Object.prototype.toString.call([]) // "[object Array]"
```

利用这个特性，可以写出一个比`typeof`运算符更准确的类型判断函数。
```js
var type = function (o){
  var s = Object.prototype.toString.call(o);
  return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};

type({}); // "object"
type([]); // "array"
type(5); // "number"
type(null); // "null"
type(); // "undefined"
type(/abcd/); // "regex"
type(new Date()); // "date"
```
在上面这个`type`函数的基础上，还可以加上专门判断某种类型数据的方法。
```js
var type = function (o){
  var s = Object.prototype.toString.call(o);
  return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};

['Null',
 'Undefined',
 'Object',
 'Array',
 'String',
 'Number',
 'Boolean',
 'Function',
 'RegExp'
].forEach(function (t) {
  type['is' + t] = function (o) {
    return type(o) === t.toLowerCase();
  };
});

type.isObject({}) // true
type.isNumber(NaN) // true
type.isRegExp(/abc/) // true
```
#### Object.prototype.toLocaleString()
`Object.prototype.toLocaleString`方法与`toString`的返回结果相同，也是返回一个值的字符串形式。
```js
var obj = {};
obj.toString(obj) // "[object Object]"
obj.toLocaleString(obj) // "[object Object]"
```
这个方法的主要作用是**留出一个接口**，让各种不同的对象实现自己版本的`toLocaleString`，用来返回**针对某些地域的特定的值**。
```js
var person = {
  toString: function () {
    return 'Henry Norman Bethune';
  },
  toLocaleString: function () {
    return '白求恩';
  }
};

person.toString() // Henry Norman Bethune
person.toLocaleString() // 白求恩
```
上面代码中，`toString()`方法返回对象的一般字符串形式，`toLocaleString()`方法返回本地的字符串形式。

目前，主要有三个对象自定义了`toLocaleString`方法。
- Array.prototype.toLocaleString()
- Number.prototype.toLocaleString()
- Date.prototype.toLocaleString()

举例来说，日期的实例对象的`toString`和`toLocaleString`返回值就不一样，而且`toLocaleString`的返回值跟用户设定的所在地域相关。
```js
var date = new Date();
date.toString() // "Tue Jan 01 2018 12:01:33 GMT+0800 (CST)"
date.toLocaleString() // "1/01/2018, 12:01:33 PM"
```
#### Object.prototype.hasOwnProperty() **「常用」**
`Object.prototype.hasOwnProperty`方法接受一个字符串作为参数，返回一个布尔值，表示该实例对象**自身是否具有该属性**。
```js
var obj = {
  p: 123
};

obj.hasOwnProperty('p') // true
obj.hasOwnProperty('toString') // false
```

上面代码中，对象`obj`自身具有`p`属性，所以返回`true`。`toString`属性是继承的，所以返回`false`。
# Object
## `Object` 是所有类的基类
### 为什么 `Object` 是所有类的基类？
- 所有对象的原型链最终都会指向 `Object.prototype`，因此 `Object` 是 JavaScript 中所有对象的“根类”。
- **原型链的终点**：`Object.prototype.__proto__` 是 `null`，表示原型链的尽头。
- **示例**：
```js
const obj = {};
console.log(obj.__proto__); // Object.prototype
console.log(obj.__proto__.__proto__); // null
```
## 构造函数
```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.tostring = this.name;
}
const p = new Person('z3', 18);
for (const i in p) {
    console.log(i);
}
/*
name
age
tostring
*/
console.log(p.tostring); // z3
```
## 原型对象
隐式原型属性「**实例对象的隐式原型对象指向的是缔造者的原型对象**」
```js
// 定义一个构造函数
function Demo() {
    this.a = 1
    this.b = 2
}
// 创建一个Demo的实例对象
const d = new Demo()
console.log(Demo.prototype) // 显示原型属性「原型对象才有此属性」
console.log(d.__proto__) // 隐式原型属性「实例对象的隐式原型对象指向的是缔造者的原型对象」
console.log(Demo.prototype === d.__proto__) // true

// 程序员通过显示原型属性操作原型对象，追加一个x属性，值为99
Demo.prototype.x = 99

console.log(d) // Demo { a: 1, b: 2 }
console.log(d.x) // 99
```
### 构造函数的原型链
构造函数（如 `Demo`）本身也是对象，它的原型链如下：
```js
Demo.__proto__ === Function.prototype; // true
Function.prototype.__proto__ === Object.prototype; // true
Object.prototype.__proto__ === null; // true
```
- 因此，所有构造函数（包括 `Demo`、`Array`、`String` 等）的原型链最终都指向 `Object.prototype`，这使得 `Object` 成为所有类的基类。
### 自定义对象方法
驼峰转下划线
```js
let str = "bestRiven";
String.prototype.humpToUnderline = function () {
    return this.replace(/[A-Z]/g, function ($0) {
        return "_" + $0.toLowerCase();
    })
}
console.log(str.humpToUnderline()); // best_riven
```
### `prototype` 和 `__proto__` 的区别与联系
#### 关系图解
```
Person 构造函数
│
├── prototype (Person.prototype)                // 构造函数的原型对象
│   ├── 属性/方法（如 `sayHello`）
│   └── __proto__ → Object.prototype            // Person.prototype 是 Object 的实例
│
└── __proto__ → Function.prototype              // Person 是 Function 的实例

person1 实例
│
└── __proto__ → Person.prototype                // person1 的原型是 Person.prototype
```
#### `prototype`：显式原型（构造函数的属性）
- **定义**：
	`prototype` 是 **构造函数** 的一个属性，它指向一个对象（称为“**原型对象**”）。所有通过该构造函数创建的实例对象都会共享这个原型对象上的属性和方法。
- **作用**：用于实现继承和属性共享。
- **示例**：
```js
function Demo() {
	this.a = 1;
	this.b = 2;
}
Demo.prototype.c = 3; // 所有 Demo 实例共享属性 c
const d = new Demo();
console.log(d.c); // 3
```
#### `__proto__`：隐式原型（实例的属性）
- **定义**：
	`__proto__` 是 **实例对象** 的一个内置属性（非标准属性，但大多数浏览器支持），它指向该实例对象的原型对象（即构造函数的 `prototype`）。
- **作用**：构成原型链，用于属性查找。当访问实例的某个属性时，如果实例自身没有该属性，JavaScript 引擎会沿着 `__proto__` 向上查找原型链，直到找到或到达原型链顶端（`null`）。
- **示例**：
```js
const d = new Demo();
console.log(d.__proto__); // 指向 Demo.prototype
```
#### 两者的关系
- 实例的 `__proto__` 指向其构造函数的 `prototype`：
```javascript
d.__proto__ === Demo.prototype; // true
```
- 构造函数的 `__proto__` 指向 `Function.prototype`（因为构造函数本身是函数，由 `Function` 构造）：
```js
Demo.__proto__ === Function.prototype; // true
```
#### 原型链关系
- `d` 的原型链：
```js
d.__proto__ === Demo.prototype; // true
Demo.prototype.__proto__ === Object.prototype; // true
Object.prototype.__proto__ === null; // true
```
- `Demo` 的原型链：
```js
Demo.__proto__ === Function.prototype; // true
Function.prototype.__proto__ === Object.prototype; // true
Object.prototype.__proto__ === null; // true
```
#### 总结
| 概念          | 所属对象     | 作用             | 访问方式                                         | 示例                                    |
| ----------- | -------- | -------------- | -------------------------------------------- | ------------------------------------- |
| `prototype` | **构造函数** | 用于继承和共享属性/方法   | `构造函数.prototype`                             | `Demo.prototype.c = 3`                |
| `__proto__` | **实例对象** | 指向原型对象，用于原型链查找 | `对象.__proto__` 或 `Object.getPrototypeOf(对象)` | `d.__proto__ === Demo.prototype`      |
| `Object`    | 根类       | 所有对象的原型链终点     |                                              | `Object.prototype.__proto__ === null` |
#### 注意事项
1. **避免直接修改 `__proto__`**  
    修改原型链可能导致性能问题，推荐使用 `Object.setPrototypeOf()`（谨慎使用）。
2. **`__proto__` 已被标准化但不推荐**  
    虽然 `__proto__` 在 ES6 中被标准化，但官方推荐使用 `Object.getPrototypeOf()` 和 `Object.setPrototypeOf()`。
3. **原型链的终点**  
    所有原型链最终都会指向 `Object.prototype`，其 `__proto__` 为 `null`，表示原型链的终点。
### hasOwnProperty
判断是否具有指定属性「不包含继承的属性」
```js
function My() {
    this.name = "z3";
    this.age = 18;
}
// 这个函数只创建了一次(提升性能)
My.prototype = {
    address: "china"
}
let test = new My();
for (let v in test) {
    // 判断是否为test对象的属性
    if (test.hasOwnProperty(v)) {
        console.log(test[v]);
    }
}
/*
z3
18
*/
```
### Object.hasOwn()
判断是否具有指定属性「是否为自身的属性」
```javascript
const foo = Object.create({ a: 123 });
foo.b = 456;

Object.hasOwn(foo, 'a') // false
Object.hasOwn(foo, 'b') // true
```

`Object.hasOwn()`的一个好处是，对于不继承`Object.prototype`的对象不会报错，而`hasOwnProperty()`是会报错的。
```javascript
const obj = Object.create(null);

obj.hasOwnProperty('foo') // 报错
Object.hasOwn(obj, 'foo') // false
```
上面示例中，`Object.create(null)`返回的对象`obj`是没有原型的，不继承任何属性，这导致调用`obj.hasOwnProperty()`会报错，但是`Object.hasOwn()`就能正确处理这种情况。
## 构造函数与原型对象的关系
- **构造函数**（如 `Demo`）是一个函数，用于创建对象实例。
- **原型对象**（`Demo.prototype`）是一个普通对象，由 JavaScript 自动创建。它包含所有通过 `new Demo()` 创建的实例共享的属性和方法。
- **关键区别**：
  - `Demo` 是一个函数（类型为 `function`）。
  - `Demo.prototype` 是一个对象（类型为 `object`）。
### 代码示例与验证
```javascript
function Demo() {
  this.a = 1;
  this.b = 2;
}

console.log(typeof Demo);         // "function"
console.log(typeof Demo.prototype); // "object"
```
- **结论**：`Demo` 是函数，`Demo.prototype` 是对象，两者类型不同，**不可能是同一个实体**。
### 原型对象的特殊属性
原型对象 `Demo.prototype` 本身有一个 `constructor` 属性，它指向构造函数 `Demo`：
```javascript
console.log(Demo.prototype.constructor === Demo); // true
```
- **含义**：`Demo.prototype.constructor` 指向 `Demo`，但 `Demo.prototype` 本身仍然是一个独立的对象。
### 总结
| 项目                    | 类型         | 作用                     | 示例                     |
| --------------------- | ---------- | ---------------------- | ---------------------- |
| 构造函数 `Demo`           | `function` | 创建对象实例的模板              | `new Demo()`           |
| 原型对象 `Demo.prototype` | `object`   | 存储所有实例共享的属性和方法         | `Demo.prototype.c = 3` |
| 实例 `d`                | `object`   | 实例对象，自身属性 + 原型链上的属性/方法 | `d.a`, `d.__proto__.c` |
## `Object` 构造函数和 `Function` 构造函数
在 JavaScript 中，`Object` 构造函数和 `Function` 构造函数是 **两个不同的概念**，但它们之间存在紧密的关联。以下是详细解释和示例：
#### 1. `Object` 构造函数
- **用途**：`Object` 是 JavaScript 中所有对象的基类，用于创建普通对象。
- **特点**：
  - 所有对象（包括函数、数组、日期等）最终都继承自 `Object.prototype`。
  - `Object` 本身是一个构造函数，它也是一个函数对象（由 `Function` 构造函数实例化）。
```javascript
// 创建一个普通对象
const obj = new Object();
console.log(obj); // {}

// 使用字面量语法创建对象（本质是 new Object()）
const objLiteral = {};
console.log(objLiteral); // {}
```
#### 2. `Function` 构造函数
- **用途**：`Function` 是 JavaScript 中所有函数的基类，用于动态创建函数。
- **特点**：
  - 所有函数（包括 `Object` 构造函数本身）都是 `Function` 的实例。
  - `Function` 本身也是一个函数对象（由 `Function` 构造函数实例化，形成“自指”的循环）。
```javascript
// 动态创建函数
const add = new Function('a', 'b', 'return a + b');
console.log(add(2, 3)); // 5
```
#### 3. `Object` 和 `Function` 的关系
- **相互依赖**：
  - `Object` 是 `Function` 的实例（`Object instanceof Function === true`）。
  - `Function` 的 `prototype` 是 `Object` 的实例（`Function.prototype instanceof Object === true`）。

- **原型链关系**：
  - `Object` 和 `Function` 的原型链最终都指向 `Object.prototype`，形成一个循环依赖的结构。
  - `Function` 的 `__proto__` 指向 `Function.prototype`，而 `Function.prototype.__proto__` 指向 `Object.prototype`。
```javascript
// Object 是 Function 的实例
console.log(Object instanceof Function); // true

// Function 的 prototype 是 Object 的实例
console.log(Function.prototype instanceof Object); // true

// Object 的 prototype 链顶端是 null
console.log(Object.prototype.__proto__); // null
```
#### 4. 举例说明
#### **示例 1：`Object` 和 `Function` 的构造函数关系**
```javascript
// Object 是 Function 的实例
console.log(Object instanceof Function); // true

// Function 是 Object 的实例？
console.log(Function instanceof Object); // true

// 为什么？因为 Function 的 __proto__ 是 Function.prototype，
// 而 Function.prototype.__proto__ 是 Object.prototype
console.log(Function.__proto__ === Function.prototype); // true
console.log(Function.prototype.__proto__ === Object.prototype); // true
```
#### **示例 2：函数和对象的原型链**
```javascript
// 定义一个普通函数
function Foo() {}
console.log(Foo instanceof Function); // true
console.log(Foo instanceof Object);   // true

// 原型链关系
console.log(Foo.__proto__ === Function.prototype); // true
console.log(Foo.prototype.__proto__ === Object.prototype); // true
```
#### **示例 3：`Object` 和 `Function` 的自指性**
```javascript
// Function 是 Function 的实例？
console.log(Function instanceof Function); // true
console.log(Function.__proto__ === Function.prototype); // true

// Object 是 Object 的实例？
console.log(Object instanceof Object); // true
console.log(Object.__proto__ === Function.prototype); // true
```
#### 5. 总结
| 概念                  | 特点                                                                             |
| ------------------- | ------------------------------------------------------------------------------ |
| **`Object` 构造函数**   | 所有对象的基类，用于创建普通对象，继承自 `Function` 的实例。                                           |
| **`Function` 构造函数** | 所有函数的基类，用于动态创建函数，自身也是 `Function` 的实例（自指）。                                      |
| **关系**              | `Object` 是 `Function` 的实例，`Function` 的 `prototype` 是 `Object` 的实例。两者通过原型链相互依赖。 |
#### 6. 关键结论
- **`Object` 和 `Function` 是不同的构造函数**，但它们通过原型链形成一个循环依赖的结构。
- **`Object` 是 `Function` 的实例**，因为所有构造函数（包括 `Object`）都是 `Function` 的实例。
- **`Function` 的 `prototype` 是 `Object` 的实例**，因为 `Function.prototype` 是一个对象，继承自 `Object.prototype`。
- **JavaScript 的原型链设计**：`Function` 和 `Object` 的关系体现了 JavaScript 中“函数是对象，对象是函数”的核心思想。
## 浅拷贝、深拷贝
### 浅拷贝（Shallow Copy）「默认」
#### 定义
创建一个新对象，但新对象的**顶层属性**如果是引用类型（如对象、数组），则指向**原始对象的同一内存地址**。
**修改新对象的引用类型属性时，原始对象也会被修改。**
#### 特点
- 仅复制对象的**第一层属性**。
- 对基本类型（如 `number`, `string`）进行独立复制。
- 对引用类型（如对象、数组）仅复制引用地址，而非实际值。
```js
const original = {a: 1, b: [2], c: [3], d: {d1: [4]}, e: {e1: 5}};
const copy = {...original};

copy.a = 2;
copy.b = [3];
copy.c[0] = 4;
copy.d.d1 = [5];
copy.e.e1 = 6;

// 原始对象的 c[0]、d.d1、e.e1 也被修改了
console.log(original); // 输出: { a: 1, b: [ 2 ], c: [ 4 ], d: { d1: [ 5 ] }, e: { e1: 6 } }
console.log(copy);     // 输出: { a: 2, b: [ 3 ], c: [ 4 ], d: { d1: [ 5 ] }, e: { e1: 6 } }
```
>**浅拷贝只复制引用地址**，但**直接赋值 `copy.b = [3]`** 是在 `copy` 对象上**重新创建了一个新的引用**，而原始对象 `original` 的 `b` 属性仍然指向原来的数组 `[2]`。
#### 实现方式
##### 扩展运算符（...）
通过展开语法复制对象或数组的顶层属性「所有可遍历属性」。
```js
let obj = {  
    0: 'a', 1: 'b', 2: 'c', length: 3  
};  
let arr = ['a', 'b', 'c'];  
const objCopy = {...obj};  
const arrCopy = [...arr];  
console.log(objCopy); // { '0': 'a', '1': 'b', '2': 'c', length: 3 }  
console.log(arrCopy); // [ 'a', 'b', 'c' ]
```
##### Object.assign()
将源对象的属性复制到目标对象，返回新对象。
[参考链接](https://www.jianshu.com/p/f9ec860ecd81)
[ES6 Object.assign()](https://es6.ruanyifeng.com/#docs/object-methods)
```js
Object.assign() // 方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。
Object.assign(target, ...sources) //【target：目标对象】，【souce：源对象（可多个）】
```
###### 方式1：无法复制属性描述符
>注：
1. 如果对象相同的键，保留**后者对象的键值**
2. `Object.assign` 仅复制自身可枚举值，触发 getter/setter，不复制描述符或原型链属性。完整复制需结合 `Object.getOwnPropertyDescriptor` 和 `Object.defineProperty`。
```js
const object1 = {
    a: 1,
    b: 2,
    c: 3
};
Object.defineProperty(object1, 'secret', {
    value: '42',
    enumerable: false,    // 不可枚举
    writable: false,      // 不可写
    configurable: false   // 不可配置
});

let object2 = {c: 4, d: 5};
Object.assign(object2, object1);
console.log(object1)  // { a: 1, b: 2, c: 3 }
console.log(object2)  // { c: 3, d: 5, a: 1, b: 2 }

// 检查 object1 的属性描述符
console.log(Object.getOwnPropertyDescriptor(object1, 'secret'));
/*
{
  value: '42',
  writable: false,
  enumerable: false,
  configurable: false
}
*/
// 检查 object2 的属性描述符
console.log(Object.getOwnPropertyDescriptor(object2, 'secret')); // undefined
```
###### 方式2：正确复制属性描述符
```js
// 创建一个对象，并定义一个不可枚举、只读的属性
const source = {};
Object.defineProperty(source, 'secret', {
    value: '42',
    enumerable: false,    // 不可枚举
    writable: false,      // 不可写
    configurable: false   // 不可配置
});
// 使用 Object.getOwnPropertyDescriptor 和 Object.defineProperty
const target = {};
// 获取源对象的属性描述符
const sourceDescriptor = Object.getOwnPropertyDescriptor(source, 'secret');
// 将描述符应用到目标对象
Object.defineProperty(target, 'secret', sourceDescriptor);
// 检查 target 的属性描述符
const descriptor = Object.getOwnPropertyDescriptor(target, 'secret');
console.log(descriptor);
/*
{
  value: '42',
  writable: false,     // 保留原始设置
  enumerable: false,
  configurable: false
}
*/
```
##### Object.create()
创建一个**新对象**，并将其**原型（`Prototype`）指向指定的对象**，实现**原型继承**。
新对象会共享原型链上的属性和方法，但**不会直接复制属性值**（引用类型属性会共享）。
```js
const copy = Object.create(original);
console.log(copy.__proto__);
```
##### 数组的`slice()`
用于数组的浅拷贝，复制数组元素但引用类型元素仍共享内存地址。
```js
const copy = arr.slice();
```
### 深拷贝（Deep Copy）
#### 定义
创建一个**完全独立的新对象**，新对象的所有层级属性（包括嵌套的对象、数组）都与原始对象完全隔离。修改新对象的任何属性**不会**影响原始对象。
#### 特点
递归复制对象的所有层级属性。
支持基本类型和引用类型的独立复制。
#### 实现方式
##### 第三方库（如 Lodash）「推荐」
使用`_.cloneDeep()` 实现深度克隆，兼容性更好。
```js
const _ = require('lodash');
// 原始对象（包含多种数据类型）
const original = {
    name: 'Alice',
    age: 30,
    address: { city: 'Tokyo' },
    hobbies: ['reading'],
    birthDate: new Date(),
    map: new Map([['key', 'value']]),
    set: new Set([1, 2, 3])
};
// 深拷贝
const copy = _.cloneDeep(original);
// 修改深拷贝对象
copy.age = 35;
copy.address.city = 'Osaka';
copy.hobbies.push('swimming');
copy.birthDate.setDate(copy.birthDate.getDate() + 1);
copy.map.set('newKey', 'newValue');
copy.set.add(4);
// 输出结果
console.log('Original:', original);
console.log('Copy:', copy);
/*
Original: {
  name: 'Alice',
  age: 30,
  address: { city: 'Tokyo' },
  hobbies: [ 'reading' ],
  birthDate: 2025-05-04T02:49:10.007Z,
  map: Map(1) { 'key' => 'value' },
  set: Set(3) { 1, 2, 3 }
}
Copy: {
  name: 'Alice',
  age: 35,
  address: { city: 'Osaka' },
  hobbies: [ 'reading', 'swimming' ],
  birthDate: 2025-05-05T02:49:10.007Z,
  map: Map(2) { 'key' => 'value', 'newKey' => 'newValue' },
  set: Set(4) { 1, 2, 3, 4 }
}
*/
```
##### JSON.parse(JSON.stringify(obj))
将对象序列化为 JSON 字符串再解析为新对象。
```js
obj1 = { a: 0 , b: { c: 0}}; 
let obj3 = JSON.parse(JSON.stringify(obj1)); 
obj1.a = 4; 
obj1.b.c = 4; 
console.log(JSON.stringify(obj3)); // { a: 0, b: { c: 0}}
```
### 对比与选择

#### 1. 核心区别
| **特性**       | **浅拷贝**   | **深拷贝**                   |
| ------------ | --------- | ------------------------- |
| **复制层级**     | 仅第一层属性    | 所有层级属性                    |
| **引用类型修改影响** | 修改会影响原始对象 | 修改不会影响原始对象                |
| **特殊类型支持**   | 支持所有类型    | `JSON` 方法不支持函数、`Symbol` 等 |
#### 2. 选择场景
| **场景**               | **推荐方法**                    |
| -------------------- | --------------------------- |
| 需要共享数据或提升性能（如简单对象）   | 浅拷贝（`Object.assign`, `...`） |
| 需要兼容旧浏览器或复杂类型（如循环引用） | 第三方库（Lodash 的 `cloneDeep`）  |
## 对象排序
```javascript
function person (name, score) {
    this.name = name;
    this.socre = score;
}
p = new person(a, b);
arr.push(p);
arr.sort(compare);
function compare (a, b) {
    return b.score - a.score;
}
console.log(arr);
```
## 排序器
```javascript
var compare = function (prop) {
    return function (obj1, obj2) {
        var val1 = obj1[prop];
        var val2 = obj2[prop];
        if (!isNaN(Number(val1)) && !isNaN(Number(val2))) {
            val1 = Number(val1);
            val2 = Number(val2);
        }
        if (val1 < val2) {
            return -1;
        } else if (val1 > val2) {
            return 1;
        } else {
            return 0;
        }            
    } 
}

checked_criteria_arr.sort(compare("sort_to"));
```
## 将数组里面的对象去重
```javascript
var hash = {};
checked_criteria_arr = checked_criteria_arr.reduce(function(item, next) {
    hash[next.name] ? '' : hash[next.name] = true && item.push(next);
    return item
}, [])
```
## try……catch、throw new Error
```javascript
 window.onload = function () {
	try {
		var res = fun(5, 0);
		console.log(res);
	} catch (e) {
		alert(e.message);
	}
}
function fun(a, b) {
	if (b == 0) {
		throw new Error('除数不能为0');
	}
	return a / b;
}
```
## Date
```
Date.now()（获取从1970.1.1开始的秒数）、Date.parse(new Date())/1000;（获取从1970.1.1开始的秒数）
setFullYear(year,month,day)、getFullYear(year,month,day)
```
### toLocaleString()、toLocaleTimeString()、toLocaleDateString()
```javascript
var date = new Date();
console.log(date.toLocaleTimeString()); // 17:52:05
console.log(date.toLocaleDateString()); // 2021-5-21
console.log(date.toLocaleString()); // 2021-5-21 17:52:05
console.log(date.toLocaleDateString().split('-').join('/')); // 2021/5/21
```
### getMonth()....
```javascript
var date = new Date();
console.log(date.getMonth()); // 4 (比单签月份少1)
```
### setTime(millisec)、getTime()（获取从1970.1.1开始的毫秒数）
设置月份要比实际月份少1
```javascript
var date = new Date();
date.setTime(date.getTime() + (2 * 3600 * 1000)); // 2小时后的时间「毫秒」
console.log(date.toLocaleString());
/**
返回格式受平台影响
浏览器: 2021/5/23 12:14:33
nodejs: 5/16/2024, 12:43:41 PM
/
```
### getMonth()、getDate()、getHours()、getMinutes()、getSeconds()、getMilliseconds()
格式化时间
```javascript
function timestampToTime(timestamp) {
    var date = new Date(timestamp * 1000);//时间戳为10位需*1000，时间戳为13位的话不需乘1000
    var Y = date.getFullYear() + '-';
    var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';
    var D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' ';
    var h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
    var m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
    var s = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds();
    return Y + M + D + h + m + s;
}

console.log(timestampToTime(1403058804)); // 2014-06-18 10:33:24
```
## 属性描述对象
## Array 对象
## 包装对象
## Boolean 对象
## Number 对象
## String 对象
## Math 对象
## Date 对象
## RegExp 对象
## JSON 对象
# DOM
## document
```
write()（写入页面） 、getElementById() （用ID获取元素）
getElementsByName(用name获取元素)、body、createElement()（创建元素）
```
## Node
```shell
onclick # 单击事件
innerHTML # 获取或设置HTML内容
input.value # 获取或设置表单输入元素的值
appendChild() # 向每个匹配的元素内部追加内容
prepend() # 向每个匹配的元素内部前置内容
removeChild() # 从父节点中移除一个子节点
this # 指向当前上下文中的对象
parentNode # 获取节点的父节点
className # 获取或设置元素的 CSS 类名
style # 获取或设置元素的内联样式
style.cssText # 获取或设置元素的内联 CSS 样式字符串
```
实例
```js
// 事件处理：当元素被点击时触发
onclick = function(event) {
  // 事件处理函数
  // event：事件对象，包含有关事件的详细信息
};

// 获取或设置元素的 HTML 内容
// 获取：返回元素的 HTML 内容（字符串）
// 设置：用指定的 HTML 内容替换元素的现有内容
element.innerHTML;
element.innerHTML = "<p>新内容</p>";

// 获取或设置表单输入元素的值
// 获取：返回输入元素的当前值（字符串）
// 设置：设置输入元素的值
inputElement.value;
inputElement.value = "新的输入值";

// 将一个节点追加到指定父节点的子节点列表末尾
parentElement.appendChild(childNode);

// 将一个节点插入到指定父节点的子节点列表开头
parentElement.prepend(childNode);

// 从父节点中移除指定的子节点
parentElement.removeChild(childNode);

// 指向当前上下文中的对象
// 在事件处理函数中，this 指向触发事件的元素
element.onclick = function() {
  console.log(this); // 指向被点击的元素
};

// 获取节点的父节点
childNode.parentNode;

// 获取或设置元素的 CSS 类名
// 获取：返回元素的 CSS 类名（字符串）
// 设置：设置元素的 CSS 类名
element.className;
element.className = "new-class";

// 获取或设置元素的内联样式
// 获取：返回元素的内联样式对象
// 设置：设置元素的内联样式
element.style.backgroundColor = "red";
element.style.display = "block";

// 获取或设置元素的内联 CSS 样式字符串
// 获取：返回元素的内联 CSS 样式字符串
// 设置：设置元素的内联 CSS 样式字符串
element.style.cssText;
element.style.cssText = "background-color: blue; color: white;";
```
## window
alert()、confirm()（确定提示框）、prompt()（提示输入框）
## Table
```
table：insertRow（建立行）、deleteRow（删除行）、rows（这一行）

tr：insertCell（建立列）、deleteCell（删除这一列）、cells（某一列）
```
### 常用命令
```javascript
// 页面加载后执行js
window.onload=function(){
   //（这里写要执行的js）
}

// get访问一个链接
window.location.href='http://www.baidu.com'
// 返回上一个页面
window.history.back()
window.history.go(-1);
// 对文档写入，让字符串显示在页面上
document.write()
// 获取文档中指定ID的元素
document.getElementById()
// 获取文档中指定class的元素
document.getElementsByClassName()
// 获取文档中指定name的元素
document.getElementsByName()
// 当单击时执行一个函数
onclick
// 获取或操作某元素里面的 HTML 内容
innerHTML
// 获取或操作输入框的值（内容）
input.value
// 显示一个警告框
alert()
```
### removeChild
```javascript
stop.onclick = function () {
    var row = table.insertRow(-1);
    var cell = row.insertCell(-1);
    var but = document.createElement('button');
    but.innerHTML = '第' + i + '名' + '：' + inp.value;
    cell.appendChild(but);
    but.className = 'b';
    i++;
    but.onclick = function () {
        this.parentNode.removeChild(this);
    }
}
```
### style
```javascript
// Jframe里面元素获取Jframe外面的元素
window.parent.document.getElementById('a_popup_wrapper').style.display = 'none';
```
### JS选择器
```javascript
getElementById() // 返回指定 ID 的元素
getElementsByName() // 返回带有指定名称的对象集合
getElementsByTagName() // 返回带有指定标签名的对象集合
getElementsByClassName() // 返回文档中所有指定类名的元素
querySelector() // 返回文档中匹配指定css选择器的第一个元素
querySelectorAll() // 返回文档中匹配指定css选择器的所有元素
```
在`class="toc-list" `的元素后面添加元素
```js
const tocElement = document.querySelector('.toc-list');
if (tocElement) {
    const newListItem = document.createElement('li');
    newListItem.className = 'toc-list-item';
    newListItem.textContent = '233';
    tocElement.insertAdjacentElement('afterend', newListItem);
} else {
    console.error('未找到 class="toc" 的元素');
}
```
### 元素显示与隐藏
```js
// 获取元素
const element = document.getElementById('myElement');
// 隐藏元素
element.style.display = 'none';
// 显示元素
element.style.display = 'block'; // 或 'inline', 'flex', 等
```
# LocalStorage、SessionStorage
1. 存储内容大小一般支持**5MB**左右（不同浏览器可能还不一样）
2. 浏览器端通过 `Window.sessionStorage` 和 `Window.localStorage` 属性来实现本地存储机制。
3. 相关API：
    1. `xxxxxStorage.setItem('key', 'value');`
       该方法接受一个键和值作为参数，会把键值对添加到存储中，如果键名存在，则更新其对应的值。
    2. `xxxxxStorage.getItem('person');`
       ​ 该方法接受一个键名作为参数，返回键名对应的值。
    3. `xxxxxStorage.removeItem('key');`
       ​ 该方法接受一个键名作为参数，并把该键名从存储中删除。
    4. `xxxxxStorage.clear()`
       ​ 该方法会清空存储中的所有数据。
4. 备注：
    1. SessionStorage存储的内容会随着浏览器**窗口关闭而消失**。
    2. LocalStorage存储的内容，需要**手动清除才会消失**。
    3. `xxxxxStorage.getItem(xxx)`如果xxx对应的value获取不到，那么getItem的返回值是null。
    4. `JSON.parse(null)`的结果依然是null。
## LocalStorage
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <title>localStorage</title>
</head>

<body>
    <div>
        <button onclick="add()">添加</button>
        <button onclick="del()">删除</button>
        <button onclick="edit()">修改</button>
        <button onclick="read()">查询</button>
        <button onclick="key()">查询键</button>
        <button onclick="clean()">清空</button>
    </div>
    <script type="text/javascript">
        const k = 'person';
        function add() {
            localStorage.setItem(k, JSON.stringify({ name: "张三", age: 18 }));
        }
        function edit() {
            localStorage.setItem(k, JSON.stringify({ name: "李四", age: 19 }));
        }
        function del() {
            localStorage.removeItem(k);
        }
        function read() {
            alert(localStorage.getItem(k));
        }
        function key() {
            alert(localStorage.key(k));
        }
        function clean() {
            localStorage.clear();
        }
    </script>
</body>

</html>
```
## SessionStorage
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <title>sessionStorage</title>
</head>

<body>
    <div>
        <button onclick="add()">添加</button>
        <button onclick="del()">删除</button>
        <button onclick="edit()">修改</button>
        <button onclick="read()">查询</button>
        <button onclick="key()">查询键</button>
        <button onclick="clean()">清空</button>
    </div>
    <script type="text/javascript">
        const k = 'person';
        function add() {
            sessionStorage.setItem(k, JSON.stringify({ name: "张三", age: 18 }));
        }
        function edit() {
            sessionStorage.setItem(k, JSON.stringify({ name: "李四", age: 19 }));
        }
        function del() {
            sessionStorage.removeItem(k);
        }
        function read() {
            alert(sessionStorage.getItem(k));
        }
        function key() {
            alert(sessionStorage.key(k));
        }
        function clean() {
            sessionStorage.clear();
        }
    </script>
</body>

</html>
```
# 字符串操作
## slice
提取字符串的一个部分，并将其作为新字符串返回，而不修改原始字符串。
```js
let s = "Bearer hello world";
console.log(s.slice(7)); // hello world
```
## indexOf
方法返回调用它的 String 对象中第一次出现的指定值的索引，从 fromIndex 处进行搜索。如果未找到该值，则返回 -1。
```js
const beasts = ['ant', 'bison', 'camel', 'duck', 'bison'];

console.log(beasts.indexOf('bison'));
// Expected output: 1

// Start from index 2
console.log(beasts.indexOf('bison', 2));
// Expected output: 4

console.log(beasts.indexOf('giraffe'));
// Expected output: -1
```
# JSON
## JS数据转换
```javascript
var json = '[{"name":"z3"},{"name":"l4"},{"name":"w5"}]';
var obj = JSON.parse(json); // json字符串转json对象
var json_str = JSON.stringify(obj); // json对象转json字符串
console.log(json_str); // [{"name":"z3"},{"name":"l4"},{"name":"w5"}]
```
## js判断字符串是否为JSON格式
```javascript
function isJSON(str) {
    if (typeof str == 'string') {
        try {
            var obj=JSON.parse(str);
            if(typeof obj == 'object' && obj ){
                return true;
            }else{
                return false;
            }

        } catch(e) {
            console.log('error：'+str+'!!!'+e);
            return false;
        }
    }
    console.log('It is not a string!')
}
```
# 数组操作
## push()
向数组末尾添加一个或多个元素，并返回新的数组长度。
```javascript
// 示例 1: 添加一个元素
const fruits = ["apple", "banana"];
const newLength = fruits.push("orange");
console.log(fruits);     // 输出: ["apple", "banana", "orange"]
console.log(newLength);  // 输出: 3

// 示例 2: 添加多个元素
const animals = ["cat", "dog"];
const anotherLength = animals.push("elephant", "lion");
console.log(animals);      // 输出: ["cat", "dog", "elephant", "lion"]
console.log(anotherLength); // 输出: 4
```
## pop() 
方法从数组中删除最后一个元素，并返回该元素的值。此方法**更改数组**的长度。
```js
const plants = ['broccoli', 'cauliflower', 'cabbage', 'kale', 'tomato'];

console.log(plants.pop());
// Expected output: "tomato"

console.log(plants);
// Expected output: Array ["broccoli", "cauliflower", "cabbage", "kale"]

plants.pop();

console.log(plants);
// Expected output: Array ["broccoli", "cauliflower", "cabbage"]
```
## shift() 
方法从数组中删除第一个元素，并返回该元素的值。此方法**更改数组**的长度。
```js
const array1 = [1, 2, 3];

const firstElement = array1.shift();

console.log(array1);
// Expected output: Array [2, 3]

console.log(firstElement);
// Expected output: 1
```
## unshift() 
方法将一个或多个元素添加到数组的开头，并返回该数组的新长度(该方法**修改原有数组**)。
```js
const array1 = [1, 2, 3];

console.log(array1.unshift(4, 5));
// Expected output: 5

console.log(array1);
// Expected output: Array [4, 5, 1, 2, 3]66
```
## splice()
通过删除或替换现有元素或者原地添加新的元素来修改数组，并以数组形式返回被修改/删除的内容。
* 参数：
    * `start`: 开始修改的索引。
    * `deleteCount` (可选): 要删除的元素个数。如果为 <= 0，则不删除元素。
    * `...items` (可选): 要添加到数组中的元素，从 `start` 索引开始插入。
* 返回一个**包含被删除元素的数组**。如果没有删除任何元素，则返回空数组。
```javascript
const myFish = ["angel", "clown", "mandarin", "surgeon"];

// 示例 1: 从索引 2 开始删除 2 个元素
const removed1 = myFish.splice(2, 2);
console.log(myFish);    // 输出: ["angel", "clown"]
console.log(removed1); // 输出: ["mandarin", "surgeon"]

// 示例 2: 从索引 1 开始删除 0 个元素，并插入 "drum"
const removed2 = myFish.splice(1, 0, "drum");
console.log(myFish);    // 输出: ["angel", "drum", "clown"]
console.log(removed2); // 输出: [] (没有删除任何元素)

// 示例 3: 从索引 2 开始删除 1 个元素，并插入 "guitar" 和 "bass"
const removed3 = myFish.splice(2, 1, "guitar", "bass");
console.log(myFish);    // 输出: ["angel", "drum", "guitar", "bass"]
console.log(removed3); // 输出: ["clown"]
```
## sort()
对数组的元素进行原地排序，并返回排序后的数组。默认排序顺序是在将元素转换为字符串(基于**字符串的 Unicode 值**)，然后比较它们的 UTF-16 代码单元值序列时构建的。
```javascript
// 示例 1: 字符串数组排序 (默认)
const names = ["Charlie", "Alice", "Bob"];
names.sort();
console.log(names); // 输出: ["Alice", "Bob", "Charlie"]

// 示例 2: 数字数组排序 (需要自定义比较函数)
const numbers = [10, 5, 8, 1, 15];
numbers.sort((a, b) => a - b); // 升序排序
console.log(numbers); // 输出: [1, 5, 8, 10, 15]

const numbersDescending = [10, 5, 8, 1, 15];
numbersDescending.sort((a, b) => b - a); // 降序排序
console.log(numbersDescending); // 输出: [15, 10, 8, 5, 1]
```
## reserve()
方法将数组中元素的位置颠倒，并返回该数组。数组的第一个元素会变成最后一个，数组的最后一个元素变成第一个。**该方法会改变原数组**。
```js
const array1 = ['one', 'two', 'three'];

const reversed = array1.reverse();
console.log('reversed:', reversed);
// Expected output: "reversed:" Array ["three", "two", "one"]

// Careful: reverse is destructive -- it changes the original array.
console.log('array1:', array1);
// Expected output: "array1:" Array ["three", "two", "one"]
```
## join()
将一个数组（或一个类数组对象的所有元素）连接成一个字符串并返回这个字符串。可以使用一个可选的分隔符字符串来分隔数组元素。
* **不修改**原始数组。
```javascript
const elements = ["Fire", "Air", "Water"];

// 示例 1: 使用默认分隔符 (逗号)
console.log(elements.join()); // 输出: "Fire,Air,Water"

// 示例 2: 使用空字符串作为分隔符
console.log(elements.join("")); // 输出: "FireAirWater"

// 示例 3: 使用连字符作为分隔符
console.log(elements.join("-")); // 输出: "Fire-Air-Water"
```
## split()
 将一个字符串分割成一个子字符串数组，然后将结果作为字符串数组返回。
```javascript
const message = "Hello, world!";

// 示例 1: 使用空格作为分隔符
const words = message.split(" ");
console.log(words); // 输出: ["Hello,", "world!"]

// 示例 2: 使用逗号作为分隔符
const items = "apple,banana,orange";
const fruitsArray = items.split(",");
console.log(fruitsArray); // 输出: ["apple", "banana", "orange"]

// 示例 3: 使用空字符串作为分隔符 (分割成单个字符)
const chars = message.split("");
console.log(chars); // 输出: ["H", "e", "l", "l", "o", ",", " ", "w", "o", "r", "l", "d", "!"]

// 示例 4: 使用限制参数 (限制返回的子字符串数量)
const limitedWords = message.split(" ", 1);
console.log(limitedWords); // 输出: ["Hello,"]
```
## filter() 
方法创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。 (**过滤所有等值为false的元素**)
```js
const words = ['spray', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter((word) => word.length > 6);

console.log(result);
// Expected output: Array ["exuberant", "destruction", "present"]
```
## concat() 
方法用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组。
```js
const array1 = ['a', 'b', 'c'];
const array2 = ['d', 'e', 'f'];
const array3 = array1.concat(array2);

console.log(array3);
// Expected output: Array ["a", "b", "c", "d", "e", "f"]
```
## slice() 
方法返回一个新的数组对象，这一对象是一个由 begin 和 end 决定的原数组的浅拷贝（包括 begin，不包括end）。
**原始数组不会被改变。**
```js
const animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

console.log(animals.slice(2));
// Expected output: Array ["camel", "duck", "elephant"]

console.log(animals.slice(2, 4));
// Expected output: Array ["camel", "duck"]

console.log(animals.slice(1, 5));
// Expected output: Array ["bison", "camel", "duck", "elephant"]

console.log(animals.slice(-2));
// Expected output: Array ["duck", "elephant"]

console.log(animals.slice(2, -1));
// Expected output: Array ["camel", "duck"]

console.log(animals.slice());
// Expected output: Array ["ant", "bison", "camel", "duck", "elephant"]

```
## unique
通过set集合进行数组去重
```js
function unique (arr) {
  return Array.from(new Set(arr))
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
 //[1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {}, {}]
```
## reduce
**`reduce()`** 方法对数组中的每个元素按序执行一个提供的 **reducer** 函数，每一次运行 **reducer** 会将**先前元素的计算结果作为参数传入**，最后将其结果汇总为单个返回值。
**参数**
- [`accumulator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce#accumulator)上一次调用 `callbackFn` 的结果。在第一次调用时，如果指定了 `initialValue` 则为指定的值，否则为 `array[0]` 的值。
- [`currentValue`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce#currentvalue) 当前元素的值。在第一次调用时，如果指定了 `initialValue`，则为 `array[0]` 的值，否则为 `array[1]`。
- [`currentIndex`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce#currentindex) `currentValue` 在数组中的索引位置。在第一次调用时，如果指定了 `initialValue` 则为 `0`，否则为 `1`。
```js
const array1 = [1, 2, 3, 4];

// 0 + 1 + 2 + 3 + 4
const initialValue = 0;
const sumWithInitial = array1.reduce(
  (accumulator, currentValue) => accumulator + currentValue,
  initialValue,
);

console.log(sumWithInitial);
// Expected output: 10
```
## 获得指定区间数组
```js
// 获得指定区间数组，因为数组没有实际的值，所以`_`为undefined，start为起始值+索引就可以得到区间数组
function range({ start, end }) {
    return Array.from({ length: end - start + 1 }, (_, index) => start + index);
}
console.log(range({ start: 101, end: 105 })); // [ 101, 102, 103, 104, 105 ]
```
# 事件循环：微任务与宏任务
## 一、定义与来源

### 1. 宏任务（Macrotasks）
- **定义**：由浏览器或宿主环境触发的任务。
- **常见来源**：
  - `setTimeout`、`setInterval` 的回调。
  - DOM 事件（如点击、滚动）。
  - I/O 操作（如文件读写、网络请求）。
  - UI 渲染（如重绘、布局）。
  - 整体脚本（整个代码块本身也是一个宏任务）。
### 2. 微任务（Microtasks）
- **定义**：由 JavaScript 引擎自身触发的任务。
- **常见来源**：
  - `Promise.then()`、`Promise.catch()`、`Promise.finally()` 的回调。
  - `MutationObserver`（监听 DOM 变化的回调）。
  - `queueMicrotask()` 方法添加的任务。
  - `async/await`（基于 Promise 的语法糖）。
## 二、执行顺序与优先级

### 1. 事件循环流程
1. 执行一个宏任务（如同步代码或 `setTimeout` 回调）。
2. 执行过程中产生的微任务会被加入微任务队列。
3. **宏任务执行完毕后**，立即清空微任务队列（按顺序执行所有微任务）。
4. 更新 UI 渲染（如果需要）。
5. 继续从宏任务队列中取出下一个任务，重复上述流程。
### 2. 优先级
- **微任务优先级高于宏任务**。
- 即使 `setTimeout(func, 0)` 的延迟时间为 0，它也会在当前宏任务结束后等待所有微任务执行完毕才执行。
## 三、队列处理方式
### 1. 宏任务队列
- 每次事件循环处理一个宏任务。
- **示例**：
```javascript
setTimeout(() => console.log(1), 0);
setTimeout(() => console.log(2), 0);
// 输出顺序：1 → 2（每次循环处理一个宏任务）
```
### 2. 微任务队列
- 每次事件循环会 **清空整个微任务队列**。
- **示例**：
```javascript
Promise.resolve().then(() => console.log(1));
Promise.resolve().then(() => console.log(2));
// 输出顺序：1 → 2（微任务队列一次性清空）
```
## 四、执行顺序示例
```javascript
console.log("Start"); // 同步代码（宏任务）

setTimeout(() => {
  console.log("Timeout"); // 宏任务
}, 0);

Promise.resolve().then(() => {
  console.log("Promise"); // 微任务
});
console.log("End"); // 同步代码（宏任务）
// 输出顺序：
// Start → End → Promise → Timeout
```
**解释**：
1. 同步代码（`Start` 和 `End`）优先执行。
2. `setTimeout` 的回调被加入宏任务队列。
3. `Promise` 的回调被加入微任务队列。
4. 所有同步代码执行完毕后，清空微任务队列（输出 `Promise`）。
5. 最后执行宏任务队列中的 `setTimeout`（输出 `Timeout`）。
## 五、Node.js 中的特殊情况
- **`process.nextTick`**：  
  在 Node.js 环境中，`process.nextTick` 是一种特殊的微任务，其优先级 **高于** `Promise` 的微任务。
```javascript
process.nextTick(() => console.log(1)); // 优先级高
Promise.resolve().then(() => console.log(2)); // 优先级低
// 输出顺序：1 → 2
```
## 六、性能影响

### 1. 微任务的潜在问题
如果微任务队列中存在**大量嵌套**的微任务（如递归添加微任务），会导致宏任务和 UI 渲染被阻塞，造成界面卡顿。
```javascript
Promise.resolve().then(() => {
	Promise.resolve().then(() => {
	  // 无限嵌套微任务，阻塞后续宏任务
	});
});
```
### 2. 宏任务的延迟性
宏任务的执行时间受当前任务队列状态影响。
```javascript
// 同步代码阻塞主线程 5 秒
const start = Date.now();
while (Date.now() - start < 5000) {}

setTimeout(() => {
	console.log("Timeout"); // 实际延迟 >5 秒
}, 0);
```
## 七、总结对比
| **特性**     | **宏任务**                  | **微任务**                    |
| ---------- | ------------------------ | -------------------------- |
| **来源**     | 浏览器/宿主环境（如 `setTimeout`） | JavaScript 引擎（如 `Promise`） |
| **执行时机**   | 每个事件循环周期处理一个宏任务          | 当前宏任务结束后立即清空所有微任务          |
| **优先级**    | 较低                       | 高                          |
| **典型场景**   | 延迟操作、UI 渲染、I/O 操作        | Promise 链式调用、DOM 变化监听      |
| **对渲染的影响** | 可能延迟渲染                   | 通常不影响渲染                    |
## 八、应用建议
- **微任务**：用于需要尽快执行的高优先级操作（如状态更新、错误处理）。
- **宏任务**：用于延迟执行或非紧急操作（如动画、轮询）。
- **避免滥用微任务**：防止微任务队列过长导致性能问题。
# 内置对象
## Math
```
random()（随机数，0到1之间）、max()（最大）,min()（最小）
round(x)（四舍五入）、ceil(x)（向上取整，四入五入）、floor(x)（向下取整，四舍五舍）
```
## window
```
eval()、onload()、open()(访问链接或页面)、close()(关闭连接或页面)

setInterval()（设置时间重复执行）、clearInterval()（暂停执行）

setTimeout()（延迟）、clearTimeout()（清除延迟）
```
### setInterval()、clearInterval()
- `setInterval()`: 是 `Window/Worker` 接口提供的方法，用于**按固定间隔重复执行函数/代码片段**，返回**唯一ID**（用于 `clearInterval()` 清除），由 `WindowOrWorkerGlobalScope` 混合接口定义。
- clearInterval(): 用于取消先前通过调用 setInterval() 设置的重复操作。需要将 setInterval() 返回的间隔 ID 传递给 clearInterval()，以停止相应的定时器。
```html
<button id="open">open</button>
<button id="close">close</button>
<script type="text/javascript">
    let open = document.getElementById('open');
    let close = document.getElementById('close');
    let clearId;
    // 开始执行任务
    open.onclick = function () {
        clearId = setInterval(() => console.log('execute'), 100);
    }
    // 清除间歇任务
    close.onclick = function () {
        console.log('clear');
        clearInterval(clearId);
    }
</script>
```
#### 注意事项
##### 不可控的执行顺序
`setInterval` 的任务会无条件添加到队列，即使前一个任务还在运行。这与事件循环的机制无关，仅由定时器驱动。
**任务堆积**：  
如果 `setInterval(func, delay)` 的 `func` 执行时间超过 `delay`，下一次任务会被直接添加到队列，导致多个 `func` 同时运行。例如：
```js
setInterval(() => {
  // 模拟耗时 500ms 的任务
  const start = Date.now();
  while (Date.now() - start < 500) {}
  console.log("执行任务");
}, 100); // 设置 100ms 间隔
```
输出结果可能是：
```
执行任务（第1次）
执行任务（第2次） // 在第1次未完成时，第2次已开始
...
```
##### 推荐
通常情况下，很少真正使用间歇调用，因为后一个间歇调用可能在前一个间歇调用结束之前调用。因此，我们通常会使用超时调用来模拟间歇调用
```js
let num = 0;
const max = 10;
function incrementNumber() {
    num++;
    if (num < max) {
        setTimeout(incrementNumber, 500);
        console.log(num);
    } else {
        console.log("Done");
    }
}
setTimeout(incrementNumber, 500);
```
### setTimeout()、clearTimeout()
```js
function log(message) {
    console.log(message + '--' + new Date().toLocaleString());
}
log("开始");
setTimeout(() => log("3 秒后执行"), 3000);
/*
开始--2025/4/3 17:23:54
3 秒后执行--2025/4/3 17:23:57
*/
```
### open()、close()
Window接口的open()方法将指定的资源加载到新的或现有的具有指定名称的浏览上下文(Window、`<iframe>`、tab)中。
如果名称不存在，则在新选项卡或新窗口中打开一个新的浏览上下文，并将指定的资源加载到其中。
```js
// 在新窗口中打开一个 URL
window.open("https://www.example.com");
// 在名为 "myWindow" 的新窗口中打开一个 URL
window.open("https://www.example.com", "myWindow");
// 在名为 "myWindow" 的新窗口中打开一个 URL，并设置窗口的特性（宽度、高度等）
window.open("https://www.example.com", "myWindow", "width=600,height=400");

// +----------------------------------------------------------------------  
// |  使用 `window.opene` 与新窗口通信
// +----------------------------------------------------------------------
// 在父窗口中
const newWindow = window.open("child.html");
newWindow.onload = () => {
  newWindow.postMessage("Hello from parent!", "*");
};
// 在 child.html 中
window.addEventListener("message", (event) => {
  console.log("Message from parent:", event.data);
  event.source.postMessage("Hello from child!", event.origin);
});
```
# JQuery实现原理
选择器实现
Element实现
each核心实现
**扩展**
对核心扩展
**典型方法**
html()、css()
```javascript
<meta charset="UTF-8">
</meta>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
<script type="text/javascript">
    window.onload = function () {
        // 添加自定义方法
        $.fn.css = function (css, value) {
            // 传一个以上的值，修改值
            if (arguments.length > 1) {
                return this.each(function (i, dom) {
                    dom.style[css] = value;
                })
            }
            // 传一个字符串是查询css样式
            if (typeof (css) == "string" || css instanceof String) {
                console.log(this[0], style[css]);
                // 返回当前值的属性
                return this[0].style[css];
            }
            // 对象型修改
            return this.each(function (i, dom) {
                for (var v in css) {
                    dom.style[v] = css[v];
                }
            })
        }
        var doms = $(".d2");
        doms.css({ color: "red", background: "blue" }).css("text-align", "center");
    }
</script>
<body>
<div class="d2">test</div>
</body>
```

```javascript
(function () {
    // 全局变量设置为当前方法
    var mq = window.Mq = window.$ = function (selector) {
        return new Element(findDom(selector));
    }
    var Elements = function (doms) {
        for (var i = 0, d; d = doms[i++];) {
            this.push(d);
        }
    }
    // 将对象原型暴露出去
    mq.fn = Elements.prototype = {
        length: 0,
        push: Array.prototype.push, // 拿到Array数组的方法给它
        each: function (callback) {
            for (var i = 0, d; d = this[i++];) {
                callback.call(d, i, d);
            }
            return this; // 返回这个函数「链式编程」
        }
    }
    function findDom(selector) {
        if (selector.charAt(0) == "#") {
            // 从索引为1开始截取到最后
            return [document.getElementById(selector.substr(1))];
        }
        if (selector.charAt(0) == ".") {
            var doms = [];
            var className = selector.substr(1);
            // 拿到所有的标签
            var ads = document.getElementsByTagName("*");
            for (var i = 0, d; d = ads[i++];) {
                if (d.className == className) {
                    doms.push(d);
                }
            }
            return doms;
        }
        // 拿到指定标签元素
        return document.getElementsByTagName(selector);
    }
})
```
input text的所有事件
```javascript
onBlur // 当 text field失去聚焦的时候执行一些代码
onChange // 当 text field失去聚焦并且它的值发生变动的时候执行一些代码
onClick // 当用户在 text field中用鼠标左键点击时执行一些代码
onFocus // 当 text field获得聚焦的时候执行一些代码
onKeyDown // 在 text field中有键按住的时候执行一些代码
onKeyUp // 当 text field 中按键释放则执行一些代码
onSelect // 当 text field里当前选中的内容发生变化时执行一些代码
onSelectStart // 当 text field中一些文字被选中则执行一些代码
```
