# LAB-09：简化复杂条件语句

## 实验标题和概述

### 学习目标
- 掌握条件语句重构技巧（guard clause、switch expression、方法提取）
- **核心亮点**：学会创建和使用自定义 @code-reviewer Agent
- 理解项目级 Agent 与全局 Agent 的区别和使用场景

### 预计时间
35-45 分钟

### 前置条件
- .NET 8.0+ SDK
- OpenCode 已安装并完成基础配置
- 已完成 LAB-08（熟悉 OpenCode 基本操作）

---

## 环境准备

### 1. 验证开发环境
```bash
# 验证 .NET SDK 版本
dotnet --version

# 验证 OpenCode 已安装
opencode --version
```

### 2. 获取示例项目
从指定位置下载 `GHCopilotEx9LabApps.zip` 并解压：
```bash
# 进入解压后的目录
cd ECommercePricingEngine
```

### 3. 修改目标框架
如果项目使用 net9.0，需改为 net8.0：
```xml
<!-- ECommercePricingEngine.csproj -->
<TargetFramework>net8.0</TargetFramework>
```

### 4. 保存基准输出
```bash
# 运行程序，保存重构前的输出
dotnet run > ACTUAL_OUTPUT_BEFORE.txt
```

---

## 核心概念：自定义 Agent

### 为什么需要自定义 Agent？

| 场景 | 传统方式 | 自定义 Agent |
|------|----------|--------------|
| 代码审查 | 每次重复输入审查标准 | 一次配置，永久复用 |
| 分析聚焦 | 通用回答，不够深入 | 专属领域专家，输出标准化 |
| 效率 | 每次重新描述需求 | 调用即执行，无需重复说明 |

### Agent 配置文件位置

- **项目级**：`.opencode/agent/` → 只在本项目生效
- **全局级**：`~/.config/opencode/agent/` → 所有项目可用

**建议**：审查标准因项目而异，推荐使用项目级 Agent。

### Agent 配置文件格式

```markdown
---
description: 简短描述（在 @agent 列表中显示）
mode: subagent
---
# Agent 名称

Agent 的系统提示内容...
```

**mode: subagent 含义**：只读分析模式，专注于审查和建议，不直接修改代码。

---

## 任务一：创建 @code-reviewer Agent

### 1. 创建 Agent 目录
```bash
mkdir -p .opencode/agent
```

### 2. 创建 Agent 配置文件

在 OpenCode 中切换到 **Build 模式**，使用以下 prompt 创建配置文件：

```
请创建 .opencode/agent/code-reviewer.md 文件，内容如下：
```

文件内容：

```markdown
---
description: 条件语句复杂度审查专家
mode: subagent
---

# Code Reviewer Agent

你是一位专注于代码可读性和条件语句复杂度的代码审查专家。

## 审查重点

### 条件语句复杂度
- [ ] 嵌套层级是否超过 3 层？
- [ ] 是否有可以用 early return / guard clause 替代的嵌套？
- [ ] 是否有可以用 switch expression 替代的 if/else 链？
- [ ] 是否有重复的条件判断可以提取为 bool 变量？

### 方法职责
- [ ] 单个方法是否承担了多个职责？
- [ ] 是否有可以提取为独立 helper 方法的代码段？
- [ ] 方法长度是否超过 50 行？

### 可维护性
- [ ] 业务规则是否清晰可读？
- [ ] 条件分支的意图是否通过命名表达清楚？

## 输出格式

对于每个问题：
- **位置**：方法名:大概行号
- **问题**：具体描述
- **严重程度**：高 / 中 / 低
- **建议**：具体的重构建议（含代码示例）

最后给出**整体可读性评分**（1-10 分）和**重构优先级列表**。
```

### 3. 重启 OpenCode 使 Agent 生效

```bash
# 退出并重启 OpenCode
# 或执行 :reload 命令（如果支持）
```

### 4. 验证 Agent 可用

```
@code-reviewer 你好，请介绍你的审查能力
```

