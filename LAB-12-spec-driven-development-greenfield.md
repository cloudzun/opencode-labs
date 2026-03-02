# LAB-12：规格驱动开发（绿地项目）

> **版本**：OpenCode 原生实现 · 零依赖  
> **预计时间**：60-70 分钟  
> **前置条件**：.NET 8.0+、OpenCode、已完成 LAB-11

---

## 一、实验概述

### 学习目标

完成本实验后，你将掌握：

1. **SDD 方法论**：理解规格驱动开发（Specification-Driven Development）的核心价值
2. **四层文档体系**：掌握 constitution→spec→plan→tasks 的递进关系
3. **@spec-writer Agent**：学会创建和使用自定义 Agent 生成结构化文档
4. **文档驱动开发**：从"自然语言需求"到"可执行任务"的完整转化流程

### 核心问题

| 传统开发 | 规格驱动开发 |
|---------|-------------|
| 拿到需求直接写代码 | 先写规格，再写代码 |
| 边做边想，容易返工 | 设计先行，减少变更 |
| 需求在脑子里 | 需求在文档里 |
| 新人难上手 | 文档即设计，新人友好 |

### SDD 四层文档体系

```
stakeholder 文档（PM/业务方提供的需求）
    ↓ 分析提炼
constitution.md  ← "我们的原则是什么"（项目治理）
    ↓ 约束指导
spec.md          ← "我们要做什么"（功能规格）
    ↓ 设计转化
plan.md          ← "我们怎么做"（技术方案）
    ↓ 任务分解
tasks.md         ← "具体做哪些步骤"（可执行任务）
    ↓ 逐步实现
代码实现          ← Build 模式按 tasks.md 执行
```

**教学价值**：
- **constitution.md**：团队共识，避免后期争论
- **spec.md**：需求边界，防止范围蔓延
- **plan.md**：技术设计，统一实现思路
- **tasks.md**：执行清单，进度可追踪

### 与 GitHub Spec Kit 的对比

| GitHub Copilot 版 | OpenCode 版 |
|------------------|-------------|
| 依赖 `specify` CLI | 零依赖，原生实现 |
| 需要 GitHub 账号 | 无需账号 |
| VSCode 插件 | OpenCode 通用 |
| **@spec-writer Agent** | 自定义 Agent 实现同等能力 |

---

## 二、环境准备

### 步骤 1：创建项目目录

```bash
mkdir RSSFeedReader
cd RSSFeedReader
```

### 步骤 2：准备 stakeholder 文档目录

```bash
mkdir StakeholderDocuments
mkdir specs
mkdir .opencode/agent
```

### 步骤 3：初始化 .NET 项目结构（暂不创建具体项目）

```bash
mkdir backend
mkdir frontend
```

### 步骤 4：验证 .NET 版本

```bash
dotnet --version
# 应输出：8.x.x
```

---

## 三、任务一：准备 stakeholder 文档

> **说明**：stakeholder 文档模拟真实项目中 PM/业务方提供的需求文档。在真实场景中，这些文档由非技术人员编写；本实验中，我们直接使用附录中的完整内容。

### 3.1 创建 ProjectGoals.md

将**附录 A**的内容复制到 `StakeholderDocuments/ProjectGoals.md`

### 3.2 创建 AppFeatures.md

将**附录 B**的内容复制到 `StakeholderDocuments/AppFeatures.md`

### 3.3 创建 TechStack.md

将**附录 C**的内容复制到 `StakeholderDocuments/TechStack.md`

### 3.4 审查要点

✅ **合格标准**：
- 三个文件都位于 `StakeholderDocuments/` 目录下
- 文件内容完整，无乱码
- ProjectGoals.md 包含项目愿景和成功标准
- AppFeatures.md 包含功能列表和用户场景
- TechStack.md 包含技术选型和约束

❌ **常见问题**：
- 文件路径错误（应放在 StakeholderDocuments/ 内）
- 内容被截断（确保完整复制）
- 编码问题（使用 UTF-8）

