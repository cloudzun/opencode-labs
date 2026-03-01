# LAB-02：使用 OpenCode 分析与理解 Python 代码库

## 1. 实验标题和概述

### 学习目标
完成本实验后，你将能够：
- 使用 OpenCode 终端 TUI 工具分析和理解 Python 项目代码
- 掌握 Plan 模式和 Build 模式的切换时机
- 使用 `@explore` agent 探索项目架构
- 通过文件引用（@file）深入分析特定模块
- 使用 OpenCode 生成项目文档并完成 Git 提交

### 预计时间
30-40 分钟

### 前置条件
- 具备基础 Python 开发经验
- 已安装 Python 3.8+ 和 pip
- 熟悉基本 Git 操作（clone、add、commit、diff）
- 初次接触 OpenCode 工具

---

## 2. 环境准备

### 2.1 安装并验证 OpenCode

确保 OpenCode 已正确安装：

```bash
opencode --version
```

如果未安装，请参考官方文档进行安装。

### 2.2 配置模型

OpenCode 需要配置有效的模型才能工作。检查可用模型：

```bash
# 查看可用模型列表
opencode models
```

确保已配置可用的 LLM 模型（如 Claude、GPT-4、Qwen 等）。如需添加模型，参考 `opencode auth login` 命令。

### 2.3 设置全局 AGENTS.md（可选）

AGENTS.md 是 OpenCode 的全局规则配置文件，可以定义 AI 助手的编码规范。

在用户主目录创建或编辑 `~/.config/opencode/AGENTS.md`：

```markdown
# 全局编码规范

## Python 风格
- 遵循 PEP 8 代码风格
- 使用类型注解
- 函数和类必须有文档字符串

## 提交规范
- 提交信息使用中文
- 遵循"类型：描述"格式，如"feat: 添加用户登录功能"
```

---

## 3. 项目配置

### 3.1 下载示例项目

本实验使用一个 Python 图书馆管理系统作为分析对象。创建项目目录并准备代码：

```bash
# 创建项目目录
mkdir -p library
cd library

# 初始化 Git 仓库
git init
```

### 3.2 下载示例项目代码

本实验使用微软提供的图书馆管理系统示例代码：

```bash
# 下载项目 zip 包
curl -L -o AZ2007LabAppM2Python.zip \
  "https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM2Python.zip"

# 解压
unzip AZ2007LabAppM2Python.zip

# 进入项目目录
cd AccelerateDevGHCopilot/library
```

解压后的项目结构如下：

```
AccelerateDevGHCopilot/library/
├── application_core/
├── console/
│   ├── console_app.py
│   └── main.py
├── infrastructure/
│   ├── json_data.py
│   ├── json_loan_repository.py
│   └── json_patron_repository.py
└── tests/
```

### 3.3 初始化 AGENTS.md

在项目根目录创建 AGENTS.md，让 OpenCode 了解项目规范：

```bash
# 使用 OpenCode 初始化 AGENTS.md
opencode
```

在 OpenCode 中输入：

```
/init
```

OpenCode 会自动分析项目并生成适合的 AGENTS.md 文件。

### 3.4 验证项目可运行

```bash
# 安装依赖（如有 requirements.txt）
pip install -r requirements.txt

# 运行测试验证
python -m pytest tests/ -v
```

---

## 4. 任务一：使用 OpenCode 分析代码库

### 4.1 启动 OpenCode 并初始化项目

```bash
# 在项目根目录启动 OpenCode
opencode
```

启动后你会看到 OpenCode 的终端界面。首次进入时，输入：

```
/init
```

**你应该看到**：OpenCode 分析项目结构并生成 AGENTS.md 文件，包含项目的编码规范和上下文信息。

### 4.2 全局探索项目架构（Plan 模式 + @explore）

**切换到 Plan 模式**：按 `Tab` 键，状态栏会显示当前模式（`plan` 或 `build`）。

Plan 模式用于分析和规划，不会直接修改代码。

在 Plan 模式下输入：

```
@explore 帮我梳理这个项目的整体架构和技术栈，包括：
1. 项目的目录结构和各模块的职责
2. 使用的核心技术框架和库
3. 数据流和模块间的依赖关系
4. 入口程序和主要执行流程
```

