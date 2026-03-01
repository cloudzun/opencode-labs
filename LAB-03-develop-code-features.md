# LAB-03: 使用 OpenCode 开发代码功能

## 1. 实验概述

### 学习目标
- 掌握 OpenCode 的 **Plan 模式** 和 **Build 模式** 切换
- 学会使用 `@文件路径` 引用多个文件进行上下文感知开发
- 实践使用 `@explore` agent 理解项目结构
- 完成一个完整的"图书可用性查询"功能开发

### 预计时间
40-50 分钟

### 前置条件
- 已完成 LAB-02，熟悉 OpenCode 基本操作
- 具备 Python 基础，了解枚举、类、JSON 数据处理
- 已安装 OpenCode 并配置好模型

---

## 2. 环境准备

### 2.1 验证 OpenCode 安装
```bash
opencode --version
```

### 2.2 检查模型配置
```bash
opencode config
```
确保已配置支持长上下文的模型（推荐 qwen3.5-plus 或类似）。

---

## 3. 项目配置

### 3.1 下载并解压项目
从课程资源下载 `AZ2007LabAppM3Python.zip`（注意是 **M3** 版本）：

```bash
unzip AZ2007LabAppM3Python.zip -d library
cd library
```

### 3.2 初始化 Git 仓库
```bash
git init
git add .
git commit -m "Initial commit: library management system"
```

### 3.3 验证项目可运行
```bash
python3 -m unittest discover tests
```
所有测试应通过。

---

## 4. 任务一：创建功能分支

### 4.1 创建并切换到功能分支
```bash
git checkout -b book-availability
```

### 4.2（可选）推送到远程
如果已配置 GitHub remote：
```bash
git push -u origin book-availability
```

---

## 5. 任务二：理解现有代码结构（Plan 模式）

### 5.1 使用 @explore 理解项目

启动 OpenCode：
```bash
opencode
```

**Step 1:** 使用 `@explore` agent 快速了解项目结构：

```
@explore 请分析这个图书馆管理项目的代码结构，重点说明：
1. console/ 目录下的主要文件及其职责
2. infrastructure/ 目录下的 JSON 数据文件有哪些
3. 数据流向：从用户输入到查询 JSON 文件的完整路径
```

### 5.2 理解 JSON 数据结构

**Step 2:** 分析三个核心 JSON 文件的关联关系：

```
@infrastructure/Json/Books.json @infrastructure/Json/BookItems.json @infrastructure/Json/Loans.json
请分析这三个 JSON 文件的数据结构和关联关系，用于实现图书可用性查询功能。
具体说明：
- Books.json 中每条记录包含哪些字段
- BookItems.json 如何关联 Books
- Loans.json 如何关联 BookItems 和 Patron
```

> **预期输出**：你会看到类似这样的关联关系：
> - `Books.id` → `BookItems.bookId`
> - `BookItems.id` → `Loans.bookItemId`
> - 通过这三层关联可以判断某本书是否有可借阅的副本

### 5.3 理解代码修改点

**Step 3:** 分析需要修改的代码文件：

```
@console/common_actions.py @console/console_app.py
我需要添加一个"图书可用性查询"功能。请分析现有代码结构，列出：
1. 需要在 CommonActions 枚举中添加什么新值
2. 需要在 console_app.py 中修改哪些方法
3. 需要新增什么方法

**只给出规划，不要写代码。**
```

> **关键提示**：这是典型的 **Plan 模式**——让 AI 先分析、规划，列出修改方案，不急于写代码。

---

## 6. 任务三：使用 OpenCode 开发功能

### 6.1 添加 SEARCH_BOOKS 枚举值

**Step 4:** 切换到 **Build 模式**，修改 `common_actions.py`：

```
@console/common_actions.py
在 CommonActions 枚举中添加 SEARCH_BOOKS = auto()，位置在 SEARCH_PATRONS 之后。
直接修改文件。
```

验证修改：
```bash
!git diff console/common_actions.py
```

### 6.2 更新菜单显示

**Step 5:** 修改 `write_input_options` 方法：

```
@console/console_app.py
找到 write_input_options 方法，在菜单选项中添加：
"{CommonActions.SEARCH_BOOKS.value}. Search Book Availability"
位置与 SEARCH_PATRONS 选项相邻。直接修改文件。
```

### 6.3 更新输入处理

**Step 6:** 修改 `_handle_patron_details_selection` 方法：

```
@console/console_app.py
找到 _handle_patron_details_selection 方法，在处理 patron_details 的 if 条件中：
在 options 列表中添加 CommonActions.SEARCH_BOOKS，与 SEARCH_PATRONS 在同一层级。
直接修改文件。
```

### 6.4 更新 patron_details 方法

**Step 7:** 确保 patron_details 方法的 options 包含新枚举值：

```
@console/console_app.py
检查 patron_details 方法中的 options 定义，确保包含 CommonActions.SEARCH_BOOKS。
如果已有则跳过，没有则添加。
```

### 6.5 实现 search_books 方法（核心功能）

**Step 8:** 实现完整的图书查询功能：