---

## 四、任务二：创建 @spec-writer Agent

> **说明**：@spec-writer 是本实验的核心自定义 Agent，负责将 stakeholder 文档转化为 SDD 四层文档体系。

### 4.1 创建 Agent 定义文件

创建文件 `.opencode/agent/spec-writer.md`，内容如下：

```markdown
---
description: 规格驱动开发文档专家
mode: subagent
---

# Spec Writer Agent

你是一位专注于规格驱动开发（SDD）的文档专家。

## 核心职责
将 stakeholder 需求文档转化为结构化的 SDD 文档体系：
constitution.md → spec.md → plan.md → tasks.md

## constitution.md 生成规则
包含以下部分：
- 项目名称和描述
- 核心原则（5 条，具体可执行，非泛泛而谈）
- 技术标准（语言、框架、架构约束）
- 质量要求（测试、安全、性能）
- 治理规则（变更管理、文档要求）

## spec.md 生成规则
包含以下部分：
- 项目概述
- 用户场景与测试（Given-When-Then 格式）
- 功能需求（MUST/SHOULD/MAY 分级）
- 成功标准（可量化）
- 范围外（明确排除项）

## plan.md 生成规则
包含以下部分：
- 架构概述（后端/前端职责划分）
- 技术选型（含选型理由）
- 数据模型
- API 设计（端点、请求/响应格式）
- 实现策略（MVP 优先）

## tasks.md 生成规则
按阶段组织：
- Phase 1: Setup（项目初始化）
- Phase 2: Foundation（基础设施）
- Phase 3: US1（第一个用户故事）
- ...
每个任务：编号（T001 起）、描述、验收标准
最后包含 Implementation Strategy（MVP First 策略）

## 输出原则
- 具体可执行，不写泛泛而谈的原则
- 每个需求都有对应的验收标准
- tasks.md 中的任务粒度：单个任务 1-4 小时完成
```

### 4.2 重启 OpenCode 使 Agent 生效

保存文件后，重启 OpenCode 或重新加载工作区。

### 4.3 验证 Agent 是否生效

在 OpenCode 中输入：

```
@spec-writer 你好，请介绍你的能力
```

**预期输出**：@spec-writer 应自我介绍其 SDD 文档生成能力。

### 4.4 审查要点

✅ **合格标准**：
- `.opencode/agent/spec-writer.md` 文件存在
- 文件包含完整的 frontmatter（`---` 包裹的元数据）
- `mode: subagent` 设置正确
- 核心职责和生成规则描述清晰

❌ **常见问题**：
- frontmatter 格式错误（缺少 `---`）
- mode 设置错误（应为 subagent）
- 文件路径错误（必须在 `.opencode/agent/` 下）

---

## 五、任务三：生成 constitution.md（项目治理原则）

> **说明**：constitution.md 是 SDD 文档体系的顶层文档，定义项目的治理原则和技术标准。

### 5.1 调用 @spec-writer 生成文档

在 OpenCode 中输入以下 prompt：

```
@spec-writer
请分析以下 stakeholder 文档，生成项目治理原则文件：
@StakeholderDocuments/ProjectGoals.md
@StakeholderDocuments/AppFeatures.md
@StakeholderDocuments/TechStack.md

要求：
- 提取 5 条具体可执行的核心原则（不要泛泛而谈）
- 包含技术标准、质量要求、治理规则
- 保存到 specs/constitution.md
```

### 5.2 审查 constitution.md

生成完成后，阅读 `specs/constitution.md`，检查以下内容：

**✅ 合格标准**：

| 检查项 | 合格示例 | 不合格示例 |
|-------|---------|-----------|
| 核心原则数量 | 恰好 5 条 | 少于 5 条或泛泛而谈 |
| 原则具体性 | "使用 C# 12.0+ 特性" | "使用现代 C#" |
| 技术标准 | ".NET 8.0 LTS" | "使用.NET" |
| 质量要求 | "单元测试覆盖率≥80%" | "保证代码质量" |
| 治理规则 | "变更需更新 tasks.md" | "做好文档管理" |

