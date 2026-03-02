# 任务：起草 LAB-02 OpenCode 实验手册

## 背景
我们正在将微软 GitHub Copilot 课程的实验手册改造为 OpenCode 版本。
原版 Lab 使用 VSCode + GitHub Copilot，我们需要改造为使用 OpenCode（终端 TUI）完成相同学习目标。

## 原版 Lab 核心内容（LAB_AK_02）
原版 Lab 做三件事：
1. 下载并配置一个 Python 图书馆管理系统项目
2. 用 GitHub Copilot 分析和理解代码库（@workspace、#usages、/explain）
3. 用 GitHub Copilot 生成 README.md 文档

## 改造映射关系
| VSCode + Copilot | OpenCode 等效操作 |
|---|---|
| `@workspace describe this project` | Plan 模式 + `@explore 帮我梳理这个项目的整体架构和技术栈` |
| `@workspace #usages How is ConsoleApp used?` | `@src/console_app.py 这个类的用途和调用关系是什么？` |
| `/explain main.py` | Plan 模式 + `@console/main.py 解释这个文件的启动流程` |
| 拖拽文件到 Chat 上下文 | prompt 里直接 `@infrastructure/json_data.py @infrastructure/json_loan_repository.py` |
| 生成内容后复制粘贴到文件 | Build 模式直接让 AI 写文件 |

## OpenCode 特有内容（需要加入）
- `/init` 自动生成项目 AGENTS.md
- Plan ↔ Build 模式切换时机
- `@explore` agent 使用
- `/undo` 撤销机制
- Leader 快捷键（Leader+N/M/L/U 等）
- `!git diff` 审查 + commit 节奏

## OpenCode 最佳实践参考
重点参考：
- 第一节：全局规则设置（AGENTS.md 配置）
- 第二节：上手新项目的推进流程（5步走）
- 第四节：日常维护与操作习惯

## 实验手册要求
请生成一份完整的中文实验手册，文件名 `LAB-02-analyze-document-code.md`，包含以下结构：

1. **实验标题和概述**（学习目标、预计时间、前置条件）
2. **环境准备**（安装验证、模型配置、全局 AGENTS.md 设置）
3. **项目配置**（下载代码、初始化 git、运行测试验证）
4. **任务一：使用 OpenCode 分析代码库**
   - 4.1 启动 OpenCode 并初始化项目
   - 4.2 全局探索项目架构（Plan 模式 + @explore）
   - 4.3 深入分析核心类（文件引用 @file）
   - 4.4 理解数据访问层（多文件上下文）
5. **任务二：运行并理解应用**（终端运行 Python 应用）
6. **任务三：生成项目文档**（Build 模式生成 README.md，git commit）
7. **实验总结**（学到了什么）
8. **附录：快速参考**（常用命令和快捷键）

### 写作规范
- 全程中文
- 每个操作步骤要有具体的命令/prompt 示例（代码块格式）
- 截图说明用文字描述替代（如"你应该看到..."）
- 预计完成时间：30-40分钟
- 面向对象：有基础 Python 经验的开发者，初次接触 OpenCode
- 语气：专业但友好

## 项目代码结构（供参考）
```
AccelerateDevGHCopilot/library
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

## 输出
请直接创建文件 `LAB-02-analyze-document-code.md`，内容完整，可以直接交给学生使用。