```
@console/console_app.py @infrastructure/json_data.py
@infrastructure/Json/Books.json @infrastructure/Json/BookItems.json @infrastructure/Json/Loans.json

请实现 search_books 方法，步骤如下：

1. 提示用户输入书名："Enter a book title to search for: "
2. 使用 self._json_data.books 加载书籍数据
3. 在 books 中查找匹配的书籍（不区分大小写，使用 title 字段）
4. 如果找到书籍，使用 self._json_data.book_items 找到对应的物理副本
5. 使用 self._json_data.loans 检查是否有未归还的借阅记录
6. 判断逻辑：
   - 如果所有副本都已借出（return_date 为 null）：显示已借出状态
   - 否则：显示可借阅状态
7. 输出格式：
   - 可借："{book.title} is available for loan"
   - 已借出："{book.title} is on loan to another patron. The return due date is {due_date}"
   （注意：due_date 需要格式化为 "YYYY-MM-DD" 格式，使用 due_date.strftime('%Y-%m-%d')）

注意：需要在 ConsoleApp 的 __init__ 中添加 json_data 参数，并在 main 函数中传递。

直接修改 console_app.py 文件，添加 search_books 方法。
```

> **关键提示**：这是典型的 **多文件上下文开发**——在一个 prompt 中引用多个相关文件（代码 + 数据），让 AI 理解完整的数据流。

### 6.6 更新 ConsoleApp 构造函数和 main 函数

**Step 8.5:** 由于 search_books 方法需要访问 JSON 数据，需要更新 ConsoleApp 的构造函数：

```
@console/console_app.py
在 ConsoleApp 的 __init__ 方法中添加 json_data 参数，并存储为 self._json_data。
同时更新 main 函数，在创建 ConsoleApp 实例时传递 json_data 参数。
```

---

## 7. 任务四：验证功能

### 7.1 运行单元测试
```bash
python3 -m unittest discover tests
```
确保所有测试通过。

### 7.2 手动测试

**Step 9:** 运行应用测试功能：

```bash
python3 console/main.py
```

测试流程：
1. 选择菜单中的 "Search Book Availability"
2. 输入书名（如 "The Great Gatsby"）
3. 验证输出：
   - 可借书籍显示 "is available for loan"
   - 已借出书籍显示 "is on loan to another patron. The return due date is XXXX-XX-XX"

### 7.3 审查代码改动

**Step 10:** 使用 `!git diff` 审查所有改动：

```
!git diff
```

确认修改符合预期后，可以使用 `/undo` 撤销不满意的改动。

---

## 8. 任务五：合并分支

### 8.1 提交功能分支
```
!git add . && git commit -m "feat: add book availability search"
```

### 8.2 合并到 main 分支
```
!git checkout main
!git merge book-availability
```

### 8.3（可选）删除功能分支
```
!git branch -d book-availability
```

---

## 9. 实验总结

### 9.1 Plan→Build→验证→Commit 工作流

本次实验完整实践了 OpenCode 的开发流程：

| 阶段 | 模式 | 工具/命令 | 目的 |
|------|------|-----------|------|
| **理解** | Plan | `@explore`、`@文件路径` | 理解项目结构和数据关系 |
| **规划** | Plan | 直接提问（不写代码） | 让 AI 列出修改方案 |
| **实现** | Build | `@文件路径` + 直接修改指令 | 让 AI 直接修改文件 |
| **验证** | - | `!git diff`、单元测试 | 审查改动、确保正确 |
| **提交** | - | `!git commit` | 提交到版本控制 |

### 9.2 OpenCode 相对于 VSCode + Copilot 的优势

1. **直接修改文件**：不需要"inline chat → accept → apply"多步骤确认
2. **多文件上下文**：一个 prompt 引用多个文件，AI 理解完整上下文
3. **Plan 模式先行**：先规划再执行，减少错误
4. **安全撤销**：`/undo` 基于 git 快照，比传统 Undo 更安全
5. **终端一体化**：`!git` 命令直接在 OpenCode 中执行

---

## 10. 附录：快速参考

### 10.1 常用命令
| 命令 | 用途 |
|------|------|
| `@explore` | 理解项目结构和文件关系 |
| `@文件路径` | 引用特定文件作为上下文 |
| `!git diff` | 审查所有改动 |
| `!git add/commit` | 提交代码 |
| `/undo` | 撤销上一次修改 |

### 10.2 关键 Prompt 模板

**理解数据结构**：
```
@文件 1 @文件 2 @文件 3
请分析这些数据文件的数据结构和关联关系，用于实现 XXX 功能
```

**规划功能实现（Plan 模式）**：
```
@代码文件 1 @代码文件 2
我需要实现 XXX 功能。请分析现有代码结构，列出需要修改的文件和方法，以及修改方案。不要写代码，先给出规划。
```

**实现功能（Build 模式）**：
```
@目标文件
请实现 XXX 功能：
1. 步骤一
2. 步骤二
3. 步骤三
直接修改文件。
```

### 10.3 项目文件结构参考
```
library/
├── console/
│   ├── common_actions.py   ← 添加枚举值
│   ├── console_app.py      ← 主要修改（菜单、处理、方法），main() 函数在此文件末尾
│   ├── console_state.py    ← 状态枚举定义
│   └── main.py
├── infrastructure/
│   ├── Json/
│   │   ├── Books.json      ← 查询数据源
│   │   ├── BookItems.json  ← 查询数据源
│   │   ├── Loans.json      ← 查询数据源
│   │   └── Patrons.json
│   ├── json_data.py        ← 统一数据访问（books, book_items, loans 属性）
│   ├── json_loan_repository.py
│   └── json_patron_repository.py
└── tests/
```

---

**恭喜你完成 LAB-03！** 你已经掌握了使用 OpenCode 进行多文件功能开发的完整工作流。