**❌ 常见问题**：
- 原则过于抽象，无法执行
- 缺少技术标准的具体版本号
- 质量要求不可量化
- 治理规则模糊

### 5.3 如需重新生成

如果审查不通过，可以要求 @spec-writer 重新生成：

```
@spec-writer
constitution.md 的原则不够具体，请重新生成。
要求：每条原则必须是可执行、可验证的具体规则，不要写"保证质量"这类空话。
```

---

## 六、任务四：生成 spec.md（功能规格说明）

> **说明**：spec.md 定义项目的功能需求和验收标准，是需求的正式契约。

### 6.1 调用 @spec-writer 生成文档

在 OpenCode 中输入以下 prompt：

```
@spec-writer
基于以下文档生成功能规格说明：
@specs/constitution.md
@StakeholderDocuments/ProjectGoals.md
@StakeholderDocuments/AppFeatures.md

要求：
- 用户场景使用 Given-When-Then 格式
- 需求使用 MUST/SHOULD/MAY 分级
- 明确列出 MVP 范围和范围外内容
- 保存到 specs/spec.md
```

### 6.2 审查 spec.md

生成完成后，阅读 `specs/spec.md`，检查以下内容：

**✅ 合格标准**：

| 检查项 | 合格示例 | 不合格示例 |
|-------|---------|-----------|
| 用户场景格式 | Given 无订阅，When 加载页面，Then 显示空状态 | "用户可以查看订阅列表" |
| 需求分级 | "MUST：添加订阅功能" | 所有需求混在一起 |
| MVP 范围 | 明确列出"仅内存存储" | 范围模糊 |
| 范围外 | 明确排除"用户认证、持久化存储" | 未说明范围外内容 |
| 成功标准 | "添加订阅响应时间<2 秒" | "性能良好" |

**❌ 常见问题**：
- 用户场景未使用 Given-When-Then 格式
- 需求没有 MUST/SHOULD/MAY 分级
- MVP 范围不清晰，包含过多功能
- 缺少明确的范围外说明

### 6.3 关键：MVP 范围确认

spec.md 中的 MVP 范围必须与 TASK.md 一致：
- ✅ **包含**：添加订阅 URL、显示订阅列表、仅内存存储
- ❌ **排除**：用户认证、数据库持久化、订阅分类、RSS 内容解析

---

## 七、任务五：生成 plan.md（技术实现计划）

> **说明**：plan.md 将功能规格转化为技术设计，定义架构、API、数据模型等。

### 7.1 调用 @spec-writer 生成文档

在 OpenCode 中输入以下 prompt：

```
@spec-writer
基于规格说明和技术文档生成技术实现计划：
@specs/spec.md
@specs/constitution.md
@StakeholderDocuments/TechStack.md

要求：
- 明确后端/前端职责划分
- 定义 API 端点（路径、方法、请求/响应格式）
- 定义数据模型
- 端口配置：后端 5151，前端 5213
- 保存到 specs/plan.md
```

### 7.2 审查 plan.md

生成完成后，阅读 `specs/plan.md`，检查以下内容：

**✅ 合格标准**：

| 检查项 | 合格示例 | 不合格示例 |
|-------|---------|-----------|
| 架构概述 | 明确后端 API/前端 UI 职责 | "前后端分离"（太模糊） |
| API 端点 | `POST /api/subscriptions`，请求/响应格式完整 | 只写"添加订阅接口" |
| 数据模型 | `FeedSubscription { Id, Url, Title, AddedAt }` | 无数据模型定义 |
| 端口配置 | 后端 5151，前端 5213 | 端口未指定或不一致 |
| CORS 配置 | 明确允许的前端 origin | 未提及 CORS |

**❌ 常见问题**：
- API 设计缺少请求/响应格式
- 数据模型字段不完整
- 端口配置与 TASK.md 不一致
- 缺少 CORS 配置说明

### 7.3 关键：端口一致性

