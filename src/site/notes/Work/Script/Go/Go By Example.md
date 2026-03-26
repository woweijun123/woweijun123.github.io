---
{"dg-publish":true,"permalink":"/Work/Script/Go/Go By Example/","title":"Go By Example","tags":["flashcards"],"noteIcon":"","created":"2026-03-25T23:52:47.000+08:00","updated":"2026-03-26T10:47:36.114+08:00","dg-note-properties":{"title":"Go By Example","tags":["flashcards"],"links":"[Go by Example 中文版](https://gobyexample-cn.github.io/)"}}
---

# Hello World
```go
package main

import (
	"fmt"
)

func main() {
	//var a = "init"
	fmt.Println("hello world")
}
```
输出
```
hello world
```
# 值
```go
package main

import "fmt"

func main() {
	// 1. 运算符
	fmt.Println("go" + "lang")

	fmt.Println("1+1 =", 1+1)
	fmt.Println("7.0/3.0 =", 7.0/3.0)

	fmt.Println(true && false)
	fmt.Println(true || false)
	fmt.Println(!true)

	// 2. 位运算
	var a uint = 60 /* 60 = 0011 1100 */
	var b uint = 13 /* 13 = 0000 1101 */
	var c uint = 0
	c = a & b /* 12 = 0000 1100 */
	fmt.Printf("第一行 - cc 的值为 %d\n", c)
	c = a | b /* 61 = 0011 1101 */
	fmt.Printf("第二行 - cc 的值为 %d\n", c)
	c = a ^ b /* 49 = 0011 0001 */
	fmt.Printf("第三行 - cc 的值为 %d\n", c)
	c = a << 2 /* 240 = 1111 0000 */
	fmt.Printf("第四行 - cc 的值为 %d\n", c)
	c = a >> 2 /* 15 = 0000 1111 */
	fmt.Printf("第五行 - cc 的值为 %d\n", c)

	// 3. 赋值运算符
	var aa int = 21
	var cc int

	cc = aa
	fmt.Printf("第 1 行 - =  运算符实例，cc 值为 = %d\n", c)
	cc += aa
	fmt.Printf("第 2 行 - += 运算符实例，cc 值为 = %d\n", c)
	cc -= aa
	fmt.Printf("第 3 行 - -= 运算符实例，cc 值为 = %d\n", c)
	cc *= aa
	fmt.Printf("第 4 行 - *= 运算符实例，cc 值为 = %d\n", c)
	cc /= aa
	fmt.Printf("第 5 行 - /= 运算符实例，cc 值为 = %d\n", c)

	cc = 200
	cc <<= 2
	fmt.Printf("第 6行  - <<= 运算符实例，cc 值为 = %d\n", c)
	cc >>= 2
	fmt.Printf("第 7 行 - >>= 运算符实例，cc 值为 = %d\n", c)
	cc &= 2
	fmt.Printf("第 8 行 - &= 运算符实例，cc 值为 = %d\n", c)
	cc ^= 2
	fmt.Printf("第 9 行 - ^= 运算符实例，cc 值为 = %d\n", c)
	cc |= 2
	fmt.Printf("第 10 行 - |= 运算符实例，cc 值为 = %d\n", c)
}
```
输出
```
golang
1+1 = 2
7.0/3.0 = 2.3333333333333335
false
true
false
第一行 - cc 的值为 12
第二行 - cc 的值为 61
第三行 - cc 的值为 49
第四行 - cc 的值为 240
第五行 - cc 的值为 15
第 1 行 - =  运算符实例，cc 值为 = 15
第 2 行 - += 运算符实例，cc 值为 = 15
第 3 行 - -= 运算符实例，cc 值为 = 15
第 4 行 - *= 运算符实例，cc 值为 = 15
第 5 行 - /= 运算符实例，cc 值为 = 15
第 6行  - <<= 运算符实例，cc 值为 = 15
第 7 行 - >>= 运算符实例，cc 值为 = 15
第 8 行 - &= 运算符实例，cc 值为 = 15
第 9 行 - ^= 运算符实例，cc 值为 = 15
第 10 行 - |= 运算符实例，cc 值为 = 15
```
# 变量
```go
package main

import "fmt"

// 全局变量声明后不使用不会报错
var global = true

func main() {
	// 局部变量声明后不使用会报错
	var a = "initial"
	fmt.Println(a)
	// 同一行多个赋值
	var b, c int = 1, 2
	fmt.Println(b, c)

	var d = true
	fmt.Println(d)

	var e int
	fmt.Println(e)

	f := "short"
	fmt.Println(f)
}
```
# 常量
常量中的数据类型只可以是? <?:?> 布尔型、数字型（整数型、浮点型和复数）和字符串型。
<!--SR:!2026-05-15,78,288-->
常量不能用什么语法声明? <?:?> :=
<!--SR:!2026-04-20,64,288-->
常量可以用函数计算表达式的值，但要求是？<?:?> 函数必须是`len(), cap(), unsafe.Sizeof()`这样的内置函数，否则编译不过
<!--SR:!2026-07-02,106,288-->
<?e?>
常量的枚举写法？
iota 特殊常量的写法、特性、隐式重复？
<?l?>
```go
package main

import (
	"fmt"
	"math"
	"unsafe"
)

// 1. 声明全局常量
const s string = "constant"

func main() {
	// +----------------------------------------------------------------------
	// | 1.常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。
	// | 2.常量不能用 := 语法声明
	// +----------------------------------------------------------------------
	fmt.Println("1.GlobalConst:", s)

	// 2. 声明局部常量、常量表达式
	// 💡注意：数值常量在未指定类型前是“无类型”的（Typeless），具有高精度
	const n = 500000000
	// 3e20 是科学计数法。Go 的常量在编译期可以处理极大数值，不会溢出
	const d = 3e20 / n
	fmt.Println("2.ConstantExpr:", d)

	// 3. 显式类型转换
	// 数值常量在使用时可以根据上下文自动转换或手动转换
	fmt.Println("3.ExplicitConvert:", int64(d))

	// 4. 在函数中使用常量
	// math.Sin 接收 float64 参数，常量 n 会根据需要自动转换
	fmt.Println("4.MathFunction:", math.Sin(n))

	// 5. 常量用作枚举
	const (
		Fail    = "失败"
		Success = "成功"
	)
	fmt.Println("5.Enum:", Fail, Success)

	// 6. 常量可以用len(), cap(), unsafe.Sizeof()函数计算表达式的值。常量表达式中，函数必须是内置函数，否则编译不过
	const (
		Default = "abc"
		Len     = len(Default)
		// 获取 Default 变量的内存大小（以字节为单位）
		SizeOf = unsafe.Sizeof(Default)
	)
	fmt.Println("6.expression:", Default, Len, SizeOf)

	// +----------------------------------------------------------------------
	// |iota，特殊常量，可以认为是一个可以被编译器修改的常量。
	// +----------------------------------------------------------------------
	const (
		A = iota // 0：第一行
		B        // 1：第二行，隐式重复上行表达式
		C = "hi" // 2：第三行，虽然没用 iota，但计数继续
		D = iota // 3：第四行，恢复计数
	)
	/* iota 特性
	   自动重置：每当遇到 const 关键字，iota 就会被重置为 0。
	   行索引器：在同一个 const 代码块中，iota 代表当前行的索引（从 0 开始计数）。
	   逐行自增：代码块中每新增一行常量声明，iota 计数便自动加 1，无论该行是否使用了 iota。
	*/
	fmt.Println("7.iota:", A, B, C, D)

	const (
		i = 1 << iota // iota=0: 1左移0位 = 1
		j = 3 << iota // iota=1: 3左移1位 = 6
		k             // iota=2: 隐式重复 (3 << 2) = 12
		l             // iota=3: 隐式重复 (3 << 3) = 24
	)
	// 输出结果: 8.iota: 1 6 12 24
	fmt.Println("8.iota:", i, j, k, l)
}
```
输出
```
1.GlobalConst: constant
2.ConstantExpr: 6e+11
3.ExplicitConvert: 600000000000
4.MathFunction: -0.28470407323754404
5.Enum: 失败 成功
6.expression: abc 3 16
7.iota: 0 1 hi 3
8.iota: 1 6 12 24
```
<!--SR:!2026-04-19,63,288-->
<?e?>
# For 循环
<?e?>
类 While 模式?
经典三段式?
无限循环与 break?
循环控制：continue?
goto 的使用频率非常低，尽量避免使用?
<?l?>
```go
package main

import "fmt"

func main() {
	// 1. 类 While 模式
	// 只有一个条件的循环，类似于其他语言中的 while 循环
	i := 1
	for i <= 3 {
		fmt.Println("1.ConditionLoop:", i)
		i = i + 1
	}

	// 2. 经典三段式
	// 初始化; 条件判断; 后置操作
	for j := 7; j <= 9; j++ {
		fmt.Println("2.ClassicLoop:", j)
	}

	// 3. 无限循环与 break
	// 不带条件的 for 即为死循环，常配合 break 使用来跳出
	for {
		fmt.Println("3.InfiniteLoop: loop once")
		break
	}

	// 4. 循环控制：continue
	// 配合 if 使用，跳过本次循环的剩余部分，进入下一次迭代
	for n := 0; n <= 5; n++ {
		if n%2 == 0 {
			// 如果是偶数，则跳过后续的打印步骤
			continue
		}
		fmt.Println("4.ContinueLoop(Odd):", n)
	}

	// +----------------------------------------------------------------------
	// | goto 的使用频率非常低，尽量避免使用
	// +----------------------------------------------------------------------
	/*
	   精简总结：
	   1. 标签跳转：直接移动执行流到同函数的指定标签。
	   2. 场景应用：常用于简化深层循环退出或统一资源清理。
	   3. 结构风险：易导致逻辑混乱，调试困难，应优先使用 break/return。
	   4. 语法约束：禁止跳过变量声明以防止初始化失效。
	*/
	i2 := 0
LOOP: // 定义标签
	if i2 < 5 {
		fmt.Println(i2)
		i2++
		goto LOOP // 1. 构成循环：跳转回标签处
	}
	if i2 == 5 {
		goto EXIT // 2. 条件转移：跳转到结束
	}
EXIT: // 3. 集中出口
	fmt.Println("Done")
}
```
输出
```
1.ConditionLoop: 1
1.ConditionLoop: 2
1.ConditionLoop: 3
2.ClassicLoop: 7
2.ClassicLoop: 8
2.ClassicLoop: 9
3.InfiniteLoop: loop once
4.ContinueLoop(Odd): 1
4.ContinueLoop(Odd): 3
4.ContinueLoop(Odd): 5
5.goto: 0
5.goto: 1
5.goto: 2
5.goto: 3
5.goto: 4
5.goto:Done
```
<!--SR:!2026-05-14,77,288-->
<?e?>
# If/Else 分支
```go
package main

import "fmt"

func main() {
    // 1. 基础 if-else
    // 💡注意：Go 的条件表达式不需要圆括号 ()，但大括号 {} 是强制必须写的
    if 7%2 == 0 {
        fmt.Println("1.Result:7 is even")
    } else {
        fmt.Println("1.Result:7 is odd")
    }

    // 2. 只有 if 的情况
    if 8%4 == 0 {
        fmt.Println("2.Result:8 is divisible by 4")
    }

    // 3. if 中的条件初始化语句
    // 在条件判断前可以声明变量，该变量的作用域仅限于这个 if/else 块
    if num := 9; num < 0 {
        fmt.Println("3.Result:", num, "is negative")
    } else if num < 10 {
        fmt.Println("3.Result:", num, "has 1 digit")
    } else {
        fmt.Println("3.Result:", num, "has multiple digits")
    }
}
```
输出
```
1.Result:7 is odd
2.Result:8 is divisible by 4
3.Result: 9 has 1 digit
```
# Switch 分支结构
<?e?>
1. 基础 switch?
2. 一个 case 匹配多个值?
3. 无表达式 switch (类似于 if-else 链)?
4. 类型 switch (Type Switch)?
5. fallthrough?
<?l?>
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 1. 基础 switch
	// 在 Go 中，每个 case 后面默认自带 break，匹配成功执行后会自动退出 switch
	i := 2
	fmt.Print("1.Basic: write ", i, " as ")
	switch i {
	case 1:
		fmt.Println("one")
	case 2:
		fmt.Println("two")
	case 3:
		fmt.Println("three")
	}

	// 2. 一个 case 匹配多个值
	// 使用逗号分隔多个表达式。此外还展示了 default 分支的处理
	switch time.Now().Weekday() {
	case time.Saturday, time.Sunday:
		fmt.Println("2.MultiCase: It's the weekend")
	default:
		fmt.Println("2.MultiCase: It's a weekday")
	}

	// 3. 无表达式 switch (类似于 if-else 链)
	// 当 switch 后面没有跟变量时，它等同于 switch true。这种写法比长串 if-else 更整洁
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("3.Expressionless: It's before noon")
	default:
		fmt.Println("3.Expressionless: It's after noon")
	}

	// 4. 类型 switch (Type Switch)
	// 用于在运行时判断一个接口值 (interface{}) 的具体类型
	whatAmI := func(i interface{}) {
		switch t := i.(type) {
		case bool:
			fmt.Println("4.TypeSwitch: I'm a bool")
		case int:
			fmt.Println("4.TypeSwitch: I'm an int")
		default:
			fmt.Printf("4.TypeSwitch: Don't know type %T\n", t)
		}
	}
	whatAmI(true)
	whatAmI(1)
	whatAmI("hey")

	// 5. 使用 fallthrough 会强制执行后面的 case 语句，fallthrough 不会判断下一条 case 的表达式结果是否为 true。
	/*
	   核心规则：
	   1. fallthrough 必须位于 case 块的最后一行。
	   2. 它会强制执行紧邻的下一个 case 代码块，而不再判断该 case 的条件。
	   3. 它不会跨越没有 fallthrough 的 case。
	*/
	switch {
	case false:
		fmt.Println("5.1")
		fallthrough
	case true: // 1. 匹配成功，进入执行
		fmt.Println("5.2case 条件语句为 true")
		fallthrough // 2. 强制穿透：无视下一行 case 的条件直接执行
	case false: // 3. 虽然是 false，但受上层 fallthrough 影响仍会执行
		fmt.Println("5.3case 条件语句为 false")
		fallthrough // 4. 继续强制穿透
	case true:
		fmt.Println("5.4case 条件语句为 true")
		// 5. 无 fallthrough，逻辑在此处中断，跳出 switch
	case false:
		fmt.Println("5.5")
		fallthrough
	default:
		fmt.Println("5.6")
	}
}
```
输出
```
1.Basic: write 2 as two
2.MultiCase: It's a weekday
3.Expressionless: It's after noon
4.TypeSwitch: I'm a bool
4.TypeSwitch: I'm an int
4.TypeSwitch: Don't know type string
5.2case 条件语句为 true
5.3case 条件语句为 false
5.4case 条件语句为 true
```
<!--SR:!2026-05-11,75,288-->
<?e?>
# 数组
<?e?>
1. 声明数组?
2. 设置与获取?
3. 获取长度?
4. 一行声明并初始化?
5. 多维数组?
6. 一行声明并初始化多维数组?
7. 可以使用 什么符号 代替数组的长度，编译器会根据元素个数自行推断数组的长度?
8. 数组可以直接比较要求是?
9. 数组 什么 是类型的一部分?
<?l?>
```go
package main

import "fmt"

// 多维数组可以作为函数参数，但要注意数组是值类型，传递大数组会有性能开销：
func processMatrix(matrix [3][3]int) [3][3]int {
	// 处理矩阵...
	matrix[0][0] = 100
	return matrix
}

// 更好的方式是使用指针或切片
func processMatrixPtr(matrix *[3][3]int) {
	matrix[0][0] = 100
}