**你应该看到**：
- `@explore` agent 会遍历项目文件
- 输出一份详细的项目架构分析报告
- 识别出核心模块：`application_core`（领域模型）、`console`（用户界面）、`infrastructure`（数据持久化）

**提示**：可以使用 `/compact` 命令压缩上下文，避免对话过长导致响应变慢。

### 4.3 深入分析核心类（文件引用 @file）

现在让我们深入分析具体的核心类。

**分析控制台应用入口**：

```
@console/main.py 解释这个文件的启动流程，它是如何初始化并运行应用程序的？
```

**分析核心应用类**：

```
@console/console_app.py 这个类的用途和调用关系是什么？
它提供了哪些主要功能？其他模块是如何使用它的？
```

**你应该看到**：
- OpenCode 会解析该文件的代码结构
- 说明 `ConsoleApp` 类的主要职责
- 列出调用该类的方法有哪些

### 4.4 理解数据访问层（多文件上下文）

数据访问层涉及多个文件的协作。使用多文件引用：

```
@infrastructure/json_data.py @infrastructure/json_loan_repository.py @infrastructure/json_patron_repository.py

请分析这三个文件如何协作实现数据持久化：
1. json_data.py 提供了什么基础功能？
2. json_loan_repository.py 和 json_patron_repository.py 如何继承和扩展基础功能？
3. 数据存储的格式和结构是怎样的？
```

**你应该看到**：
- OpenCode 会同时读取三个文件的内容
- 分析它们之间的继承关系和调用链
- 解释 JSON 数据存储的结构

**深入理解业务逻辑**（在 Plan 模式下）：

```
@application_core/loan.py 请详细解释这个文件中贷款业务逻辑的实现，包括主要类、方法和数据结构。
```

Plan 模式下 OpenCode 会仔细分析代码但不做任何修改。

---

## 5. 任务二：运行并理解应用

### 5.1 在终端运行应用

保持在 OpenCode 界面中，你可以直接在内置终端运行应用：

```bash
# 在 OpenCode 中执行命令，使用 ! 前缀
!python console/main.py
```

**你应该看到**：图书馆管理系统的控制台界面启动，显示主菜单选项。

### 5.2 交互式探索

回到对话模式（按 `Tab` 切换回 Plan 模式），提问：

```
根据刚才运行的输出，这个应用程序提供了哪些用户功能？
用户如何通过菜单进行借书、还书和查询操作？
```

OpenCode 会结合运行输出和代码分析给出详细解答。

### 5.3 使用 /undo 撤销不当操作

如果你在分析过程中让 AI 执行了某些操作（如生成了不需要的代码），可以使用：

```
/undo
```

这会撤销最近一次 AI 执行的文件修改操作。

---

## 6. 任务三：生成项目文档

### 6.1 切换到 Build 模式生成 README

**切换到 Build 模式**：按 `Tab` 键切换到 Build 模式，状态栏显示 `build`

Build 模式允许 AI 直接编写和修改文件。

输入以下 prompt：

```
请为这个图书馆管理系统项目生成一份完整的 README.md 文档，包括：
1. 项目简介和功能概述
2. 安装和运行说明
3. 项目结构说明
4. 核心模块介绍
5. 使用示例
6. 测试方法
```

**你应该看到**：
- OpenCode 在 Build 模式下会直接创建 `README.md` 文件
- 内容基于之前对项目分析的理解
- 文件创建后会显示在对话框中供你确认

### 6.2 审查生成的内容

使用 Git 命令审查变化：

```bash
# 在 OpenCode 中执行
!git diff
```

**你应该看到**：新增 README.md 文件的完整内容。

### 6.3 提交代码

如果满意生成的内容，进行 Git 提交：

```bash
# 暂存文件
!git add README.md

# 提交
!git commit -m "docs: 生成项目 README 文档"
```

或者让 OpenCode 帮你完成：

```
请帮我将生成的 README.md 提交到 git，提交信息为"docs: 生成项目 README 文档"
```

### 6.4 查看提交历史

```bash
!git log --oneline -5
```

**你应该看到**：最近的提交记录，包括刚才创建的 README 提交。

---

## 7. 实验总结

