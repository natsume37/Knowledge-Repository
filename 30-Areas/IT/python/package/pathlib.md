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
