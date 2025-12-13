# 个人知识库 (Personal Knowledge Base)

这是一个知识平台备份仓库，用来备份平时的笔记、学习记录、日常思考以及 BUG 修复记录。

本仓库采用改良版的 **PARA** （Projects, Areas, Resources, Archives）架构，结合 Obsidian 的双向链接特性，旨在打造一个井井有条且具有生命周期的知识管理系统。

## 📂 目录结构说明

本仓库采用以下目录结构：

- **00-Inbox (收集箱)**: 临时存放处。还没想好放哪里的想法、手机端快速记录的内容都先丢这里，定期整理。
- **10-Daily (日常记录)**: 存放每日笔记 (Daily Notes)。用于记录当天的流水账、待办事项和突发灵感。
- **20-Projects (当前项目)**: 正在进行的具体任务（如 "搭建个人博客", "学习 Go 语言"）。
- **30-Areas (长期领域)**: 长期关注的领域和沉淀的知识（如 IT 技术栈、生活感悟）。
- **40-Resources (资源库)**: 外部引用的资源、摘录的文章、代码片段、读书笔记。
- **99-Archives (归档)**: 已完成的项目或不再维护的资料。
- **Templates (模板)**: 存放 Obsidian 的笔记模板（如 BUG 报告、日记模板、通用笔记模板）。
- **Assets (附件)**: 统一存放图片、PDF 等附件，保持根目录整洁。

## ⚙️ Obsidian 设置指南

为了获得最佳体验，请在 Obsidian 中进行以下设置：

### 1. 附件管理

让图片自动归档到 `Assets` 文件夹，避免污染笔记目录。

- 进入 **设置 (Settings)** -> **文件与链接 (Files & Links)**
- **新建附件的默认位置 (Default location for new attachments)**: 选择 **"指定的文件夹 (In the folder specified below)"**
- **附件文件夹路径**: 输入 `Assets`

### 2. 模板设置 (Templates)

启用模板功能，快速创建标准化的笔记。

- 进入 **设置 (Settings)** -> **核心插件 (Core Plugins)** -> 开启 **模板 (Templates)**
- 点击齿轮图标 ⚙️ 配置插件：
  - **模板文件夹位置 (Template folder location)**: 输入 `Templates`

### 3. 日记设置 (Daily Notes)

配置每日笔记的自动存放位置和格式。

- 进入 **设置 (Settings)** -> **核心插件 (Core Plugins)** -> 开启 **日记 (Daily notes)**
- 点击齿轮图标 ⚙️ 配置插件：
  - **新文件位置 (New file location)**: 输入 `10-Daily`
  - **模板文件位置 (Template file location)**: 选择 `Templates/Daily_Note_Template.md`

## 📝 使用工作流建议

1. **快速记录**: 有想法直接 `Ctrl+N` (或手机端) 写入 `00-Inbox`，或者写在当天的 `Daily Note` 中。
2. **遇到 BUG**: 新建笔记 -> 插入模板 `Bug_Report_Template` -> 记录并打上标签（如 `#bug/python`）。
3. **定期回顾**: 每周回顾 `00-Inbox` 和 `10-Daily`，将有价值的内容移动到 `30-Areas` 或 `20-Projects` 中深化。