func main() {
	// 1. 声明数组
	// var 变量名 [长度]类型。数组长度是类型的一部分
	// 默认情况下，数组会自动初始化为零值（int 默认为 0）
	var a [5]int
	fmt.Println("1.EmptyArray:", a)

	// 2. 设置与获取
	// 使用 索引 (Index) 来操作，索引从 0 开始
	a[4] = 100
	fmt.Println("2.SetElement:", a)
	fmt.Println("3.GetElement:", a[4])

	// 3. 获取长度
	// 内置函数 len() 返回数组的长度
	fmt.Println("4.Length:", len(a))

	// 4. 一行声明并初始化
	b := [5]int{1, 2, 3, 4, 5}
	fmt.Println("5.DeclareAndInit:", b)

	// 5. 多维数组
	// 数组可以嵌套，例如 [2][3]int 表示 2 行 3 列
	var twoD [2][3]int
	for i := 0; i < 2; i++ {
		for j := 0; j < 3; j++ {
			// 填充数据：索引 i 为外层，j 为内层
			twoD[i][j] = i + j
		}
	}
	fmt.Println("6.2DArray: ", twoD)

	// 6. 初始化多维数组
	multiArr := [3][4]int{
		{0, 1, 2, 3},   /*  第一行索引为 0 */
		{4, 5, 6, 7},   /*  第二行索引为 1 */
		{8, 9, 10, 11}, /* 第三行索引为 2 */
	}
	fmt.Println("6.MultiArray:", multiArr)

	// 7. 可以使用 ... 代替数组的长度，编译器会根据元素个数自行推断数组的长度
	balance := [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
	fmt.Println("7.ArrayWithoutLength:", balance)

	// 8. 数组比较
	// 创建两个相同的二维数组
	aa := [2][2]int{{1, 2}, {3, 4}}
	bb := [2][2]int{{1, 2}, {3, 4}}
	cc := [2][2]int{{1, 2}, {3, 5}}

	// 数组可以直接比较（只有当维度完全相同时）
	fmt.Println("8.aa == bb:", aa == bb) // 输出: true
	fmt.Println("8.aa == cc:", aa == cc) // 输出: false

	// 💡注意：不同维度的数组不能比较
	// d := [2][3]int{{1, 2, 3}, {4, 5, 6}}
	// fmt.Println(a == d)  // 编译错误：类型不匹配

	// 9. 数组长度是类型的一部分
	// 这两个是不同的类型！
	var a1 [2][3]int
	var b1 [3][2]int

	// 以下代码会编译错误
	// a1 = b1  // 错误：类型不匹配

	fmt.Printf("9.a1 的类型: %T\n", a1) // [2][3]int
	fmt.Printf("9.b1 的类型: %T\n", b1) // [3][2]int
}
```
输出
```
1.EmptyArray: [0 0 0 0 0]
2.SetElement: [0 0 0 0 100]
3.GetElement: 100
4.Length: 5
5.DeclareAndInit: [1 2 3 4 5]
6.2DArray:  [[0 1 2] [1 2 3]]
6.MultiArray: [[0 1 2 3] [4 5 6 7] [8 9 10 11]]
7.ArrayWithoutLength: [1000 2 3.4 7 50]
8.aa == bb: true
8.aa == cc: false
9.a1 的类型: [2][3]int
9.b1 的类型: [3][2]int
```
<!--SR:!2026-06-04,100,308-->
<?e?>
# 切片
<?e?>
1. 使用 make 申明?
2. 字面量声明?
3. 赋值与获取?
4. 长度 (Length)?
5. 追加元素 (Append)?
6. 复制切片的注意事项是?
7. 切片操作 (Slicing)?
8. 多维切片?
9. make 的不同参数含义?
10. 长度(len) vs 容量(cap)?
11. 索引访问限制?
12. 切片删除操作(保留顺序)?
<?l?>
```go
package main

import "fmt"

func main() {
	/*
	   精简总结：
	   1. 本质：切片是数组的包装，提供动态扩容能力。
	   2. 扩容：append 是核心，容量不足时会成倍（或按比例）增加。
	   3. 结构：包含指针 (Pointer)、长度 (Length)、容量 (Capacity)。
	   4. 内存：切片之间可能共享底层数组，修改需注意副作用。
	*/

	// 1. 使用 make 申明
	// 使用 make(类型, 初始长度)。初始元素会被赋为该类型的零值
	s := make([]string, 3)
	fmt.Println("1.Make:", s)

	// 2. 字面量声明
	t := []string{"g", "h", "i"}
	fmt.Println("2.Literal:", t)

	// 3. 赋值与获取
	s[0] = "a"
	s[1] = "b"
	s[2] = "c"
	fmt.Println("3.SetAndGet:", s, "Index2:", s[2])

	// 4. 长度 (Length)
	fmt.Println("4.Length:", len(s))

	// 5. 追加元素 (Append)
	// append 会返回一个新的切片，因为如果底层数组容量不足，它会分配新内存
	s = append(s, "d")
	s = append(s, "e", "f")
	fmt.Println("5.AfterAppend:", s)

	// 6. 复制切片 (Copy)
	/* 必须先创建一个长度足够的切片作为目标
	   copy函数工作原理:
	       1.copy(dst, src) 会复制 min(len(dst), len(src)) 个元素
	       2.当 len(t2) = 0 时，min(0, 1) = 0，所以不会复制任何元素
	*/
	// 错误示例
	c := make([]string, 0)
	copy(c, s)
	fmt.Println("6.1CopySlice:", c)
	// 正确的复制方式
	c = make([]string, len(s))
	copy(c, s)
	fmt.Println("6.2CopySlice:", c)

	// 7. 切片操作 (Slicing)
	// 语法: s[low:high] -> 包含索引 low，但不包含 high (左闭右开)
	l := s[2:5] // 获取索引 2, 3, 4 的元素
	fmt.Println("7.1Slice[2:5]:", l)
	l = s[:5] // 从开头到索引 4
	fmt.Println("7.2Slice[:5]:", l)
	l = s[2:] // 从索引 2 到结尾
	fmt.Println("7.3Slice[2:]:", l)

	// 8. 多维切片
	// 8.1 独立切片，与数组不同，多维切片的内层切片长度可以是不固定的（锯齿状）
	twoD := make([][]int, 3)
	for i := 0; i < 3; i++ {
		innerLen := i + 1
		twoD[i] = make([]int, innerLen)
		for j := 0; j < innerLen; j++ {
			twoD[i][j] = i + j
		}
	}
	fmt.Println("8.12D_JaggedSlice: ", twoD)

	// 8.2 共享内存多维切片（高性能连续内存）
	h, w := 3, 4
	// 1. 一次性分配连续的底层数组
	pixels := make([]int, h*w)
	// 2. 初始化外层切片
	matrix := make([][]int, h)
	for i := range matrix {
		// 3. 将连续内存划片赋值，所有行共用同一个底层数组
		matrix[i] = pixels[i*w : (i+1)*w]
	}
	fmt.Println("8.22D_SharedSlice: ", matrix)

	// 9. make 的不同参数含义
	// 注意: len=0, cap=3 的切片，长度为0，所以不能用索引访问元素
	ss := make([]int, 0, 3)
	// ss[0] = 1        // ❌ 错误：Panic! 索引越界，因为长度为 0
	ss = append(ss, 1) // ✅ 正确：使用 append 增加长度并赋值
	fmt.Println("9.SimpleAppend:", ss)

	// 10. 长度(len) vs 容量(cap)
	s1 := make([]int, 3)    // len=3, cap=3, 元素自动填充零值 [0, 0, 0]
	s2 := make([]int, 0, 3) // len=0, cap=3, 为空但预留了 3 个位置
	fmt.Printf("10.1s1: len=%d cap=%d %v\n", len(s1), cap(s1), s1)
	fmt.Printf("10.2s2: len=%d cap=%d %v\n", len(s2), cap(s2), s2)

	// 11. 索引访问限制
	// s1 可以直接通过索引修改，因为它已经有了 3 个位置
	s1[0] = 100
	fmt.Println("11.1s1[0] =", s1[0])
	// 自动扩容机制
	s2 = append(s2, 1, 2)
	fmt.Printf("11.2AfterAppend: len=%d cap=%d %v\n", len(s2), cap(s2), s2)
	// 当 append 超过当前容量 (cap=3) 时，Go 会自动分配更大的底层数组
	s2 = append(s2, 3, 4)
	fmt.Printf("11.3AfterGrowth: len=%d cap=%d %v\n", len(s2), cap(s2), s2)

	// 12. 切片删除操作（保留顺序）
	// 利用 append 的特性：将“索引 1 之前”和“索引 2 之后”的部分拼接
	list := []string{"a", "b", "c"}
	// list[:1] 是 {"a"}，list[2:]... 是 "c"
	list = append(list[:1], list[2:]...)
	fmt.Println("12.AfterDelete:", list)
}
```
输出
```
1.Make: [  ]
2.Literal: [g h i]
3.SetAndGet: [a b c] Index2: c
4.Length: 3
5.AfterAppend: [a b c d e f]
6.1CopySlice: []
6.2CopySlice: [a b c d e f]
7.1Slice[2:5]: [c d e]
7.2Slice[:5]: [a b c d e]
7.3Slice[2:]: [c d e f]
8.2D_JaggedSlice:  [[0] [1 2] [2 3 4]]
9.SimpleAppend: [1]
10.1s1: len=3 cap=3 [0 0 0]
10.2s2: len=0 cap=3 []
11.1s1[0] = 100
11.2AfterAppend: len=2 cap=3 [1 2]
11.3AfterGrowth: len=4 cap=6 [1 2 3 4]
12.AfterDelete: [a c]
```
<!--SR:!2026-04-12,47,268-->
<?e?>
# Map
```go
package main

import "fmt"

func main() {
	// 1. 创建映射
	// 使用 make(map[键类型]值类型)
	m := make(map[string]int)

	// 2. 设置键值对
	// 使用 名字[键] = 值 语法
	m["k1"] = 7
	m["k2"] = 13

	// 3. 打印整个映射
	// 💡注意：Map 的打印输出顺序是随机的
	fmt.Println("1.MapInstance:", m)

	// 4. 获取特定键的值
	v1 := m["k1"]
	fmt.Println("2.GetValue:", v1)

	// 5. 获取长度
	// 返回映射中键值对的数量
	fmt.Println("3.Length:", len(m))

	// 6. 删除键值对
	// 使用内置函数 delete(映射名, 键名)
	delete(m, "k2")
	fmt.Println("4.AfterDelete:", m)

	// 7. 检查键是否存在 (Comma OK 模式)
	// 第一个返回值是该键对应的值，第二个返回值 (prs) 是布尔值，表示键是否存在
	// 即使键不存在，第一个返回值也会是类型的零值（如 0）
	_, prs := m["k2"]
	fmt.Println("5.IsPresent:", prs)

	// 8. 字面量声明并初始化
	n := map[string]int{"foo": 1, "bar": 2}
	fmt.Println("6.LiteralDecl:", n)
}
```
输出
```
1.MapInstance: map[k1:7 k2:13]
2.GetValue: 7
3.Length: 2
4.AfterDelete: map[k1:7]
5.IsPresent: false
6.LiteralDecl: map[bar:2 foo:1]
```
# Range 遍历
range遍历字符串返回的是什么?<?:?>迭代的是 Unicode 码点 (Rune) 而非字节
<!--SR:!2026-05-07,72,289-->
```go
package main

import "fmt"

func main() {
	// 1. 遍历切片（求和）
	// range 返回两个值：索引和元素值。这里使用下划线 _ 忽略索引
	nums := []int{2, 3, 4}
	sum := 0
	for _, num := range nums {
		sum += num
	}
	fmt.Println("1.SliceSum:", sum)

	// 2. 遍历切片（获取索引）
	// 如果需要索引，将其分配给变量（如 i）
	for i, num := range nums {
		if num == 3 {
			fmt.Println("2.FindIndex:", i)
		}
	}

	// 3. 遍历映射 (Map)
	// 对于 Map，range 返回键 (Key) 和值 (Value)
	kvs := map[string]string{"a": "apple", "b": "banana"}
	for k, v := range kvs {
		fmt.Printf("3.MapKV: %s -> %s\n", k, v)
	}

	// 4. 仅遍历映射的键
	// 如果只需要键，可以只写一个变量
	for k := range kvs {
		fmt.Println("4.MapKey:", k)
	}

	// 5. 遍历字符串
	// 💡注意：range 遍历字符串时，迭代的是 Unicode 码点 (Rune) 而非字节
	// 第一个值是字符的起始字节索引，第二个值是字符的 Unicode 编码 (int32)
	for i, c := range "go" {
		fmt.Println("5.StringIter:", i, c)
	}
}
```
输出
```
1.SliceSum: 9
2.FindIndex: 1
3.MapKV: a -> apple
3.MapKV: b -> banana
4.MapKey: b
4.MapKey: a
5.StringIter: 0 103
5.StringIter: 1 111
```
# 函数
```go
package main

import "fmt"

// 1. 基础函数声明
// 格式：func 函数名(参数名 参数类型) 返回类型
func plus(a int, b int) int {
	// Go 需要显式 return，不会自动返回最后一个表达式的值
	return a + b
}

// 2. 参数类型合并
// 如果多个连续参数类型相同，可以只在最后一个参数后写类型
func plusPlus(a, b, c int) int {
	return a + b + c
}

func main() {
	// 3. 调用函数
	// 使用 name(args) 形式调用
	res := plus(1, 2)
	fmt.Println("1.BasicPlus: 1+2 =", res)

	res = plusPlus(1, 2, 3)
	fmt.Println("2.MultiPlus: 1+2+3 =", res)
}
```
输出
```
1.BasicPlus: 1+2 = 3
2.MultiPlus: 1+2+3 = 6
```
# 多返回值
```go
package main

import "fmt"

func multipleReturn(a, b, c int) (int, int) {
	return a + b, c
}

func main() {
	i, i2 := multipleReturn(2, 2, 3)
	fmt.Println("1.", i, i2)

	i, _ = multipleReturn(2, 2, 3)
	fmt.Println("2.", i)

	_, i2 = multipleReturn(2, 2, 3)
	fmt.Println("3.", i2)
}
```
输出
```
1. 4 3
2. 4
3. 3
```
# 变参函数
<?e?>
申明时 ... 分别在php和go函数的哪边？调用时 ... 分别在php和go的哪边？
<?l?>
左边，php左、go右

```go
package main

import "fmt"

// 1. 定义变参函数
// 在参数类型前加上省略号 ...
// 在函数体内部，nums 的类型实际上是 []int（即切片）
func sum(nums ...int) {
	fmt.Print("1.InputParams:", nums, " ")
	total := 0
	// 可以像遍历切片一样遍历变参
	for _, num := range nums {
		total += num
	}
	fmt.Println("2.ResultSum:", total)
}

func main() {
	// 2. 传入不同数量的参数
	sum(1, 2)
	sum(1, 2, 3)

	// 3. 将切片作为变参传入
	// 如果你已经有一个切片，不能直接传给变参函数
	// 必须在切片名后加上 ... 展开切片，将其拆散为独立的参数
	nums := []int{1, 2, 3, 4}
	sum(nums...)
}
```
输出
```
1.InputParams:[1 2] 2.ResultSum: 3
1.InputParams:[1 2 3] 2.ResultSum: 6
1.InputParams:[1 2 3 4] 2.ResultSum: 10
```
<!--SR:!2026-06-14,106,290-->
<?e?>
# 闭包
```go
package main

import "fmt"

// 1. 定义一个返回函数的函数
// intSeq 的返回类型是一个函数：func() int
func intSeq() func() int {
	i := 0 // 这是一个局部变量，将被闭包“捕获”

	// 2. 返回一个匿名函数
	// 这个匿名函数直接引用了外部的变量 i
	return func() int {
		i++ // 修改被捕获的变量
		return i
	}
}

func main() {
	// 3. 创建闭包实例
	// nextInt 是一个包含了变量 i 引用和匿名函数代码的闭包
	nextInt := intSeq()

	// 4. 调用闭包
	// 每次调用，i 的值都会在内存中保留并递增
	fmt.Println("1.FirstInstance:", nextInt()) // 输出 1
	fmt.Println("1.FirstInstance:", nextInt()) // 输出 2
	fmt.Println("1.FirstInstance:", nextInt()) // 输出 3

	// 5. 闭包的状态隔离
	// 重新调用 intSeq 会创建一个全新的变量 i
	newInts := intSeq()
	fmt.Println("2.NewInstance:", newInts()) // 输出 1（从头开始）

	// 6. 闭包做为变量传递调用
	tmp := func(i int) int {
		return i + 1
	}
	f2 := make([]func(i int) int, 0, 5)
	for i := 0; i < 5; i++ {
		f2 = append(f2, tmp)
	}
	for k, v := range f2 {
		fmt.Println("3.ClosureAsVariable:", v(k))
	}
}
```
输出
```
1.FirstInstance: 1
1.FirstInstance: 2
1.FirstInstance: 3
2.NewInstance: 1
3.ClosureAsVariable: 1
3.ClosureAsVariable: 2
3.ClosureAsVariable: 3
3.ClosureAsVariable: 4
3.ClosureAsVariable: 5
```
# 递归
匿名函数定义递归时的注意事项?<?:?>必须先声明变量名，再进行赋值
<!--SR:!2026-06-08,103,309-->
```go
package main

import "fmt"

// 1. 经典递归：计算阶乘 (Factorial)
// n! = n * (n-1) * ... * 1
func fact(n int) int {
	// 递归基准情况 (Base Case)：必须有退出条件，否则会无限循环导致栈溢出
	if n == 0 {
		return 1
	}
	// 递归步骤：调用自身并缩小问题规模
	return n * fact(n-1)
}

func main() {
	// 计算 7 的阶乘 (7 * 6 * 5 * 4 * 3 * 2 * 1)
	fmt.Println("1.Factorial(7):", fact(7))

	// 2. 闭包递归：计算斐波那契数列 (Fibonacci)
	// 在函数内部定义递归时，必须先声明变量名，再进行赋值
	var fib func(n int) int

	fib = func(n int) int {
		// 基准情况：F(0)=0, F(1)=1
		if n < 2 {
			return n
		}
		// 递归调用：F(n) = F(n-1) + F(n-2)
		return fib(n-1) + fib(n-2)
	}

	// 计算斐波那契数列索引为7的数
	// (0, 1, 1, 2, 3, 5, 8, 13...)
	// (0, 1, 2, 3, 4, 5, 6, 7...)
	fmt.Println("2.Fibonacci(7):", fib(7))
}
```
输出
```
1.Factorial(7): 5040
2.Fibonacci(7): 13
```
# 指针
打印 `&i` 会输出什么样的结果? <?:?> 会输出类似 `0xc000012088` **十六进制**的**内存地址**
<!--SR:!2026-05-16,79,288-->
Go 语言中仍有指针的存在，但注意事项是?<?:?>不允许进行指针运算
<!--SR:!2026-06-03,99,309-->
<?e?>
1. 使用指针变量存储实际变量的地址?
2. 多级指针?
3. unsafe?
<?l?>
```go
package main

import (
	"fmt"
	"unsafe"
)

// 1. 值传递 (Pass by Value)
// ival 是原变量的一个副本。在函数内修改 ival 不会影响外部原始变量。
func zeroval(ival int) {
	ival = 0
}

// 2. 指针传递 (Pass by Reference/Pointer)
// iptr 的类型是 *int，表示它指向一个整数的内存地址。
// 通过 *iptr (解引用) 可以修改该地址存放的实际值。
func zeroptr(iptr *int) {
	*iptr = 0
}

func main() {
	/*
	   精简总结：
	   1. & (取址)：获取变量在内存中的地址。
	   2. * (取值)：访问指针指向的内存空间（解引用）。
	   3. 多级指针：二级指针 ptr2 存储的是一级指针 ptr1 的地址，寻址链路为：ptr2 -> ptr1 -> base。「三级同理」
	*/
	// 基础申明
	i := 1
	fmt.Println("1.InitialValue:", i)

	// 调用 zeroval，传入的是 i 的值 (1)
	zeroval(i)
	fmt.Println("2.AfterZeroval:", i) // 依然是 1

	// 3. 取地址符 &
	// &i 会获取变量 i 在内存中的地址，并传给 zeroptr
	zeroptr(&i)
	fmt.Println("3.AfterZeroptr:", i) // 变为 0

	// 4. 打印指针
	// 直接打印 &i 会输出类似 0xc000012088 的十六进制内存地址
	fmt.Println("4.PointerAddress:", &i)

	// 5. 指针
	var a int = 20 /* 声明实际变量 */
	var ip *int    /* 声明指针变量 */
	ip = &a        /* 使用指针变量存储实际变量的地址 */
	fmt.Printf("5.1a变量的地址是: %x\n", &a)
	/* 指针变量的存储地址 */
	fmt.Printf("5.2ip变量储存的指针地址: %x\n", ip)
	/* 使用指针访问值 */
	fmt.Printf("5.3*ip变量的值: %d\n", *ip)

	// 6. 指针数组「节省内存空间」
	const MAX int = 3
	var ptr [MAX]*int
	a1 := []int{10, 100, 200}

	for i := 0; i < MAX; i++ {
		ptr[i] = &a1[i] /* 整数地址赋值给指针数组 */
	}

	for i := 0; i < MAX; i++ {
		fmt.Printf("6.a[%d] = %d\n", i, *ptr[i])
	}

	// 7. 多级指针
	base := 3000
	ptr1 := &base // 一级指针：存储 base 的地址
	ptr2 := &ptr1 // 二级指针：存储 ptr1 的地址
	ptr3 := &ptr2 // 三级指针：存储 ptr2 的地址

	// 访问结果
	fmt.Printf("7.1base = %d\n", base)      // 3000
	fmt.Printf("7.2*ptr1 = %d\n", *ptr1)    // 3000（解引用一次）
	fmt.Printf("7.3**ptr2 = %d\n", **ptr2)  // 3000（解引用两次）
	fmt.Printf("7.4**ptr3 = %d\n", ***ptr3) // 3000（解引用两次）

	// +----------------------------------------------------------------------
	// | Go 语言中仍有指针的存在，但并不允许进行指针运算
	// +----------------------------------------------------------------------
	/*
	   内存安全与垃圾回收（GC）
	   在 C/C++ 中，你可以通过 p++ 移动指针来访问数组的下一个元素。如果计算错误，指针可能会指向非法的内存区域，导致程序崩溃（Buffer Overflow）。
	   更重要的是，Go 拥有自动垃圾回收机制。如果允许指针随意运算，GC 就很难追踪每一个指针究竟指向哪里，这会极大增加内存管理的复杂度和不确定性。

	   简洁性
	   Go 提倡“一种任务只有一种做法”。在 C 语言中，访问数组元素既可以用 a[i] 也可以用 *(a + i)。Go 强制要求使用索引访问，提高了代码的可读性和一致性。
	*/

	/* 禁止的操作：指针运算
	   在 Go 中，你不能对指针进行加减运算。
	   arr := [3]int{1, 2, 3}
	   p := &arr[0]
	   // 编译错误：invalid operation: p + 1 (mismatched types *int and int)
	   // p = p + 1
	*/

	/* 特殊情况：unsafe 包
	   虽然标准 Go 不允许指针运算，但 Go 提供了一个名为 unsafe 的包。通过 unsafe.Pointer，你可以将指针转换为 uintptr 类型（一个足以容纳地址的整数），
	   进行运算后再转回指针。

	   警告：正如其名，这通常是不安全的。除非在极致的性能优化或底层系统调用中，否则不建议使用。
	*/
	// 1. 定义一个包含两个整数的数组
	// 在内存中，这两个 int 是连续排列的
	arr := [2]int{10, 20}
	// 2. 获取数组第一个元素的通用指针
	// unsafe.Pointer 是一种特殊指针类型，它可以指向任意变量，也可以转换回 uintptr
	p := unsafe.Pointer(&arr[0])
	// 3. 强制进行指针运算：移动到下一个 int 的位置
	// 核心步骤解析：
	// - uintptr(p): 将通用指针转为数值（uintptr），因为只有数值才能进行加法运算
	// - unsafe.Sizeof(arr[0]): 计算一个 int 占用多少字节（通常是 8 字节）
	// - uintptr(p) + offset: 在内存地址数值上往后移动 8 个字节
	// - unsafe.Pointer(...): 将运算后的数值地址重新转回通用指针
	nextP := unsafe.Pointer(uintptr(p) + unsafe.Sizeof(arr[0]))
	// 4. 读取运算后的指针对应的值
	// *(*int)(nextP):
	//   - (*int)(nextP) 将通用指针强转为具体的整型指针
	//   - * 取值操作，访问该内存地址上的数据
	fmt.Printf("8.%d\n", *(*int)(nextP)) // 输出 20
}
```
输出
```
1.InitialValue: 1
2.AfterZeroval: 1
3.AfterZeroptr: 0
4.PointerAddress: 0x14000092020
5.1a变量的地址是: 14000092028
5.2ip变量储存的指针地址: 14000092028
5.3*ip变量的值: 20
6.a[0] = 10
6.a[1] = 100
6.a[2] = 200
7.1base = 3000
7.2*ptr1 = 3000
7.3**ptr2 = 3000
7.4**ptr3 = 3000
8.20
```
<!--SR:!2026-06-03,90,289-->
<?e?>
# 字符串和rune类型
<?e?>
1. 打印原始字符串?
2. 字符串切片（按字节切取）?
3. len(s) 返回的是什么长度?
4. 循环打印每个字节的十六进制值?
5. 获取字符数量?
6. range 遍历字符串：它会自动按字符（Rune）迭代，并返回字节起始索引?
7. 手动解码字符?
8. 将字节切片解码为单个字符（Rune）?
<?l?>
```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
	// Go 语言中的字符串由字节组成，而“字符”概念由 **rune** 表示，其本质是一个代表 Unicode 码点的整数。
	// 泰语词汇 "สวัสดี" (Sawasdee)
	const s = "สวัสดี"

	// 1. 打印原始字符串
	fmt.Println("1.String:", s)

	// 2. 字符串切片（按字节切取）
	// 💡注意：泰语和中文一样是多字节字符，截取不当会导致乱码
	fmt.Println("2.ByteSlice:", s[0:3])

	// 3. len(s) 返回的是字节长度（Byte Length）
	fmt.Println("3.ByteLen:", len(s))

	// 4. 循环打印每个字节的十六进制值
	fmt.Print("4.HexBytes: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
	fmt.Println()

	// 5. 获取字符数量（Rune Count），这才是视觉上的字数
	fmt.Println("5.RuneCount:", utf8.RuneCountInString(s))

	// 6. range 遍历字符串：它会自动按字符（Rune）迭代，并返回字节起始索引
	fmt.Println("6.RangeIter:")
	for k, v := range s {
		fmt.Printf("6.%#U starts at %d\n", v, k)
	}

	// 7. 手动使用 DecodeRuneInString 解码字符
	fmt.Println("7.DecodeRuneInString:")
	for i, w := 0, 0; i < len(s); i += w {
		// 函数只解码字符串中的第一个 rune（Unicode 字符）
		// 虽然传入的是整个字符串 "你好"，但 DecodeRuneInString 只处理第一个字符
		// 返回字符的值及其占用的字节宽度
		runeValue, width := utf8.DecodeRuneInString(s[i:])
		fmt.Printf("7.Value:%v Width:%d Char:%#U\n", runeValue, width, runeValue)
		w = width
		// 检查字符
		examineRune(runeValue)
	}

	// 8. 将字节切片解码为单个字符（Rune）
	r, _ := utf8.DecodeRune([]byte(s[0:3]))
	if r == 'ส' {
		fmt.Printf("8.MatchRune: %#U\n", r)
	}
}

// examineRune 演示如何通过 rune 类型进行字符比较
func examineRune(r rune) {
	if r == 't' {
		fmt.Println("7.-> found tee")
	} else if r == 'ส' {
		fmt.Println("7.-> found so sua (Thai letter)")
	}
}
```
输出
```
1.String: สวัสดี
2.ByteSlice: ส
3.ByteLen: 18
4.HexBytes: e0 b8 aa e0 b8 a7 e0 b8 b1 e0 b8 aa e0 b8 94 e0 b8 b5 
5.RuneCount: 6
6.RangeIter:
6.U+0E2A 'ส' starts at 0
6.U+0E27 'ว' starts at 3
6.U+0E31 'ั' starts at 6
6.U+0E2A 'ส' starts at 9
6.U+0E14 'ด' starts at 12
6.U+0E35 'ี' starts at 15
7.DecodeRuneInString:
7.Value:3626 Width:3 Char:U+0E2A 'ส'
7.-> found so sua (Thai letter)
7.Value:3623 Width:3 Char:U+0E27 'ว'
7.Value:3633 Width:3 Char:U+0E31 'ั'
7.Value:3626 Width:3 Char:U+0E2A 'ส'
7.-> found so sua (Thai letter)
7.Value:3604 Width:3 Char:U+0E14 'ด'
7.Value:3637 Width:3 Char:U+0E35 'ี'
8.MatchRune: U+0E2A 'ส'
```
<!--SR:!2026-04-27,52,249-->
<?e?>
# 结构体
<?e?>
1. 按顺序初始化?
2. 按字段名初始化?
3. 零值初始化?
4. 指针初始化方式?
5. 结构体指针的自动解引用?
6. 匿名结构体?
7. 公私有字段怎么区别? 包外使用?
<?l?>
```go
package main

import (
	"example/example2"
	"fmt"
)

// 定义结构体:使用 type 关键字定义一个新地复合类型 person
type person struct {
	name    string // 私有字段
	age     int    // 私有字段
	Address string // 公有字段
}

// 构造函数惯用法
// Go 习惯提供一个以 New 开头的函数来封装复杂的初始化逻辑，并返回指针
func newPerson(name string) *person {
	p := person{name: name}
	p.age = 42
	// 💡 知识点：Go 编译器会自动进行逃逸分析（Escape Analysis）
	// 返回局部变量的指针是完全安全的
	return &p
}

func main() {
	// 1. 基础字面量初始化
	// 方式 A：按顺序赋值（不推荐，字段变动容易出错）
	fmt.Println("1.1按顺序初始化:", person{"Bob", 20, "sc"})
	// 方式 B：指定字段名赋值（推荐，清晰且健壮）
	fmt.Println("1.2按字段名初始化:", person{name: "Alice", age: 30})
	// 方式 C：部分赋值，未指定的字段将保持该类型的零值 (0, "", nil)
	fmt.Println("1.3零值初始化:", person{name: "Fred"})

	// 2. 指针初始化方式
	// 方式 A：使用内置 new 关键字，分配内存并返回指针，字段均为零值
	p2 := new(person)
	p2.name = "Loren"
	p2.age = 18
	fmt.Println("2.1使用 new 关键字:", p2)
	// 方式 B：对字面量取地址 &，可同时指定字段值更灵活，这是获取结构体指针最快捷的方式
	fmt.Println("2.2字面量取地址:", &person{name: "Ann", age: 40})
	// 方式 C：调用自定义构造函数
	fmt.Println("2.3调用构造函数:", newPerson("Jon"))

	// 3. 访问与修改
	s := person{name: "Sean", age: 50}
	fmt.Println("3.访问字段:", s.name)

	// 4. 结构体指针的自动解引用
	sp := &s
	// 💡 知识点：Go 语法糖：sp.age 实际被编译器转换为 (*sp).age。即使 sp 是指针，也可以直接用 . 访问字段
	// 无需像 C 语言那样使用 sp->age，也不必写 (*sp).age
	fmt.Println("4.1指针自动解引用访问:", sp.age)
	// 修改操作会直接作用于底层原始数据
	sp.age = 51
	fmt.Println("4.2修改后的原对象值:", s.age)

	// 5. 匿名结构体
	var st = struct{ Name string }{Name: "张三"}
	fmt.Println("5.匿名结构体 ", st.Name) // 张三

	// 6. 包外使用
	// ✅ 可修改公有字段
	pp := example2.PrivatePerson{Address: "cd"}
	fmt.Println("6.可修改公有字段:", pp.Address)
	// ❌ 编译错误，无法访问私有字段
	//pp.age = 20
}
```
输出
```
1.1按顺序初始化: {Bob 20 sc}
1.2按字段名初始化: {Alice 30 }
1.3零值初始化: {Fred 0 }
2.1使用 new 关键字: &{Loren 18 }
2.2字面量取地址: &{Ann 40 }
2.3调用构造函数: &{Jon 42 }
3.访问字段: Sean
4.1指针自动解引用访问: 50
4.2修改后的原对象值: 51
5.匿名结构体  张三
6.可修改公有字段: cd
```
<!--SR:!2026-06-09,104,309-->
<?e?>
# 方法
<?e?>
1. 公私有方法区别?
2. 非结构体类型的方法?
3. 什么类型不能添加方法?
4. 方法调用自动转换: 值调用指针方法? 指针调用值方法?
5. 函数调用: 区别方法调用?
6. 方法表达式: 实例直接调用? 类型名调用?
7. 分配接口时的规则?
<?l?>
```go
package main

import (
	"fmt"
	"math"
)

// =============================================================
// 权限规则说明：
// 1. 公有方法 (Exported): 首字母大写，可以被其他包 (package) 访问。
// 2. 私有方法 (Unexported): 首字母小写，仅限当前包内部调用。
// =============================================================
// --- 1. 结构体与方法定义 ---
type rect2 struct {
	width, height int
}

// Area 是公有方法，其他包可以通过 rect2 实例访问
// 指针接收者 (Pointer Receiver)
// 语法：func (接收者变量名 *类型) 方法名() 返回类型
// 优点：可以修改结构体内部的值，且避免了大对象的拷贝开销
func (r *rect2) Area() int {
	return r.width * r.height
}

// perim 是私有方法，外部包无法直接调用 (常用于内部逻辑封装)
// 值接收者 (Value Receiver)
// 语法：func (接收者变量名 类型) 方法名() 返回类型
// 特点：方法内部拿到的是原结构体的一个副本，修改 r 不会影响外部原值
func (r rect2) perim() int {
	return 2*r.width + 2*r.height
}

// --- 2. 非结构体类型的方法 ---
// Go 不允许直接给 int/float 加方法，但可以给“自定义别名类型”加方法
type TmpFloat float64

func (f TmpFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

/*
// 错误示例：尝试为基本类型 int 添加方法（非法）
func (i int) Square() int {  // 编译错误
    return i * i
}

// 错误示例：尝试为字符串类型添加方法（非法）
func (s string) Reverse() string {  // 编译错误
    return reverse(s)
}
*/

// --- 3. 方法与函数的对比 (指针重定向) ---
type Vertex struct {
	X, Y float64
}

// 方法：接收者既可是值也可是指针「反之亦然：以值为接收者的方法被调用时，接收者既能为值又能为指针」
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

// 普通函数：参数类型必须严格匹配「反之亦然：接受一个值作为参数的函数必须接受一个指定类型的值」
func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

// --- 4. 接口契约 ---
type Abser interface {
	Abs() float64
}

// 为 Vertex 实现接口（💡注意：这里使用的是指针接收者）
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	// --- 自动转换 (Syntactic Sugar) ---
	r := rect2{width: 10, height: 5}
	// 值调用指针方法：Go 自动转换为 (&r).Area()
	fmt.Println("1.值调指针方法:", r.Area())
	rp := &r
	// 指针调用值方法：Go 语法糖 自动转换为 (*rp).perim()
	fmt.Println("2.指针调值方法:", rp.perim())

	// --- Method Expressions (方法表达式) ---
	// 方式 A: 实例直接调用
	fmt.Println("3.实例直接调用:", rect2{1, 1}.perim())
	// 方式 B: 类型名调用（需显式传递实例作为第一个参数）
	fmt.Println("4.类型名调用:", rect2.perim(rect2{2, 2}))

	// --- 指针重定向实战 ---
	// 接收者值类型
	v := Vertex{3, 4}
	// 对于方法：v 是值，Scale 需要指针，Go 自动取地址 &v
	v.Scale(2)
	// 对于函数：必须手动传入地址，否则编译报错
	ScaleFunc(&v, 10)

	// 接收者指针类型
	p := &Vertex{4, 3}
	p.Scale(3) // 指针直接匹配
	ScaleFunc(p, 8)
	fmt.Printf("5.最终结果: v=%v, p=%v\n", v, p)

	// --- 接口赋值与方法集 (Method Sets) ---
	/* 💡 核心笔记：
	   	   虽然在【调用方法】时，Go 会自动处理指针/值的转换（语法糖）；
	   	   但在【分配接口】时，规则非常严格：
	              - 如果方法是 (v T) 接收，则 T 和 *T 都实现了接口。
	              - 如果方法是 (v *T) 接收，则只有 *T 实现了接口。
	*/
	var a Abser
	f := TmpFloat(-math.Sqrt2)
	v2 := Vertex{3, 4}
	// 场景 A：
	a = f  // 值类型 TmpFloat 实现了 Abs (值接收者)，匹配成功
	a = &f // 引用类型 TmpFloat 实现了 Abs (值接收者)，匹配成功
	// 场景 B：
	a = &v2 // 指针类型 *Vertex 实现了 Abs (指针接收者)，匹配成功
	// a = v2 // 值类型 ❌ 错误：Vertex 未实现 Abser (Abs 方法需要指针接收者)
	fmt.Printf("6.最终结果: v=%v, a_result=%.2f\n", v, a.Abs())
}
```
输出
```
1.值调指针方法: 50
2.指针调值方法: 30
3.实例直接调用: 4
4.类型名调用: 8
5.最终结果: v={60 80}, p=&{96 72}
6.最终结果: v={60 80}, a_result=5.00
```
<!--SR:!2026-04-14,49,269-->
<?e?>
# 接口
实现了接口的全部方法，就**自动实现**了 geometry 接口
<?e?>
1. 多态应用
2. 空接口
3. 类型断言: 直接断言? 安全断言?
4. 类型选择
<?l?>
```go
package main

import (
	"fmt"
	"math"
)

// 1. 定义标准接口
// 接口规定：任何类型实现了area()和perim()方法，它就被视为一个 geometry。
type geometry interface {
	area() float64
	perim() float64
}

// 2. 结构体实现
type rect struct {
	width, height float64
}
type circle struct {
	radius float64
}

// Go 的隐式实现：实现了接口的全部方法，就自动实现了 geometry 接口
func (r rect) area() float64 {
	return r.width * r.height
}
func (r rect) perim() float64 {
	return 2*r.width + 2*r.height
}
func (c circle) area() float64 {
	return math.Pi * c.radius * c.radius
}
func (c circle) perim() float64 {
	return 2 * math.Pi * c.radius
}

// 3. 多态应用 (Polymorphism)
// 接收 geometry 接口作为参数，实现了对不同形状的通用处理
func measure(g geometry) {
	fmt.Printf("1. 实例数据: %v\n", g)
	fmt.Printf("2. 面积: %.2f\n", g.area())
	fmt.Printf("3. 周长: %.2f\n", g.perim())
}

// 4. 空接口 (Empty Interface / any)
// interface{} 可以接收任何类型，是 Go 实现泛化处理的基础
func printValue(val interface{}) {
	fmt.Printf("4. 值: %v, 动态类型: %T\n", val, val)
}

// 5. 接口组合 (Interface Embedding)
// 接口可以嵌套，子接口自动继承父接口的所有方法要求
type Parent interface {
	getName() string
}
type Child interface {
	Parent
}
type ins struct {
	name string
}

func (i ins) getName() string {
	return i.name
}

func main() {
	// --- 接口基础用法 ---
	r := rect{width: 3, height: 4}
	var c geometry = circle{radius: 5}

	measure(r)
	measure(c)

	printValue(42)
	printValue("Go")

	// --- 6. 类型断言 (Type Assertion) ---
	// 从接口中安全地提取其底层具体值
	var i interface{} = "hello"

	// 方式 A：直接断言（不匹配会 panic）
	str := i.(string)
	fmt.Println("5. 直接断言:", str)

	// 方式 B：安全断言（ok-idiom 模式）
	var i2 interface{} = 42
	if val, ok := i2.(string); ok {
		fmt.Println("String:", val)
	} else {
		fmt.Println("6. 安全断言: 转换失败，不是 string 类型")
	}

	// --- 7. 类型选择 (Type Switch) ---
	// 根据接口变量的具体类型执行不同分支逻辑
	typeCheck := func(v interface{}) {
		switch t := v.(type) {
		case int:
			fmt.Printf("7. 类型为 int: %d\n", t)
		case float64:
			fmt.Printf("7. 类型为 float64: %f\n", t)
		case string:
			fmt.Printf("7. 类型为 string: %s\n", t)
		default:
			fmt.Println("7. 未知类型")
		}
	}
	typeCheck(10)
	typeCheck("Golang")

	// --- 8. 接口组合实战 ---
	var child Child = ins{name: "z3"}
	fmt.Println("8. 组合接口调用:", child.getName())

	// --- 9. 动态性观察 ---
	// 接口变量实际上由 (Dynamic Type, Dynamic Value) 组成
	var i3 interface{} = 3.14
	fmt.Printf("9. 动态类型: %T, 动态值: %v\n", i3, i3)
}
```
输出
```
1.多态应用:
1.1实例数据: {3 4}
1.2面积: 12.00
1.3周长: 14.00
1.1实例数据: {5}
1.2面积: 78.54
1.3周长: 31.42
2.空接口:
2.值: 42, 动态类型: int
2.值: Go, 动态类型: string
3.1直接断言: hello
3.2安全断言: 转换失败，不是 string 类型
4.类型为 int: 10
4.类型为 string: Golang
5.组合接口调用: z3
6.动态类型: float64, 动态值: 3.14
```
<!--SR:!2026-05-12,70,270-->
<?e?>
# Embedding
<?e?>
1. 字段提升 (Field Promotion)
2. 方法提升 (Method Promotion)
3. 接口实现
<?l?>
```go
package main

import "fmt"

// 定义基础结构体
type base struct {
	num int
}

func (b base) describe() string {
	return fmt.Sprintf("base with num=%v", b.num)
}

// 定义容器结构体并“内嵌” base
type container struct {
	base // 这是一个匿名字段（内嵌），没有显示的名字
	str  string
}

func main() {
	// 初始化内嵌结构体
	// 必须显式地为内嵌的结构体类型赋初值
	co := container{
		base: base{
			num: 1,
		},
		str: "some name",
	}

	// 1. 字段提升 (Field Promotion)
	// 我们可以直接通过 co.num 访问 base 的字段，就像它是 container 自己的字段一样
	fmt.Printf("1.1PromotedField: co={num: %v, str: %v}\n", co.num, co.str)
	// 也可以使用完整的路径来访问
	fmt.Println("1.2FullRoute:", co.base.num)

	// 2. 方法提升 (Method Promotion)
	// 因为 container 内嵌了 base，所以 container 也自动拥有了 base 的所有方法
	fmt.Println("2.PromotedMethod:", co.describe())

	// 3. 接口实现
	// 由于方法得到了提升，container 现在也自动实现了所有 base 实现过的接口
	type describer interface {
		describe() string
	}
	var d describer = co
	fmt.Println("3.InterfaceUsage:", d.describe())
}

```
输出
```
1.1PromotedField: co={num: 1, str: some name}
1.2FullRoute: 1
2.PromotedMethod: base with num=1
3.InterfaceUsage: base with num=1
```
<!--SR:!2026-06-01,86,270-->
<?e?>
# 泛型
<?e?>
1. 泛型中：函数、结构体、方法
2. 实例化泛型结构体?
3. 联合约束应用?
4. 自定义接口约束应用? 类型约束中符号`~`的含义?
5. 复杂约束调用?
<?l?>
## Go 指针赋值 vs 属性修改
泛型方法的**指针地址替换**操作`lst.tail = &element[T]{val: v}`类似[[Work/Frontend/JS/JS basic#浅拷贝、深拷贝\|JS basic#浅拷贝、深拷贝]]的直接赋值`copy.b = [3]`
### 1. 属性修改 (Field Assignment)
- **代码**: `p.next = n`
- **逻辑**: 修改指针所指向的结构体==1;;内部成员==。
- **JS类比**: `obj.next = newObj` (修改对象属性)
- **影响**: 所有指向该结构体的指针都能看到此变化。
### 2. 引用替换 (Pointer Reassignment)
- **代码**: `p = n`
- **逻辑**: 改变指针变量本身存储的==1;;地址==。
- **JS类比**: `p = newObj` (重新赋值)
- **影响**: 仅改变当前指针的指向，不影响原对象。
### 3. 链表 Push 经典套路
1. `tail.next = n`: 把新节点链在旧尾巴后面。
2. `tail = n`: 把尾巴标签移到新节点上。
```go
package main

import (
	"fmt"
)

// --- 泛型函数与类型参数 ---
// [K comparable, V any] 是类型参数列表
// K comparable: 约束键必须支持可比较运算 (==, !=)，这是 map 键的必要条件
// V any: 约束值可以是任何类型 (any 是 interface{} 的别名)
func MapKeys[K comparable, V any](m map[K]V) []K {
	r := make([]K, 0, len(m))
	for k := range m {
		r = append(r, k)
	}
	return r
}

// --- 泛型结构体 ---
// List[T] 可以存储任何指定类型 T 的元素
type List[T any] struct {
	head, tail *element[T]
}
type element[T any] struct {
	// 内部节点 element 同样需要类型参数传递
	next *element[T]
	val  T
}

// --- 泛型方法 ---
// 接收者必须包含类型参数 [T]
func (lst *List[T]) Push(v T) {
	n := &element[T]{val: v}
	if lst.head == nil {
		lst.head = n
	} else {
		lst.tail.next = n // 因head、tail指向同一地址，故修改tail的next时，head的next也一并会修改
	}
	lst.tail = n // 替换tail指针指向最新变量地址
}
func (lst *List[T]) GetAll() []T {
	var elems []T
	for e := lst.head; e != nil; e = e.next {
		elems = append(elems, e.val)
	}
	return elems
}

// --- 类型约束 (Type Constraints) ---
// 使用 | 符号定义联合约束 (Union Constraints)
type Number interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64 |
	~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
	~float32 | ~float64
	// 💡 提示：使用 ~ 前缀表示支持该类型的衍生类型（如 type MyInt int）
}