plan.md 中的端口配置必须与后续实现一致：
- 后端：`http://localhost:5151`
- 前端：`http://localhost:5213`
- CORS：后端必须允许 `http://localhost:5213`

---

## 八、任务六：生成 tasks.md（可执行任务清单）

> **说明**：tasks.md 是 SDD 文档体系的最终输出，将技术计划分解为可执行的任务。

### 8.1 调用 @spec-writer 生成文档

在 OpenCode 中输入以下 prompt：

```
@spec-writer
基于规格说明和技术计划生成可执行任务清单：
@specs/spec.md
@specs/plan.md
@specs/constitution.md

要求：
- 按阶段组织（Phase 1: Setup, Phase 2: Foundation, Phase 3: US1）
- 每个任务编号（T001 起），包含描述和验收标准
- 最后包含 Implementation Strategy（MVP First 策略）
- 保存到 specs/tasks.md
```

### 8.2 审查 tasks.md

生成完成后，阅读 `specs/tasks.md`，检查以下内容：

**✅ 合格标准**：

| 检查项 | 合格示例 | 不合格示例 |
|-------|---------|-----------|
| 阶段组织 | Phase 1/2/3 清晰划分 | 所有任务混在一起 |
| 任务编号 | T001, T002, T003... | 无编号或编号混乱 |
| 验收标准 | "编译通过，无警告" | "完成功能" |
| 任务粒度 | 单个任务 1-4 小时 | 单个任务需要一整天 |
| MVP 策略 | 明确指定哪些任务是 MVP 范围 | 未区分 MVP 和后续功能 |

**❌ 常见问题**：
- 任务未按阶段组织
- 缺少任务编号
- 验收标准不可验证
- 任务粒度过大（>8 小时）
- 缺少 MVP First 策略说明

### 8.3 关键：MVP First 策略

tasks.md 最后必须包含 Implementation Strategy，明确：
- **MVP 范围**：Phase 1 + Phase 2 + Phase 3（US1）
- **后续功能**：US2、US3 等（本实验不实现）
- **验收条件**：完成 MVP 范围即可演示

---

## 九、任务七：Build 模式实现 MVP

> **说明**：本阶段使用 OpenCode Build 模式，按 tasks.md 逐步实现 MVP 功能。

### 9.1 调用 OpenCode 实现 Phase 1 和 Phase 2

在 OpenCode 中输入以下 prompt：

```
请按照 specs/tasks.md 中的 MVP First 策略实现应用：
@specs/tasks.md
@specs/plan.md

实现 Phase 1（Setup）和 Phase 2（Foundation）：
1. 创建后端项目：dotnet new webapi -n RSSFeedReader.Api -o backend/RSSFeedReader.Api
2. 创建前端项目：dotnet new blazorwasm -n RSSFeedReader.UI -o frontend/RSSFeedReader.UI
3. 清理 Blazor 模板页面（删除 Home.razor、Counter.razor、Weather.razor）
4. 配置端口（后端 5151，前端 5213）
5. 配置 CORS

完成后运行 dotnet build 确认编译通过
```

### 9.2 关键：清理 Blazor 模板页面

Blazor 模板默认包含 Home、Counter、Weather 页面，必须删除以避免路由冲突：

```bash
# 验证清理结果
ls frontend/RSSFeedReader.UI/Pages/
# 应该只有：Subscriptions.razor（不应有 Home.razor、Counter.razor、Weather.razor）
```

### 9.3 关键：端口配置三处一致

检查以下配置：

| 位置 | 配置项 | 值 |
|-----|-------|-----|
| 后端 `launchSettings.json` | `applicationUrl` | `http://localhost:5151` |
| 前端 `wwwroot/appsettings.json` | `ApiBaseUrl` | `http://localhost:5151/api/` |
| 后端 `Program.cs` | CORS `WithOrigins` | `http://localhost:5213` |

### 9.4 审查要点

