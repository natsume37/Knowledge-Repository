---
created: 2026-01-07 13:42
author: natsume37
category:
tags:
  - 
---

# pathlib

>  object oriented filesystem paths
>  pathlib 是 Python 3.4+ 引入的**面向对象路径处理库**，旨在替代传统的 `os.path` 模块，提供更简洁、直观、跨平台的路径操作能力。


### 1. 核心类区分

|类名|作用|特点|
|---|---|---|
|`PurePath`（纯路径）|仅处理路径字符串，不涉及实际文件系统 IO|跨平台（自动适配 Windows/Unix 路径分隔符）、无 IO 副作用|
|`Path`（具体路径）|继承 `PurePath`，增加文件系统 IO 操作|可判断文件是否存在、创建 / 删除文件 / 目录、读写文件等|

### 2. 常用操作示例

#### （1）路径创建与拼接

路径拼接用 ** `/` 运算符 **（替代 `os.path.join`），直观且无字符串拼接错误：

```
from pathlib import Path

# 1. 绝对路径
path1 = Path("/home/user/data")  # Unix
path2 = Path("C:\\Users\\user\\data")  # Windows（或Path("C:/Users/user/data")，自动兼容）

# 2. 相对路径
path3 = Path("project") / "config" / "settings.yaml"  # 拼接：project/config/settings.yaml

# 3. 结合系统路径（用户目录、当前目录）
current_dir = Path.cwd()  # 当前工作目录（等价于 os.getcwd()）
home_dir = Path.home()    # 用户主目录（等价于 os.path.expanduser("~")）
data_dir = home_dir / "data" / "csv"  # 跨平台兼容：无需关心分隔符
```

注意：


#### （2）路径属性与拆分

直接通过对象属性获取路径组件（替代 `os.path.splitext`/`os.path.dirname` 等嵌套函数）：


```
file_path = Path("project/data/report_2024.csv")

file_path.parent          # 父目录：PosixPath('project/data')（等价于 os.path.dirname）
file_path.parent.parent   # 上层父目录：PosixPath('project')
file_path.name            # 文件名（含后缀）：'report_2024.csv'
file_path.stem            # 文件名（不含后缀）：'report_2024'
file_path.suffix          # 文件后缀：'.csv'
file_path.suffixes        # 多后缀（如 .tar.gz）：['.csv']
file_path.root            # 根目录：'/'（Unix）或 'C:\\'（Windows）
file_path.resolve()       # 绝对路径：/xxx/project/data/report_2024.csv
```

#### （3）文件 / 目录 IO 操作

`Path` 内置 IO 方法，无需手动调用 `open`/`os.makedirs` 等，代码更简洁：


```
# 1. 判断路径类型
file_path.exists()        # 是否存在（文件/目录）
file_path.is_file()       # 是否为文件
file_path.is_dir()        # 是否为目录
file_path.is_absolute()   # 是否为绝对路径

# 2. 文件操作
file_path.touch()         # 创建空文件（等价于 os.mknod）
file_path.write_text("hello")  # 写入文本（自动关闭文件，替代 open+write）
content = file_path.read_text()  # 读取文本（自动关闭文件）

# 3. 目录操作
dir_path = Path("project/logs")
dir_path.mkdir(parents=True, exist_ok=True)  # 创建多级目录（parents=True：父目录不存在则创建；exist_ok=True：已存在不报错，等价于 os.makedirs）
dir_path.rmdir()          # 删除空目录（等价于 os.rmdir）
file_path.unlink()        # 删除文件（等价于 os.remove）
```

#### （4）路径遍历与匹配

通过 `glob`/`rglob` 实现文件匹配（替代 `os.listdir`/`os.walk`），支持通配符：

```
# 1. 匹配当前目录下所有 .csv 文件
for csv_file in Path("data").glob("*.csv"):
    print(csv_file)  # data/report_2024.csv, data/input.csv...

# 2. 递归匹配所有子目录下的 .csv 文件（r=recursive）
for csv_file in Path("data").rglob("*.csv"):
    print(csv_file)  # data/2024/report.csv, data/old/input.csv...

# 3. 匹配特定模式（如 2024 开头的 .txt 文件）
for txt_file in Path("logs").glob("2024-*.txt"):
    pass
```

## 二、pathlib 与 os/os.path 模块的核心区别

`os` 是 Python 内置的传统系统模块，`os.path` 是其路径处理子模块；pathlib 是面向对象的现代化替代方案，核心差异体现在「编程范式」「可读性」「功能集成」上：

|对比维度|pathlib（推荐）|os/os.path（传统）|
|---|---|---|
|编程范式|面向对象（路径是对象）|函数式（路径是字符串，通过函数处理）|
|路径拼接|`/` 运算符（直观简洁）：`Path("a") / "b"`|`os.path.join(a, b)`（嵌套繁琐）|
|路径属性获取|直接通过属性：`path.parent`/`path.suffix`|嵌套函数：`os.path.dirname(path)`/`os.path.splitext(path)[1]`|
|文件 IO 集成|内置 `read_text()`/`write_text()`/`mkdir()`|需结合 `open()`/`os.makedirs()` 等，分开调用|
|跨平台兼容性|自动适配路径分隔符（`\`/`/`）|需手动用 `os.path.join`，否则易出错|
|代码可读性|链式调用（`path.parent.parent.stem`）|多层嵌套（`os.path.splitext(os.path.dirname(path))[0]`）|
|遍历与匹配|`glob()`/`rglob()`（简洁）|`os.listdir()`+ 手动过滤，或 `os.walk()`（代码冗长）|
|异常处理|抛出具体异常（如 `FileNotFoundError`）|部分函数返回 `None` 或 `False`，需手动判断|
|适用场景|新项目、跨平台开发、文件密集型操作|旧 Python 版本（<3.4）、底层系统调用场景|

### 直观代码对比

#### 示例 1：路径拼接与属性获取


```
# pathlib
path = Path.home() / "project" / "data.csv"
print(path.parent)  # /home/user/project
print(path.suffix)  # .csv