// 定义需要特定方法的接口约束
type Stringer interface {
	String() string
}

// 约束方法参数类型为Number
func Add[T Number](a, b T) T {
	return a + b
}

// 约束方法参数类型为Stringer
func PrintString[T Stringer](value T) string {
	return value.String()
}

// 实现符合 Stringer 约束的结构体
type Person struct {
	Name string
	Age  int
}

// Person 实现接口 Stringer
func (p Person) String() string {
	return fmt.Sprintf("%s (%d years old)", p.Name, p.Age)
}

// --- 复合约束 ---
// NumericStringer 要求 T 类型既是Number，又实现 String() 方法
type NumericStringer interface {
	Number
	String() string
}

// 实现符合 NumericStringer 约束的自定义类型
type MyFloat float64

// 实现 String() 方法
func (mf MyFloat) String() string {
	return fmt.Sprintf("%.2f", float64(mf))
}
func PrintNumericValue[T NumericStringer](v T) string {
	return fmt.Sprintf("Value: %s, Doubled: %v\n", v.String(), Add(v, v))
}

func main() {
	// 1. 类型推导 (Inference)
	m := map[int]string{1: "Apple", 2: "Banana"}
	// 自动推导出 K 为 int, V 为 string
	fmt.Println("1.泛型函数调用:", MapKeys(m))

	// 也可以显式指定类型参数（虽然通常没必要）
	_ = MapKeys[int, string](m)

	// 2. 实例化泛型结构体
	// 结构体声明时必须显式指定具体类型 [int]
	lst := List[int]{}
	lst.Push(10)
	lst.Push(20)
	fmt.Println("2.泛型链表数据:", lst.GetAll())

	// 3. 联合约束应用
	fmt.Println("3.1整数相加:", Add(10, 20))
	fmt.Println("3.2浮点相加:", Add(3.14, 2.71))

	// 4. 自定义接口约束应用
	person := Person{Name: "Alice", Age: 25}
	fmt.Println("4.自定义接口约束应用:", PrintString(person))

	// 5. 复杂约束调用
	var val MyFloat = 123.456
	fmt.Println("5.复杂约束调用:", PrintNumericValue(val))
}

```
输出
```
1.泛型函数调用: [1 2]
2.泛型链表数据: [10 20]
3.1整数相加: 30
3.2浮点相加: 5.85
4.自定义接口约束应用: Alice (25 years old)
5.复杂约束调用: Value: 123.46, Doubled: 246.91
```
<!--SR:!2026-05-21,69,250-->
<?e?>
# 错误处理
<?e?>
错误处理
这些代码是怎么写的?
<?l?>
```go
package main

import (
	"errors"
	"fmt"
)

// 基础错误处理
// 核心原则：错误是值。函数习惯上将 error 作为最后一个返回值
func general(num int) (int, error) {
	if num == 0 {
		// errors.New 用于创建静态的简单错误字符串
		return -1, errors.New("不能为0")
	}
	return num, nil
}

// 自定义错误类型
// error 是包含 Error() string 方法的接口，可通过结构体携带更多上下文(如 Code)
type argsError struct {
	Code int
	Msg  string
}

// 实现 error 接口的唯一方法
func (e *argsError) Error() string {
	return fmt.Sprintf("Error Code: %d, Message: %s", e.Code, e.Msg)
}

// 使用自定义错误
func custom(num int) (int, error) {
	if num == 0 {
		// 返回自定义结构体的指针，它隐式实现了 error 接口
		return -1, &argsError{Code: 500, Msg: "can't equal 0"}
	}
	return num, nil
}

// New返回一个错误，格式化为给定的文本，每次调用New都会返回一个不同的错误值，即使文本是相同的。
var ErrNotFound = errors.New("not found")

// 使用错误变量 ErrNotFound
func findItem(id int) error {
	return fmt.Errorf("val: %v error: %w", id, ErrNotFound)
}

type MyError struct {
	Code int
	Msg  string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("Code: %d, Msg: %s", e.Code, e.Msg)
}

func getError() error {
	return &MyError{Code: 404, Msg: "Not Found"}
}

func main() {
	// 1. 标准错误检查流程
	// Go 推荐“尽早返回”：先处理错误逻辑，正常逻辑放在函数主体
	for _, v := range []int{0, 1} {
		if i, err := general(v); err != nil {
			fmt.Println("1.1 捕获常规错误:", err)
		} else {
			fmt.Println("1.2 成功返回值:", i)
		}
	}

	// 2. 处理自定义错误
	_, e := custom(0)
	fmt.Println("2. 打印完整错误信息:", e)
	// 3. 错误断言 (Error Assertion)
	// 如果需要访问 error 接口之外的自定义字段（如 Code），需进行类型转换
	// 类型断言 e.(*argsError)「不推荐」
	if instance, ok := e.(*argsError); ok {
		fmt.Println("3.1 类型断言:", instance.Code)
		fmt.Println("3.1 类型断言:", instance.Msg)
	}
	// 💡注意：生产环境使用 errors.As 进行更稳健的类型转换
	var target *argsError
	// 将错误类型的内存地址赋值给指针变量target的内存地址
	if errors.As(e, &target) {
		fmt.Println("3.2 errors.As:", target.Code)
		fmt.Println("3.2 errors.As:", target.Msg)
	}

	// 4.errors.Is
	err := findItem(1)
	if errors.Is(err, ErrNotFound) {
		fmt.Println("4.Item not found")
	} else {
		fmt.Println("4.Other error:", err)
	}

	// 5.errors.As
	// 返回的是 *MyError，但被接收为 error 接口
	err2 := getError()
	// 声明目标类型的nil指针变量并提取出的具体类型是 *MyError
	var myErr *MyError
	// 使用 errors.As 进行探测和转换
	// 参数1：原始错误 (err2) 参数2：目标变量的地址 (&myErr)
	if errors.As(err2, &myErr) {
		// 转换成功后，myErr 不再是 nil，而是指向了原始错误中的具体结构体实例
		// 此时可以直接访问 MyError 特有的字段：Code 和 Msg
		fmt.Printf("5.Custom error - Code: %d, Msg: %s\n", myErr.Code, myErr.Msg)
	}
}

```
<!--SR:!2026-04-16,60,288-->
<?e?>
输出
```
1.1 捕获常规错误: 不能为0
1.2 成功返回值: 1
2. 打印完整错误信息: Error Code: 500, Message: can't equal 0
3.1 类型断言: 500
3.1 类型断言: can't equal 0
3.2 errors.As: 500
3.2 errors.As: can't equal 0
4.Item not found
5.Custom error - Code: 404, Msg: Not Found
```
# 协程
```go
package main

import (
	"fmt"
	"time"
)

func f(from string) {
	for i := 0; i < 3; i++ {
		fmt.Println(from, ":", i)
	}
}

func main() {
	// 直接调用
	f("direct")
	// 协程调用
	go f("goroutine")
	// 协程调用闭包
	go func(from string) {
		fmt.Println(from)
	}("going")
	// 睡眠一秒钟
	time.Sleep(time.Second)
	fmt.Println("done")
}
```
输出
```
direct : 0
direct : 1
direct : 2
going
goroutine : 0
goroutine : 1
goroutine : 2
done
```
# 通道
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	message := make(chan string)
	// 从协程读取消息
	go func() { message <- "ping" }()
	msg := <-message
	fmt.Println(msg)

	// 从协程写入消息
	msg2 := make(chan string)
	// 协程1 写入
	go func() {
		msg2 <- "pong"
	}()
	// 协程2 读取
	go func() {
		msg2 := <-msg2
		fmt.Println("go2: ", msg2)
	}()
	time.Sleep(time.Second)
}
```
输出
```
ping
go2:  pong
```
# 通道缓冲
无缓冲通道的发送操作是 (?)  必须等到 (?) 才能完成发送? <?:?> 同步的; 有接收者准备好接收时
<!--SR:!2026-08-04,131,290-->
在**主协程**使用无缓冲通道并且**没有接收者**准备好接收时(在**子协程**中使用无缓冲通道不会触发死锁)
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// +----------------------------------------------------------------------
	// |默认无缓冲通道
	// +----------------------------------------------------------------------
	/*
	   运行结果：
	   程序会在执行 messages <- "hello" 这一行时永久阻塞（deadlock）。

	   证明： 这证明了无缓冲通道的发送操作是同步的，必须等到有接收者准备好接收时才能完成发送。
	*/
	// 默认无缓冲通道
	messages := make(chan string)
	// 尝试发送第一个值「同步阻塞，只有在发送消息前，创建了接收者才不阻塞」
	messages <- "hello"
	// 如果程序能运行到这里，说明成功发送
	fmt.Println("Sent a value")
	// 尝试接收值
	fmt.Println(<-messages)

	// +----------------------------------------------------------------------
	// |有缓冲通道（Buffered Channel）
	// +----------------------------------------------------------------------
	/*
	   运行结果：
	   两个值已成功发送到通道，程序未阻塞，说明它们已被缓存。
	   buffered
	   channel

	   证明：
	   1. 代码在主协程中是顺序执行的，messages2 <- "buffered" 和 messages2 <- "channel" 之间没有启动任何其他协程来接收数据。
	   2. 如果没有缓冲机制，第一个发送操作就会立即阻塞（如场景 1 所示）。
	   3. 程序能够顺序执行完两个发送操作，并打印中间的确认信息，这直接证明这两个值已被通道临时存储（即缓存），而无需等待对应的接收操作。
	*/
	// 缓冲区大小为 2
	messages2 := make(chan string, 2)
	// 发送第一个值 (未阻塞)
	messages2 <- "buffered"
	// 发送第二个值 (未阻塞)
	messages2 <- "channel"
	// 如果尝试在这里发送第三个值，程序将会阻塞或死锁
	// messages2 <- "third"
	fmt.Println("两个值已成功发送到通道，程序未阻塞，说明它们已被缓存。")
	// 接收值
	fmt.Println(<-messages2)
	fmt.Println(<-messages2)

	// +----------------------------------------------------------------------
	// |没有阻塞主协程不会触发死锁
	// +----------------------------------------------------------------------
	r := make(chan int)
	go func() {
		<-r // 这个 goroutine 阻塞，但主协程仍可继续
	}()
	time.Sleep(time.Second)
}
```
输出
```
两个值已成功发送到通道，程序未阻塞，说明它们已被缓存。
buffered
channel
```
# 通道同步
```go
package main

import (
	"fmt"
	"time"
)

// 定义工作协程
// 接收一个只读或读写通道 c 用于通知完成状态
func worker(c chan bool) {
	fmt.Println("working...")
	// 模拟耗时任务
	time.Sleep(2 * time.Second)
	fmt.Println("done")
	// 发送信号
	// 任务完成后，向通道发送一个值，通知 main 协程
	c <- true
}

func main() {
	// 1. 创建通道
	c := make(chan bool)
	
	// 2. 启动协程
	// 使用 go 关键字异步执行 worker
	go worker(c)
	
	// 3. 阻塞等待
	// 程序执行到这一行会停下来，直到 worker 往通道 c 里发数据
	// 如果没有这一行，main 会直接结束，worker 可能还没跑完就被强制终止了
	fmt.Println("终于执行完了：", <-c)
}
```
输出
```
working...
done
终于执行完了： true
```
# 通道方向
```go
package main

import "fmt"

// 只写通道 (Send-only)
// chan<- string 表示此函数只能向 pings 发送数据
// 如果在此函数内部尝试 <-pings (读取)，编译器会报错
func ping(pings chan<- string, msg string) {
	pings <- msg
}

// 双向转换与单向限制
// pings <-chan string: 只能从中读取数据
// pongs chan<- string: 只能向其发送数据
func pong(pings <-chan string, pongs chan<- string) {
	// 从第一个通道取出，存入第二个通道
	msg := <-pings
	pongs <- msg
}

func main() {
	// 1. 在 main 中创建的是双向通道
	pings := make(chan string, 1)
	pongs := make(chan string, 1)
	
	// 2. 类型转换
	// 当双向通道作为参数传递给接收单向通道的函数时，Go 会自动将其转换为受限的单向通道
	go ping(pings, "traverse")
	go pong(pings, pongs)
	// 最终从 pongs 获取结果
	fmt.Println("from to pong:", <-pongs)
}

```
输出
```
from to pong: traverse
```
# 通道选择器
若多个 case 同时就绪，执行顺序是怎样的?<?:?>随机挑选一个执行
<!--SR:!2026-06-20,105,310-->
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	/*
	   精简总结：
	   1. 仅限 Channel：每个 case 必须是通道发送或接收。
	   2. 随机公平：若多个 case 同时就绪，随机挑选一个执行。
	   3. 阻塞机制：无 default 且通道均未就绪时，协程进入阻塞。
	   4. 单次触发：一次 select 仅执行一个 case 就会退出。
	*/
	// 1. 初始化两个通道
	c1 := make(chan string)
	c2 := make(chan string)

	// 2. 模拟两个异步任务
	// 任务一：1秒后向 c1 发送数据
	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "one"
	}()
	// 任务二：2秒后向 c2 发送数据
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "two"
	}()

	// 3. 使用 select 同时监听多个通道
	// 我们知道会有两个消息，所以循环两次
	for i := 0; i < 2; i++ {
		// select 会阻塞，直到其中一个 case 准备就绪
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
		}
	}
}
```
输出
```
received one
received two
```
# 超时处理
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 1. 场景一：任务执行时间 2s，超时限制 1s
	c1 := make(chan string, 1)
	go func() {
		// 模拟一个耗时 2 秒的操作
		time.Sleep(2 * time.Second)
		c1 <- "result 1"
	}()

	select {
	case res := <-c1:
		fmt.Println(res)
	case <-time.After(1 * time.Second):
		// time.After 会在指定时间后发送当前时间到返回的通道中
		// 因为 1s 早于 2s 到达，所以会执行这个 case
		fmt.Println("timeout 1")
	}

	// 2. 场景二：任务执行时间 2s，超时限制 3s
	c2 := make(chan string, 1)
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "result 2"
	}()

	select {
	case res := <-c2:
		// 因为 2s 早于 3s 得到结果，所以会正常打印结果
		fmt.Println(res)
	case <-time.After(3 * time.Second):
		fmt.Println("timeout 2")
	}
}
```
输出
```
timeout 1
result 2
```
# 非阻塞通道操作
```go
package main

import "fmt"

func main() {
	// 默认创建的是无缓冲通道 (Unbuffered Channels)
	messages := make(chan string)
	signals := make(chan bool)

	// 1. 非阻塞接收
	// 如果 messages 通道中有数据，则执行第一个 case
	// 如果没有数据，它不会等待，而是立即执行 default
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	default:
		fmt.Println("no message received")
	}

	// 2. 非阻塞发送
	// 因为 messages 是无缓冲通道且当前没有接收者，
	// 所以发送操作无法立即完成。
	// select 会直接跳到 default，从而避免程序卡死。
	msg := "hi"
	select {
	case messages <- msg:
		fmt.Println("sent message", msg)
	default:
		fmt.Println("no message sent")
	}

	// 3. 多路非阻塞选择
	// 同时尝试从多个通道接收数据，若都不可用，则执行 default
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	case sig := <-signals:
		fmt.Println("received signal", sig)
	default:
		fmt.Println("no activity")
	}
}
```
输出
```
no message received
no message sent
no activity
```
# 通道的关闭
```go
package main

import "fmt"

func main() {
	// 1. 创建带缓冲的通道，模拟任务队列
	jobs := make(chan int, 5)
	done := make(chan bool)

	// 2. 启动一个工作协程接收任务
	go func() {
		for {
			// 使用特殊的双返回值语法：
			// j 是接收到的值
			// more 是一个布尔值：如果通道已关闭且数据已读完，则为 false
			j, more := <-jobs
			if more {
				fmt.Println("received job", j)
			} else {
				// 当 more 为 false，说明通道已关闭且没有残留数据
				fmt.Println("received all jobs")
				done <- true
				return
			}
		}
	}()

	// 3. 发送 3 个任务
	for j := 1; j <= 3; j++ {
		jobs <- j
		fmt.Println("sent job", j)
	}

	// 4. 关闭通道
	// close 告诉接收者不会再有新数据了。
	// 💡注意：关闭后的通道依然可以读取其中残留的数据（1, 2, 3）。
	close(jobs)
	fmt.Println("sent all jobs")

	// 阻塞等待工作协程完成信号
	<-done
}

```
输出
```
sent job 1
sent job 2
sent job 3
sent all jobs
received job 1
received job 2
received job 3
received all jobs
```
# 通道遍历
更底层的写法<?:?>`if msg, bl := <-queue2; bl {...}`
<!--SR:!2026-06-22,109,311-->
```go
package main

import "fmt"

func main() {
	// 1. 使用 range 遍历通道
	queue := make(chan string, 2)
	queue <- "one"
	queue <- "two"

	// 关键点：必须关闭通道，否则下方的 range 循环会永远阻塞（死锁）
	close(queue)

	// range 会不断从通道中弹出数据，直到通道被关闭且数据被取完
	for msg := range queue {
		fmt.Println("range:", msg)
	}

	// 2. 使用 for 配合解构赋值遍历
	queue2 := make(chan string, 2)
	queue2 <- "three"
	queue2 <- "four"
	close(queue2)

	// 这是一种更底层的写法，效果与 range 相同
	for {
		// bl (通常命名为 ok) 表示通道是否尚未关闭或仍有数据
		if msg, bl := <-queue2; bl {
			fmt.Println("for:", msg)
		} else {
			// 当 bl 为 false，说明通道已关且空，跳出循环或结束程序
			return
		}
	}
}
```
输出
```
range: one
range: two
for: three
for: four
```
# Timer
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 1. 基础用法：一次性触发
	// NewTimer 返回一个结构体，其中的 C 属性是一个通道
	timer1 := time.NewTimer(time.Second)

	// 阻塞式等待：直到 1 秒后，系统向 timer1.C 发送当前时间
	<-timer1.C
	fmt.Println("1.Timer 1 fired: 1秒后执行")

	// 2. 取消定时器
	// 创建一个原本计划 1 秒后触发的计时器
	timer2 := time.NewTimer(time.Second)
	go func() {
		<-timer2.C
		fmt.Println("Timer 2 fired") // 这行代码将不会被执行
	}()

	// 在计时器到期前调用 Stop()
	// 如果你只是想等待，Sleep 更简单；但如果你需要取消，必须用 Timer
	stop2 := timer2.Stop()
	if stop2 {
		fmt.Println("2.Timer 2 stopped: 我取消了定时器")
	}

	// 留出时间观察结果，确认子协程 2 确实没有执行打印
	time.Sleep(2 * time.Second)
}
```
输出
```
1.Timer 1 fired: 1秒后执行
2.Timer 2 stopped: 我取消了定时器
```
# Ticker
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	/*
	   想“延迟一次”用 Timer；
	   想“定时重复”用 Ticker。
	*/

	// Ticker：周期触发
	ticker := time.NewTicker(time.Second)
	done := make(chan bool)
	count := 1
	go func() {
		for {
			select {
			case t := <-ticker.C:
				fmt.Println("Tick at：", t.Format("2006-01-02 15:4:05"))
				fmt.Println("执行次数：", count)
				count++
			case <-done:
				fmt.Println("停止执行")
				return
			}
		}
	}()

	// 2秒后 手动停止ticker
	selectSeconds := time.Duration(2)
	time.Sleep(selectSeconds * time.Second)
	ticker.Stop()

	// 主协程同步等待3秒
	seconds := 3
	time.Sleep(time.Duration(seconds) * time.Second)
	fmt.Println(seconds, "秒后停止")

	// AfterFunc 示例
	time.AfterFunc(300*time.Millisecond, func() {
		fmt.Println("定时回调一次函数")
		close(done)
	})
	<-done
}
```
输出
```
Tick at： 2026-01-22 11:38:07
执行次数： 1
Tick at： 2026-01-22 11:38:08
执行次数： 2
3 秒后停止
定时回调一次函数
停止执行
```
# 工作池
```go
package main

import (
	"fmt"
	"time"
)

// 定义工作者函数
// jobs 通道仅用于接收任务 (<-chan)
// results 通道仅用于发送结果 (chan<-)
func workerPool(id int, jobs <-chan int, results chan<- int) {
	// 多个 worker 竞争读取同一个 jobs 通道
	// Go 确保每个任务只会被一个 worker 领取
	for j := range jobs {
		fmt.Println("worker", id, "started  job", j)
		// 模拟耗时的业务逻辑处理
		time.Sleep(time.Second)
		fmt.Println("worker", id, "finished job", j)
		// 将处理结果发回结果通道
		results <- j * 2
	}
}

func main() {
	const numJobs = 5
	// 1. 创建带缓冲的通道
	// 缓冲区大小设为任务总数，防止发送端因为接收端处理慢而阻塞
	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)

	// 2. 启动固定数量的协程 (池化)
	// 这里启动了 3 个工作协程，它们会并发地执行任务
	for w := 1; w <= 3; w++ {
		go workerPool(w, jobs, results)
	}

	// 3. 分派任务
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	// 任务发送完毕后必须关闭通道，否则 worker 的 range 循环永远不会结束
	close(jobs)

	// 4. 收集结果
	// 这里的阻塞等待确保了 main 函数在所有任务完成前不会退出
	for a := 1; a <= numJobs; a++ {
		<-results
	}
}
```
输出
```
worker 3 started  job 2
worker 1 started  job 1
worker 2 started  job 3
worker 2 finished job 3
worker 2 started  job 4
worker 3 finished job 2
worker 3 started  job 5
worker 1 finished job 1
worker 3 finished job 5
worker 2 finished job 4
```
# WaitGroup
方法传参 WaitGroup 必须是(?)类型 <?:?> 指针 (`*sync.WaitGroup`)，以确保操作的是同一个计数器
<!--SR:!2026-07-20,119,291-->
<?e?>
go 1.22前的问题
<?l?>
 `i := i` 避免在每个协程闭包中重复利用相同的 i 值 更多细节可以参考 the FAQ(https://go.dev/doc/faq#closures_and_goroutines)
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	// 1. 声明 WaitGroup 变量
	var wg sync.WaitGroup

	// 2. 定义任务函数
	// 必须传入 WaitGroup 的指针 (*sync.WaitGroup)，以确保操作的是同一个计数器
	daemon := func(id int, wg *sync.WaitGroup) {
		// 5. 任务完成后递减计数器
		// defer 确保即使函数中间发生了 panic，Done() 也会被执行
		defer wg.Done()

		fmt.Printf("Worker %d starting\n", id)
		time.Sleep(time.Second) // 模拟耗时操作
		fmt.Printf("Worker %d done\n", id)
	}

	for i := 1; i <= 5; i++ {
		// 3. 增加计数器
		// 每次启动协程前，调用 Add(1) 告诉 WaitGroup 多了一个任务
		wg.Add(1)

		// 💡注意：循环变量的捕获问题「go<=1.22」
		// 避免在每个协程闭包中重复利用相同的 i 值 更多细节可以参考 the FAQ(https://go.dev/doc/faq#closures_and_goroutines)
		//i := i

		// 4. 调用函数
		go daemon(i, &wg)
	}

	// 6. 阻塞等待
	// Wait() 会停在这里，直到 WaitGroup 的内部计数器归零
	wg.Wait()
	fmt.Println("All workers finished")
}
```
输出
```
Worker 1 starting
Worker 2 starting
Worker 5 starting
Worker 4 starting
Worker 3 starting
Worker 5 done
Worker 3 done
Worker 2 done
Worker 4 done
Worker 1 done
All workers finished
```
# 速率限制
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// +----------------------------------------------------------------------
	// | 限速器：200ms触发一次
	// +----------------------------------------------------------------------
	// 创建一个缓冲大小为5的请求通道并放入5个请求，随后关闭通道表示不再发送请求
	requests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		requests <- i
	}
	close(requests)
	// 限速器：使用 time.Tick 每200ms产生一个时间信号，作为速率控制的令牌
	// 💡注意：time.Tick 会返回一个不能停止的 ticker，如需停止应使用 time.NewTicker 并调用 Stop()
	limiter := time.Tick(200 * time.Millisecond)
	// 按固定速率处理请求：每处理一个请求先从 limiter 获取一个令牌（阻塞直到有令牌）
	for req := range requests {
		<-limiter
		fmt.Println("request", req, time.Now())
	}

	// +----------------------------------------------------------------------
	// | 支持突发的限速器（突发容量为3）
	// +----------------------------------------------------------------------
	// burstyLimiter 是一个带缓冲的通道，预先放入3个令牌，允许一次性处理最多3个突发请求
	burstyLimiter := make(chan time.Time, 3)
	for i := 0; i < 3; i++ {
		burstyLimiter <- time.Now()
	}
	// 后续以每200ms补充一个令牌到 burstyLimiter，允许持续处理
	// 同样这里使用 time.Tick 简化示例；实际中若需要在将来停止应使用 time.NewTicker 并 Stop()
	go func() {
		for t := range time.Tick(200 * time.Millisecond) {
			burstyLimiter <- t
		}
	}()
	// 模拟一组突发请求并使用 burstyLimiter 控制速率
	burstyRequests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		burstyRequests <- i
	}
	close(burstyRequests)
	for req := range burstyRequests {
		<-burstyLimiter
		fmt.Println("request", req, time.Now())
	}
}
```
输出
```
request 1 2026-01-22 11:48:25.20078 +0800 CST m=+0.200960043
request 2 2026-01-22 11:48:25.400908 +0800 CST m=+0.401085626
request 3 2026-01-22 11:48:25.600038 +0800 CST m=+0.600214668
request 4 2026-01-22 11:48:25.799952 +0800 CST m=+0.800126543
request 5 2026-01-22 11:48:26.000192 +0800 CST m=+1.000364335
request 1 2026-01-22 11:48:26.000405 +0800 CST m=+1.000577168
request 2 2026-01-22 11:48:26.000415 +0800 CST m=+1.000587751
request 3 2026-01-22 11:48:26.000419 +0800 CST m=+1.000592001
request 4 2026-01-22 11:48:26.20147 +0800 CST m=+1.201640668
request 5 2026-01-22 11:48:26.401497 +0800 CST m=+1.401665960
```
# 原子计数器
<!--SR:!2026-03-28,3,271-->
<?e?>
原子计数器关键代码
<?l?>
```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {
	// 1. 定义原子变量
	// 我们使用无符号 64 位整型来承载计数
	var ops uint64
	// 使用 WaitGroup 确保所有协程执行完毕
	var wg sync.WaitGroup
	// 2. 启动 50 个并发协程
	for i := 0; i < 50; i++ {
		wg.Add(1)
		go func() {
			for c := 0; c < 1000; c++ {
				// 3. 原子自增操作
				// 不能直接使用 ops++，因为那不是线程安全的（读取-修改-写入三步可能被中断）
				// atomic.AddUint64 通过 CPU 指令保证这一行操作是“不可分割”的
				atomic.AddUint64(&ops, 1)
			}
			wg.Done()
		}()
	}
	// 等待 50 * 1000 次累加全部完成
	wg.Wait()
	// 4. 读取结果
	// 虽然此时协程已结束，但在高并发读取中，
	// 建议也使用 atomic.LoadUint64(&ops) 来保证读取到的值是最新的
	fmt.Println("ops:", atomic.LoadUint64(&ops))
}
```
输出
```
ops: 50000
```
<!--SR:!2026-03-30,34,251-->
<?e?>
# 互斥锁
<?e?>
互斥锁的关键代码?
<?l?>
```go
package main

import (
	"fmt"
	"sync"
)

// 1. 定义包含锁的结构体
// 将 Mutex 和它要保护的数据封装在一起是 Go 的最佳实践
type Container struct {
	mu       sync.Mutex // 互斥锁，保护下方的 counters
	counters map[string]int
}

// 2. 在方法中使用锁
func (c *Container) inc(name string) {
	// Lock() 确保同一时刻只有一个 Goroutine 能进入临界区
	c.mu.Lock()
	// 使用 defer 在方法返回时自动解锁，防止因忘记解锁导致的死锁
	defer c.mu.Unlock()
	// 在 Go 中，直接并发读写 Map 是非法的，会导致程序崩溃 (fatal error)
	c.counters[name]++
}

func main() {
	c := Container{
		// 💡注意：Map 必须初始化后才能使用
		counters: map[string]int{"a": 0, "b": 0},
	}
	var wg sync.WaitGroup
	doIncrement := func(name string, n int) {
		for i := 0; i < n; i++ {
			c.inc(name)
		}
		wg.Done()
	}
	// 3. 并发执行
	wg.Add(3)
	go doIncrement("a", 10000) // 协程1 修改 "a"
	go doIncrement("a", 10000) // 协程2 修改 "a" (竞态点)
	go doIncrement("b", 10000) // 协程3 修改 "b"
	wg.Wait()
	fmt.Println(c.counters)
}
```
输出
```
map[a:20000 b:10000]
```
<!--SR:!2026-05-19,77,272-->
<?e?>
# 状态协程
```go
package main

