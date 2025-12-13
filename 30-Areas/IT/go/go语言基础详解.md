# go语言基础详解

## Hello Goland

```go
package main

import "fmt"  
  
func main() {  
    fmt.Println("hello world")  
}
```

以上是最基本的go语言程序。

package main用来标记当前包（同属于同一包下的不同.go文件相当于在同一个文件内、不需要 import引入）

## 如何运行

打开 CMD1输入命令 _go run hello.go_ 并按回车执行代码。

```cmd
go run hello.go
```

我们还可以使用 go build 命令来生成二进制文件：

```cmd
$ go build hello.go 
$ ls
hello    hello.go
$ ./hello 
Hello, World!
```

### 注意

需要注意的是 <kbd>{ </kbd>不能单独放在一行，所以以下代码在运行时会产生错误：

```go
package main

func main()
{  // 不能单独空行
	println("这是错误的")
}
```


## 分隔符

在 go 程序中、一行代表一个语句的结束、不需要一定要以 <kbd>；</kbd>结尾
如果你打算将多个语句写在同一行，它们则必须使用 ; 人为区分。

## 注释

```go
// 单行注释

/* 多行注释 
多行
*/
```

## 标识符

标识符由字母数字和下划线组成，标识符开头必须以**字母**和**下划线**开头

## 数据类型

- 布尔型：bool
- 整型
- 浮点型
- 字符串类型
- 派生类型
	- 指针类型 pointer
	- 数组类型 array
	- 结构化类型  struct
	- Channel类型
	- 函数类型 function
	- 切片类型 Slice
	- 接口类型 Interface
	- 映射类型 Map：对应pyhton的字典

## 语言量

### 变量定义

```go
var v_name v_type

var a int // 这种定义的时候没有赋值所以默认为0
a = 3
var b int = 3

var v = 3
```

初始化不赋值：

- 数值类型（包括complex64/128）为 **0**
    
- 布尔类型为 **false**
    
- 字符串为 **""**（空字符串）
    
- 以下几种类型为 **nil**：
    
    var a *int
    var a []int
    var a map[string] int
    var a chan int
    var a func(string) int
    var a error // error 是接口

### 常量定义

```go
// const const identifier [type] = value
```

你可以省略类型说明符 [type]，因为编译器可以根据变量的值来推断其类型。

- 显式类型定义： `const b string = "abc"`  
    
- 隐式类型定义： `const b = "abc"`

多个相同类型的声明可以简写为：

```go
const c_name1, c_name2 = value1, value2
```

## Go 语言内置的运算符有：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 位运算符
- 赋值运算符
- 其他运算符

### 算术运算符

下表列出了所有Go语言的算术运算符。假定 A 值为 10，B 值为 20。

| 运算符 | 描述  | 实例             |
| --- | --- | -------------- |
| +   | 相加  | A + B 输出结果 30  |
| -   | 相减  | A - B 输出结果 -10 |
| *   | 相乘  | A * B 输出结果 200 |
| /   | 相除  | B / A 输出结果 2   |
| %   | 求余  | B % A 输出结果 0   |
| ++  | 自增  | A++ 输出结果 11    |
| --  | 自减  | A-- 输出结果 9     |

## 条件语句

Go语言主要有以下条件语句：

- If
- If-else
- If 嵌套语句
- switch语句
- select语句

> 注意：Go 没有三目运算符，所以不支持 **?:** 形式的条件判断。