# os.path
path = os.path.join(os.path.expanduser("~"), "project", "data.csv")
print(os.path.dirname(path))  # /home/user/project
print(os.path.splitext(path)[1])  # .csv
```

#### 示例 2：递归遍历所有 .txt 文件


```
# pathlib（1行遍历）
for txt_file in Path("logs").rglob("*.txt"):
    pass

# os.path（需嵌套循环）
for root, dirs, files in os.walk("logs"):
    for file in files:
        if file.endswith(".txt"):
            txt_file = os.path.join(root, file)
            pass
```

## 三、pathlib 在实际开发中的核心用处

pathlib 解决了传统 `os.path` 的「代码繁琐」「易出错」「跨平台兼容差」等问题，在以下场景中尤为实用：

### 1. 项目路径统一管理（最常用）

开发中需避免硬编码绝对路径，用 pathlib 基于项目根目录动态生成路径，确保项目可移植：

python

运行

```
# 项目结构：project/
#   ├─ src/
#   │  └─ config.py（当前文件）
#   ├─ data/
#   ├─ config/
#   └─ logs/

from pathlib import Path

# 1. 获取项目根目录（src 的上层目录）
PROJECT_ROOT = Path(__file__).parent.parent  # __file__ 是当前 config.py 的路径

# 2. 动态生成各模块路径
CONFIG_PATH = PROJECT_ROOT / "config" / "settings.yaml"
DATA_PATH = PROJECT_ROOT / "data"
LOG_PATH = PROJECT_ROOT / "logs"
MODEL_PATH = PROJECT_ROOT / "models" / "trained_model.pkl"

# 3. 确保目录存在（启动时执行）
LOG_PATH.mkdir(parents=True, exist_ok=True)
DATA_PATH.mkdir(parents=True, exist_ok=True)
```

### 2. 批量文件处理（数据 / 日志 / 报表）

数据分析、日志归档、报表生成等场景中，需遍历 / 匹配大量文件，pathlib 的 `glob`/`rglob` 大幅简化代码：


```
# 示例：批量读取 data 目录下所有子目录的 .csv 文件，合并为 DataFrame
import pandas as pd
from pathlib import Path

def merge_csv_data(data_dir: str) -> pd.DataFrame:
    dir_path = Path(data_dir)
    # 递归匹配所有 .csv 文件
    csv_files = list(dir_path.rglob("*.csv"))
    # 批量读取并合并
    df_list = [pd.read_csv(file) for file in csv_files]
    return pd.concat(df_list, ignore_index=True)

merged_df = merge_csv_data("data")
```

### 3. 跨平台脚本开发

如果脚本需要在 Windows、Linux、Mac 上运行，pathlib 自动适配路径分隔符，无需手动处理：


```
# 跨平台创建用户目录下的配置文件
config_path = Path.home() / ".myapp" / "config.json"
config_path.parent.mkdir(parents=True, exist_ok=True)
config_path.write_text('{"theme": "dark"}')  # 无需关心路径是 ~/.myapp/ 还是 C:\Users\user\.myapp\
```

### 4. 简化文件 IO 操作

无需手动管理 `open` 的上下文（或忘记关闭文件），`read_text()`/`write_text()` 直接读写文本文件：


```
# 读取配置文件
config = Path("config.yaml").read_text(encoding="utf-8")

# 写入日志
log_content = "2024-01-01: 程序启动成功"
Path("app.log").write_text(log_content, encoding="utf-8")

# 二进制文件（read_bytes()/write_bytes()）
image_data = Path("photo.jpg").read_bytes()
```

### 5. 路径验证与处理（无 IO 场景）

用 `PurePath` 处理路径字符串（如用户输入、配置文件中的路径），无需访问文件系统：


```
from pathlib import PurePath

# 验证用户输入的路径是否为 .csv 后缀
def is_csv_path(path_str: str) -> bool:
    return PurePath(path_str).suffix == ".csv"

print(is_csv_path("data.csv"))  # True
print(is_csv_path("data.txt"))  # False
```

## 四、总结与推荐

- **核心优势**：pathlib 以「面向对象」设计简化路径操作，代码更简洁、可读性更高、跨平台性更强，还集成了文件 IO，大幅减少路径相关 bug。
- **适用场景**：新项目、跨平台开发、文件密集型操作（数据处理、日志管理、项目配置）。
- **兼容建议**：若需兼容 Python 2 或依赖 `os` 的底层功能（如 `os.fork()`），可继续使用 `os`；否则优先选择 pathlib。

**一句话推荐**：Python 3.4+ 开发中，pathlib 是路径处理的首选，能显著提升开发效