预期响应：Agent 应介绍其专注于条件语句复杂度审查的能力。

---

## 任务二：使用 @code-reviewer 分析代码

### 1. 进入项目目录并启动 OpenCode

```bash
cd ECommercePricingEngine
opencode
```

### 2. 调用 @code-reviewer 审查代码

```
@code-reviewer @ECommercePricingDemo.cs
请审查 CalculateFinalPrice 方法的条件语句复杂度，给出重构优先级列表
```

### 3. 理解审查报告

审查报告应包含：

- **问题清单**：每个问题的位置、描述、严重程度
- **嵌套层级分析**：指出最深的嵌套路径
- **重构机会**：可提取的方法、可简化的条件
- **可读性评分**：1-10 分
- **优先级列表**：高/中/低优先级的重构任务

### 4. 与原版 Ask 模式对比

| 对比项 | Ask 模式 | @code-reviewer Agent |
|--------|----------|---------------------|
| 输入成本 | 每次重复审查标准 | 一次配置，永久复用 |
| 输出格式 | 不固定 | 标准化模板 |
| 分析深度 | 通用分析 | 专注条件复杂度 |
| 效率 | 低 | 高 |

---

## 任务三：Build 模式执行重构

### 1. 切换到 Build 模式

确保 OpenCode 处于 Build 模式（允许修改代码）。

### 2. 基于审查报告执行重构

使用以下 prompt 执行主要重构：

```
@ECommercePricingDemo.cs
基于刚才 @code-reviewer 的审查报告，重构 CalculateFinalPrice 方法：
1. 提取 ApplyMembershipDiscounts 方法，内部用 switch expression 按会员级别分发
2. 为每个会员级别创建独立方法（ApplyPremiumDiscounts/ApplyGoldDiscounts/ApplySilverDiscounts/ApplyFirstTimeBuyerDiscounts）
3. 提取 ApplyCouponDiscounts 方法
4. 提取 ApplyBulkDiscounts 方法
5. 在每个 helper 方法中使用 guard clause 减少嵌套
6. 保持所有业务逻辑和输出格式不变
完成后运行 dotnet run 验证输出一致
```

### 3. 重构后预期结构

```
CalculateFinalPrice（主方法，约 30 行）
├── ApplyMembershipDiscounts
│   ├── ApplyPremiumDiscounts
│   ├── ApplyGoldDiscounts
│   ├── ApplySilverDiscounts
│   └── ApplyFirstTimeBuyerDiscounts
├── ApplyCouponDiscounts
├── ApplyBulkDiscounts
└── ApplyCategorySpecificDiscounts（已有）
```

### 4. 逐步验证

每完成一次重构，运行：

```bash
!dotnet run
```

确保输出与重构前一致。

---

## 任务四：@code-reviewer 验证重构质量

### 1. 重构后再次审查

```
@code-reviewer @ECommercePricingDemo.cs
验证重构后的代码质量：
1. 嵌套层级是否已降低到 3 层以内？
2. CalculateFinalPrice 是否已简化为方法调用链？
3. 给出重构前后的可读性评分对比
```

### 2. 对比审查报告

| 指标 | 重构前 | 重构后 |
|------|--------|--------|
| 嵌套层级 | ? 层 | ≤3 层 |
| 方法长度 | >50 行 | <30 行 |
| 可读性评分 | ?/10 | ?/10 |
| 高优先级问题 | ? 个 | 0 个 |

### 3. 确认问题解决

确保所有标记为"高"严重程度的问题已解决。

---

## 任务五：输出对比验证

### 1. 保存重构后输出

```bash
dotnet run > ACTUAL_OUTPUT_AFTER.txt
```

### 2. 对比前后输出

```bash
diff ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt
```

### 3. 预期结果

**无差异** → 业务逻辑完全保留，重构成功。

---

## （可选）任务六：LoanApprovalWorkflow

### 1. 进入第二个项目

```bash
cd ../LoanApprovalWorkflow
```