import (
	"fmt"
	"math/rand"
	"sync/atomic"
	"time"
)

// 1. 定义请求结构体
// 除了包含要操作的数据外，还包含一个用于接收回复的通道 (resp)
type readOp struct {
	key  int
	resp chan int
}
type writeOp struct {
	key  int
	val  int
	resp chan bool
}

func main() {
	var readOps uint64
	var writeOps uint64

	// 2. 定义请求队列通道
	reads := make(chan readOp)
	writes := make(chan writeOp)

	// 3. 状态持有协程 (State Manager)
	// 这个协程独占 state 变量，没有任何其他协程可以直接访问它
	go func() {
		var state = make(map[int]int)
		for {
			select {
			case read := <-reads:
				// 处理读请求：从私有 state 中取值，并通过 resp 通道发回
				read.resp <- state[read.key]
			case write := <-writes:
				// 处理写请求：更新私有 state，并通过 resp 通道发回确认
				state[write.key] = write.val
				write.resp <- true
			}
		}
	}()

	// 4. 模拟大量读操作协程 (100个)
	for r := 0; r < 100; r++ {
		go func() {
			for {
				read := readOp{
					key:  rand.Intn(5),
					resp: make(chan int),
				}
				reads <- read // 发送请求
				<-read.resp   // 等待结果（阻塞）
				atomic.AddUint64(&readOps, 1)
				time.Sleep(time.Millisecond)
			}
		}()
	}

	// 5. 模拟少量写操作协程 (10个)
	for w := 0; w < 10; w++ {
		go func() {
			for {
				write := writeOp{
					key:  rand.Intn(5),
					val:  rand.Intn(100),
					resp: make(chan bool),
				}
				writes <- write // 发送请求
				<-write.resp    // 等待确认
				atomic.AddUint64(&writeOps, 1)
				time.Sleep(time.Millisecond)
			}
		}()
	}

	time.Sleep(time.Second)

	// 打印最终统计数据
	fmt.Println("readOps:", atomic.LoadUint64(&readOps))
	fmt.Println("writeOps:", atomic.LoadUint64(&writeOps))
}
```
输出
```
readOps: 88144
writeOps: 8820
```
# 排序
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	// int 排序
	integer := []int{5, 2, 3, 1, 0}
	sort.Ints(integer)
	fmt.Println(integer)

	// string 排序
	strings := []string{"e", "d", "a", "c", "b"}
	sort.Strings(strings)
	fmt.Println(strings)

	// sort 来检查一个切片是否为有序的
	fmt.Println("sorted:", sort.IntsAreSorted(integer))
	fmt.Println("sorted:", sort.StringsAreSorted(strings))
}
```
输出
```
[0 1 2 3 5]
[a b c d e]
sorted: true
sorted: true
```
# 使用函数自定义排序
```go
package main

import (
	"fmt"
	"sort"
)

// 定义一个自定义类型 byLen，它是 []string 的别名
// 用于实现按字符串长度排序的接口
type byLength []string

// Len sort.Interface约束方法：返回集合中的元素数量
func (s byLength) Len() int {
	return len(s)
}

// Swap sort.Interface约束方法：交换集合中索引为 a 和 b 的两个元素
func (s byLength) Swap(a, b int) {
	s[a], s[b] = s[b], s[a]
}

// Less sort.Interface约束方法：判断索引 a 处的元素是否应该排在索引 b 处的元素之前
func (s byLength) Less(a, b int) bool {
	return len(s[a]) < len(s[b]) // 元素长度比较: 索引 a 元素 < 索引 b 元素为 true
}

func main() {
	// 初始化一个字符串切片 fruits
	fruits := []string{"peach", "strawberry", "banana"}

	// 使用 sort.Sort() 方法进行排序
	// 将 fruits 转换为 byLen 类型，使其拥有自定义的排序逻辑
	// 排序逻辑在 Less 方法中定义：按字符个数升序
	sort.Sort(byLength(fruits))

	// 输出排序后的切片
	fmt.Println(fruits) // 输出: [peach banana strawberry]
}

```
输出
```
[peach banana strawberry]
```
# Panic
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	panic("a problem")

	create, err := os.Create("./t1.txt")
	if err != nil {
		panic(err)
	}
	write, err := create.Write([]byte("2344"))
	if err != nil {
		return
	}
	// 返回写入的字节数
	fmt.Println(write)
}
```
输出
```
panic: a problem
```
# Defer
<?e?>
Defer 三大规则?
循环defer的坑?
<?l?>
```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

// --- 文件操作与 Defer 确保资源释放 ---
func createFile(p string) *os.File {
	fmt.Println("1. 创建文件...")
	create, err := os.Create(p)
	if err != nil {
		panic(err) // 如果无法创建文件，程序停止执行
	}
	return create
}

func writeFile(f *os.File, c string) {
	fmt.Println("2. 写入数据...")
	// 使用 fmt.Fprintln 写入数据，它接收一个 io.Writer 接口
	l, err := fmt.Fprintln(f, c)
	if err != nil {
		fmt.Println("写入失败:", err)
		return
	}
	fmt.Printf("成功写入字节数: %d\n", l)
}

func closeFile(f *os.File) {
	fmt.Println("3. 关闭文件 (由 defer 调用)")
	err := f.Close()
	if err != nil {
		// 如果关闭失败，将错误信息输出到标准错误流
		fmt.Fprintf(os.Stderr, "error: %v \n", err)
		os.Exit(1)
	}
}

// --- Defer 三大规则 ---
func rule1() {
	for i := 0; i < 3; i++ {
		defer fmt.Println(i)
	}
}

func rule2() {
	i := 0
	// 💡 重点：此时 i 的当前值 (0) 已被求值并保存。
	defer fmt.Printf("rule 2: %d \n", i)

	i++ // 函数体内的修改不会改变已压入 defer 栈的参数
}

func rule3() (i int) { // 具名返回值修改函数的最终结果
	defer func() {
		// 💡 重点：defer 后的匿名函数 在 return 赋值后、函数彻底退出前执行
		i++
	}()
	// 执行过程：1. i = 1 (赋值) -> 2. 执行 defer (i++) -> 3. 退出 (返回 2)
	return 1
}

func main() {
	// --- 文件操作与 Defer 确保资源释放 ---
	fmt.Println("\n--- 文件操作与 Defer 确保资源释放 ---")
	file := createFile("t2.txt")
	// 💡 最佳实践：获取资源后立即声明 defer，防止后续代码报错导致资源泄漏
	defer closeFile(file)
	writeFile(file, "hello go defer")

	// --- Defer 三大规则 ---
	fmt.Println("\n--- Defer 三大规则 ---")
	// 规则 1：栈结构 后进先出 (LIFO Order)
	fmt.Println("rule 1:")
	rule1()

	// 规则 2：参数预计算 (快照效应：Snapshot Effect)
	// 这是 defer 最容易踩坑的地方：defer 函数的参数在声明时就会被求值并固定，而不是在执行时。
	// 💡注意：如果 defer 后面跟的是匿名函数且内部引用了外部变量（闭包），则会捕获变量的引用，此时会看到更新后的值。
	rule2()

	// 规则 3：修改命名返回值 (Named Return Modification)
	fmt.Printf("rule 3: %v \n", rule3())

	fmt.Println("\n--- 避坑指南 ---")
	// 循环中应通过匿名函数即时触发 `defer`，避免因延迟执行导致文件描述符耗尽。
	matches, _ := filepath.Glob(`/Users/weichengjun/dnmp/www/test/go/example/*.go`)
	for k, file := range matches {
		func() {
			f, _ := os.Open(file)
			if k <= 3 {
				r := []byte{5}
				_, _ = f.Read(r)
				fmt.Println("避坑指南", r)
			}
			defer f.Close() // 每次匿名函数结束就会关闭，不会堆积
		}()
	}
}

```
输出
```
--- 文件操作与 Defer 确保资源释放 ---
1. 创建文件...
2. 写入数据...
成功写入字节数: 15

--- Defer 三大规则 ---
rule 1:
2
1
0
rule 2: 0 
rule 3: 2 

--- 避坑指南 ---
避坑指南 [112]
避坑指南 [112]
避坑指南 [112]
避坑指南 [112]
3. 关闭文件 (由 defer 调用)
```
<!--SR:!2026-05-02,67,272-->
<?e?>
# Recover
```go
package main

import "fmt"

// mayPanic 函数：会立即触发一个恐慌（panic）
func mayPanic() {
	panic("a problem") // 使用 panic 抛出一个字符串类型的恐慌
}

func main() {
	// defer 延迟调用一个匿名函数。
	// 无论函数 main() 是正常返回还是遭遇 panic，这个延迟函数都会执行。
	defer func() {
		// recover() 必须在延迟函数（deferred function）中调用，它会捕获最近一次的 panic。
		// 如果捕获到了 panic，它返回恐慌的值（即 "a problem"）；
		// 如果没有 panic 或 panic 已被捕获，它返回 nil。
		if r := recover(); r != nil {
			// 如果 r 不为 nil，说明发生了 panic 且已被成功捕获/恢复。
			fmt.Println("Recovered. Error:\n", r) // 打印恢复信息和恐慌的值
		}
	}()

	// 调用 mayPanic() 时，会触发 panic("a problem")。
	// 此时程序执行流会立即停止，并跳转到上面的 defer 函数中执行 recover。
	mayPanic()

	// 因为 panic 已经被触发，并且程序流被中断并跳转到了 defer，
	// 所以这行代码永远不会被执行。
	fmt.Println("After myPanic()")
}

```
输出
```
Recovered. Error:
 a problem
```
# 字符串函数
```go
package main

import (
	"fmt"
	"io"
	sg "strings" // 导入 strings 包，并给它一个别名 pf
)

var p = fmt.Println // 将 fmt.Println 赋值给变量 p，简化输出

func main() {
	// --- 常用 strings 包函数示例 ---
	// 检查是否包含子串
	p("Contains:  ", sg.Contains("test", "es"))
	// 统计子串出现次数
	p("Count:     ", sg.Count("test", "t"))
	// 检查是否有前缀
	p("HasPrefix: ", sg.HasPrefix("test", "te"))
	// 检查是否有后缀
	p("HasSuffix: ", sg.HasSuffix("test", "st"))
	// 查找子串首次出现的索引
	p("Index:     ", sg.Index("test", "e"))
	// 使用分隔符连接切片元素
	p("Join:      ", sg.Join([]string{"a", "b"}, "-"))
	// 重复字符串
	p("Repeat:    ", sg.Repeat("a", 5))
	// 全部替换
	p("Replace:   ", sg.Replace("foo", "o", "0", -1))
	// 替换第 1 个匹配项
	p("Replace:   ", sg.Replace("foo", "o", "0", 1))
	// 按分隔符分割字符串，返回切片
	p("Split:     ", sg.Split("a-b-c-d-e", "-"))
	// 转换为小写
	p("ToLower:   ", sg.ToLower("TEST"))
	// 转换为大写
	p("ToUpper:   ", sg.ToUpper("test"))
	p()

	// --- 基本字符串操作 ---
	p("Len: ", len("hello")) // 获取字符串长度（字节数）
	p("Char:", "hello"[1])   // 获取指定索引处的字节值

	// --- 字符串读取 ---
	var str = "hello world"
	reader := sg.NewReader(str)
	b := make([]byte, 8)
	for {
		n, err := reader.Read(b)
		fmt.Println(string(b[:n]))
		if err == io.EOF {
			break
		}
	}
}

```
输出
```
Contains:   true
Count:      2
HasPrefix:  true
HasSuffix:  true
Index:      1
Join:       a-b
Repeat:     aaaaa
Replace:    f00
Replace:    f0o
Split:      [a b c d e]
ToLower:    test
ToUpper:    TEST

Len:  5
Char: 101
hello wo
rld
```
# 字符串格式化
<?e?>
1. 宽度为 6 的整数，右对齐
2. 宽度为 6，小数点后 2 位的浮点数
3. 左对齐的宽度为 6，小数点后 2 位的浮点数
4. 宽度为 6 的字符串，右对齐
5. 左对齐的宽度为 6 的字符串
<?l?>
```go
package main

import (
	"fmt" // 导入 fmt 包，用于格式化 I/O
	"os"  // 导入 os 包，用于访问操作系统功能，这里用于输出到标准错误
)

// 定义一个结构体 point，包含两个整型字段 x 和 y
type point struct {
	x, y int
}

func main() {
	// 初始化结构体变量 p
	p := point{1, 2}
	// %v：输出结构体的值
	fmt.Printf("struct1: %v\n", p)

	// %+v：输出结构体的值，并包含字段名
	fmt.Printf("struct2: %+v\n", p)

	// %#v：输出结构体的 Go 语法表示 (即 type{field: value})
	fmt.Printf("struct3: %#v\n", p)

	// %T：输出值的类型
	fmt.Printf("type: %T\n", p)

	// %t：输出布尔值
	fmt.Printf("bool: %t\n", true)

	// %d：输出十进制整数
	fmt.Printf("int: %d\n", 123)

	// %b：输出二进制整数
	fmt.Printf("bin: %b\n", 14)

	// %c：输出 Unicode 字符（对应整数码点）
	fmt.Printf("char: %c\n", 33)

	// %x：输出十六进制整数
	fmt.Printf("hex: %x\n", 456)

	// %f：输出默认格式的浮点数
	fmt.Printf("float1: %f\n", 78.9)

	// %e / %E：输出科学记数法浮点数（小写 e / 大写 E）
	fmt.Printf("float2: %e\n", 123400000.0)
	fmt.Printf("float3: %E\n", 123400000.0)

	// %s：输出字符串
	fmt.Printf("str1: %s\n", "\"string\"")

	// %q：带双引号的字符串，字符带有转义
	fmt.Printf("str2: %q\n", "\"string\"")

	// %x：将字符串的每个字节表示为两个字符的十六进制
	fmt.Printf("str3: %x\n", "hex this")

	// %p：输出指针地址
	fmt.Printf("pointer: %p\n", &p)

	// %6d：宽度为 6 的整数，右对齐
	fmt.Printf("width1: |%6d|%6d|\n", 12, 345)

	// %6.2f：宽度为 6，小数点后 2 位的浮点数
	fmt.Printf("width2: |%6.2f|%6.2f|\n", 1.2, 3.45)

	// %-6.2f：左对齐的宽度为 6，小数点后 2 位的浮点数
	fmt.Printf("width3: |%-6.2f|%-6.2f|\n", 1.2, 3.45)

	// %6s：宽度为 6 的字符串，右对齐
	fmt.Printf("width4: |%6s|%6s|\n", "foo", "b")

	// %-6s：左对齐的宽度为 6 的字符串
	fmt.Printf("width5: |%-6s|%-6s|\n", "foo", "b")

	// fmt.Sprintf：格式化字符串并返回结果（不打印）
	s := fmt.Sprintf("sprintf: a %s", "string")
	fmt.Println(s)

	// fmt.Fprintf：格式化字符串并写入指定的 io.Writer（这里是标准错误 os.Stderr）
	fmt.Fprintf(os.Stderr, "io: an %s\n", "error")
}

```
输出
```
struct1: {1 2}
struct2: {x:1 y:2}
struct3: main.point{x:1, y:2}
type: main.point
bool: true
int: 123
bin: 1110
char: !
hex: 1c8
float1: 78.900000
float2: 1.234000e+08
float3: 1.234000E+08
str1: "string"
str2: "\"string\""
str3: 6865782074686973
pointer: 0x140000100f0
width1: |    12|   345|
width2: |  1.20|  3.45|
width3: |1.20  |3.45  |
width4: |   foo|     b|
width5: |foo   |b     |
sprintf: a string
io: an error
```
# 文本模板
<!--SR:!2026-05-13,71,272-->
<?e?>
1. 使用 Must 方法简化错误处理，重新解析模板?
2. 使用结构体作为数据源?
3. 使用 map 作为数据源?
4. 创建条件判断模板?
5. 创建循环遍历模板?
<?l?>
```go
// +----------------------------------------------------------------------
// | 创建模板「基础方法」
// +----------------------------------------------------------------------
// 创建一个名为 "t1" 的新模板
base := template.New("t1")
// 解析模板字符串，{{.}} 表示当前传入的数据
base, err := base.Parse("Value is {{.}}\n")
if err != nil {
	panic(err)
}

// +----------------------------------------------------------------------
// | 创建模板「使用 Must 方法简化错误处理」
// +----------------------------------------------------------------------
// 定义一个创建模板的辅助函数
Create := func(name, t string) *template.Template {
	return template.Must(template.New(name).Parse(t))
}
// 创建模板
t1 := Create("t1", "Value: {{.}}\n")
// 💡解析并输出到标准输出
// 字符串
t1.Execute(os.Stdout, "some text")
// 数字
t1.Execute(os.Stdout, 5)
// 💡解析并输出到变量
out := new(strings.Builder)
// 字符串切片
t1.Execute(out, []string{"Go", "Rust", "C++", "C#"})
fmt.Println(out)

// 创建模板
t2 := Create("t2", "Name: {{.Name}}\n")
// 1. 使用结构体作为数据源
t2.Execute(os.Stdout, struct{ Name string }{"Jane Doe"})
// 2. 使用 map 作为数据源
t2.Execute(os.Stdout, map[string]string{"Name": "Mickey Mouse"})

// 3. 创建条件判断模板
// 使用 - 在动作中去除空格
t3 := Create("t3", "{{if . -}} yes {{else -}} no {{end}}\n")
// 根据传入数据是否为空执行不同分支
t3.Execute(os.Stdout, "not empty") // 非空数据
t3.Execute(os.Stdout, "")          // 空数据

// 4. 创建循环遍历模板
t4 := Create("t4", "Range: {{range .}}{{.}} {{end}}\n")
// 遍历并输出字符串切片中的每个元素
t4.Execute(os.Stdout, []string{"Go", "Rust", "C++", "C#"})
```
输出
```
Value: some text
Value: 5
Value: [Go Rust C++ C#]
Name: Jane Doe
Name: Mickey Mouse
yes 
no 
Range: Go Rust C++ C#
```
<!--SR:!2026-04-09,19,232-->
<?e?>
## 代码自动生成
```go
// 1. 定义上下文数据 (类似 PHP 的 $context)
context := map[string]any{
	"modelName":  "My",
	"stBaseName": "MyStruct",
	"numberField": map[string]map[string]string{
		"id": {
			"COLUMN_TYPE":    "int",
			"COLUMN_NAME":    "id",
			"COLUMN_DEFAULT": "0",
			"COLUMN_COMMENT": "ID",
		},
		"name": {
			"COLUMN_TYPE":    "string",
			"COLUMN_NAME":    "name",
			"COLUMN_DEFAULT": "''",
			"COLUMN_COMMENT": "姓名",
		},
	},
}
// 2. 自定义函数映射: 首字母大写
funcMap := template.FuncMap{
	"title": func(s string) string {
		if len(s) == 0 {
			return ""
		}
		return strings.ToUpper(s[:1]) + s[1:]
	},
}

// 3. 解析模板 (确保 New 的名字和文件名一致，或者执行时指定文件名)
/* 💡注意：在 Go 的 text/template 包中，一个模板对象其实是一个集合（Set）。
   1.执行 template.New("struct") 时，创建了一个名为 "struct" 的空模板。
   2.调用 ParseFiles("struct.tmpl") 时，它解析了文件，并在集合中创建了一个名为 "struct.tmpl"（带后缀）的新模板，并关联了文件内容。
   3.调用 tmpl.Execute(...) 时，它默认尝试执行集合中 "最顶层（即刚 New 出来的那个）" 模板，也就是那个名为 "struct" 的空模板。
   4.因为 "struct" 是空的，内容全在 "struct.tmpl" 里，所以报错 incomplete or empty。
*/
path := "/Users/weichengjun/dnmp/www/test/go/example/struct.tmpl"
tmpl, err := template.New(filepath.Base(path)).Funcs(funcMap).ParseFiles(path)
if err != nil {
	panic(err)
}

// 4. 执行到缓冲区
var buf strings.Builder
err = tmpl.Execute(&buf, context)
if err != nil {
	panic(err)
}

// 5. 自动格式化（这一步非常关键，会让生成的 Go 代码像手写的一样整齐）
formattedCode, err := format.Source([]byte(buf.String()))
if err != nil {
	fmt.Println("格式化失败，请检查模板语法:", err)
	// 格式化失败时，打印原始代码方便调试
	fmt.Println(buf.String())
	return
}

// 6. 写入文件
os.WriteFile("generated_struct.go", formattedCode, 0644)
```
<?e?>
# 正则表达式
<?e?>
1. 快速匹配判断
2. 字符串是否匹配
3. 查找首个匹配字符串
4. 查找首个匹配的索引位置
5. 查找首个匹配及其分组内容
6. 查找首个匹配及其分组的索引
7. 查找所有字符串匹配项
8. 查找所有字符串匹配项的前 2 个匹配项
9. 查找所有匹配及其分组的索引
10. 匹配 []byte 类型数据
11. 强制解析正则
12. 替换所有匹配为固定字符串
13. 使用函数转换匹配项并替换
14. 下划线转驼峰
15. 驼峰转下划线
<?l?>
```go
package main

import (
	"bytes"
	"fmt"
	"regexp"
	"strings"
)

func main() {
	// 1. 快速匹配判断
	match, _ := regexp.MatchString("p([a-z]+)ch", "peach")
	fmt.Println("1.Match:", match)

	// 解析正则对象
	r, _ := regexp.Compile("p([a-z]+)ch")

	// 2. 正则对象匹配判断
	fmt.Println("2.MatchString:", r.MatchString("peach"))

	// 3. 查找首个匹配字符串
	fmt.Println("3.FindString:", r.FindString("peach punch"))

	// 4. 查找首个匹配的索引位置
	fmt.Println("4.FindStringIndex:", r.FindStringIndex("peach punch"))

	// 5. 查找首个匹配及其分组内容
	fmt.Println("5.FindStringSubmatch:", r.FindStringSubmatch("peach punch"))

	// 6. 查找首个匹配及其分组的索引
	fmt.Println("6.FindStringSubmatchIndex:", r.FindStringSubmatchIndex("peach punch"))

	// 7. 查找所有匹配项
	fmt.Println("7.FindAllString:", r.FindAllString("peach punch pinch", -1))

	// 8. 查找前 2 个匹配项
	fmt.Println("8.FindAllString:", r.FindAllString("peach punch pinch", 2))

	// 9. 查找所有匹配及其分组的索引
	fmt.Println("9.FindAllStringSubmatch:", r.FindAllStringSubmatch("peach punch pinch", -1))
	fmt.Println("9.2FindAllStringSubmatchIndex:", r.FindAllStringSubmatchIndex("peach punch pinch", -1))

	// 10. 匹配 []byte 类型数据
	fmt.Println("10.Match:", r.Match([]byte("peach")))

	// 11. 强制解析正则
	r = regexp.MustCompile("p([a-z]+)ch")
	fmt.Println("11.MustCompile:", r)

	// 12. 替换所有匹配为固定字符串
	fmt.Println("12.ReplaceAllString:", r.ReplaceAllString("a peach a punch", "<fruit>"))

	// 13. 使用函数转换匹配项并替换
	in := []byte("a peach")
	out := r.ReplaceAllFunc(in, bytes.ToUpper)
	fmt.Println("13.ReplaceAllFunc:", string(out))

	// 14.下划线转驼峰
	allFunc := func(str string) string {
		// 正则匹配 [-_] 及其紧跟的一个字母
		reg := regexp.MustCompile(`[-_]+([a-z])`)
		//reg, _ := regexp.Compile("[-_]+([a-z])")
		// 使用 ReplaceAllStringFunc 处理匹配到的部分
		result := reg.ReplaceAllStringFunc(str, func(matched string) string {
			// matched 的值类似于 "_w" 或 "-w"
			// 取最后一位字符转大写并返回
			return strings.ToUpper(matched[len(matched)-1:])
		})
		return result
	}
	// 14.1 测试基础下划线转换
	fmt.Println("14.1.Underline:", allFunc("hello_world")) // helloWorld
	// 14.2 测试中划线转换
	fmt.Println("14.2.Hyphen:", allFunc("api-test-case")) // apiTestCase
	// 14.3 测试多个连接符
	fmt.Println("14.3.Multi:", allFunc("data___source")) // dataSource

	// 15.驼峰转下划线
	humpToLine := func(str string) string {
		result := regexp.MustCompile(`[A-Z]{1}`).ReplaceAllStringFunc(str, func(matched string) string {
			return "_" + strings.ToLower(matched)
		})
		return result
	}
	// 15.1 测试基础下划线转换
	fmt.Println("15.1.helloWorld:", humpToLine("helloWorld")) // helloWorld
	// 15.2 测试中划线转换
	fmt.Println("15.2.apiTestCase:", humpToLine("apiTestCase")) // apiTestCase
	// 15.3 测试多个连接符
	fmt.Println("15.3.dataSource:", humpToLine("dataSource")) // dataSource
}
```
输出
```
1.Match: true
2.MatchString: true
3.FindString: peach
4.FindStringIndex: [0 5]
5.FindStringSubmatch: [peach ea]
6.FindStringSubmatchIndex: [0 5 1 3]
7.FindAllString: [peach punch pinch]
8.FindAllString: [peach punch]
9.FindAllStringSubmatch: [[peach ea] [punch un] [pinch in]]
9.2FindAllStringSubmatchIndex: [[0 5 1 3] [6 11 7 9] [12 17 13 15]]
10.Match: true
11.MustCompile: p([a-z]+)ch
12.ReplaceAllString: a <fruit> a <fruit>
13.ReplaceAllFunc: a PEACH
14.1.Underline: helloWorld
14.2.Hyphen: apiTestCase
14.3.Multi: dataSource
15.1.helloWorld: hello_world
15.2.apiTestCase: api_test_case
15.3.dataSource: data_source
```
<!--SR:!2026-05-10,68,272-->
<?e?>
# JSON
<?e?>
1. Struct 序列化为 JSON 对象
2. JSON 对象 反序列化为 Struct
<?l?>
```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

// 基础结构体：默认使用字段名作为 JSON 的 Key
type response1 struct {
	Page   int
	Fruits []string
}

// 标签结构体：使用 `json:"key"` 自定义 JSON 中的键名
type response2 struct {
	Page   int      `json:"page"`
	Fruits []string `json:"fruits"`
}

func main() {
	// --- 序列化部分 ---
	// 1. 基础类型序列化 (bool)
	bolB, _ := json.Marshal(true)
	fmt.Println("1.Bool:", string(bolB))
	// 2. 基础类型序列化 (int)
	intB, _ := json.Marshal(1)
	fmt.Println("2.Int:", string(intB))
	// 3. 基础类型序列化 (float)
	fltB, _ := json.Marshal(2.34)
	fmt.Println("3.Float:", string(fltB))
	// 4. 基础类型序列化 (string)
	strB, _ := json.Marshal("gopher")
	fmt.Println("4.String:", string(strB))
	// 5. 切片序列化为 JSON 数组
	slcD := []string{"apple", "peach", "pear"}
	slcB, _ := json.Marshal(slcD)
	fmt.Println("5.Slice:", string(slcB))
	// 6. Map 序列化为 JSON 对象
	mapD := map[string]int{"apple": 5, "lettuce": 7}
	mapB, _ := json.Marshal(mapD)
	fmt.Println("6.Map:", string(mapB))
	// 7. 结构体序列化 (使用默认字段名, 字段名要大写才能序列化)
	res1D := &response1{
		Page:   1,
		Fruits: []string{"apple", "peach", "pear"}}
	res1B, _ := json.Marshal(res1D)
	fmt.Println("7.Struct1:", string(res1B))
	// 8. 结构体序列化 (使用 json 标签 可以自定义json 键名)
	res2D := &response2{
		Page:   1,
		Fruits: []string{"apple", "peach", "pear"}}
	res2B, _ := json.Marshal(res2D)
	fmt.Println("8.Struct2:", string(res2B))

	// --- 反序列化部分 ---
	byt := []byte(`{"num":6.13,"strs":["a","b"]}`)
	// 9. 解析到通用 map (未知结构时使用)
	var dat map[string]interface{}
	if err := json.Unmarshal(byt, &dat); err != nil {
		panic(err)
	}
	fmt.Println("9.UnmarshalMap:", dat)
	// 10. 类型断言获取 map 中的数值 (JSON 数字默认解析为 float64)
	num := dat["num"].(float64)
	fmt.Println("10.AssertNum:", num)
	// 11. 嵌套数据的类型断言
	strs := dat["strs"].([]interface{})
	str1 := strs[0].(string)
	fmt.Println("11.AssertStr:", str1)
	// 12. 解析到具体结构体 (推荐做法)
	str := `{"page": 1, "fruits": ["apple", "peach"]}`
	res := response2{}
	json.Unmarshal([]byte(str), &res)
	fmt.Println("12.UnmarshalStruct:", res)
	fmt.Println("13.AccessField:", res.Fruits[0])
	// 13. 使用 Encoder 直接流式输出到标准输出 (os.Stdout)
	enc := json.NewEncoder(os.Stdout)
	d := map[string]int{"apple": 5, "lettuce": 7}
	fmt.Print("14.EncoderOutput: ")
	enc.Encode(d)
}
```
输出
```
1.Bool: true
2.Int: 1
3.Float: 2.34
4.String: "gopher"
5.Slice: ["apple","peach","pear"]
6.Map: {"apple":5,"lettuce":7}
7.Struct1: {"Page":1,"Fruits":["apple","peach","pear"]}
8.Struct2: {"page":1,"fruits":["apple","peach","pear"]}
9.UnmarshalMap: map[num:6.13 strs:[a b]]
10.AssertNum: 6.13
11.AssertStr: a
12.UnmarshalStruct: {1 [apple peach]}
13.AccessField: apple
14.EncoderOutput: {"apple":5,"lettuce":7}
```
<!--SR:!2026-07-03,111,292-->
<?e?>
# XML
<?e?>
1. 序列化 XML (带缩进格式)
2. 序列化并添加 XML 标准头部声明
3. 反序列化：将 XML 字节流转回结构体
4. 序列化嵌套结构
5. 反序列化时字段转换失败的原因?
<?l?>
```go
package main

import (
	"encoding/xml"
	"fmt"
)

// Plant 结构体定义了 XML 的映射规则
type Plant struct {
	XMLName xml.Name `xml:"plant"`   // 指定根元素标签名为 plant「💡必须，否则不能序列化」
	Id      int      `xml:"id,attr"` // id 映射为 XML 属性 (attr)
	Name    string   `xml:"name"`    // Name 映射为子元素 <name>
	Origin  []string `xml:"origin"`  // 切片映射为多个 <origin> 标签
}

// String 方法用于自定义结构体的字符串输出格式
func (p Plant) String() string {
	return fmt.Sprintf("Plant id=%v, name=%v, origin=%v",
		p.Id, p.Name, p.Origin)
}

func main() {
	coffee := &Plant{Id: 27, Name: "Coffee"}
	coffee.Origin = []string{"Ethiopia", "Brazil"}

	// 1. 序列化 XML (带缩进格式)
	out, _ := xml.MarshalIndent(coffee, " ", "  ")
	fmt.Println("1.Marshal:\n"+string(out), "\n")

	// 2. 序列化并添加 XML 标准头部声明
	fmt.Println("2.WithHeader:\n"+xml.Header+string(out), "\n")

	// 3. 反序列化：将 XML 字节流转回结构体
	var p Plant
	if err := xml.Unmarshal(out, &p); err != nil {
		panic(err)
	}
	fmt.Println("3.Unmarshal:", p, "\n")

	tomato := &Plant{Id: 81, Name: "Tomato"}
	tomato.Origin = []string{"Mexico", "California"}

	// Nesting 演示复杂的嵌套结构映射
	type Nesting struct {
		XMLName xml.Name `xml:"nesting"`
		// 关键点：使用 > 符号创建嵌套层级 <parent><child><plant>...
		Plants []*Plant `xml:"parent>child>plant"`
	}

	nesting := &Nesting{}
	nesting.Plants = []*Plant{coffee, tomato}

	// 4. 序列化嵌套结构
	out, _ = xml.MarshalIndent(nesting, " ", "  ")
	fmt.Println("4.Nested:\n"+string(out), "\n")
	
	/* 5. 灯泡反序列化时字段转换失败的原因?
		1. 字段未导出：结构体字段首字母必须大写，否则 `json` 包通过反射无法访问和写入。
		2. Tag 标签不匹配：JSON 中的 Key 与结构体字段名或 `json:"..."` 标签定义的名称不一致。
		3. 数据类型不兼容：例如 JSON 传的是字符串 `"100"`，而 Go 字段定义为 `int`（且未加 `,string` 标签）。
		4. 嵌套层次错误：JSON 的层级结构与 Go 结构体的嵌套层级（Struct nesting）没有对应上。
		5. 空值处理异常：JSON 传了 `null`，但 Go 目标字段是非指针类型（如 `int`、`struct`），无法接收空值。
		6. 自定义解析报错：字段实现了 `UnmarshalJSON` 接口，但自定义的逻辑执行时抛出了错误。
	*/
}
```
输出
```
1.Marshal:
 <plant id="27">
   <name>Coffee</name>
   <origin>Ethiopia</origin>
   <origin>Brazil</origin>
 </plant> 

2.WithHeader:
<?xml version="1.0" encoding="UTF-8"?>
 <plant id="27">
   <name>Coffee</name>
   <origin>Ethiopia</origin>
   <origin>Brazil</origin>
 </plant> 

3.Unmarshal: Plant id=27, name=Coffee, origin=[Ethiopia Brazil] 

4.Nested:
 <nesting>
   <parent>
     <child>
       <plant id="27">
         <name>Coffee</name>
         <origin>Ethiopia</origin>
         <origin>Brazil</origin>
       </plant>
       <plant id="81">
         <name>Tomato</name>
         <origin>Mexico</origin>
         <origin>California</origin>
       </plant>
     </child>
   </parent>
 </nesting> 
```
<!--SR:!2026-04-23,29,232-->
<?e?>
# 时间
<?e?>
1. 获取当前本地时间
2. 构造一个特定的时间点 (年, 月, 日, 时, 分, 秒, 纳秒, 时区)
3. 提取时间分量
4. 获取星期几
5. 时间点比较 (早于、晚于、相等)
6. 计算两个时间点的时间差 (Duration)
7. 时间点的加法与减法偏移
<?l?>
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 1. 获取当前本地时间
	now := time.Now()
	fmt.Println("1.Now:", now)

	// 2. 构造一个特定的时间点 (年, 月, 日, 时, 分, 秒, 纳秒, 时区)
	then := time.Date(2025, 11, 17, 20, 34, 58, 651387237, time.UTC)
	fmt.Println("2.Then:", then)

	// 3. 提取时间分量
	fmt.Println("3.1Year:", then.Year())
	fmt.Println("3.2Month:", then.Month())
	fmt.Println("3.3Day:", then.Day())
	fmt.Println("3.4Hour:", then.Hour())
	fmt.Println("3.5Minute:", then.Minute())
	fmt.Println("3.6Second:", then.Second())
	fmt.Println("3.7Nanosecond:", then.Nanosecond())
	fmt.Println("3.8Location:", then.Location()) // 获取时区
	fmt.Println("3.9Weekday:", then.Weekday())

	// 4. 时间点比较 (早于、晚于、相等)
	fmt.Println("4.1Before:", then.Before(now))
	fmt.Println("4.2After:", then.After(now))
	fmt.Println("4.3Equal:", then.Equal(now))

	// 5. 计算两个时间点的时间差 (Duration)
	diff := now.Sub(then)
	fmt.Println("5.Diff:", diff)
	fmt.Println("5.Diff Hours:", int(diff.Hours()))
	fmt.Println("5.Diff Minutes:", int(diff.Minutes()))
	fmt.Println("5.Diff Seconds:", int(diff.Seconds()))
	fmt.Println("5.Diff Nanoseconds:", int(diff.Nanoseconds()))

	// 6. 时间点的加法与减法偏移
	fmt.Println("6.AddDiff:", then.Add(diff))  // 加上时间差
	fmt.Println("6.SubDiff:", then.Add(-diff)) // 减去时间差
}
```
输出
```
1.Now: 2026-01-22 15:48:56.254889 +0800 CST m=+0.000038543
2.Then: 2025-11-17 20:34:58.651387237 +0000 UTC
3.1Year: 2025
3.2Month: November
3.3Day: 17
3.4Hour: 20
3.5Minute: 34
3.6Second: 58
3.7Nanosecond: 651387237
3.8Location: UTC
3.9Weekday: Monday
4.1Before: true
4.2After: false
4.3Equal: false
5.Diff: 1571h13m57.603501763s
5.Diff Hours: 1571
5.Diff Minutes: 94273
5.Diff Seconds: 5656437
5.Diff Nanoseconds: 5656437603501763
6.AddDiff: 2026-01-22 07:48:56.254889 +0000 UTC
6.SubDiff: 2025-09-13 09:21:01.047885474 +0000 UTC
```
<!--SR:!2026-04-25,58,252-->
<?e?>
# 时间戳
<?e?>
1. 获取当前时间对象
2. 获取 Unix 时间戳（自 1970-01-01 以来的秒数）
3. 获取纳秒级时间戳
4. 将纳秒转换为毫秒 (1毫秒 = 1,000,000纳秒)
5. 将秒级时间戳还原为 time.Time 对象
6. 将纳秒级时间戳还原为 time.Time 对象
7. 将字符串还原为 time.Time 对象
<?l?>
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 1. 获取当前时间对象
	now := time.Now()
	fmt.Println("1.Now:", now)
	// 2. 获取 Unix 时间戳（自 1970-01-01 以来的秒数）
	secs := now.Unix()
	// 3. 获取纳秒级时间戳
	nanos := now.UnixNano()
	// 4. 将纳秒转换为毫秒 (1毫秒 = 1,000,000纳秒)
	millis := nanos / 1000000
	fmt.Println("2.Seconds:", secs)
	fmt.Println("3.Milliseconds:", millis)
	fmt.Println("4.Nanoseconds:", nanos)
	// 5. 将秒级时间戳还原为 time.Time 对象
	// time.Unix(秒, 纳秒)
	fmt.Println("5.Seconds Transform Time:", time.Unix(secs, 0))
	// 6. 将纳秒级时间戳还原为 time.Time 对象
	fmt.Println("6.Nanos Transform Time:", time.Unix(0, nanos))
	// 7. 将字符串还原为 time.Time 对象
	parse, _ := time.Parse(time.DateTime, "2026-01-01 00:00:00")
	fmt.Println("7.String Transform Time:", parse)
}
```
输出
```
1.Now: 2026-01-22 15:49:43.084436 +0800 CST m=+0.000038792
2.Seconds: 1769068183
3.Milliseconds: 1769068183084
4.Nanoseconds: 1769068183084436000
5.Seconds Transform Time: 2026-01-22 15:49:43 +0800 CST
6.Nanos Transform Time: 2026-01-22 15:49:43.084436 +0800 CST
7.String Transform Time: 2026-01-01 00:00:00 +0000 UTC
```
<!--SR:!2026-03-29,42,292-->
<?e?>
# 时间的格式化和解析
<?e?>
1. 使用预定义的 RFC3339 标准格式输出
2. 解析字符串为时间对象
3. 自定义格式化：只输出 12 小时制的时间
4. 自定义格式化：全日期时间（含星期、月份、日、时、分、秒、年）
5. 自定义格式化：精确到微秒及偏移量
6. 错误处理演示：解析格式不匹配时会返回错误
7. 获取时间对象的各个字段
<?l?>
```go
t := time.Now()
// 1. 使用预定义的 RFC3339 标准格式输出
fmt.Println("1.RFC3339:", t.Format(time.RFC3339))

