# Go 语言基础 (写给 Python 开发者)

欢迎来到 Go 语言的世界！这份指南专为有 Python 基础的开发者设计，参考了 [TopGoer](https://www.topgoer.com/) 的知识体系，通过对比两种语言的异同，帮助你快速上手 Go。

![700](assets/go语言基础/file-20251211191407254.png)
## 1. 核心差异概览

| 特性         | Python               | Go                             |
| :----------- | :------------------- | :----------------------------- |
| **类型系统** | 动态类型 (Dynamic)   | 静态类型 (Static)              |
| **运行方式** | 解释型 (Interpreter) | 编译型 (Compiler)              |
| **编程范式** | 面向对象 (OOP) 为主  | 过程式 + 接口 (组合优于继承)   |
| **并发模型** | 线程/协程 (GIL 限制) | Goroutine (轻量级线程，无 GIL) |
| **代码风格** | 缩进 (Indentation)   | 大括号 `{}` (Braces)           |
| **异常处理** | try-except           | 多返回值 (value, err)          |

---

## 2. 基础语法

### Hello World

Go 程序必须属于一个包 (`package`)。可执行程序的入口包必须是 `main`。

```go
package main // 声明包名

import "fmt" // 导入格式化包

func main() {
    fmt.Println("Hello, World!")
}
```

### 变量与常量

Go 是静态类型，但支持类型推导。

**Python:**

```python
name = "Gopher"
age = 10
const_val = 3.14 # Python 没有真正的常量
```

**Go:**

```go
// 1. 标准声明
var name string = "Gopher"

// 2. 类型推导
var age = 10

// 3. 短变量声明 (仅限函数内)
isCool := true

// 4. 批量声明
var (
    a int
    b string
)

// 5. 常量 (真正的常量)
const Pi = 3.14
const (
    StatusOk = 200
    StatusNotFound = 404
)
```

---

## 3. 复合数据类型 (重点)

这是 Python 开发者最需要适应的部分。

### 数组 (Array) vs 切片 (Slice)

Python 的 `list` 对应 Go 的 `Slice`。Go 的 `Array` 是固定长度的，很少直接使用。

**Python (List):**

```python
nums = [1, 2, 3]
nums.append(4)
print(nums[1:3])
```

**Go (Slice):**

```go
// 数组 (固定长度)
var arr [3]int = [3]int{1, 2, 3}

// 切片 (动态长度，引用类型)
nums := []int{1, 2, 3}
nums = append(nums, 4) // append 可能返回新的 slice 引用，必须赋值回去

// 切片操作
sub := nums[1:3] // [2, 3]
```

### 字典 (Map)

Python 的 `dict` 对应 Go 的 `map`。

**Python:**

```python
scores = {"Alice": 90, "Bob": 85}
scores["Charlie"] = 95
if "Alice" in scores:
    print(scores["Alice"])
```

**Go:**

```go
// 初始化 map[KeyType]ValueType
scores := map[string]int{
    "Alice": 90,
    "Bob":   85,
}
scores["Charlie"] = 95

// 查找 (Go 特有的 "comma ok" 惯用法)
val, ok := scores["Alice"]
if ok {
    fmt.Println(val)
}
```

---

## 4. 流程控制

### If / Else

Go 的 `if` 条件不需要括号，且支持在判断前执行一个简单的语句。

```go
if x := 10; x > 5 {
    fmt.Println("x is big")
}
// x 在这里已经不可用了
```

### 循环 (For)

Go 只有 `for`，没有 `while`。

**Python (While):**

```python
while i < 5: ...
```

**Go (While 风格):**

```go
for i < 5 { ... }
```

**Go (Range 遍历):**
类似于 Python 的 `enumerate`。

```go
nums := []int{10, 20, 30}
for index, value := range nums {
    fmt.Println(index, value)
}

// 遍历 map
for key, value := range scores { ... }
```

### Switch

Go 的 `switch` 默认不需要 `break`，除非使用 `fallthrough`。

```go
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("OS X")
case "linux":
    fmt.Println("Linux")
default:
    fmt.Println("Other")
}
```

---

## 5. 函数与错误处理

### 多返回值

Go 函数支持多返回值，常用于返回 `(result, error)`。

```go
func div(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

### Defer (延迟执行)

类似于 Python 的 `try...finally` 或 `with` 语句，用于资源释放。

**Python:**

```python
f = open("file")
try:
    process(f)
finally:
    f.close()
```

**Go:**

```go
f, _ := os.Open("file")
defer f.Close() // 函数退出前执行，无论是否 panic
process(f)
```

### Panic & Recover

Go 不推荐滥用异常。`panic` 类似于 `raise`，`recover` 类似于 `except`，通常只在严重错误时使用。

---

## 6. 面向对象 (Structs & Interfaces)

Go 没有 `class`，没有继承。它使用 **组合 (Composition)** 和 **接口 (Interface)**。

### 结构体 (Struct)

```go
type User struct {
    Name string
    Age  int
}

// 方法 (接收者 Receiver)
func (u *User) Grow() {
    u.Age++
}
```

### 接口 (Interface) - 鸭子类型

Go 的接口是**隐式实现**的。只要一个类型实现了接口定义的所有方法，它就实现了该接口。

**Python:**

```python
# 只要有 quack 方法就行
def let_it_quack(duck):
    duck.quack()
```

**Go:**

```go
type Quacker interface {
    Quack()
}

type Duck struct{}
func (d Duck) Quack() { fmt.Println("Quack!") }

type Dog struct{}
func (d Dog) Quack() { fmt.Println("Woof... I mean Quack!") }

func LetItQuack(q Quacker) {
    q.Quack()
}
```

---

## 7. 并发编程 (Concurrency)

### Goroutine

轻量级线程，启动成本极低。

```go
go func() {
    fmt.Println("I'm running in background")
}()
```

### Channel (通道)

Python 使用 `Queue` 通信，Go 使用 `Channel`。**不要通过共享内存来通信，而要通过通信来共享内存。**

```go
// 创建一个 int 类型的 channel
ch := make(chan int)

// 发送方
go func() {
    ch <- 42 // 发送数据
}()

// 接收方
val := <-ch // 阻塞等待数据
fmt.Println(val)
```

### Select

用于处理多个 Channel 的操作，类似于 I/O 多路复用。

```go
select {
case msg1 := <-ch1:
    fmt.Println("Received from ch1", msg1)
case ch2 <- 1:
    fmt.Println("Sent to ch2")
default:
    fmt.Println("No communication")
}
```

---

## 8. 包管理 (Go Modules)

Go 1.11+ 推出了官方的依赖管理工具 Go Modules。

- `go mod init <module_name>`: 初始化项目 (生成 go.mod)
- `go mod tidy`: 自动下载依赖并清理无用依赖
- `go get <package>`: 安装依赖

**Python (pip/requirements.txt):**

```bash
pip install requests
```

**Go:**

```bash
go get github.com/gin-gonic/gin
```

---

## 9. 常用标准库速查

| Python        | Go               | 用途          |
| :------------ | :--------------- | :------------ |
| `sys.argv`    | `os.Args`        | 命令行参数    |
| `time.sleep`  | `time.Sleep`     | 休眠          |
| `json.dumps`  | `json.Marshal`   | JSON 序列化   |
| `json.loads`  | `json.Unmarshal` | JSON 反序列化 |
| `threading`   | `go` keyword     | 并发          |
| `http.server` | `net/http`       | HTTP 服务     |

---

## 学习资源推荐

- [TopGoer](https://www.topgoer.com/) - 系统的 Go 语言中文文档
- [Go by Example](https://gobyexample.com/) - 通过例子学 Go
- [Go Tour](https://tour.go.dev/) - 官方交互式教程
