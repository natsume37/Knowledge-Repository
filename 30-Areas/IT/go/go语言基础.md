# Go 语言基础 (写给 Python 开发者)

欢迎来到 Go 语言的世界！这份指南专为有 Python 基础的开发者设计，通过对比两种语言的异同，帮助你快速上手 Go。

## 1. 核心差异概览

| 特性       | Python            | Go                      |
| :------- | :---------------- | :---------------------- |
| **类型系统** | 动态类型 (Dynamic)    | 静态类型 (Static)           |
| **运行方式** | 解释型 (Interpreter) | 编译型 (Compiler)          |
| **编程范式** | 面向对象 (OOP) 为主     | 过程式 + 接口 (非传统 OOP)      |
| **并发模型** | 线程/协程 (GIL 限制)    | Goroutine (轻量级线程，无 GIL) |
| **代码风格** | 缩进 (Indentation)  | 大括号 `{}` (Braces)       |

---

## 2. Hello World

**Python:**
```python
print("Hello, World!")
```

**Go:**
Go 程序必须属于一个包 (`package`)。可执行程序的入口包必须是 `main`。
```go
package main // 声明包名

import "fmt" // 导入格式化包 (类似于 import sys)

// main 函数是程序的入口
func main() {
    fmt.Println("Hello, World!")
}
```
> **注意**：Go 语句结尾不需要分号（编译器会自动添加），但必须使用大括号 `{}` 包裹代码块。

---

## 3. 变量与类型 (Variables & Types)

Python 是动态类型，变量不需要声明类型。Go 是静态类型，变量必须先声明（或推导）类型。

**Python:**
```python
name = "Gopher"
age = 10
is_cool = True
```

**Go:**
```go
// 方式 1: 标准声明 (var 变量名 类型)
var name string = "Gopher"

// 方式 2: 类型推导 (编译器自动判断类型)
var age = 10 

// 方式 3: 短变量声明 (最常用，仅限函数内部)
isCool := true 

// 常量
const Pi = 3.14
```

---

## 4. 控制流 (Control Flow)

### If / Else
Go 的 `if` 条件不需要括号 `()`。

**Python:**
```python
if age > 18:
    print("Adult")
else:
    print("Minor")
```

**Go:**
```go
if age > 18 {
    fmt.Println("Adult")
} else {
    fmt.Println("Minor")
}
```

### 循环 (Loops)
Go 只有 `for` 关键字，没有 `while`。`for` 可以胜任所有循环场景。

**Python (While):**
```python
count = 0
while count < 5:
    print(count)
    count += 1
```

**Go (While 风格):**
```go
count := 0
for count < 5 {
    fmt.Println(count)
    count++ // Go 中自增是语句，不是表达式
}
```

**Python (For range):**
```python
for i in range(5):
    print(i)
```

**Go (经典 C 风格):**
```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

---

## 5. 函数 (Functions)

Go 函数必须声明参数类型和返回值类型。Go 支持**多返回值**，这在 Python 中通常通过返回元组实现。

**Python:**
```python
def add(a, b):
    return a + b

def swap(a, b):
    return b, a
```

**Go:**
```go
func add(a int, b int) int {
    return a + b
}

// 多返回值
func swap(a, b string) (string, string) {
    return b, a
}
```

---

## 6. 结构体与方法 (Structs vs Classes)

Go 没有 `class`（类），而是使用 `struct`（结构体）来定义数据结构。方法通过“接收者” (Receiver) 绑定到结构体上。

**Python:**
```python
class Person:
    def __init__(self, name):
        self.name = name

    def greet(self):
        print(f"Hi, I'm {self.name}")

p = Person("Alice")
p.greet()
```

**Go:**
```go
type Person struct {
    Name string
}

// (p Person) 是接收者，类似于 Python 的 self
func (p Person) Greet() {
    fmt.Println("Hi, I'm", p.Name)
}

func main() {
    p := Person{Name: "Alice"}
    p.Greet()
}
```

---

## 7. 错误处理 (Error Handling)

Python 使用 `try-except` 异常机制。Go 提倡显式处理错误，将错误作为函数的最后一个返回值返回。

**Python:**
```python
try:
    f = open("file.txt")
except FileNotFoundError as e:
    print(e)
```

**Go:**
```go
import ("os"; "fmt")

f, err := os.Open("file.txt")
if err != nil {
    // 处理错误
    fmt.Println("Error:", err)
    return
}
// 成功打开
```

---

## 8. 并发 (Concurrency)

这是 Go 的杀手级特性。Go 使用 `Goroutine`，它比 Python 的线程轻量得多。

**Python (Threading):**
```python
import threading

def task():
    print("Running")

t = threading.Thread(target=task)
t.start()
```

**Go (Goroutine):**
只需在函数调用前加 `go` 关键字。
```go
func task() {
    fmt.Println("Running")
}

func main() {
    go task() // 启动一个新的 Goroutine
    // 注意：主程序退出时，所有 Goroutine 也会立即停止，实际使用中通常配合 Channel 或 WaitGroup
}
```

## 总结

- **简洁**：Go 只有 25 个关键字。
- **显式**：没有隐式类型转换，错误处理显式。
- **工具链**：Go 自带格式化 (`go fmt`)、测试 (`go test`) 和依赖管理 (`go mod`) 工具。

祝你在 Go 的学习之路上玩得开心！