✅ **合格标准**：
- 后端项目编译通过：`dotnet build backend/RSSFeedReader.Api`
- 前端项目编译通过：`dotnet build frontend/RSSFeedReader.UI`
- Blazor 模板页面已清理
- 端口配置三处一致
- CORS 已配置

❌ **常见问题**：
- 编译失败（检查依赖和命名空间）
- 端口冲突（5151 或 5213 被占用）
- CORS 未配置（前端无法调用后端）
- 模板页面未清理（路由冲突）

---

## 十、任务八：验证 MVP

> **说明**：启动应用，使用 spec.md 中的 Given-When-Then 场景进行验收测试。

### 10.1 启动后端

```bash
dotnet run --project backend/RSSFeedReader.Api
```

**预期输出**：
```
Now listening on: http://localhost:5151
```

### 10.2 启动前端（新终端）

```bash
dotnet run --project frontend/RSSFeedReader.UI
```

**预期输出**：
```
Now listening on: http://localhost:5213
```

### 10.3 验收测试场景

打开浏览器访问 `http://localhost:5213`，执行以下测试：

| 场景 | Given | When | Then |
|-----|-------|------|------|
| 场景 1 | 无订阅 | 加载页面 | 显示空状态提示 |
| 场景 2 | 输入有效 URL（如 https://example.com/feed.xml） | 点击添加 | 订阅出现在列表 |
| 场景 3 | 添加成功 | 查看列表 | 输入框已清空 |
| 场景 4 | 输入为空 | 点击添加 | 阻止提交，显示提示 |

### 10.4 审查要点

✅ **合格标准**：
- 后端和前端都能正常启动
- 前端能成功调用后端 API
- 四个验收场景都能通过
- 无 CORS 错误（浏览器控制台无报错）

❌ **常见问题**：
- 前端无法连接后端（检查端口和 CORS）
- 添加订阅后列表不刷新（检查状态管理）
- 控制台有 CORS 错误（检查 WithOrigins 配置）

---

## 十一、实验总结

### SDD 文档体系的价值

| 文档 | 价值 | 类比 |
|-----|------|-----|
| constitution.md | 团队共识，避免后期争论 | 宪法 |
| spec.md | 需求边界，防止范围蔓延 | 合同 |
| plan.md | 技术设计，统一实现思路 | 蓝图 |
| tasks.md | 执行清单，进度可追踪 | 施工图 |

**核心理念**：文档即设计，设计先行于代码。

### @spec-writer 与其他 Agent 的区别

| Agent | 职责 | 使用场景 |
|-------|------|---------|
| **@spec-writer** | 生成 SDD 文档 | 绿地项目、新功能、大型重构 |
| @code-reviewer | 审查代码质量 | 代码提交前、PR 审查 |
| @performance-auditor | 性能分析和优化 | 性能瓶颈排查 |

**关键区别**：@spec-writer 在写代码之前工作，其他 Agent 在代码完成后工作。

### SDD 适用场景

✅ **适用**：
- 绿地项目（从零开始）
- 新功能开发（需求明确）
- 大型重构（需要设计文档）
- 团队协作（需要共识）

❌ **不适用**：
- 紧急 hotfix（时间不允许）
- 探索性原型（需求不明确）
- 个人小工具（文档成本过高）

---

## 十二、关键 Prompt 汇总

### 生成 constitution.md

```
@spec-writer
请分析以下 stakeholder 文档，生成项目治理原则文件：
@StakeholderDocuments/ProjectGoals.md
@StakeholderDocuments/AppFeatures.md
@StakeholderDocuments/TechStack.md

要求：
- 提取 5 条具体可执行的核心原则（不要泛泛而谈）
- 包含技术标准、质量要求、治理规则
- 保存到 specs/constitution.md
```

### 生成 spec.md

```
@spec-writer
基于以下文档生成功能规格说明：
@specs/constitution.md
@StakeholderDocuments/ProjectGoals.md
@StakeholderDocuments/AppFeatures.md

要求：
- 用户场景使用 Given-When-Then 格式
- 需求使用 MUST/SHOULD/MAY 分级
- 明确列出 MVP 范围和范围外内容
- 保存到 specs/spec.md
```