### 2. 复用 @code-reviewer Agent

由于 Agent 是项目级的，需要在当前项目重新创建 `.opencode/agent/code-reviewer.md`（或配置为全局 Agent）。

### 3. 执行相同流程

- 运行 `dotnet run > ACTUAL_OUTPUT_BEFORE.txt`
- 调用 `@code-reviewer` 分析
- 执行重构
- 调用 `@code-reviewer` 验证
- 运行 `diff` 对比输出

### 4. 体会 Agent 复用价值

相同的审查标准，无需重新输入，直接调用即可。

---

## 实验总结

### 自定义 Agent vs 每次手动分析

| 维度 | 手动分析 | 自定义 Agent |
|------|----------|--------------|
| 配置时间 | 每次 2-3 分钟 | 一次性 5 分钟 |
| 输出一致性 | 低 | 高 |
| 分析深度 | 不稳定 | 稳定深入 |
| 学习成本 | 每次重新描述 | 一次配置，团队共享 |

### 何时使用项目级 vs 全局 Agent

| 场景 | 推荐类型 | 示例 |
|------|----------|------|
| 项目特定规范 | 项目级 | 本 Lab 的代码审查标准 |
| 个人编码偏好 | 全局 | 个人代码风格检查 |
| 团队统一规范 | 项目级（纳入版本控制） | 团队 Code Review 标准 |
| 通用工具 | 全局 | 单元测试生成器 |

### 重构前后对比

| 指标 | 重构前 | 重构后 | 改善 |
|------|--------|--------|------|
| CalculateFinalPrice 行数 | ~80 行 | ~30 行 | -62% |
| 最大嵌套深度 | 5-6 层 | 2-3 层 | -50% |
| 条件语句数量 | ~15 个 | ~5 个 | -67% |
| 可读性评分 | ?/10 | ?/10 | +?分 |

---

## 附录：快速参考

### 常用命令

```bash
# 创建 Agent 目录
mkdir -p .opencode/agent

# 运行程序
dotnet run

# 保存输出
dotnet run > OUTPUT.txt

# 对比输出
diff BEFORE.txt AFTER.txt
```

### 常用 Prompt

```
# 调用 @code-reviewer 审查
@code-reviewer @文件名.cs
请审查 方法名 方法的条件语句复杂度，给出重构优先级列表

# Build 模式重构
@文件名.cs
基于审查报告，重构 方法名 方法：
1. 提取 Xxx 方法
2. 使用 switch expression 替代 if/else 链
3. 使用 guard clause 减少嵌套
4. 保持业务逻辑不变

# @code-reviewer 验证
@code-reviewer @文件名.cs
验证重构后的代码质量，给出前后评分对比
```

### Agent 配置文件模板

```markdown
---
description: 条件语句复杂度审查专家
mode: subagent
---

# Code Reviewer Agent

你是一位专注于代码可读性和条件语句复杂度的代码审查专家。

## 审查重点
- 嵌套层级是否超过 3 层？
- 是否有可以用 guard clause 替代的嵌套？
- 是否有可以用 switch expression 替代的 if/else 链？

## 输出格式
- 位置：方法名：行号
- 问题：具体描述
- 严重程度：高/中/低
- 建议：具体重构建议

最后给出可读性评分（1-10 分）和重构优先级列表。
```

---

## 关键要点回顾

1. **自定义 Agent 价值**：一次配置，永久复用，输出标准化
2. **Agent 配置位置**：项目级 `.opencode/agent/` vs 全局 `~/.config/opencode/agent/`
3. **mode: subagent**：只读分析模式，不直接修改代码
4. **调用名规则**：文件名（不含 .md）即调用名，`code-reviewer.md` → `@code-reviewer`
5. **重构验证**：必须通过 `diff` 确保业务逻辑不变
6. **重构目标**：嵌套≤3 层、方法<50 行、职责单一

---

*实验手册版本：1.0 | 基于 LAB_AK_09 改造 | OpenCode Labs*