### 你学到了什么

完成本实验后，你应该掌握了：

| 技能 | OpenCode 操作 |
|------|-------------|
| 项目初始化 | `/init` 自动生成 AGENTS.md |
| 架构探索 | Plan 模式 + `@explore` |
| 文件分析 | `@path/to/file.py` 引用具体文件 |
| 多文件上下文 | 在 prompt 中引用多个文件 |
| 模式切换 | `Tab` 在 Plan/Build 间切换 |
| 命令执行 | `!command` 在 OpenCode 中运行终端命令 |
| 撤销操作 | `/undo` 撤销 AI 的文件修改 |
| Git 工作流 | `!git diff` 审查 + `!git commit` 提交 |

### OpenCode 与 GitHub Copilot 对比

| 操作 | GitHub Copilot (VSCode) | OpenCode (Terminal) |
|------|------------------------|---------------------|
| 项目探索 | `@workspace describe` | `@explore` + Plan 模式 |
| 文件分析 | `#usages` | `@file` 引用 |
| 解释代码 | `/explain` | `/explain` (Plan 模式) |
| 生成文件 | Chat 后手动粘贴 | Build 模式直接写入 |
| 上下文管理 | 侧边栏拖拽 | prompt 中 @ 引用 |

---

## 8. 附录：快速参考

### 8.1 常用命令

| 命令 | 说明 |
|------|------|
| `/init` | 初始化项目，生成 AGENTS.md |
| `/undo` | 撤销最近的文件修改 |
| `/compact` | 压缩上下文，减少 token 消耗 |
| `/new` | 新建会话 |
| `/sessions` | 查看历史会话 |
| `/help` | 显示帮助信息 |
| `/exit` | 退出 OpenCode |
| `!command` | 执行终端命令（如 `!git status`） |

### 8.2 Agent 引用

| Agent | 用法 |
|-------|------|
| `@explore` | 探索项目架构和技术栈 |
| `@file.py` | 引用具体文件进行分析 |
| `@dir/` | 引用整个目录 |

### 8.3 快捷键（Leader 键默认是空格键 Space）

| 快捷键 | 功能 |
|--------|------|
| `Tab` | 切换 Plan/Build 模式 |
| `Leader → N` | 新建会话 |
| `Leader → L` | 会话列表 |
| `Leader → M` | 模型列表（切换模型） |
| `Leader → U` | 撤销（同 `/undo`） |
| `Leader → C` | 压缩上下文（同 `/compact`） |
| `Escape` | 中断 AI 响应 |
| `Ctrl + C` | 清空输入框/退出 |
| `Ctrl + D` 或 `/exit` | 退出 OpenCode |

### 8.4 工作流建议

1. **首次接触项目**：
   ```
   /init → @explore 项目架构 → @核心文件深入分析
   ```

2. **日常开发**：
   ```
   Plan 模式分析问题 → Build 模式实现代码 → !git diff 审查 → !git commit 提交
   ```

3. **代码审查**：
   ```
   @修改的文件 请审查这些改动是否合理 → /undo (如有问题) → !git commit
   ```

### 8.5 常见问题

**Q: Plan 模式和 Build 模式有什么区别？**
- Plan 模式：只分析和建议，不修改文件
- Build 模式：可以直接创建和修改文件

**Q: 如何确认 AI 修改的内容？**
- 使用 `!git diff` 查看所有变更
- 满意后再 `!git add` 和 `!git commit`

**Q: 上下文太长怎么办？**
- 使用 `/compact` 压缩当前会话上下文
- 开始新会话 `/new` 清空上下文
- 精确引用需要的文件（`@file.py`），避免一次引用整个目录

---

## 实验完成检查清单

- [ ] 成功启动 OpenCode 并执行 `/init`
- [ ] 使用 `@explore` 完成项目架构分析
- [ ] 使用 `@file` 引用分析了至少 3 个核心文件
- [ ] 在 OpenCode 中运行了 Python 应用（`!python ...`）
- [ ] 切换到 Build 模式生成了 README.md
- [ ] 使用 `!git diff` 审查了更改
- [ ] 完成了 Git 提交

---

**祝你在 OpenCode 的学习之旅中收获满满！** 🚀

</content>