### 生成 plan.md

```
@spec-writer
基于规格说明和技术文档生成技术实现计划：
@specs/spec.md
@specs/constitution.md
@StakeholderDocuments/TechStack.md

要求：
- 明确后端/前端职责划分
- 定义 API 端点（路径、方法、请求/响应格式）
- 定义数据模型
- 端口配置：后端 5151，前端 5213
- 保存到 specs/plan.md
```

### 生成 tasks.md

```
@spec-writer
基于规格说明和技术计划生成可执行任务清单：
@specs/spec.md
@specs/plan.md
@specs/constitution.md

要求：
- 按阶段组织（Phase 1: Setup, Phase 2: Foundation, Phase 3: US1）
- 每个任务编号（T001 起），包含描述和验收标准
- 最后包含 Implementation Strategy（MVP First 策略）
- 保存到 specs/tasks.md
```

### Build 模式实现

```
请按照 specs/tasks.md 中的 MVP First 策略实现应用：
@specs/tasks.md
@specs/plan.md

实现 Phase 1（Setup）和 Phase 2（Foundation）：
1. 创建后端项目
2. 创建前端项目
3. 清理 Blazor 模板页面
4. 配置端口
5. 配置 CORS

完成后运行 dotnet build 确认编译通过
```

---

## 十三、附录：stakeholder 文档内容

### 附录 A：StakeholderDocuments/ProjectGoals.md

```markdown
# 项目目标：RSS Feed Reader

## 项目愿景

创建一个简洁、高效的 RSS/Atom 订阅管理器，帮助用户集中追踪多个博客、新闻网站和技术资讯源。

## 目标用户

- 技术从业者：追踪技术博客、官方文档更新
- 新闻爱好者：关注多个新闻源
- 研究人员：跟踪学术期刊、行业动态

## 成功标准

1. **用户体验**：添加订阅到显示内容的操作不超过 3 步
2. **性能**：订阅列表加载时间 < 2 秒
3. **可靠性**：MVP 阶段无崩溃、无数据丢失
4. **可扩展性**：架构支持后续添加用户认证、持久化存储

## MVP 范围

**包含**：
- 添加订阅（输入 URL）
- 显示订阅列表
- 内存存储（重启后数据丢失可接受）

**不包含**：
- 用户认证
- 数据库持久化
- 订阅分类/标签
- RSS 内容解析和阅读

## 项目约束

- 开发周期：2 周 MVP
- 团队规模：2-3 人
- 技术栈：.NET 8.0 生态
```

### 附录 B：StakeholderDocuments/AppFeatures.md

```markdown
# 应用功能：RSS Feed Reader

## 功能列表

### MVP 功能（必须实现）

#### F1：订阅管理

**F1.1 添加订阅**
- 用户输入 RSS/Atom Feed URL
- 系统验证 URL 格式
- 系统保存订阅信息
- 系统显示添加结果

**F1.2 查看订阅列表**
- 显示所有已添加的订阅
- 显示订阅标题（如能获取）
- 显示订阅 URL
- 显示添加时间

**F1.3 删除订阅**
- 用户可以从列表中移除订阅
- 删除后列表立即刷新

### 后续功能（MVP 后实现）

- F2：订阅分类/标签
- F3：RSS 内容解析和阅读
- F4：用户认证和多用户支持
- F5：数据库持久化

## 用户场景与验收测试

### 场景 1：首次使用

**Given** 用户首次访问应用，没有任何订阅  
**When** 用户加载主页  
**Then** 显示空状态提示，引导用户添加第一个订阅

### 场景 2：添加订阅

**Given** 用户有一个有效的 RSS Feed URL  
**When** 用户在输入框中输入 URL 并点击"添加"  
**Then** 订阅出现在列表中，输入框清空

### 场景 3：添加无效 URL

**Given** 用户输入了一个无效的 URL 格式  
**When** 用户点击"添加"  
**Then** 系统阻止提交，显示错误提示

### 场景 4：删除订阅

**Given** 列表中有至少一个订阅  
**When** 用户点击某个订阅的删除按钮  
**Then** 订阅从列表中移除，列表刷新

## 非功能需求

### 性能

- 订阅列表加载时间 < 2 秒
- 添加订阅响应时间 < 1 秒

### 可用性

- 支持主流浏览器（Chrome、Edge、Firefox）
- 响应式布局，支持桌面和移动端

### 可维护性

- 代码结构清晰，便于后续扩展
- 关键逻辑有单元测试覆盖
```