// 2. 解析字符串为时间对象
t1, _ := time.Parse(time.RFC3339, "2012-11-01T22:08:41+00:00")
fmt.Println("2.Parse:", t1)

// 3. 自定义格式化：只输出 12 小时制的时间
fmt.Println("3.Format-Time:", t.Format(time.Kitchen))

// 4. 自定义格式化：全日期时间（含星期、月份、日、时、分、秒、年）
fmt.Println("4.Format-Full:", t.Format(time.ANSIC))

// 5. 自定义格式化：精确到微秒及偏移量
fmt.Println("5.Format-Micro:", t.Format(time.RFC3339Nano))

// 6. 使用自定义布局解析字符串
t2, _ := time.Parse("3 04 PM", "8 41 PM")
fmt.Println("6.CustomParse:", t2)

// 7. 使用 Printf 进行传统的手动格式化拼接
fmt.Printf(
	"7.Printf: %d-%02d-%02dT%02d:%02d:%02d-00:00\n",
	t.Year(), t.Month(), t.Day(), t.Hour(), t.Minute(), t.Second(),
)

// 8. 错误处理演示：解析格式不匹配时会返回错误
_, e := time.Parse(time.ANSIC, "8:41PM")
fmt.Println("8.ParseError:", e) // 格式不匹配，输出错误信息

// 9. 获取时间对象的各个字段
timeStr := "2025-12-10 13:09:24"
t3, _ := time.Parse(time.DateTime, timeStr)
fmt.Printf(
	"9.Custome %d-%02d-%02d %02d:%02d:%02d",
	t3.Year(), t3.Month(), t3.Day(), t3.Hour(), t3.Minute(), t3.Second(),
)
```
输出
```
1.RFC3339: 2026-02-03T17:51:12+08:00
2.Parse: 2012-11-01 22:08:41 +0000 +0000
3.Format-Time: 5:51PM
4.Format-Full: Tue Feb  3 17:51:12 2026
5.Format-Micro: 2026-02-03T17:51:12.522564+08:00
6.CustomParse: 0000-01-01 20:41:00 +0000 UTC
7.Printf: 2026-02-03T17:51:12-00:00
8.ParseError: parsing time "8:41PM" as "Mon Jan _2 15:04:05 2006": cannot parse "8:41PM" as "Mon"
9.Custome 2025-12-10 13:09:24
```
<!--SR:!2026-05-04,40,252-->
<?e?>
# 随机数
<?e?>
1. 生成 0-99 之间的随机整数（使用默认种子，多次运行结果相同）
2. 生成 0.0 到 1.0 之间的随机浮点数
3. 生成指定范围的浮点数，例如 `[5.0, 10.0)`
4. 使用当前纳秒时间戳作为种子创建私有的随机数生成器
5. 使用固定的种子（如 42）创建生成器
6. 相同种子生成的随机序列完全一致
<?l?>
```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	// 1. 生成 0-99 之间的随机整数（使用默认种子，多次运行结果相同）
	fmt.Println("1.DefaultSource: ", rand.Intn(100))

	// 2. 生成 0.0 到 1.0 之间的随机浮点数
	fmt.Println("2.Float64: ", rand.Float64())

	// 3. 生成指定范围的浮点数，例如 [5.0, 10.0)
	fmt.Println("3.RangeFloat: ", (rand.Float64()*5)+5)

	// 4. 使用当前纳秒时间戳作为种子创建私有的随机数生成器
	// 这样每次程序运行生成的随机数序列都会不同
	r1 := rand.New(rand.NewSource(time.Now().UnixNano()))
	fmt.Println("4.TimeSeed: ", r1.Intn(100))

	// 5. 使用固定的种子（如 42）创建生成器
	r2 := rand.New(rand.NewSource(42))
	fmt.Println("5.FixedSeed1: ", r2.Intn(100))

	// 6. 相同种子生成的随机序列完全一致
	r3 := rand.New(rand.NewSource(42))
	fmt.Println("6.FixedSeed2: ", r3.Intn(100))
}
```
输出
```
1.DefaultSource:  42
2.Float64:  0.12875004654472783
3.RangeFloat:  5.922369276340139
4.TimeSeed:  49
5.FixedSeed1:  5
6.FixedSeed2:  5
```
<!--SR:!2026-04-10,45,252-->
<?e?>
# 数字解析
<?e?>
1. 将字符串解析为 64 位浮点数
2. 解析整数
3. 解析十六进制字符串
4. 解析无符号整数
5. Atoi：常用的快捷函数，将字符串转为十进制 int (等价于 ParseInt(s, 10, 0))
6. 错误处理演示：解析非数字字符串会返回错误
<?l?>
```go
package main

import (
	"fmt"
	"strconv" // 导入字符串转换包
)

func main() {
	// 1. 将字符串解析为 64 位浮点数
	f, _ := strconv.ParseFloat("1.234", 64)
	fmt.Println("1.ParseFloat:", f)

	// 2. 解析整数：strconv.ParseInt(字符串, 进制, 位数)
	// 进制设为 0 表示通过字符串前缀推断（如 0x 是十六进制）
	i, _ := strconv.ParseInt("123", 0, 64)
	fmt.Println("2.ParseInt:", i)

	// 3. 解析十六进制字符串
	d, _ := strconv.ParseInt("0x1c8", 0, 64)
	fmt.Println("3.ParseHex:", d)

	// 4. 解析无符号整数
	u, _ := strconv.ParseUint("789", 0, 64)
	fmt.Println("4.ParseUint:", u)

	// 5. Atoi：常用的快捷函数，将字符串转为十进制 int (等价于 ParseInt(s, 10, 0))
	k, _ := strconv.Atoi("135")
	fmt.Println("5.Atoi:", k)

	// 6. 错误处理演示：解析非数字字符串会返回错误
	_, e := strconv.Atoi("wat")
	fmt.Println("6.Error:", e)
}
```
输出
```
1.ParseFloat: 1.234
2.ParseInt: 123
3.ParseHex: 456
4.ParseUint: 789
5.Atoi: 135
6.Error: strconv.Atoi: parsing "wat": invalid syntax
```
<!--SR:!2026-05-29,84,272-->
<?e?>
# URL 解析
<?e?>
1. 解析 URL 字符串为 url.URL 对象
2. 获取协议方案 (Scheme)
3. 获取用户信息 (User)，包含用户名和密码
4. 获取主机名、端口
5. 获取路径、片段 (`#`之后的内容)
6. 获取原始查询参数字符串
7. 将查询参数解析为 `map[string][]string` 结构
<?l?>
```go
package main

import (
	"fmt"
	"net"
	"net/url"
)

func main() {
	// 定义一个复杂的测试 URL
	s := "postgres://user:pass@host.com:5432/path?k=v#f"

	// 1. 解析 URL 字符串为 url.URL 对象
	u, _ := url.Parse(s)
	fmt.Println("1.Parse:", u)

	// 2. 获取协议方案 (Scheme)
	fmt.Println("2.Scheme:", u.Scheme)

	// 3. 获取用户信息 (User)，包含用户名和密码
	fmt.Println("3.1UserObj:", u.User)
	fmt.Println("3.2Username:", u.User.Username())
	p, _ := u.User.Password()
	fmt.Println("3.3Password:", p)

	// 4. 获取主机名和端口
	fmt.Println("4.1HostWithPort:", u.Host)
	// 使用 net.SplitHostPort 分离主机名和端口号
	host, port, _ := net.SplitHostPort(u.Host)
	fmt.Println("4.2HostOnly:", host)
	fmt.Println("4.3PortOnly:", port)

	// 5. 获取路径和片段 (#之后的内容)
	fmt.Println("5.1Path:", u.Path)
	fmt.Println("5.2Fragment:", u.Fragment)

	// 6. 获取原始查询参数字符串
	fmt.Println("6.1RawQuery:", u.RawQuery)
	// 将查询参数解析为 map[string][]string 结构
	m, _ := url.ParseQuery(u.RawQuery)
	fmt.Println("6.2QueryMap:", m)
	fmt.Println("6.3QueryValue:", m["k"][0])
}

```
输出
```
1.Parse: postgres://user:pass@host.com:5432/path?k=v#f
2.Scheme: postgres
3.1UserObj: user:pass
3.2Username: user
3.3Password: pass
4.1HostWithPort: host.com:5432
4.2HostOnly: host.com
4.3PortOnly: 5432
5.1Path: /path
5.2Fragment: f
6.1RawQuery: k=v
6.2QueryMap: map[k:[v]]
6.3QueryValue: v
```
<!--SR:!2026-04-10,16,232-->
<?e?>
# SHA256 散列
<?e?>
1. 创建一个新的 sha256 算法句柄 (hash.Hash 接口)
2. 写入数据：将字符串转换为字节切片并传入
3. 计算最终哈希值
4. 格式化输出：使用 %x 将字节切片打印为十六进制字符串
<?l?>
```go
package main

import (
	"crypto/sha256" // 导入加密级哈希包
	"fmt"
)

func main() {
	s := "sha256 this string"

	// 1. 创建一个新的 sha256 算法句柄 (hash.Hash 接口)
	h := sha256.New()

	// 2. 写入数据：将字符串转换为字节切片并传入
	// 可以多次调用 Write 来追加数据
	h.Write([]byte(s))

	// 3. 计算最终哈希值
	// Sum 的参数用于追加现有字节切片，通常传 nil 即可
	bs := h.Sum(nil)

	// 4. 格式化输出：使用 %x 将字节切片打印为十六进制字符串
	fmt.Printf("SHA256: %x\n", bs)
}
```
输出
```
SHA256: 1af1dfa857bf1d8814fe1af8983c18080019922e557f15a8a0d3db739d77aacb
```
<!--SR:!2026-07-24,123,292-->
<?e?>
# Base64 编码
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	// 准备包含特殊字符的原始数据
	data := "abc123!?$*&()'-=@~+/"

	// 1. 标准 Base64 编码 (Standard Encoding)
	// 常用于电子邮件等传统场景，包含 + 和 / 字符
	sEnc := base64.StdEncoding.EncodeToString([]byte(data))
	fmt.Println("1.StdEncode:", sEnc)

	// 2. 标准 Base64 解码
	sDec, _ := base64.StdEncoding.DecodeString(sEnc)
	fmt.Println("2.StdDecode:", string(sDec))
	fmt.Println()

	// 3. URL 安全型 Base64 编码 (URL Encoding)
	// 将标准编码中的 + 替换为 -，/ 替换为 _，适合放在 URL 中
	uEnc := base64.URLEncoding.EncodeToString([]byte(data))
	fmt.Println("3.URLEncode:", uEnc)

	// 4. URL 安全型 Base64 解码
	uDec, _ := base64.URLEncoding.DecodeString(uEnc)
	fmt.Println("4.URLDecode:", string(uDec))
}
```
输出
```
1.StdEncode: YWJjMTIzIT8kKiYoKSctPUB+Ky8=
2.StdDecode: abc123!?$*&()'-=@~+/

3.URLEncode: YWJjMTIzIT8kKiYoKSctPUB-Ky8=
4.URLDecode: abc123!?$*&()'-=@~+/
```
# 读文件
<?e?>
1. 一次性读取：直接将整个文件内容读入内存
2. 打开文件：获取文件句柄以进行更多控制
3. 基础读取：从文件起始位置读取 5 个字节
4. 指定位置读取 (Seek)
5. 确保读取最少数量：`io.ReadAtLeast`
6. 重置文件指针到开头
7. 带缓冲读取 (bufio)
<?l?>
```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

// 辅助函数：简化错误检查逻辑
func check(e error) {
	if e != nil {
		panic(e)
	}
}

func main() {
	// 1. 一次性读取：直接将整个文件内容读入内存
	// 适用于小文件，最简单的方法
	dat, err := os.ReadFile("t2.txt")
	check(err)
	fmt.Print("1.ReadFile: ", string(dat))

	// 2. 打开文件：获取文件句柄以进行更多控制
	f, err := os.Open("t2.txt")
	check(err)

	// 3. 基础读取：从文件起始位置读取 5 个字节
	b1 := make([]byte, 5)
	n1, err := f.Read(b1)
	check(err)
	fmt.Printf("3. Read5Bytes: %d bytes: %s\n", n1, string(b1[:n1]))

	// 4. 指定位置读取 (Seek)：
	// f.Seek(偏移量, 起始位置)；0 表示从文件头开始，6 表示跳过前 6 字节
	o2, err := f.Seek(6, 0)
	check(err)
	b2 := make([]byte, 2)
	n2, err := f.Read(b2)
	check(err)
	fmt.Printf("4. SeekAndRead: %d bytes @ %d: %s\n", n2, o2, string(b2[:n2]))

	// 5. 确保读取最少数量：io.ReadAtLeast
	// 至少读满 b3 (2字节)，否则会报错
	o3, err := f.Seek(6, 0)
	check(err)
	b3 := make([]byte, 2)
	n3, err := io.ReadAtLeast(f, b3, 2)
	check(err)
	fmt.Printf("5. ReadAtLeast: %d bytes @ %d: %s\n", n3, o3, string(b3))

	// 6. 重置文件指针到开头
	_, err = f.Seek(0, 0)
	check(err)

	/* 7. 带缓冲读取 (bufio)
	   原理：预从磁盘拉取 4096 字节存入内存，后续读取优先消耗内存，减少系统调用。
	   默认 4096 字节的缓冲区，等同于 bufio.NewReaderSize(f, 4096)
	*/
	r4 := bufio.NewReader(f)
	/* 7.1 首个 Peek
	   1. 物理 IO：触发磁盘读取，填充 4KB 缓冲区。
	   2. 文件指针：OS 句柄指针后移 4096 字节（或文件末尾）。
	   3. 读取指针：bufio 内部偏移量保持 0（数据未被消耗）。
	   4. 💡注意：Peek 长度必须 ≤ 缓冲区容量（4096）。
	*/
	b4, err := r4.Peek(5)
	check(err)
	fmt.Printf("7.1 Peek(5): %s\n", string(b4))
	s, _ := f.Seek(0, io.SeekCurrent)
	fmt.Printf("7.1 OS指针位置: %d\n", s) // 此时通常显示为 4096

	// 重置底层文件指针（💡注意：此时 bufio 缓冲区内仍存有旧数据）
	f.Seek(0, 0)

	/* 7.2 后续 Peek
	   1. 内存操作：直接从现有的 4KB 缓冲区进行切片拷贝。
	   2. 独立性：由于数据已在内存，即便底层文件指针被移动，Peek 结果依然是缓冲区旧数据。
	*/
	b5, err := r4.Peek(5)
	check(err)
	fmt.Printf("7.2 Peek(5): %s\n", string(b5))
	s2, _ := f.Seek(0, io.SeekCurrent)
	fmt.Printf("7.2 OS指针位置: %d\n", s2) // 依然是之前 Seek(0,0) 的位置

	// 8. 必须手动关闭文件句柄，释放系统资源
	f.Close()
}
```
输出
```
1.ReadFile: hello go defer
3. Read5Bytes: 5 bytes: hello
4. SeekAndRead: 2 bytes @ 6: go
5. ReadAtLeast: 2 bytes @ 6: go
7.1 Peek(5): hello
7.1 OS指针位置: 15
7.2 Peek(5): hello
7.2 OS指针位置: 0
```
<!--SR:!2026-03-31,35,232-->
<?e?>
# 写文件
<?e?>
1. 快速写入文件：直接将字节切片写入指定路径
2. 创建文件：获取一个可写的句柄
3. 写入字节切片 (Byte Slice)
4. 直接写入字符串 (String)
5. 刷新到磁盘：将操作系统的缓冲区内容物理写入硬盘
6. 带缓冲的写入 (bufio)：
7. 必须调用 Flush：bufio 只是把数据写到了内存缓冲区，只有调用 Flush 才会真正写给底层的 f (os.File)
<?l?>
```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"time"
)

// 辅助函数：简化错误检查
func check2(e error) {
	if e != nil {
		panic(e)
	}
}

func main() {
	// 1. 快速写入文件：直接将字节切片写入指定路径
	// 0644 是 Unix 权限位（属主读写，其他读）
	d1 := []byte("hello\ngo\n")
	err := os.WriteFile("/tmp/dat1", d1, 0644)
	check2(err)

	// 2. 创建文件：获取一个可写的句柄
	f, err := os.Create("/tmp/dat2")
	check2(err)

	// 使用 defer 确保在函数退出时关闭文件，释放资源
	defer f.Close()

	// 3. 写入字节切片 (Byte Slice)
	d2 := []byte{115, 111, 109, 101, 10} // 对应 "some\n"
	n2, err := f.Write(d2)
	check2(err)
	fmt.Printf("3. WriteBytes: wrote %d bytes\n", n2)

	// 4. 直接写入字符串 (String)
	n3, err := f.WriteString("writes\n")
	check2(err)
	fmt.Printf("4. WriteString: wrote %d bytes\n", n3)

	// 5. 刷新到磁盘：将操作系统的缓冲区内容物理写入硬盘
	f.Sync()

	// 6. 带缓冲的写入 (bufio)：
	// 适用于频繁进行细小写入的操作，能减少系统调用次数，提高性能
	w := bufio.NewWriter(f)
	n4, err := w.WriteString("buffered\n")
	check2(err)
	fmt.Printf("6.1 BufioWrite: wrote %d bytes\n", n4)

	// 此时去查看文件，内容是空的
	time.Sleep(5 * time.Second)

	// 7. 必须调用 Flush：bufio 只是把数据写到了内存缓冲区，只有调用 Flush 才会真正写给底层的 f (os.File)
	w.Flush()
	// 此时查看文件，内容出现了
	fmt.Println("6.2 Flush 已调用，现在能看到内容了")
}
```
输出
```
3.WriteBytes: wrote 5 bytes
4.WriteString: wrote 7 bytes
6.1BufioWrite: wrote 9 bytes
6.2Flush 已调用，现在能看到内容了
```
<!--SR:!2026-05-09,67,271-->
<?e?>
# 行过滤器
<?e?>
1. 创建文件输入扫描器
2. 循环逐行读取输入
3. 检查扫描过程中是否发生错误
<?l?>
```go
// 定义主包
package main

// 导入缓冲区 IO、格式化输出和系统相关包
import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

// 主函数入口
func main() {
	fd, _ := os.OpenFile("t1.txt", os.O_RDWR, 0644)
	defer fd.Close()
	// 创建文件输入扫描器
	scanner := bufio.NewScanner(fd)
	//scanner := bufio.NewScanner(os.Stdin) // 创建标准输入扫描器
	// 循环逐行读取输入
	for scanner.Scan() {
		// 将读取的文本转换为大写
		ucl := strings.ToUpper(scanner.Text())
		// 输出转换后的字符串
		fmt.Println(ucl)
	}
	// 检查扫描过程中是否发生错误
	if err := scanner.Err(); err != nil {
		// 将错误信息输出到标准错误流
		_, _ = fmt.Fprintln(os.Stderr, "error:", err)
		// 以状态码 1 退出程序
		os.Exit(1)
	}
}
```
输出
```
AAAA
BBBB
CCCC
DDDD
```
<!--SR:!2026-04-28,60,251-->
<?e?>
# 文件路径
<?e?>
1. 路径拼接与清理
2. 路径成分提取「目录名、文件名」
3. 相对路径检查
4. 文件名与扩展名处理
5. 计算相对路径
6. 文件状态与元数据
<?l?>
```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
	"strings"
)

