# OpenCode Labs

> 一套面向开发者的 **AI 协同编程**实战手册，系统掌握以 OpenCode 为代表的终端 AI 编程工具。

📖 [完整课程介绍与设计原则 →](COURSE-OVERVIEW.md)

---

## 实验列表

| 编号 | 实验名称 | 简介 | 状态 |
|------|----------|------|------|
| LAB-00 | [OpenCode 安装与环境准备](LAB-00-setup-and-install.md) | Windows/Linux/macOS 三平台安装，百炼 Coding Plan 配置，Prompt 驱动安装开发环境 | ✅ 已发布 |
| LAB-01 | [OpenCode 快速上手](LAB-01-opencode-quickstart.md) | TUI 界面、三种模式（explore/plan/build）、@ 文件引用、/init 命令、自定义 Agent 基础 | ✅ 已发布 |
| LAB-02 | [分析与文档化代码](LAB-02-analyze-document-code.md) | 用 explore 模式分析 Python 代码库，生成架构文档与 README | ✅ 已验证 |
| LAB-03 | [开发新功能](LAB-03-develop-code-features.md) | 用 build 模式为患者记录管理系统添加新功能 | ✅ 已验证 |
| LAB-04 | [开发单元测试](LAB-04-develop-unit-tests.md) | 生成覆盖正常路径、边界条件、异常处理的单元测试 | ✅ 已验证 |
| LAB-05 | [重构改进代码质量](LAB-05-refactor-improve-code.md) | 消除重复、提取函数、改善命名，渐进式重构 | ✅ 已验证 |
| LAB-06 | [快速原型：Vibe Coding 电商应用](LAB-06-vibe-coding-ecommerce.md) | 从零 Vibe Coding 一个电商原型，体验 AI 协同编程的速度上限 | ✅ 已验证 |
| LAB-07 | [并行 Agent 重构重复代码](LAB-07-consolidate-duplicate-code.md) | 并行启动多个 OpenCode 实例，同时重构多个独立模块 | ✅ 已验证 |
| LAB-08 | [子代理分析 + 主代理执行](LAB-08-refactor-large-functions.md) | 子代理深度分析大型函数，主代理按报告执行重构 | ✅ 已验证 |
| LAB-09 | [自定义 @code-reviewer Agent](LAB-09-simplify-complex-conditionals.md) | 创建代码审查 Agent，简化复杂条件语句（295 行 → 30 行） | ✅ 已验证 |
| LAB-10 | [自定义 @performance-auditor Agent](LAB-10-implement-performance-profiling.md) | 创建性能审计 Agent，识别并修复性能瓶颈 | ✅ 已验证 |
| LAB-11 | [自定义 @security-auditor Agent](LAB-11-security-vulnerability-detection-remediation.md) | 创建安全审计 Agent，三阶段并行安全修复流水线 | ✅ 已验证 |
| LAB-12 | [SDD Greenfield：规格驱动开发新项目](LAB-12-spec-driven-development-greenfield.md) | 从零创建 RSS Feed Reader，@spec-writer Agent 生成四层规格文档 | ✅ 已验证 |
| LAB-13 | [SDD Brownfield：为现有项目添加新功能](LAB-13-spec-driven-development-brownfield.md) | 为 ContosoDashboard 添加文档管理功能，分析现有架构约束后集成 | ✅ 已验证 |
| LAB-14 | [综合实战：AI 驱动完整软件工程周期](LAB-14-capstone-project.md) | 五阶段完整周期：原型 → 迭代 → 优化 → SDD 重构 → 复盘 | ✅ 已发布 |

---

## OpenCode 与 GitHub Copilot（VS Code）的使用场景差异

两者都是优秀的 AI 辅助编程工具，面向不同场景，互补而非竞争。

| 维度 | OpenCode（终端 TUI）| GitHub Copilot（VS Code）|
|------|---------------------|--------------------------|
| 交互方式 | 终端对话，全键盘 | IDE 内嵌，实时补全 |
| 适用场景 | 大范围重构、多文件协同、自动化任务 | 行级补全、函数生成、即时建议 |
| Agent 能力 | 自定义 Agent、并行多实例 | 内置 Copilot Chat |
| Provider 灵活性 | 75+ 提供商，支持国内服务（百炼等）| 主要使用 GitHub/Azure 托管模型 |
| 工作流集成 | 适合 CI/CD、脚本化、无头环境 | 适合 GUI 开发、调试、可视化 |

**一句话**：GitHub Copilot 是"在编辑器里随时获得建议"，OpenCode 是"把一个完整任务交给 AI 执行"。很多开发者同时使用两者。

---

## 关于本课程

### 致谢

本课程基于微软 **[mslearn-github-copilot-dev](https://github.com/MicrosoftLearning/mslearn-github-copilot-dev)** 课程改编。感谢微软学习团队提供的优质原始素材和开放的学习资源。原课程围绕 GitHub Copilot 和 VS Code 设计，本课程在保留核心学习目标的基础上，将工具链替换为 OpenCode，并扩展了多代理协同、自定义 Agent、规格驱动开发等进阶内容。

### 三方协同创作

本课程手册由**人类 + OpenClaw + OpenCode** 三方协同完成：

- **人类（课程设计者）**：课程架构、实验目标、质量把关、最终审核
- **[OpenClaw](https://openclaw.ai)**：任务编排、子代理调度、进度管理、发布流程
- **[OpenCode](https://opencode.ai)**：手册内容生成、代码验证、问题修复

这种创作方式本身就是 **AI 协同编程**（AI-Collaborative Programming）的体现——人类定义"做什么"和"为什么"，AI 负责"怎么做"的执行细节。整个课程的生产过程，就是课程所教内容的最佳示范。

> **AI 协同编程**：人类保持架构决策权与质量判断，AI 负责代码执行与方案生成，两者在每个开发环节持续交互、相互增强。区别于"Vibe Coding"（放手让 AI 写），协同编程中人类始终是决策主体。

### 工具与资源

- **OpenCode**：[opencode.ai](https://opencode.ai) — 开源终端 AI 编程助手
- **阿里云百炼**：[bailian.console.aliyun.com](https://bailian.console.aliyun.com) — 推荐的国内 LLM 服务
- **原版课程**：[mslearn-github-copilot-dev](https://github.com/MicrosoftLearning/mslearn-github-copilot-dev)

---

*最后更新：2026-03-02 | 实验数量：15（LAB-00 ~ LAB-14）*