### 附录 C：StakeholderDocuments/TechStack.md

```markdown
# 技术栈：RSS Feed Reader

## 后端技术选型

### 核心框架

- **.NET 8.0 LTS**：长期支持版本，性能优异
- **ASP.NET Core Web API**：RESTful API 服务

### 架构模式

- **Controller-based API**：适合 CRUD 场景，易于理解
- **In-Memory Storage**：MVP 阶段使用 `List<T>` 存储，后续可替换为数据库
- **Dependency Injection**：内置 DI 容器

### 端口配置

- **开发环境**：5151
- **生产环境**：可配置（默认 80/443）

## 前端技术选型

### 核心框架

- **Blazor WebAssembly**：.NET 全栈，代码复用
- **Razor Components**：组件化开发

### 状态管理

- **组件级状态**：MVP 阶段使用组件内部状态
- **服务注入**：后续可引入 Fluxor 等状态管理库

### 端口配置

- **开发环境**：5213
- **生产环境**：静态文件托管（可部署到 CDN）

## 前后端通信

### API 调用

- **HttpClient**：Blazor 内置 HTTP 客户端
- **Base Address**：`http://localhost:5151/api/`（开发环境）
- **CORS**：后端配置允许前端 origin

### 数据格式

- **请求/响应**：JSON
- **序列化**：System.Text.Json（.NET 内置）

## 开发工具

### 必备工具

- **.NET SDK 8.0+**
- **代码编辑器**：VS Code / Rider / Visual Studio
- **API 测试**：curl / Postman / httpie

### 可选工具

- **Swagger UI**：API 文档和测试
- **Browser DevTools**：前端调试

## 技术约束

### 必须遵守

1. 使用 C# 12.0+ 特性（如 primary constructors、collection expressions）
2. 后端 API 遵循 RESTful 命名规范
3. 前端组件遵循 Blazor 最佳实践
4. 代码注释使用中文

### 明确排除

1. 不使用 Entity Framework（MVP 阶段无需 ORM）
2. 不使用 Authentication/Authorization（MVP 无用户系统）
3. 不使用第三方 UI 库（原生 Blazor 组件足够）
4. 不引入复杂状态管理（MVP 组件状态即可）

## 后续扩展预留

### 数据库集成

- 预留 Repository 接口，后续可实现 EF Core

### 用户认证

- 预留 Auth 服务接口，后续可实现 JWT

### RSS 解析

- 预留 Feed Parser 服务接口，后续可引入 System.ServiceModel.Syndication
```

---

## 十四、文件结构总览

完成本实验后，项目结构应如下：

```
RSSFeedReader/
├── StakeholderDocuments/
│   ├── ProjectGoals.md
│   ├── AppFeatures.md
│   └── TechStack.md
├── specs/
│   ├── constitution.md
│   ├── spec.md
│   ├── plan.md
│   └── tasks.md
├── .opencode/
│   └── agent/
│       └── spec-writer.md
├── backend/
│   └── RSSFeedReader.Api/
├── frontend/
│   └── RSSFeedReader.UI/
└── LAB-12-spec-driven-development-greenfield.md
```

---

**实验完成！🎉**

你已掌握规格驱动开发的完整流程。继续探索：
- 尝试实现 US2（删除订阅功能）
- 尝试添加持久化存储（SQLite）
- 尝试引入用户认证系统