func main() {
	// 1. 路径拼接与清理
	// filepath.Join 会根据操作系统自动选择分隔符 (Unix: /, Windows: \)
	p := filepath.Join("dir1", "dir2", "filename")
	fmt.Printf("拼接路径: %s\n", p)

	fmt.Printf("自动清理多余斜杠: %s\n", filepath.Join("dir1//", "filename"))
	fmt.Printf("自动解析 .. 路径: %s\n", filepath.Join("dir1/../dir1", "filename"))

	fmt.Println(strings.Repeat("-", 30))

	// 2. 路径成分提取
	fmt.Printf("目录部分 (Dir): %s\n", filepath.Dir(p))
	fmt.Printf("文件名部分 (Base): %s\n", filepath.Base(p))

	// 3. 相对路径检查
	fmt.Printf("Is 'dir/file' absolute? %v\n", filepath.IsAbs("dir/file"))
	fmt.Printf("Is '/dir/file' absolute? %v\n", filepath.IsAbs("/dir/file"))

	// 4. 文件名与扩展名处理
	filename := "config.json"
	ext := filepath.Ext(filename) // 结果包含点号 "."
	fmt.Printf("扩展名: %s\n", ext)
	fmt.Printf("主文件名: %s\n", strings.TrimSuffix(filename, ext))

	fmt.Println(strings.Repeat("-", 30))

	// 5. 计算相对路径
	// 计算从 base 到 target 的相对路径
	rel1, _ := filepath.Rel("a/b", "a/b/t/file")
	fmt.Printf("同级相对路径: %s\n", rel1)

	rel2, _ := filepath.Rel("a/b", "a/c/t/file")
	fmt.Printf("跨级相对路径: %s\n", rel2)

	fmt.Println(strings.Repeat("-", 30))

	// 6. 文件状态与元数据
	// 建议先检查错误，再使用 stat 对象
	targetFile := "t1.txt"
	stat, err := os.Stat(targetFile)
	if err != nil {
		if os.IsNotExist(err) {
			fmt.Printf("错误: 文件 '%s' 不存在\n", targetFile)
		} else {
			fmt.Printf("读取错误: %v\n", err)
		}
		return
	}

	fmt.Println("文件详细信息:")
	fmt.Printf("  名称:     %v\n", stat.Name())
	fmt.Printf("  大小:     %v 字节\n", stat.Size())
	fmt.Printf("  权限模式: %v\n", stat.Mode())
	fmt.Printf("  修改时间: %v\n", stat.ModTime())
	fmt.Printf("  是否目录: %v\n", stat.IsDir())
	fmt.Printf("  系统底层数据: %v\n", stat.Sys())
}
```
输出
```
拼接路径: dir1/dir2/filename
自动清理多余斜杠: dir1/filename
自动解析 .. 路径: dir1/filename
------------------------------
目录部分 (Dir): dir1/dir2
文件名部分 (Base): filename
Is 'dir/file' absolute? false
Is '/dir/file' absolute? true
扩展名: .json
主文件名: config
------------------------------
同级相对路径: t/file
跨级相对路径: ../c/t/file
------------------------------
文件详细信息:
  名称:     t1.txt
  大小:     19 字节
  权限模式: -rw-r--r--
  修改时间: 2026-01-09 14:53:44.091188997 +0800 CST
  是否目录: false
  系统底层数据: &{16777231 33188 1 61802447 501 20 0 [0 0 0 0] {1767991750 990384404} {1767941624 91188997} {1767941624 91188997} {1767869716 625129705} 19 8 4096 0 0 0 [0 0]}
```
<!--SR:!2026-04-24,52,251-->
<?e?>
# 目录
<?e?>
1. 基础目录操作
2. 构建复杂的层级结构
3. 读取单层目录
4. 工作目录切换
5. 递归遍历目录树
<?l?>
```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
	"time"
)

// checkErr 统一错误处理逻辑
func checkErr(e error) {
	if e != nil {
		panic(e)
	}
}

func main() {
	// 1. 基础目录操作 (Mkdir & RemoveAll)
	const rootDir = "subdir"
	// 创建单级目录，设置权限为 0755
	err := os.Mkdir(rootDir, 0755)
	checkErr(err)

	// 使用 defer 确保程序退出时清理环境（即使发生 panic 也会执行）
	defer os.RemoveAll(rootDir)

	// 定义创建空文件的辅助闭包
	createEmptyFile := func(path string) {
		checkErr(os.WriteFile(path, []byte(""), 0644))
	}

	// 2. 构建复杂的层级结构
	createEmptyFile(filepath.Join(rootDir, "file1"))

	// 递归创建：如果父目录不存在会自动创建
	hierarchy := filepath.Join(rootDir, "parent", "child")
	err = os.MkdirAll(hierarchy, 0755)
	checkErr(err)

	createEmptyFile(filepath.Join(rootDir, "parent", "file2"))
	createEmptyFile(filepath.Join(rootDir, "parent", "file3"))
	createEmptyFile(filepath.Join(hierarchy, "file4"))

	// 3. 读取单层目录 (os.ReadDir)
	// ReadDir 返回的是 []os.DirEntry，比旧版的 ReadDirNames 性能更高
	fmt.Println("--- 正在列出 subdir/parent 内容 ---")
	entries, err := os.ReadDir(filepath.Join(rootDir, "parent"))
	checkErr(err)

	for _, entry := range entries {
		fmt.Printf("名称: %-10s | 是否目录: %v\n", entry.Name(), entry.IsDir())
	}

	// 4. 工作目录切换 (Chdir)
	// 💡注意：频繁切换 Chdir 在大型项目中容易造成路径逻辑混乱，建议优先使用绝对路径
	originalDir, _ := os.Getwd()
	err = os.Chdir(hierarchy)
	checkErr(err)

	fmt.Printf("\n--- 切换后的当前目录: %s ---\n", hierarchy)
	currentEntries, _ := os.ReadDir(".")
	for _, entry := range currentEntries {
		fmt.Printf("名称: %-10s | 是否目录: %v\n", entry.Name(), entry.IsDir())
	}

	// 切回原始目录
	os.Chdir(originalDir)

	// 5. 递归遍历目录树 (filepath.Walk)
	fmt.Println("\n--- Walk 递归访问 subdir ---")
	startTime := time.Now()
	err = filepath.Walk(rootDir, visit)
	fmt.Println("5.1 耗时:", time.Now().Sub(startTime))

	fmt.Println("\n--- WalkDir 递归访问 subdir ---")
	startTime2 := time.Now()
	// WalkDir 是 Walk 的现代高性能替代版，它通过避免对每个文件进行多余的系统调用（Lstat），在处理大量文件时速度显著更快。
	err = filepath.WalkDir(rootDir, visit2)
	fmt.Println("5.2 耗时:", time.Now().Sub(startTime2))
	checkErr(err)
}

// visit 函数：处理遍历中的每一个节点
func visit(path string, info os.FileInfo, err error) error {
	if err != nil {
		return err // 如果发生错误，停止遍历并返回
	}
	fmt.Printf("访问路径: %-30s | 是否目录: %v\n", path, info.IsDir())
	return nil
}

// visit 函数：处理遍历中的每一个节点
func visit2(path string, info os.DirEntry, err error) error {
	if err != nil {
		return err // 如果发生错误，停止遍历并返回
	}
	fmt.Printf("访问路径: %-30s | 是否目录: %v\n", path, info.IsDir())
	return nil
}
```
输出
```
--- 正在列出 subdir/parent 内容 ---
名称: child      | 是否目录: true
名称: file2      | 是否目录: false
名称: file3      | 是否目录: false

--- 切换后的当前目录: subdir/parent/child ---
名称: file4      | 是否目录: false

--- Walk 递归访问 subdir ---
访问路径: subdir                         | 是否目录: true
访问路径: subdir/file1                   | 是否目录: false
访问路径: subdir/parent                  | 是否目录: true
访问路径: subdir/parent/child            | 是否目录: true
访问路径: subdir/parent/child/file4      | 是否目录: false
访问路径: subdir/parent/file2            | 是否目录: false
访问路径: subdir/parent/file3            | 是否目录: false
5.1 耗时: 93.875µs

--- WalkDir 递归访问 subdir ---
访问路径: subdir                         | 是否目录: true
访问路径: subdir/file1                   | 是否目录: false
访问路径: subdir/parent                  | 是否目录: true
访问路径: subdir/parent/child            | 是否目录: true
访问路径: subdir/parent/child/file4      | 是否目录: false
访问路径: subdir/parent/file2            | 是否目录: false
访问路径: subdir/parent/file3            | 是否目录: false
5.2 耗时: 70.75µs
```
<!--SR:!2026-04-09,45,251-->
<?e?>
# 临时文件和目录
<?e?>
1. 创建临时文件
2. 创建临时目录
3. 在临时目录内操作
<?l?>
```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

func checkTmp(e error) {
	if e != nil {
		panic(e)
	}
}

func main() {
	// 1. 创建临时文件
	// 第一个参数为空字符串 ""，表示使用操作系统默认的临时目录（如 Linux 的 /tmp）
	// 第二个参数 "sample" 是文件名前缀，后面会自动补齐随机字符串以保证唯一性
	f, err := os.CreateTemp("", "sample_*.log")
	checkTmp(err)

	// 打印生成的路径，例如: /tmp/sample_123456.log
	fmt.Printf("临时文件路径: %s\n", f.Name())

	// 最佳实践：创建后立即 defer 删除，防止磁盘空间泄露
	defer os.Remove(f.Name())

	// 写入数据
	_, err = f.Write([]byte{1, 2, 3, 4})
	checkTmp(err)

	// 2. 创建临时目录
	// 同样使用默认临时路径，前缀为 "sampledir"
	dName, err := os.MkdirTemp("", "sampledir_*")
	checkTmp(err)
	fmt.Printf("临时目录路径: %s\n", dName)

	// 使用 RemoveAll 递归删除整个临时目录及其中的文件
	defer os.RemoveAll(dName)

	// 3. 在临时目录内操作
	fName := filepath.Join(dName, "workspace_file.txt")
	err = os.WriteFile(fName, []byte{10, 20, 30}, 0644)
	checkTmp(err)

	fmt.Printf("在临时目录内创建了文件: %s\n", fName)
}
```
输出
```
临时文件路径: /var/folders/27/r0n6fv0103b790l89g7wlsr40000gn/T/sample_2973583092.log
临时目录路径: /var/folders/27/r0n6fv0103b790l89g7wlsr40000gn/T/sampledir_2178860841
在临时目录内创建了文件: /var/folders/27/r0n6fv0103b790l89g7wlsr40000gn/T/sampledir_2178860841/workspace_file.txt
```
<!--SR:!2026-03-30,13,211-->
<?e?>
# 单元测试和基准测试
<?e?>
1. 基础单元测试 (Basic Unit Test)
2. 表格驱动测试 (Table-Driven Tests)
3. 基准测试 (Benchmark)
<?l?>
```go
package main

import (
	"testing"
)

// IntMin 是被测函数：返回两个整数中的最小值
func IntMin(a, b int) int {
	if a < b {
		return a
	}
	return b
}

// ---------------------------------------------------------
// 1. 基础单元测试 (Basic Unit Test)
// ---------------------------------------------------------
func TestIntMinBasic(t *testing.T) {
	ans := IntMin(2, -2)
	if ans != -2 {
		// Errorf 会标记测试失败但继续执行后续逻辑
		t.Errorf("基础测试失败: IntMin(2, -2) = %d; 期望值 -2", ans)
	}
}

// ---------------------------------------------------------
// 2. 表格驱动测试 (Table-Driven Tests)
// ---------------------------------------------------------
// 这是 Go 推荐的测试实践，可以一次性测试多种输入组合
func TestIntMinTableDriven(t *testing.T) {
	// 定义匿名结构体切片作为测试矩阵
	tests := []struct {
		name string
		a, b int
		want int
	}{
		{"正数与负数", 2, -2, -2},
		{"零值测试", 0, 1, 0},
		{"相等数值", 5, 5, 5},
		{"两个负数", -1, -5, -5},
	}

	for _, tt := range tests {
		// 使用 t.Run 创建子测试，可以在 IDE 中单独运行某个用例
		t.Run(tt.name, func(t *testing.T) {
			ans := IntMin(tt.a, tt.b)
			if ans != tt.want {
				t.Errorf("用例 [%s] 失败: got %d, want %d", tt.name, ans, tt.want)
			}
		})
	}
}

// ---------------------------------------------------------
// 3. 基准测试 (Benchmark)
// ---------------------------------------------------------
// 运行命令: go test -bench=.
func BenchmarkIntMin(b *testing.B) {
	// b.N 会由 Go 运行时动态调整，直到获得稳定的统计结果
	for i := 0; i < b.N; i++ {
		IntMin(1, 2)
	}
}
```
输出
```
=== RUN   TestIntMinTableDriven
=== RUN   TestIntMinTableDriven/正数与负数
--- PASS: TestIntMinTableDriven/正数与负数 (0.00s)
=== RUN   TestIntMinTableDriven/零值测试
--- PASS: TestIntMinTableDriven/零值测试 (0.00s)
=== RUN   TestIntMinTableDriven/相等数值
--- PASS: TestIntMinTableDriven/相等数值 (0.00s)
=== RUN   TestIntMinTableDriven/两个负数
--- PASS: TestIntMinTableDriven/两个负数 (0.00s)
--- PASS: TestIntMinTableDriven (0.00s)
PASS
```
<!--SR:!2026-04-08,43,251-->
<?e?>
# 命令行参数
<?e?>
1. 获取原始参数切片
2. 获取用户输入的纯参数
3. 安全地访问特定位置的参数
4. 遍历参数的推荐写法
<?l?>
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 1. 获取原始参数切片
	// os.Args[0] 始终是二进制文件本身的路径或名称
	allArgs := os.Args

	// 2. 获取用户输入的纯参数
	// 使用切片操作 [1:] 剥离程序名，这是最常见的做法
	userArgs := os.Args[1:]

	fmt.Printf("程序路径: %s\n", allArgs[0])
	fmt.Printf("参数总数: %d\n", len(userArgs))
	fmt.Printf("参数内容: %v\n", userArgs)

	// 3. 安全地访问特定位置的参数
	// 专家提示：直接通过索引访问 os.Args 前必须检查长度，否则会引发 panic
	index := 3
	if len(os.Args) > index {
		val := os.Args[index]
		fmt.Printf("索引为 [%d] 的参数是: %s\n", index, val)
	} else {
		fmt.Printf("提示: 提供的参数不足 %d 个\n", index)
	}

	// 4. 遍历参数的推荐写法
	fmt.Println("\n遍历所有参数:")
	for i, arg := range userArgs {
		fmt.Printf("  参数 [%d]: %s\n", i, arg)
	}
}
```
输出
```
程序路径: /Users/weichengjun/Library/Caches/JetBrains/GoLand2025.2/tmp/GoLand/___go_build_command_go
参数总数: 0
参数内容: []
提示: 提供的参数不足 3 个

遍历所有参数:
```
<!--SR:!2026-05-22,79,271-->
<?e?>
# 命令行标志
<?e?>
1. 定义标志 (Defining Flags)
2. 解析标志 (Parsing)
3. 使用数据 (Accessing Data)
4. 处理尾部非标志参数 (Trailing Arguments)
<?l?>
```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	// ---------------------------------------------------------
	// 1. 定义标志 (Defining Flags)
	// ---------------------------------------------------------
	// 方式 A: 返回指针 (推荐，定义和初始化同步)
	// 格式: flag.Type(name, defaultValue, usage)
	wordPtr := flag.String("word", "foo", "一个字符串参数")
	numbPtr := flag.Int("numb", 42, "一个整数参数")
	forkPtr := flag.Bool("fork", false, "一个布尔标志")

	// 方式 B: 绑定到现有变量 (适用于结构体字段赋值)
	var svar string
	flag.StringVar(&svar, "svar", "bar", "另一个字符串参数")

	// ---------------------------------------------------------
	// 2. 解析标志 (Parsing)
	// ---------------------------------------------------------
	// 必须在定义之后、使用之前调用 Parse()
	// 它会从 os.Args[1:] 中解析出已定义的标志
	flag.Parse()

	// ---------------------------------------------------------
	// 3. 使用数据 (Accessing Data)
	// ---------------------------------------------------------
	// 💡注意: 指针类型的标志需要解引用 (*) 才能获取实际值
	fmt.Printf("标志 word: %s\n", *wordPtr)
	fmt.Printf("标志 numb: %d\n", *numbPtr)
	fmt.Printf("标志 fork: %v\n", *forkPtr)
	fmt.Printf("变量 svar: %s\n", svar)

	// ---------------------------------------------------------
	// 4. 处理尾部非标志参数 (Trailing Arguments)
	// ---------------------------------------------------------
	// 例如运行: ./app -word=test hello world
	// "hello" 和 "world" 就是非标志参数
	tailArgs := flag.Args()
	if len(tailArgs) > 0 {
		fmt.Printf("尾部参数数量: %d\n", flag.NArg())
		fmt.Printf("尾部参数内容: %v\n", tailArgs)
	}
}
```
输出
```
标志 word: foo
标志 numb: 42
标志 fork: false
变量 svar: bar
```
<!--SR:!2026-04-15,50,251-->
<?e?>
# 命令行子命令
<?e?>
1. 定义子命令及其独立的 FlagSet
2. 参数长度校验
3. 根据第一个参数分发逻辑
<?l?>
```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	// 1. 定义子命令及其独立的 FlagSet
	// 每个子命令都有自己的一套参数，互不干扰
	fooCmd := flag.NewFlagSet("foo", flag.ExitOnError)
	fooEnable := fooCmd.Bool("enable", false, "是否启用")
	fooName := fooCmd.String("name", "", "名称")

	barCmd := flag.NewFlagSet("bar", flag.ExitOnError)
	barLevel := barCmd.Int("level", 0, "等级")

	// 2. 参数长度校验
	// 需要至少两个参数：[程序名, 子命令]
	if len(os.Args) < 2 {
		fmt.Println("错误: 需要子命令 'foo' 或 'bar'")
		os.Exit(1)
	}

	// 3. 根据第一个参数分发逻辑
	switch os.Args[1] {
	case "foo":
		// 解析从第三个参数开始的内容
		fooCmd.Parse(os.Args[2:])
		fmt.Println("--- 执行子命令: foo ---")
		fmt.Printf("启用状态: %v\n", *fooEnable)
		fmt.Printf("用户名称: %s\n", *fooName)
		fmt.Printf("剩余参数: %v\n", fooCmd.Args())

	case "bar":
		barCmd.Parse(os.Args[2:])
		fmt.Println("--- 执行子命令: bar ---")
		fmt.Printf("当前等级: %d\n", *barLevel)
		fmt.Printf("剩余参数: %v\n", barCmd.Args())

	default:
		fmt.Printf("错误: 未知子命令 '%s'\n", os.Args[1])
		fmt.Println("可用命令: foo, bar")
		os.Exit(1)
	}
}
```
输出
`go run command_sub.go foo -enable -name=aa`
```
--- 执行子命令: foo ---
启用状态: true
用户名称: aa
剩余参数: []
```
<!--SR:!2026-06-23,95,271-->
<?e?>
# 环境变量
<?e?>
1. 写入环境变量
2. 读取环境变量
3. 遍历所有环境变量
<?l?>
```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	// 1. 写入环境变量
	// 💡注意：os.Setenv 仅影响当前进程及其派生的子进程，不会修改操作系统的全局配置
	os.Setenv("APP_PORT", "8080")
	os.Setenv("APP_STAGE", "production")

	// 2. 读取环境变量
	// Getenv 在变量不存在时返回空字符串。如果需要区分“空值”和“不存在”，请使用 os.LookupEnv
	port := os.Getenv("APP_PORT")
	fmt.Printf("1. 获取已知变量: APP_PORT = %s\n", port)

	// 演示不存在的情况
	dbURL, exists := os.LookupEnv("DATABASE_URL")
	if !exists {
		fmt.Println("2. 状态检查: DATABASE_URL 未定义")
	} else {
		fmt.Printf("2. 状态检查: DATABASE_URL = %s\n", dbURL)
	}

	fmt.Println("\n3. 系统所有变量概览 (部分):")
	// 3. 遍历所有环境变量
	// os.Environ() 返回格式为 "KEY=VALUE" 的切片
	for _, env := range os.Environ() {
		pair := strings.SplitN(env, "=", 2)
		// 为了示例整洁，仅打印以 APP_ 开头的变量
		if strings.HasPrefix(pair[0], "APP_") {
			fmt.Printf("   -> %s : %s\n", pair[0], pair[1])
		}
	}
}
```
输出
```
1.获取已知变量: APP_PORT = 8080
2.1状态检查: DATABASE_URL 未定义

3.系统所有变量概览 (部分):
   -> APP_PORT : 8080
   -> APP_STAGE : production
```
<!--SR:!2026-04-13,36,251-->
<?e?>
# HTTP 客户端
<?e?>
1. 发起 HTTP GET 请求
2. 关键：延迟关闭响应体 (Memory Safety)
3. 读取元数据
4. 读取响应体内容
5. 检查扫描器运行时的潜在错误（如网络中断）
<?l?>
```go
package main

import (
	"bufio"
	"fmt"
	"net/http"
)

func main() {
	// 1. 发起 HTTP GET 请求
	// 💡注意：http.Get 会使用默认的 http.DefaultClient
	url := "http://gobyexample.com"
	resp, err := http.Get(url)
	if err != nil {
		// 生产环境建议使用 log.Printf 记录错误而非 panic
		panic(err)
	}

	// 2. 关键：延迟关闭响应体 (Memory Safety)
	// Body 必须关闭，否则会导致底层的 TCP 连接无法重用（Keep-Alive 失效）
	defer resp.Body.Close()

	// 3. 读取元数据
	fmt.Printf("请求 URL: %s\n", url)
	fmt.Printf("状态码: %d\n", resp.StatusCode)
	fmt.Printf("状态描述: %s\n", resp.Status)
	fmt.Printf("Content-Type: %v\n", resp.Header.Get("Content-Type"))

	// 4. 读取响应体内容
	// 使用 bufio.Scanner 进行高效的流式读取
	fmt.Println("\n--- 响应前 5 行内容 ---")
	scanner := bufio.NewScanner(resp.Body)
	for i := 0; i < 5 && scanner.Scan(); i++ {
		fmt.Printf("Line %d: %s\n", i+1, scanner.Text())
	}

	// 5. 检查扫描器运行时的潜在错误（如网络中断）
	if err := scanner.Err(); err != nil {
		fmt.Printf("读取错误: %v\n", err)
	}
}
```
输出
```
请求 URL: http://gobyexample.com
状态码: 200
状态描述: 200 OK
Content-Type: text/html

--- 响应前 5 行内容 ---
Line 1: <!DOCTYPE html>
Line 2: <html>
Line 3:   <head>
Line 4:     <meta charset="utf-8">
Line 5:     <title>Go by Example</title>
```
<!--SR:!2026-04-21,56,271-->
<?e?>
# HTTP 服务端
<?e?>
1. 注册路由映射
2. 启动 HTTP 监听服务
<?l?>
```go
package main

import (
    "fmt"
    "net/http"
)

// hello 处理函数：向客户端返回简单的文本响应
func hello(w http.ResponseWriter, req *http.Request) {
    // Fprintf 可以将格式化字符串直接写入实现了 io.Writer 的 ResponseWriter
    fmt.Fprintf(w, "hello\n")
}

// headers 处理函数：回显客户端发送的所有 HTTP 头部信息
func headers(w http.ResponseWriter, req *http.Request) {
    // 专家提示：req.Header 的类型是 map[string][]string
    // 同一个 Header 键可能对应多个值（如 Accept, Set-Cookie）
    for name, values := range req.Header {
        for _, value := range values {
            fmt.Fprintf(w, "%v: %v\n", name, value)
        }
    }
}

func main() {
    const addr = ":8090"

    // 1. 注册路由映射
    // HandleFunc 将 URL 路径模式与处理逻辑关联起来
    http.HandleFunc("/hello", hello)
    http.HandleFunc("/headers", headers)

    // 2. 启动 HTTP 监听服务
    fmt.Printf("HTTP 服务已启动，正在监听 %s...\n", addr)

    // ListenAndServe 会阻塞 main 协程
    // 第二个参数为 nil 时，系统使用 http.DefaultServeMux (默认多路复用器)
    err := http.ListenAndServe(addr, nil)
    if err != nil {
        // 如果端口被占用，程序会在这里报错
        fmt.Printf("服务启动失败: %v\n", err)
    }
}
```
输出
```
HTTP 服务已启动，正在监听 :8090...
```
<!--SR:!2026-04-22,57,271-->
<?e?>
# Context
<?e?>
1. 获取请求关联的 Context
2. 模拟业务逻辑中的元数据传递
3. 核心：监听取消信号
<?l?>
```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"time"
)

func helloHandler(w http.ResponseWriter, req *http.Request) {
	// 1. 获取请求关联的 Context
	// 当客户端断开连接或请求超时，该 ctx 会自动触发 Done 信号
	ctx := req.Context()

	fmt.Println("--- 处理器启动 ---")
	defer fmt.Println("--- 处理器退出 ---")

	// 2. 模拟业务逻辑中的元数据传递
	// 💡注意：Context 应该是不可变的，WithValue 会返回一个新的 Context 副本
	userCtx := context.WithValue(ctx, "userID", 9527)

	// 3. 核心：监听取消信号
	select {
	case <-time.After(5 * time.Second):
		// 场景 A：模拟耗时 5 秒的操作成功完成
		fmt.Printf("处理成功，用户ID: %v\n", userCtx.Value("userID"))
		fmt.Fprintln(w, "Hello, your request was processed.")

	case <-ctx.Done():
		// 场景 B：在 5 秒内客户端主动断开（如点掉浏览器 X）
		// 此时应立即停止后续计算或数据库查询，释放资源
		err := ctx.Err()
		fmt.Printf("请求取消: %v\n", err)

		// 此时写入 w 已经没有意义，但记录日志非常关键
	}
}

func main() {
	http.HandleFunc("/hello", helloHandler)

	fmt.Println("服务监听于 :8090... (尝试访问并在 5s 内按 Ctrl+C 或关闭浏览器)")
	if err := http.ListenAndServe(":8090", nil); err != nil {
		fmt.Printf("启动失败: %v\n", err)
	}
}
```
输出
```
服务监听于 :8080... (尝试访问并在 5s 内按 Ctrl+C 或关闭浏览器)
--- 处理器启动 ---
处理成功，用户ID: 9527
--- 处理器退出 ---
```
<!--SR:!2026-06-13,102,291-->
<?e?>
# 生成进程
<?e?>
1. 基础用法：获取命令输出 (Blocking)
2. 进阶用法：使用管道进行实时交互 (Piping)
3. Shell 复合命令「Output 读取」
4. Shell 复合命令「StdoutPipe 读取」
5. Shell 复合命令「bytes.Buffer 读取」
<?l?>
```go
package main

import (
	"bytes"
	"fmt"
	"io"
	"os/exec"
)

func pcErr(err error) {
	if err != nil {
		panic(err)
	}
}

func main() {
	// ---------------------------------------------------------
	// 1. 基础用法：获取命令输出 (Blocking)
	// ---------------------------------------------------------
	// exec.Command 第一个参数是命令名，后续是可选的参数列表
	dateCmd := exec.Command("date")

	// Output 会阻塞直到命令执行完毕并返回 stdout
	dateOut, err := dateCmd.Output()
	if err != nil {
		panic(err)
	}
	fmt.Printf("--- 1: date ---\n%s\n", dateOut)

	// ---------------------------------------------------------
	// 2. 进阶用法：使用管道进行实时交互 (Piping)
	// ---------------------------------------------------------
	// 模拟：echo "..." | grep hello
	grepCmd := exec.Command("grep", "hello")
	// 获取 StdinPipe 允许我们向子进程喂数据
	stdin, err := grepCmd.StdinPipe()
	pcErr(err)
	// 获取 StdoutPipe 允许我们从子进程拿数据
	stdout, err := grepCmd.StdoutPipe()
	pcErr(err)
	// Start 启动子进程但不等待，允许我们在其运行时进行 IO 交互
	err = grepCmd.Start()
	pcErr(err)
	// 写入并关闭 Stdin
	// 注意：如果不 Close，grep 会一直等待输入，导致后续 Wait 无法结束
	stdin.Write([]byte("hello golang\ngoodbye world\nhello expert"))
	stdin.Close()
	// 读取输出结果
	grepResult, _ := io.ReadAll(stdout)
	// Wait 必须在读取完所有输出后调用，以释放进程资源
	err = grepCmd.Wait()
	pcErr(err)
	fmt.Printf("--- 2: grep 交互 ---\n%s\n", grepResult)

	// ---------------------------------------------------------
	// 3. 执行复杂 Shell 命令 (Bash -c)
	// ---------------------------------------------------------
	// --- 3.1: Shell 复合命令 ---
	// Go 默认不通过 Shell 运行命令（为了更安全、更快）。
	// 若需使用管道、通配符等 Shell 特性，需显式调用 bash/sh。
	shellCommand := "ls -lh | head -n 3"
	lsCmd := exec.Command("bash", "-c", shellCommand)

	lsOut, err := lsCmd.Output()
	pcErr(err)
	fmt.Printf("--- 3.1: Shell 复合命令「Output 读取」 ---\n%s\n", lsOut)

	// --- 3.2: Shell 复合命令 ---
	// 使用管道读取「启动命令 → 读取管道 → 等待命令」
	lsCmd2 := exec.Command("bash", "-c", shellCommand)
	// 声明管道
	lsCmdOut, _ := lsCmd2.StdoutPipe()
	// 先启动命令
	err = lsCmd2.Start()
	pcErr(err)
	// 再读取管道数据
	lsCmdOutRead, _ := io.ReadAll(lsCmdOut)
	// 等待命令完成
	err = lsCmd2.Wait()
	pcErr(err)
	fmt.Printf("--- 3.2: Shell 复合命令「StdoutPipe 读取」 ---\n%s\n", lsCmdOutRead)

	// --- 3.3: Shell 复合命令 ---
	lsCmd3 := exec.Command("bash", "-c", "ls -lh | head -n 3")
	stdoutPipe, _ := lsCmd3.StdoutPipe()
	lsCmd3.Start()
	// 1. 创建一个 Buffer，它本质上是一个自动扩容的字节数组
	var buf bytes.Buffer
	// 2. 直接将管道中的数据“流”进 Buffer
	// io.Copy 会自动处理读取循环、缓冲区大小和错误判断
	_, err = io.Copy(&buf, stdoutPipe)
	if err != nil && err != io.EOF {
		fmt.Println("读取出错:", err)
	}
	// 3. 此时数据已经全部在 buf 里了
	fmt.Printf("--- 3.3: Shell 复合命令「bytes.Buffer 读取」 ---\n%s\n", buf.String())
	lsCmd3.Wait()
}

```
输出
```
--- 1: date ---
2026年 2月13日 星期五 16时43分44秒 CST

--- 2: grep 交互 ---
hello golang
hello expert

--- 3.1: Shell 复合命令「Output 读取」 ---
total 768
-rw-r--r--  1 weichengjun  staff   2.4K  1 21 15:44 array.go
-rw-r--r--  1 weichengjun  staff   908B  1  8 10:38 atomic.go

--- 3.2: Shell 复合命令「StdoutPipe 读取」 ---
total 768
-rw-r--r--  1 weichengjun  staff   2.4K  1 21 15:44 array.go
-rw-r--r--  1 weichengjun  staff   908B  1  8 10:38 atomic.go

--- 3.3: Shell 复合命令「bytes.Buffer 读取」 ---
total 768
-rw-r--r--  1 weichengjun  staff   2.4K  1 21 15:44 array.go
-rw-r--r--  1 weichengjun  staff   908B  1  8 10:38 atomic.go
```
<!--SR:!2026-03-31,15,211-->
<?e?>
# 执行进程
<?e?>
1. 查找二进制文件的完整路径
2. 准备参数
3. 准备环境变量
4. 调用 syscall.Exec
5. 错误处理
<?l?>
```go
package main

import (
    "os"
    "os/exec"
    "syscall"
)

func main() {
    // 1. 查找二进制文件的完整路径
    // syscall.Exec 需要文件的绝对路径（如 /bin/ls）
    binary, err := exec.LookPath("ls")
    if err != nil {
        panic(err)
    }

    // 2. 准备参数
    // 按照惯例，第一个参数应为程序名本身
    args := []string{"ls", "-a", "-l", "-h"}

    // 3. 准备环境变量
    // 通常直接透传当前进程的环境变量
    env := os.Environ()

    // 4. 调用 syscall.Exec
    // 重点：一旦此行执行成功，当前的 Go 程序生命周期就结束了
    // 系统会重用当前进程的 PID，但将其代码、数据、堆栈全部替换为 ls 的
    execErr := syscall.Exec(binary, args, env)

    // 5. 错误处理
    // 如果 syscall.Exec 返回了错误，说明替换失败，代码才会继续向下运行
    if execErr != nil {
        panic(execErr)
    }
    // 这里永远不会被执行
}
```
输出
```
total 768
drwxr-xr-x  95 weichengjun  staff   3.0K  1 22 17:03 .
drwxr-xr-x   9 weichengjun  staff   288B  1 19 18:21 ..
```
<!--SR:!2026-05-03,65,271-->
<?e?>
# 信号
<?e?>
1. 初始化信号接收通道
2. 注册感兴趣的系统信号
3. 定义退出通知通道
4. 启动后台协程监听信号
5. 阻塞主线程
<?l?>
```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	// 1. 初始化信号接收通道
	// 专家建议：使用缓冲区大小为 1 的通道，确保信号通知是非阻塞的
	sigs := make(chan os.Signal, 1)

	// 2. 注册感兴趣的系统信号
	// syscall.SIGINT  -> 对应终端中的 Ctrl+C
	// syscall.SIGTERM -> 对应 kill 命令，是容器化环境（如 Kubernetes）的标准停止信号
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

	// 3. 定义退出通知通道
	// 用于协调异步协程与主线程之间的同步
	done := make(chan bool, 1)

	// 4. 启动后台协程监听信号
	go func() {
		// 阻塞直到信号发生
		sig := <-sigs
		fmt.Printf("\n--- 捕获到信号: %v ---\n", sig)

		// 这里是执行清理逻辑的最佳位置：
		// 例如：close(db), saveState(), stopServer()
		fmt.Println("正在执行清理工作...")

		done <- true
	}()

	fmt.Println("服务已启动，按 Ctrl+C 停止服务...")

	// 5. 阻塞主线程
	// 直到从 done 通道接收到信号，程序才会真正结束
	<-done
	fmt.Println("程序已优雅退出")
}
```
输出
```
服务已启动，按 Ctrl+C 停止服务...

--- 捕获到信号: interrupt ---
正在执行清理工作...
程序已优雅退出
```
# 退出
<!--SR:!2026-04-03,38,251-->
<?e?>
1. 立即终止进程
2. defer 还会执行吗？
<?l?>
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 1. 注册延迟执行语句
	// 在常规的 Go 逻辑中，defer 会在函数返回前执行
	defer fmt.Println("这条消息永远不会被打印！")

	fmt.Println("正在准备退出程序...")

	// 2. 立即终止进程
	// os.Exit(code) 会立即停止当前程序
	// 参数 3 表示退出状态码，通常 0 表示成功，非 0 表示各种错误

	// 重要：一旦调用 os.Exit，本函数及其他函数中的所有 defer 都将被跳过
	os.Exit(3)
}
```
输出
```
正在准备退出程序...
```
<!--SR:!2026-04-13,61,311-->
<?e?>
# 模块
Go Modules 是 Go 语言的官方依赖管理工具，自 Go 1.11 版本开始引入，在 Go 1.16 版本成为默认的依赖管理模式。
Go Modules 解决了 Go 语言长期以来在依赖管理方面的痛点，为开发者提供了版本控制、依赖隔离和可重复构建等核心功能。
Go Modules 是一组相关 Go 包的集合，它们被版本化并作为一个独立的单元进行管理。每个模块都有一个明确的版本标识，允许开发者在项目中精确指定所需依赖的版本。
### 核心概念解析
**模块（Module）** ：包含 `go.mod` 文件的目录树，该文件定义了模块的路径、Go 版本要求和依赖关系。
**版本（Version）** ：遵循语义化版本控制（Semantic Versioning）的标识符，格式为 `vMAJOR.MINOR.PATCH` 。
**依赖图（Dependency Graph）** ：模块及其所有传递依赖的层次结构，Go 工具会自动解析和维护。
## 为什么需要 Go Modules？
### 传统 GOPATH 的问题
在 Go Modules 出现之前，Go 使用 GOPATH 模式，存在以下局限性：
1. **工作空间限制** ：所有项目必须放在 GOPATH 目录下
2. **版本管理困难** ：无法精确控制依赖版本
3. **依赖冲突** ：多个项目可能使用同一依赖的不同版本
4. **可重复构建挑战** ：难以确保不同环境下的构建一致性
### GOPATH vs Go Modules
| 特性     | GOPATH 模式     | Go Modules |
| ------ | ------------- | ---------- |
| 项目位置限制 | 必须放在 GOPATH 下 | 任意位置均可     |
| 版本控制   | 有限支持          | 完整的语义化版本控制 |
| 依赖隔离   | 全局共享          | 项目级隔离      |
| 可重复构建  | 困难            | 自动保障       |
| 离线工作   | 不支持           | 支持本地缓存     |
## 核心文件解析
### go.mod 文件
`go.mod` 是模块的定义文件，包含以下主要部分：
```go
module example.com/mymodule // 模块路径

