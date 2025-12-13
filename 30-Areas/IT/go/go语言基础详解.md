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