go 1.21 // Go 版本要求

require (
    github.com/gin-gonic/gin v1.9.1
    golang.org/x/text v0.12.0
)

replace golang.org/x/text => ../local/text // 本地替换

exclude github.com/old/module v1.0.0 // 排除特定版本
```
### go.sum 文件
`go.sum` 文件记录依赖模块的加密哈希值，用于验证模块内容的完整性：
```go
github.com / bytedance / sonic v1.9.1 h1:ei0tVql02GmiYGRCTUcI6g...  
github.com / bytedance / sonic v1.9.1 / go.mod h1:iZcSUejdk5C4OW...  
```
## 基本命令详解
### 模块初始化
```go
# 创建新模块
go mod init example.com/myproject

# 在现有项目中初始化
cd /path/to/project
go mod init
```
### 依赖管理
```go
# 添加依赖（自动选择最新版本）
go get github.com/gin-gonic/gin

# 添加特定版本
go get github.com/gin-gonic/gin@v1.9.1

# 更新到最新版本
go get -u github.com/gin-gonic/gin

# 更新所有依赖
go get -u all

# 下载依赖到本地缓存
go mod download

# 整理 go.mod 文件
go mod tidy
```
### 依赖查询
```go
# 查看所有依赖
go list -m all

# 查看特定依赖的可用版本
go list -m -versions github.com/gin-gonic/gin

# 查看为什么需要某个依赖
go mod why github.com/gin-gonic/gin
```
## 实际工作流程
### 1\. 新项目初始化
```go
# 创建项目目录
mkdir myproject && cd myproject

# 初始化模块
go mod init github.com/username/myproject

# 编写代码并导入依赖
# 然后运行以下命令自动处理依赖
go mod tidy
```
### 2\. 依赖版本控制策略
```go
// go.mod 中的版本指定方式
require (
    github.com/lib/pq v1.10.9          // 精确版本
    golang.org/x/text v0.3.7           // 精确版本
    github.com/stretchr/testify v1.8.0 // 测试依赖
)

// 间接依赖由 Go 工具自动管理
```
### 3\. 版本选择机制
Go Modules 使用 **最小版本选择（MVS** ）算法：
![](https://weichengjun2.dpdns.org/i/2026/01/19/696e0a3627cef.png)
## 高级特性
### 版本替换（Replace）
```go
// 用本地路径替换远程依赖
replace github.com/some/dependency => ../local/dependency

// 用不同版本替换
replace github.com/some/dependency => github.com/some/dependency v2.0.0

// 用 fork 仓库替换
replace github.com/some/dependency => github.com/myfork/dependency v1.0.0
```
### 排除特定版本
```go
exclude (
    github.com/problematic/module v1.0.0
    github.com/another/badmodule v2.1.0
)
```
### 私有仓库支持
```go
# 配置私有仓库认证
git config --global url."https://user:token@github.com".insteadOf "https://github.com"

# 或使用环境变量
export GOPRIVATE=github.com/mycompany/*
```
## 最佳实践
### 1\. 版本管理策略
```go
# 开发阶段使用最新版本
go get -u ./...

# 发布前锁定版本
go mod tidy
go mod vendor  # 可选：创建vendor目录

# 定期更新依赖
go get -u all
go mod tidy
```
### 2\. 协作开发规范
```go
# 提交前确保 go.mod 和 go.sum 一致
go mod tidy
go mod verify

# 检查未使用的依赖
go mod tidy -v
```
### 3\. CI/CD 集成
```go
# GitHub Actions 示例
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v3
        with:
          go-version: '1.21'
      - run: go mod download
      - run: go test ./...
```
## 常见问题与解决方案
### 问题 1: 依赖下载失败
解决方案
```go
# 设置代理
go env -w GOPROXY=https://goproxy.cn,direct

# 清理缓存并重试
go clean -modcache
go mod download
```
### 问题 2: 版本冲突
解决方案
```go
# 查看依赖图
go mod graph

# 分析冲突原因
go mod why -m conflicting/package

# 使用 replace 指令解决
```
### 问题 3: 私有模块认证
解决方案
```go
# 配置 netrc 文件
machine github.com
login username
password token

# 或使用 SSH 替代 HTTPS
git config --global url."git@github.com:".insteadOf "https://github.com/"
```
## 实践练习
### 练习 1: 创建第一个模块
1、创建新目录并初始化模块：
```shell
mkdir hello-world && cd hello-world
go mod init example.com/hello
```
2、创建 `main.go` ：
```go
package main

import (
    "fmt"
    "rsc.io/quote"
)

func main() {
    fmt.Println(quote.Hello())
}
```
3、运行并观察依赖管理：
```go
go run main.go
go mod tidy
cat go.mod
```
### 练习 2: 版本控制实践
1、添加特定版本的依赖：
```shell
go get golang.org/x/text@v0.3.7
```
2、尝试更新到最新版本：
```shell
go get -u golang.org/x/text
```
3、查看版本变化：
```shell
go list -m all | grep text
```
# 反射
## 基础
<?e?>
通过反射获取结构体方法，必须保持接收者==1;;类型==一致，否则不能获取结构体的所有方法
> [!note]- 反射的三大定律？
> 1. 从`interface{}`到**反射对象**：TypeOf(i) 获取类型, ValueOf(i) 获取值。
> 	- 当我们将一个变量传递给 `reflect.TypeOf()` 或 `reflect.ValueOf()` 时，该变量会先被隐式转换为 `interface{}`。反射包会解析这个接口，并提取出它的**动态类型**和**动态值**。
> 		- **`reflect.Type`**：代表变量的**类型**（如 `int`、`struct`）。
> 		- **`reflect.Value`**：代表变量的**动态值**。
> 2. **修改**反射对象：必须是“**可寻址**”的 (通过指针 + Elem)。
> 	- 若要修改反射对象，其值必须是“**可设置的**”（Settable）。
> 		- 如果你传递的是**值**（副本），反射对象是**不可设置**的，因为修改副本没有意义。
> 		- 如果你传递的是**指针**，并且通过 **`Elem()` 方法**获取指针指向的内容，反射对象才是**可设置**的。
> 3. 从**反射对象**到`interface{}`：v.Interface() 将反射对象**重新装箱**。
> 	- 通过调用 `reflect.Value` 的 `Interface()` 方法，我们可以将反射出来的对象打包回接口类型，随后可以通过**类型断言**将其还原为**原始类型**。
<!--SR:!2026-04-15,20,273-->
<?e?>
### 基础结构
```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Name string `json:"name" doc:"用户姓名"` // 导出字段：反射可读、可修改
	age  int    `json:"age"`             // 私有字段：反射可读元数据，但不可写（CanSet=false）
}

func (u User) MyName() string {
	return fmt.Sprintf("我的名字叫 %s", u.Name)
}

func (u User) SayHello(to string, times int) string {
	return fmt.Sprintf("你好 %s，我是 %s，说了 %d 遍", to, u.Name, times)
}

type Player struct {
	Reason string `map:"extend.reason"`
}

type MyService struct{}

// 模拟业务方法：接收值类型参数
func (s *MyService) HandleValue(p Player) {
	fmt.Println("Value接收成功:", p.Reason)
}

// 模拟业务方法：接收指针类型参数
func (s *MyService) HandlePtr(p *Player) {
	fmt.Println("Pointer接收成功:", p.Reason)
}
```
#### 专家提示 (Expert Tips):
1. 性能：反射比直接调用慢 ==1;;10-100== 倍。它是将“编译时检查”推迟到“运行时计算”。
2. 缓存：在高性能 ORM 或 JSON 库中，通常会用 ==1;;sync.Map== 缓存 Type 的 ==1;;Field==索引 和 ==1;;Tag==解析结果。
3. 谨慎：反射可以==1;;读取==私有字段(PkgPath != "")，但绝对无法通过反射 SetString ==1;;修改==它们（除非使用 unsafe）。
4. 类型安全：反射绕过了==1;;编译器==检查。如果 SetString 到了一个 int 字段，程序会直接 ==1;;Panic==。
5. 对齐：Method.Call 对参数极其严格。传入 Value 还是 Pointer 必须与 ==1;;Method.Type().In(i)== 严格匹配。
<!--SR:!2026-03-27,8,273-->
<?e?>
### 元数据探索
- **`TypeOf`（说明书）**：负责描述。关注 **Kind**（本质类别，如 `struct`）与 **Type**（定义名称，如 `User`）。
- **`ValueOf`（实体内容）**：负责操作。关注 **Data**（运行时具体数值）与 **Settability**（是否可修改）。
<?l?>
#### TypeOf
>**💡注意**：当 u 是`&User{}`**(指针)** 时 不能调用 `NumField()`
> 因为：reflect.TypeOf(u) 返回的是 `*User`**(指针)** 类型，而不是 User **结构体**类型。**指针类型没有字段**，所以调用 NumField() 会 **panic**。
```go
u := User{"Gemini", 18}
t := reflect.TypeOf(u)
fmt.Printf("1. [Type分析] 名称: %v | 基础类别(Kind): %v\n", t.Name(), t.Kind())
for i := 0; i < t.NumField(); i++ {
	// Field(i) 提取该位置的“零件属性” (StructField)
	f := t.Field(i)
	fmt.Printf("--- 字段 [%d] 静态元数据 ---\n", i)
	fmt.Printf("1. Name: %-10v | 代码标识符\n", f.Name)
	fmt.Printf("2. Type: %-10v | 静态类型\n", f.Type)
	fmt.Printf("3. PkgP: %-10v | 权限判断：为空则可导出，非空则私有\n", f.PkgPath)
	fmt.Printf("4. Offs: %-10v | 内存对齐偏移字节\n", f.Offset)
	fmt.Printf("5. Index: %-10v | 嵌套寻址\n", f.Index)
	fmt.Printf("6. Anonymous: %-10v | 匿名/嵌入\n", f.Anonymous)
	fmt.Printf("7. Tag : %-10v | 原始标签\n", f.Tag)
	fmt.Printf("8. JSON: %-10v | 标签解析：Get(\"json\") 结果\n\n", f.Tag.Get("json"))
}
/*
   1. [Type分析] 名称: User | 基础类别(Kind): struct
   --- 字段 [0] 静态元数据 ---
   2. Name: Name       | 代码标识符
   3. Type: string     | 静态类型
   4. PkgP:            | 权限判断：为空则可导出，非空则私有
   5. Offs: 0          | 内存对齐偏移字节
   6. Index: [0         ] | 嵌套寻址
   7. Anonymous: false      | 匿名/嵌入
   8. Tag : json:"name" doc:"用户姓名" | 原始标签
   9. JSON: name       | 标签解析：Get("json") 结果

   --- 字段 [1] 静态元数据 ---
   10. Name: age        | 代码标识符
   11. Type: int        | 静态类型
   12. PkgP: main       | 权限判断：为空则可导出，非空则私有
   13. Offs: 16         | 内存对齐偏移字节
   14. Index: [1         ] | 嵌套寻址
   15. Anonymous: false      | 匿名/嵌入
   16. Tag : json:"age" | 原始标签
   17. JSON: age        | 标签解析：Get("json") 结果
*/
```
#### ValueOf
```go
vo := reflect.ValueOf(u)
fmt.Printf("1.2 [Value分析] 名称: %v | 基础类别(Kind): %v\n", t.Name(), t.Kind())
for j := 0; j < vo.NumField(); j++ {
	// Field(i) 提取该位置的值
	fieldVal := vo.Field(j)
	// Type() 获取字段说明书，然后Field(j) 提取该位置的“零件属性” (StructField)
	fieldType := vo.Type().Field(j)
	fmt.Printf("--- 字段 [%d] 值 ---\n", j)
	fmt.Printf("1. Value: %-10v | 值\n", fieldVal)
	fmt.Printf("1. Type: %-10v | 静态类型\n", fieldType.Type)
}
/*
   1.2 [Value分析] 名称: User | 基础类别(Kind): struct
   --- 字段 [0] 值 ---
   1. Value: Gemini     | 值
   2. Type: string     | 静态类型
   --- 字段 [1] 值 ---
   3. Value: 18         | 值
   4. Type: int        | 静态类型
*/
```
<!--SR:!2026-03-27,8,273-->
<?e?>
### 动态值操作与可设置性
💡重点：CanSet() 的前提是 `ValueOf(&x).Elem()`
<?l?>
```go
fmt.Println("\n2. [值操作与寻址]")
x := 3.4
v := reflect.ValueOf(x)
fmt.Printf("直接传值: CanSet = %v (因为是副本)\n", v.CanSet())
// Elem() 相当于 *ptr，它跨越指针直接定位到原始内存的地址
vPtr := reflect.ValueOf(&x).Elem()
fmt.Printf("指针+Elem: CanSet = %v\n", vPtr.CanSet())
if vPtr.CanSet() {
	vPtr.SetFloat(3.1415)
	fmt.Printf("修改结果: x = %v\n", x)
}
/*
   1. [值操作与寻址]
   直接传值: CanSet = false (因为是副本)
   指针+Elem: CanSet = true
   修改结果: x = 3.1415
*/
```
<!--SR:!2026-03-29,9,273-->
<?e?>
### 方法动态调用
💡重点：Call 接收并返回 `[]reflect.Value` 切片
<?l?>
#### 基本用法
```go
fmt.Println("\n3. [动态调用]")
uv := reflect.ValueOf(u)

// A. 无参调用 (按索引)
res1 := uv.Method(0).Call(nil)
fmt.Printf("无参调用 (Method 0): %v\n", res1[0].Interface())

// B. 带参调用 (按名称)
method := uv.MethodByName("SayHello")
args := []reflect.Value{
	reflect.ValueOf("Gopher"),
	reflect.ValueOf(3),
}
res2 := method.Call(args)
fmt.Printf("带参调用 (SayHello): %v\n", res2[0].String())
/*
   3. [动态调用]
   无参调用 (Method 0): 我的名字叫 Gemini
   带参调用 (SayHello): 你好 Gopher，我是 Gemini，说了 3 遍
*/
```
#### 方法调用 Method vs. Func
##### 1. 核心对比：绑定逻辑与调用差异
调用结构体方法存在**两种**本质不同的路径：**绑定态 (Method Value)** 与 **非绑定态 (Function Value)**。
###### 方式 A：`reflect.Value.Method()` (绑定态)
获取的是已绑定具体实例的“方法值”。调用时**无需**传入接收者（Receiver）。
```go
// 语义：instance.MethodName
v := reflect.ValueOf(instance)
mValue := v.MethodByName("Store") // 此时 mValue 已“锁死”在 instance 上

// 调用：直接传参
mValue.Call([]reflect.Value{arg1, arg2}) 
```
###### 方式 B：`reflect.Type.Method().Func` (非绑定态)
获取的是未绑定的底层函数。调用时**必须**将实例作为**第一个参数**传入。
```go
// 语义：Type.MethodName(instance, ...)
t := reflect.TypeOf(instance)
mFunc := t.Method(0).Func // 此时 mFunc 只是一个通用函数指针

// 调用：首个参数必须是接收者实例
mFunc.Call([]reflect.Value{reflect.ValueOf(instance), arg1, arg2})
```
##### 2. 技术参数对照表
| 维度       | `Value.Method(i)` | `Type.Method(i).Func` |
| :------- | :---------------- | :-------------------- |
| **底层类型** | Method Value      | Function Value        |
| **参数数量** | $N$ (仅业务参数)       | $N + 1$ (接收者 + 业务参数)  |
| **绑定时机** | 获取时立即绑定           | 调用时动态传入               |
| **内存开销** | 每次获取都会闭包捕获（较多）    | 共享底层函数定义（较少）          |
| **执行速度** | **快** (直接寻址)      | **慢** (多一层转换)         |
##### 3. 代码实战示例
```go
type Calc struct{ ID int }

func (c *Calc) Add(a, b int) int { return a + b }
func main() {
	c := &Calc{ID: 1}

	// --- 方式 A: 适合固定实例的缓存 ---
	// 逻辑：将方法与 c 永久绑定，存入 Map 后随时取用
	vMethod := reflect.ValueOf(c).MethodByName("Add")
	res1 := vMethod.Call([]reflect.Value{reflect.ValueOf(10), reflect.ValueOf(20)})
	fmt.Println(res1[0].Interface()) // 30

	// --- 方式 B: 适合泛型框架/DI 容器 ---
	// 逻辑：缓存函数定义，调用时再决定作用于哪个 Calc 实例
	tMethod := reflect.TypeOf(c).Method(0).Func
	res2 := tMethod.Call([]reflect.Value{
		reflect.ValueOf(c), // 显式注入实例
		reflect.ValueOf(10),
		reflect.ValueOf(20),
	})
	fmt.Println(res2[0].Interface()) // 30
}
```
##### 4. 架构级记忆要点 💡
* **90% 场景选方式 A**：处理一个具体的对象（如 Gin 的 Controller 注入），用 `Value.Method()`。更符合直觉，且参数列表与原生函数签名对齐。
* **10% 场景选方式 B**：写 ORM 或性能要求极高的底层库，且需要**同一个方法定义**作用于**成千上万**个**不同**的实例，此时缓存 `Type.Method().Func` 更有利于内存复用。
* **Panic 预警**：使用方式 B 时，如果第一个参数的类型与方法接收者不匹配（例如期望 `*Calc` 却传了 `Calc`），程序会直接崩溃。
<!--SR:!2026-03-29,9,273-->
<?e?>
### 运行时动态构造
💡重点：可以在完全不知道类型名的情况下，根据 Type 生产实物
#### struct
`reflect.New` 模拟了内建函数 `new(T)` 的分配行为。
链路：**“类型提取 -> 堆内存分配 -> 寻址解引用 -> 类型还原”**
<?l?>
```go
// 1.【类型提取】获取目标结构体的 Type 描述符（元数据中心）
tUser := reflect.TypeOf(User{})

// 2.【堆内存分配】按 Type 尺寸开辟零值内存，返回指向该地址的 Value 指针
// 语义等同于：ptr := new(User)
valPtr := reflect.New(tUser)

// 3.【寻址解引用】通过 .Elem() 获取指针指向的实际变量，使 Value 具备“可设置性”(Settability)
// 3.1.【偏移定位】根据字段名计算内存偏移量，定位目标字段
// 3.2.【原子写】校验类型安全后，将字面量写入对应的内存地址
valPtr.Elem().FieldByName("Name").SetString("NewBot")

// 4.【类型还原】将反射 Value 打包回 interface{}，触发运行时类型断言输出
fmt.Printf("动态结构体: %+v\n", valPtr.Interface()) // 动态结构体: &{Name:NewBot age:0}
```
<!--SR:!2026-03-28,5,253-->
<?e?>
#### slice
`reflect.MakeSlice` 模拟了内建函数 `make([]T, len, cap)` 的底层行为。
链路：**“类型构造 -> 切片初始化 -> 动态追加 -> 类型导出”**
<?l?>
```go
// 1.【类型构造】利用 SliceOf 将基础类型(int)升维，构造出“切片类型”的元数据描述符
// 语义等同于：reflect.TypeOf([]int{})
tSlice := reflect.SliceOf(reflect.TypeOf(0))

// 2.【切片初始化】在堆上分配 SliceHeader 结构（包含 Data 指针、Len、Cap）
// 语义等同于：vSlice := make([]int, 0, 10)
vSlice := reflect.MakeSlice(tSlice, 0, 10)

// 3.【动态追加】调用运行时 append 逻辑。注意：由于切片可能触发扩容（重新分配内存），
// 必须接收 Append 的返回值并重新赋值给 vSlice 句柄
vSlice = reflect.Append(vSlice, reflect.ValueOf(100), reflect.ValueOf(200))

// 4.【类型导出】通过 .Interface() 消除反射包装，回归普通切片视图
fmt.Printf("动态切片: %v\n", vSlice.Interface()) // 动态切片: [100 200]
```
<!--SR:!2026-03-30,9,273-->
<?e?>
#### map
`reflect.MakeMap` 模拟了内建函数 `make(map[K]V)` 的底层行为。
链路：**“键值对构造 -> 哈希表初始化 -> 键值对注入 -> 类型导出”**
<?l?>
```go
// 1.【键值对构造】提取 Key 和 Value 的类型元数据，并将其绑定至目标 Map 的类型描述符
// 语义等同于：reflect.TypeOf(map[string]int{})
tKey := reflect.TypeOf("")
tVal := reflect.TypeOf(0)
tMap := reflect.MapOf(tKey, tVal)

// 2.【哈希表初始化】在堆上分配 hmap 结构并初始化哈希种子（hash seed）
// 语义等同于：vMap := make(map[string]int)
vMap := reflect.MakeMap(tMap)

// 3.【键值对注入】执行哈希寻址并在对应的 Bucket 中写入数据。与切片不同，Map 是引用类型，无需接收返回值即可原地修改底层哈希表
vMap.SetMapIndex(reflect.ValueOf("Go"), reflect.ValueOf(2026))

// 4.【类型导出】通过 .Interface() 卸载反射外壳，回归标准的 map 视图
fmt.Printf("动态映射: %v\n", vMap.Interface()) // 动态映射: map[Go:2026]
```
<!--SR:!2026-03-28,7,273-->
<?e?>
#### chan
`reflect.MakeChan` 模拟了内建函数 `make(chan T, buffer)` 的底层行为。
链路：**“类型构造 -> hchan结构初始化 -> 同步发送 -> 同步接收 -> 类型导出”**
<?l?>
```go
// 1.【类型构造】定义通道的“双向属性”与“承载类型”，生成 Chan 类型元数据描述符
// 语义等同于：reflect.TypeOf(make(chan string)) BothDir 代表双向通道
tChan := reflect.ChanOf(reflect.BothDir, reflect.TypeOf(""))

// 2.【hchan结构初始化】在堆上分配 hchan 结构，包含循环缓冲区(buf)和等待队列(recvq/sendq)
// 语义等同于：vChan := make(chan string, 2) 缓冲大小为 2
vChan := reflect.MakeChan(tChan, 2)

// 3.【同步发送】执行运行时 chansend 操作。若缓冲区满则触发反射协程挂起，直至数据写入 buf
// 语义等同于：vChan <- "反射消息"
vChan.Send(reflect.ValueOf("反射消息"))

// 4.【同步接收】执行运行时 chanrecv 操作。从缓冲区或发送队列提取数据，返回 reflect.Value
// 语义等同于：msg := <- vChan
msg, _ := vChan.Recv()

// 5.【类型导出】通过 .String() 直接提取底层字符串，或使用 .Interface() 还原类型
fmt.Printf("动态通道接收: %s\n", msg.String()) // 动态通道接收: 反射消息
```
<!--SR:!2026-03-29,8,273-->
<?e?>
#### func
💡重点：这是反射最**黑魔法**的部分，运行时“捏”出一个逻辑函数
`reflect.MakeFunc` 允许在运行时将一个**通用的逻辑闭包**绑定到特定的**函数签名**上。
链路：**“签名元数据定义 -> 通用逻辑注入 -> 运行时绑定 -> 反射触发 -> 原生接口对齐”**
<?l?>
```go
// 1.【签名元数据定义】利用 FuncOf 构造函数的入参和出参序列，生成函数类型的运行时描述符
// 语义等同于：reflect.TypeOf(func([]int) []int { return nil })
funcType := reflect.FuncOf(
	[]reflect.Type{reflect.TypeOf([]int{})}, // 入参类型
	[]reflect.Type{reflect.TypeOf([]int{})}, // 出参类型
	false,                                   // 是否为可变参数
)

// 2.【通用逻辑注入】编写内部 Stub 实现。反射引擎会将原始调用栈的参数打包成 []Value 传入，并要求返回 []Value 以适配出参
fnLogic := func(args []reflect.Value) (results []reflect.Value) {
	inSlice := args[0] // 获取第一个入参「💡反射函数出入参类型是切片」
	// 动态构造出参容器：reflect.MakeSlice 申请内存
	outSlice := reflect.MakeSlice(reflect.TypeOf([]int{}), inSlice.Len(), inSlice.Len())
	// 循环处理入参
	for i := 0; i < inSlice.Len(); i++ {
		// 元素级寻址与计算(将入参乘以2后输出)
		outSlice.Index(i).SetInt(inSlice.Index(i).Int() * 2)
	}
	return []reflect.Value{outSlice} // 返回处理后的切片
}

// 3.【运行时绑定】将设计好的“签名”与“逻辑实体”焊接，生成一个可调用的反射函数对象
vFunc := reflect.MakeFunc(funcType, fnLogic)

// 4.【反射触发】通过 .Call 启动调用。反射引擎负责处理参数的压栈、跳转及结果弹出
values := []reflect.Value{reflect.ValueOf([]int{1, 2, 3, 4})}
call := vFunc.Call(values)
fmt.Printf("动态函数: %v\n", call[0].Interface()) // 动态函数: [2 4 6 8]

// 5.【原生接口对齐】通过反射 Set 操作将动态生成的函数句柄赋值给具名函数变量
// 语义等同于：doubleFunc = vFunc
var doubleFunc func([]int) []int
reflect.ValueOf(&doubleFunc).Elem().Set(vFunc)
fmt.Println("动态函数还原为普通函数调用")
// 方式1
fmt.Printf("%v\n", doubleFunc([]int{10, 20})) // [20 40]
// 方式2
doubleFunc2 := vFunc.Interface().(func([]int) []int)
fmt.Printf("%v\n", doubleFunc2([]int{30, 40})) // [60 80]
```
<!--SR:!2026-03-30,9,273-->
<?e?>
### 防御性检查与接口
#### 空值校验
反映指针变量本身的“合法性”与“内容”。
<?e?>
##### 1. `IsValid()` —— 判定：反射对象是否存在？
<?l?>
**底层逻辑**：检查 `reflect.Value` 内部是否持有了有效的类型（Type）信息。如果返回 `false`，代表这个 `Value` 是一个“空壳”，调用任何其他方法都会直接 **Panic**。
>**准则**: 调用任何方法前，先用 `IsValid()` 防 Panic。
* **场景一：获取不存在的 Map Key**
```go
m := map[string]int{"a": 1}
v := reflect.ValueOf(m).MapIndex(reflect.ValueOf("b"))
fmt.Println(v.IsValid()) // false (键 "b" 不存在)
```
* **场景二：获取不存在的结构体字段**
```go
v := reflect.ValueOf(User{}).FieldByName("InvalidField")
fmt.Println(v.IsValid()) // false (字段名写错了)
```
<!--SR:!2026-04-01,9,273-->
<?e?>
##### 2. `IsNil()` —— 判定：持有的地址是否为空？
<?l?>
**底层逻辑**：检查变量持有的底层内存地址（Uintptr）是否为 `0`。只适用于引用类型：**Ptr, Chan, Map, Slice, Func, Interface**。
* **场景一：声明但未初始化的切片/映射**
```go
var s []int
fmt.Println(reflect.ValueOf(s).IsNil()) // true (空切片没有底层数组)
```
* **场景二：显式的空指针**
```go
var p *int = nil
fmt.Println(reflect.ValueOf(p).IsNil()) // true
```
* **注意**：对 `int` 或 `struct` 调用 `IsNil` 会触发 **Panic**，因为它们不是引用类型。
<!--SR:!2026-04-01,9,273-->
<?e?>
##### 3. `IsZero()` —— 判定：内容是否为初始零值？
<?l?>
**底层逻辑**：检查该变量的值是否等于其类型的默认值（如 `0`, `""`, `false`）。对于结构体，会递归检查所有字段是否均为零值。
* **场景一：基础类型的零值**
```go
fmt.Println(reflect.ValueOf(0).IsZero())      // true
fmt.Println(reflect.ValueOf("").IsZero())     // true
```
* **场景二：结构体的零值（重点）**
```go
u := User{Name: "", Age: 0}
fmt.Println(reflect.ValueOf(u).IsZero())     // true (所有字段都是默认值)

u2 := User{Name: "Bot"}
fmt.Println(reflect.ValueOf(u2).IsZero())    // false (Name 被赋值了)
```
<!--SR:!2026-03-30,7,273-->
<?e?>
##### 4. 综合对比：以 `u2 := new(User)` 为例
```go
u := new(User) // 分配了内存，u 是一个 *User 指针，指向 {Name:"", Age:0}
of := reflect.ValueOf(u)
// A. of 包含 *User 类型信息吗？
fmt.Println(of.IsValid())

// B. u 指向的地址是 0x0 吗？
fmt.Println(of.IsNil())

// C. *User 类型的零值是什么？
fmt.Println(of.IsZero())

// D. 如果想看结构体内的属性是不是空的？
// 需要先解引用 Elem() 拿到 User 结构体实体
fmt.Println(of.Elem().IsZero())
```
<?l?>
true (它是一个有效的反射对象)
false (new 分配了地址，不是 nil)
false (指针的零值是 nil。因为 u != nil，所以它不是指针类型的零值。)
true (里面的 Name 和 Age 确实都是零值)
<!--SR:!2026-03-31,8,273-->
<?e?>
##### 状态判定逻辑矩阵
| 状态              | `reflect.ValueOf(nil)` | `var p *int = nil` | `p := new(int)` | `p := 10`    |
| :-------------- | :--------------------- | :----------------- | :-------------- | :----------- |
| **`IsValid()`** | ==1;;false==           | ==1;;true==        | ==1;;true==     | ==1;;true==  |
| **`IsNil()`**   | ==1;;Panic==           | ==1;;true==        | ==1;;false==    | ==1;;Panic== |
| **`IsZero()`**  | ==1;;Panic==           | ==1;;true==        | ==1;;false==    | ==1;;false== |
##### 架构级记忆要点 💡
* **防御式编程**：在处理外部传入的 `interface{}` 时，标准的检查流程是：`v.IsValid()` -> `if v.Kind() == Ptr { v.IsNil() }` -> `v.IsZero()`。
* **语义差异**：`IsNil` 关注的是**内存连通性**，而 `IsZero` 关注的是**业务状态**。
##### 通用的 `IsEmpty` 函数
它需要处理从基础类型到复杂容器（Slice, Map, Chan）的所有“空”语义。
逻辑建模为 **“三层过滤网”：无效性检查 -> 引用零值检查 -> 容器长度检查。**
```go
func IsEmpty(i interface{}) bool {
	v := reflect.ValueOf(i)

	// 1.【无效性拦截】如果 Value 本身无效（如传入 nil），判定为空
	if !v.IsValid() {
		return true
	}

	// 2.【语义化分支】根据不同 Kind 判定“空”的定义
	switch v.Kind() {
	case reflect.Ptr, reflect.Interface:
		// 指针或接口：看其是否指向 nil
		return v.IsNil()
	case reflect.Slice, reflect.Map, reflect.Chan, reflect.Array:
		// 容器类：看其元素个数是否为 0
		return v.Len() == 0
	case reflect.String:
		// 字符串：看长度是否为 0
		return v.Len() == 0
	case reflect.Struct:
		// 结构体：看是否所有字段都是初始零值
		return v.IsZero()
	default:
		// 基础类型（int, bool等）：直接看是否为该类型的初始零值
		return v.IsZero()
	}
}
```
<!--SR:!2026-03-31,8,273-->
<?e?>
#### 接口类型采样
在 Go 反射的底层实现中，**接口采样**之所以必须采用“绕路”指针的方式，是由 Go 的 **Eface（空接口）** 运行机制决定的。
##### 核心物理逻辑：类型擦除与动态提取
在 Go 中，所有传递给 `reflect.TypeOf(i interface{})` 的参数都会经历一次==1;;隐式==转换。
1. **直接传递接口的问题**：如果你传入一个 `fmt.Stringer` 接口变量，Go 会将其解构为 `(Type, Data)` 对。反射引擎会直接跳过==1;;接口==外壳，去提取 `Data` 对应的具体实现==1;;类型==。如果你传的是 `nil`，反射则完全拿不到任何 `Type` 信息。
2. **指针采样的妙处**：当使用 `(*fmt.Stringer)(nil)` 时，创建了一个==1;;指针==类型。在反射看来，这个指针的**基准类型**就是该接口==1;;本身==。
```go
// 1.【静态类型锁定】在编译期强制声明一个类型为 *fmt.Stringer 的空指针。
// 此时内存中并不存在 Stringer 实体，但存在一个指向该接口定义的“类型标记”。
ptr := (*fmt.Stringer)(nil)

// 2.【类型元数据捕获】reflect.TypeOf 接收该指针。
// 反射引擎捕获到的是 *fmt.Stringer（指针类型）的 rtype 描述符。
tPtr := reflect.TypeOf(ptr)

// 3.【解引用剥离】调用 .Elem() 移除指针层级（Pointer Dereference）。
// 这一步直接触达指针所指向的底层单元——即接口（Interface）的原始元数据定义。
// 语义等同于：reflect.TypeOf(new(fmt.Stringer)).Elem()
stringerType := tPtr.Elem()
// 4. 【合法性判定】使用获取到的“接口说明书”作为基准，判定其他类型是否满足约束。
isImplement := reflect.TypeOf(User{}).Implements(stringerType)
fmt.Printf("User 是否实现了 Stringer: %v\n", isImplement) // User 是否实现了 Stringer: false
```
##### 为什么 `Elem()` 是唯一触达路径？
在 `reflect.Type` 的内部逻辑中，**`Elem()`** 是跨越“容器”与“内容”的唯一桥梁。
- 对于**指针类型**：`Elem()` 返回其指向的 ==1;;Base Type==。
- 对于**接口指针**：其 Base Type 恰恰就是那个 ==1;;Interface==类型本身。
##### 架构级记忆要点 💡
- **采样公式**：`reflect.TypeOf((*T)(nil)).Elem()` 是获取接口 `reflect.Type` 的**工业标准写法**。
- **不可替代性**：这是判定一个动态类型是否满足某个接口约束（`Implements`）的**前置必要条件**。
<!--SR:!2026-03-31,8,273-->
<?e?>
#### 兼容性比对
扫描 User 类型的 Method Set（方法集），检查是否**完全覆盖**了 Stringer 接口定义的**所有函数**
```go
// 1.【类型采样】获取具体实现类 User 的 Type 描述符
// 该描述符记录了 User 结构体绑定的所有方法名及其签名
tUser := reflect.TypeOf(User{})

// 2.【接口基准】使用此前通过指针采样获取的 Stringer 接口 Type
// stringerType 包含了接口定义的“强制要求”方法列表：String() string
stringerType := reflect.TypeOf((*fmt.Stringer)(nil)).Elem()

// 3.【方法集扫描】执行 Implements 判定。
// 反射引擎会遍历 tUser 的方法集，检查是否完整覆盖了 stringerType 的所有定义。
// 语义等同于强类型断言：_, ok := interface{}(User{}).(fmt.Stringer)
isMatch := tUser.Implements(stringerType)

fmt.Printf("User 是否实现了 fmt.Stringer: %v\n", isMatch) // User 是否实现了 fmt.Stringer: false
```
<!--SR:!2026-03-22,3,273-->
<?e?>
### 参数动态注入与对齐
💡重点：解决在不知道**方法签名**的情况下，如何注入【值】或【指针】参数
链路：**“参数类型嗅探 -> 指针与值类型转换 -> 堆内存动态分配 -> 深层寻址赋值 -> 类型对齐与解引用 -> 栈帧调用”**
<?l?>
```go
svc := &MyService{}
vSvc := reflect.ValueOf(svc)
tSvc := vSvc.Type()
for i := 0; i < vSvc.NumMethod(); i++ {
	method := vSvc.Method(i)
	// 1.【参数类型嗅探】通过 Method 对象的元数据获取输入参数的反射类型（reflect.Type）
	originType := method.Type().In(0)
	// 2.【类型转换】检测参数是否为指针。若是指针，通过 Elem() 提取其指向的“基准类型”(Base Type)
	tmpType := originType
	if originType.Kind() == reflect.Ptr {
		tmpType = originType.Elem()
	}
	// 3.【堆内存动态分配】在内存中为基准类型申请零值空间，并返回指向该空间的指针 Value
	// 语义等同于：ptrVal := new(Player)
	ptrVal := reflect.New(tmpType)
	// 4.【深层寻址赋值】通过 .Elem() 穿透指针，在申请的内存块中定位字段并执行“原子写”操作
	ptrVal.Elem().FieldByName("Reason").SetString("M4-Core-Data")
	// 5.【类型对齐与解引用】根据目标方法对参数的要求（值传递 vs 指针传递）进行最终适配
	// 核心逻辑：若方法需要值类型，则通过 .Elem() 实现 *T -> T 的转换；否则直接传递指针
	arg := ptrVal
	if originType.Kind() != reflect.Ptr {
		arg = ptrVal.Elem() // 解引用：*Player -> Player
	}
	// 6.【反射触发调用】将适配后的参数压入反射调用栈，执行目标逻辑并输出结果
	fmt.Printf("--- 动态注入调用: %s ---\n", tSvc.Method(i).Name)
	method.Call([]reflect.Value{arg})
}
/*
   --- 动态注入调用: HandleValue ---
   Value接收成功: M4-Core-Data
   --- 动态注入调用: HandlePtr ---
   Pointer接收成功: M4-Core-Data
*/
```
<!--SR:!2026-04-03,8,255-->
<?e?>
## 实现结构体方法动态回调
<?l?>
```go
package main

import (
	"fmt"
	"reflect"
	"strings"
)

// --- 1. 数据模型定义层 ---
// 状态结构体
type StatusDO struct{ Status int }

// 订单结构体
type Order struct{ ID string }

// 扩展结构体
type Extend = map[string]any

// 订单事件
type OrderAction = string // 💡类型别名语法 OrderAction 和 string 完全等价，方法集通用

const (
	OrderPlace  OrderAction = "Place"
	OrderCancel OrderAction = "Cancel"
)

// --- 2. 业务逻辑层 ---
// 订单操作服务
type OrderActionService struct{}

// 请求参数结构体：使用 map 标签声明数据来源路径
type Req struct {
	Status *StatusDO `map:"status"` // 直接从根节点提取 status
	Order  *Order    `map:"order"`  // 直接从根节点提取 order
	Extend *Extend   `map:"extend"` // 扩展字段
}

// 实际业务方法：下单
func (a *OrderActionService) Place(req *Req) *StatusDO {
	fmt.Printf("【下单】订单ID: %s\n", req.Order.ID)
	return &StatusDO{Status: 2}
}

// 实际业务方法：取消
func (a *OrderActionService) Cancel(req *Req) *StatusDO {
	fmt.Printf("【取消】原因: %s, 订单ID: %s\n", (*req.Extend)["reason"], req.Order.ID)
	return &StatusDO{Status: 2}
}

// --- 3. 核心调度框架层 ---
// 预定义方法缓存，避免高频调用时 MethodByName 的性能损耗
var eventCache map[string]reflect.Value

// 实现了“数据映射 -> 动态实例化 -> 方法调用”的完整链路
func call(eventName string, req map[string]any) any {
	// 优先从缓存获取方法反射值，取出对应方法反射值
	m, ok := eventCache[eventName]
	if !ok {
		panic(fmt.Errorf("未找到方法: %s", eventName))
	}
	// 获取方法的第一个参数类型 (假设我们约定每个方法只有一个 Request 结构体)
	mType := m.Type()
	if mType.NumIn() != 1 {
		panic(fmt.Errorf("方法参数必须为一个 Request 结构体: %s", eventName))
	}
	// 【核心魔术】：
	// 拿到方法首参类型 (*Cancel)
	in := mType.In(0)
	var e reflect.Type
	// 若是指针类型则转成实体，若不是则直接赋值
	isPtr := in.Kind() == reflect.Ptr
	if isPtr {
		// .Elem() -> 剥掉指针，拿到基类型 (Cancel)
		e = in.Elem()
	} else {
		e = in
	}
	// reflect.New(...) -> 分配内存，得到新实例的指针 (*Cancel)
	// .Elem() -> 进入实例内部，使其字段变为“可填充”状态
	reqValue := reflect.New(e).Elem()

	// 遍历结构体字段，根据 Tag 注入数据
	for i := 0; i < reqValue.NumField(); i++ {
		fieldVal := reqValue.Field(i)         // 获取字段的反射值
		fieldType := reqValue.Type().Field(i) // 获取字段的元数据（包括 Tag）
		path := fieldType.Tag.Get("map")      // 获取 map 标签定义的路径

		// 路径解析：支持 "a.b.c" 格式的深层 map 提取
		var current any = req
		for _, part := range strings.Split(path, ".") {
			if m, ok := current.(map[string]any); ok {
				current = m[part]
			}
		}

		// 如果查找到有效数据，则注入到结构体实例中
		if current != nil {
			val := reflect.ValueOf(current)
			// 核心：检查【数据源类型】是否可以赋值给【目标字段类型】
			// fieldType.Type 是结构体定义的类型，val.Type() 是实际数据的类型
			if val.Type().AssignableTo(fieldType.Type) {
				fieldVal.Set(val) // 直接复用 val，不再重复调用 reflect.ValueOf
			} else {
				// 生产环境建议：在这里可以记录日志，说明类型不匹配
				fmt.Printf("方法参数类型不匹配: 期望 %v, 得到 %v\n", fieldType.Type, val.Type())
			}
		}
	}
	// 若是指针类型则将填充好的 reqValue 转换回地址（指针）传入
	if isPtr {
		reqValue = reqValue.Addr()
	}
	// 执行方法调用, m.Call 返回的是 []reflect.Value，我们取下标 0 并转回 any
	return m.Call([]reflect.Value{reqValue})[0].Interface()
}

// --- 4. 运行示例 ---
func main() {
	// 声明数据结构
	orderActionService := &OrderActionService{}
	eventCache = make(map[string]reflect.Value)

	// 扫描结构体的所有方法, 并加入缓存
	vo := reflect.ValueOf(orderActionService)
	numMethod := vo.NumMethod()
	for i := 0; i < numMethod; i++ {
		m := vo.Method(i)
		eventCache[vo.Type().Method(i).Name] = m
	}

	// 模拟来自上游的原始数据源
	rawData := map[string]any{
		"status": &StatusDO{Status: 1},
		"order":  &Order{ID: "ORD_M4_2026"},
		"extend": &map[string]any{
			"reason": "设备升级，重新下单",
		},
	}

	// 触发调度：指定服务、方法和数据源
	result := call(OrderCancel, rawData)
	// 最终结果转换与输出
	if finalStatus, ok := result.(*StatusDO); ok {
		fmt.Printf("【调用完成】返回状态码: %d\n", finalStatus.Status)
	}
}
```
输出
```
【业务执行】原因: 设备升级，重新下单, 订单ID: ORD_M4_2026
【调用完成】返回状态码: 2
```
<!--SR:!2026-03-26,1,194-->
<?e?>
## GO VS PHP 反射耗时对比
### Go
```go
package main

import (
	"reflect"
	"testing"
)

type UserBench struct {
	Name string
}

// 原生赋值
func BenchmarkNativeSet(b *testing.B) {
	u := UserBench{}
	for i := 0; i < b.N; i++ {
		u.Name = "Gemini"
	}
}

// 反射赋值
func BenchmarkReflectSet(b *testing.B) {
	u := UserBench{}
	v := reflect.ValueOf(&u).Elem()
	f := v.FieldByName("Name")
	for i := 0; i < b.N; i++ {
		f.SetString("Gemini")
	}
}
```
输出
```
goos: darwin
goarch: arm64
cpu: Apple M4
BenchmarkNativeSet
BenchmarkNativeSet-10     	1000000000	         0.2610 ns/op
BenchmarkReflectSet
BenchmarkReflectSet-10    	802388667	         1.580 ns/op
```
### PHP
```php
class User
{
    public $name;
}

$u     = new User();
$count = 1000000;

// 原生赋值
$start = microtime(true);
for ($i = 0; $i < $count; $i++) {
    $u->name = "Gemini";
}
echo "Native: " . (microtime(true) - $start) . "s\n";

// 反射赋值
$start = microtime(true);
$ref   = new ReflectionProperty('User', 'name');
for ($i = 0; $i < $count; $i++) {
    $ref->setValue($u, "Gemini");
}
echo "Reflect: " . (microtime(true) - $start) . "s\n";
```
输出
```
Native: 0.017929792404175s
Reflect: 0.082978010177612s
```
### 对比结果
<?e?>
#### 1. 绝对速率对比 (Absolute Speed)
为了公平，我们将 PHP 的结果换算为纳秒（ns/op）。假设 PHP 的循环次数是1,000,000次：

| 项目                 | Go (ns/op)      | PHP (ns/op)      | 性能倍数 (Go vs PHP) |
| ------------------ | --------------- | ---------------- | ---------------- |
| **原生赋值 (Native)**  | ~==1;;0.26== ns | ~==1;;17.93== ns | 约 ==1;;68== 倍    |
| **反射赋值 (Reflect)** | ~==1;;1.58== ns | ~==1;;82.98== ns | 约 ==1;;52== 倍    |
**核心发现：**
Go 的**反射调用**（==1;;~1.58== ns）比 PHP 的**直接赋值**（==1;;~17.93== ns）还要快 ==1;;11== 倍。这意味着在 Go 中即便是被认为“慢”的反射，其执行效率也远超 PHP 的常规代码。
<!--SR:!2026-04-15,21,255-->
<?e?>
#### 2. 损耗比分析 (Performance Penalty)
反射比原生慢了多少倍：
- **Go**: ==1;;$1.58/0.26 \approx 6$== 倍
- **PHP**: ==1;;$0.0829/0.0179 \approx 4.6$== 倍
虽然 Go 的绝对速度快，但从比例上看，Go 开启反射后的性能跌幅（==1;;6==倍）比 PHP（==1;;4.6==倍）更大。这是因为 Go 原生代码太快了，导致反射带来的“类型检查”和“内存寻址”开销显得异常突出。
<!--SR:!2026-04-11,19,275-->
<?e?>
#### 3. 为什么 Go 在 Apple M4 上表现如此恐怖？
##### 0.26 ns/op 是什么概念？
Apple M4 的主频大约在 $4.4\text{ GHz}$ 左右，这意味着一个时钟周期大约是 $0.22\text{ ns}$。
- 你的原生测试结果 `0.26 ns` 意味着：**单次赋值操作几乎只占用了一个 CPU 时钟周期**。
- 这说明 Go 编译器将该操作优化到了极致，甚至可能利用了 CPU 的流水线并行处理（Pipeline）。
##### 1.58 ns/op 的反射快在哪里？
在 Go 中可能使用了 `v.Field(i).Set()` 而不是 `v.FieldByName("Name").Set()`。
* `1.58 ns` 相当于 **7 个时钟周期**。
* 在这么短的时间内，Go 完成了：接口解包、安全检查、内存偏移计算、值写入。这体现了 Go 运行时（Runtime）极高的效率。
#### 4. 结论与实战建议
* **PHP 开发者视角**：反射虽然比原生慢，但在 PHP 的量级下，这几倍的损耗在 Web 请求（毫秒级）中几乎感知不到。你可以放心在框架中使用**注解**、**依赖注入**等反射功能。
* **Go 开发者视角**：反射确实很快，但如果你的业务是高频计算（例如每秒处理百万级消息），`1.58 ns` 相比 `0.26 ns` 依然是 **600%** 的成本增加。
#### 5. 进阶测试建议：内存分配 (Memory Allocation)
反射不仅消耗 CPU，有时还会引起内存分配（逃逸到堆）。建议你在运行 Go 测试时加上 `-benchmem`：
```bash
go test -bench=. -benchmem
```
如果 `B/op` 大于 0，说明反射导致了额外的内存分配，这在生产环境中会引发 GC（垃圾回收）压力，这才是 Go 反射真正的“隐形杀手”。
<?e?>
### 基准测试每次操作分配的字节数
```go
package main

import (
	"reflect"
	"testing"
)

type User struct {
	Name string
}

var globalS string

// 原生赋值
func BenchmarkNative(b *testing.B) {
	u := User{}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		u.Name = "Gemini"
		globalS = u.Name
	}
}

// 优化后：只测试写入，索引已提前拿到 (代表 1.58ns 那个级别)
func BenchmarkReflectByIndex(b *testing.B) {
	u := User{}
	v := reflect.ValueOf(&u).Elem()
	idx := 0 // 提前确定的索引
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		v.Field(idx).SetString("Gemini")
	}
}

// 模拟真实：每次都要通过字符串查找字段 (这是最常用的方式)
func BenchmarkReflectByName(b *testing.B) {
	u := User{}
	v := reflect.ValueOf(&u).Elem()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		// FieldByName 会导致内部的字符串查找逻辑
		v.FieldByName("Name").SetString("Gemini")
	}
}
```
#### 执行测试（关键指令）
在终端运行以下命令，注意 `-benchmem`：
```shell
go test -bench=. -benchmem
```
#### 结果解读
```shell
goos: darwin
goarch: arm64
pkg: benchmark
cpu: Apple M4
BenchmarkNative-10              1000000000               0.6258 ns/op          0 B/op          0 allocs/op
BenchmarkReflectByIndex-10      488833464                2.228 ns/op           0 B/op          0 allocs/op
BenchmarkReflectByName-10       48556538                25.29 ns/op            0 B/op          0 allocs/op
PASS
ok      benchmark       3.289s
```

| **测试项**            | **耗时 (ns/op)**  | **每秒执行次数 (估算)** | **性能差距**       | **含义**                            |
| ------------------ | --------------- | --------------- | -------------- | --------------------------------- |
| **Native**         | ==1;;0.62== ns  | ~16 亿次          | ==1;;1==x (基准) | 最纯粹的内存写入，几乎等同于 CPU 频率的物理极限。       |
| **ReflectByIndex** | ==1;;2.22== ns  | ~4.5 亿次         | ~==1;;3.5==x   | 缓存了索引后的反射。这是在生产环境中使用反射的**黄金标准**。  |
| **ReflectByName**  | ==1;;25.29== ns | ~0.4 亿次         | ~==1;;40==x    | 未经优化的反射。每次都靠字符串匹配。这是大部分初级框架的性能瓶颈。 |
##### 为什么内存分配（B/op）都是 0？
这是最令人欣慰的数据。`0 B/op` 和 `0 allocs/op` 表示：
- **没有堆逃逸**：Go 编译器通过“逃逸分析”，发现你的 `User` 结构体和反射对象 `v` 并没有离开 `Benchmark` 函数的作用域。
- **内存效率**：所有的反射操作都在 **栈（Stack）** 上完成，不会触发 GC（垃圾回收）。这意味着在高并发下，这种代码不会导致系统卡顿。
##### 为什么 `ByName` 比 `ByIndex` 慢了 11 倍？
即便在 Apple M4 这种强大的 CPU 上，==1;;字符串==匹配 依然是沉重的负担。
- **ByIndex (2.22 ns)**：底层只是一次简单的==1;;指针偏移==计算（Pointer Arithmetic）。CPU 知道起始地址，加上索引偏移，直接定位。
- **ByName (25.29 ns)**：
    1. 拿字符串 "Name" 去结构体的元数据表里对比。
    2. 逐个==1;;遍历==字段：是 "Age" 吗？不是。是 "Name" 吗？匹配成功。
    3. 这种遍历和字符串比较无法被 CPU 完全并行化，因此耗时陡增。
<!--SR:!2026-04-16,23,275-->
<?e?>
# 中间件
<?l?>
## 拦截错误
```go
package main

import "fmt"

func main() {
	// ✅情况 A：执行 middleA() 返回匿名函数，然后立即调用该函数
	middleA()() // 结果：打印 "recover2: 错误了2"，成功捕获

	// ❌情况 B：执行 middleB() 返回函数，然后立即调用该函数
	middleB()() // 结果：程序崩溃 (Panic)，无法捕获
}

// --- 情况 A：通过闭包将 recover 注入到执行逻辑内部 ---
func middleA() func() {
	// 步骤 1: middleA 开始运行
	// 步骤 2: 直接返回一个匿名函数（此时内部代码还未执行）
	return func() {
		// 步骤 3: main 调用此函数，开始执行
		// 步骤 4: 注册 defer，它守护的是当前这个匿名函数
		defer func() {
			if err := recover(); err != nil {
				fmt.Println("recoverA: ", err)
			}
		}()
		// 步骤 5: 触发 panic
		panic("错误了A")
		// 步骤 6: panic 向上寻找最近的守护者，触发步骤 4 的 defer -> 成功捕获
	}
}

// --- 情况 B：recover 注册在了错误的生命周期 ---
func middleB() func() {
	// 步骤 1: middleB 开始运行
	// 步骤 2: 注册 defer，但它只守护 middleB 函数本身
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("recoverB: ", err)
		}
	}()
	// 步骤 3: middleB 运行结束，返回 handlerFunc 地址
	// 步骤 4: 执行 middleB 的 defer（此时无 panic，recover 为空）
	return func() { panic("错误了B") }
	// 步骤 5: main 函数拿到地址后开始执行该函数 -> 触发 panic
	// 此时 middleB 已销毁，其 defer 无法跨越函数边界去救火
}
```
输出
```
recoverA:  错误了A
panic: 错误了B
```
## 洋葱模型 onion
```go
package main

import "fmt"

// --- Context 结构 ---
type Context struct {
	handlers []func(*Context) // 处理器链条（包含中间件和控制器）
	index    int              // 当前执行到的索引位置
}

// --- 依次执行后续的处理器，就像推倒第一块多米诺骨牌 ---
func (c *Context) Next() {
	c.index++
	for c.index < len(c.handlers) {
		c.handlers[c.index](c)
	}
}

// --- 中间件 A：异常捕获 ---
func Recovery(c *Context) {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("[Recovery] 捕获异常，返回 500 错误")
		}
	}()
	fmt.Println("[Recovery] 防护开始")
	c.Next()
	fmt.Println("[Recovery] 防护结束")
}

// --- 中间件 B：日志记录 ---
func Logger(c *Context) {
	fmt.Println("[Log] --> 请求开始") // Next 之前：请求阶段
	c.Next()                      // 暂停当前，去执行后面的
	fmt.Println("[Log] <-- 响应结束") // Next 之后：响应阶段
}

// --- 核心业务逻辑 ---
func Handler(c *Context) {
	fmt.Println("[Business] 业务逻辑开始")
	c.Next()
	fmt.Println("[Business] 业务逻辑结束")
}

func main() {
	c := &Context{
		handlers: []func(*Context){Recovery, Logger, Handler},
		index:    -1,
	}
	c.Next()
}
```
输出
```
[Recovery] 已开启防护
[Log] --> 请求进入
[Business] 执行核心业务逻辑...
[Log] <-- 响应返回
[Recovery] 防护任务结束
```
<!--SR:!2026-03-29,33,287-->
<?e?>