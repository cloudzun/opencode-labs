# LAB-07：使用 OpenCode 多代理并行模式合并重复代码

## 一、实验概述

### 1.1 实验标题
**C# 代码重构：使用 OpenCode 多代理并行模式合并电商系统中的重复代码**

### 1.2 学习目标
- 掌握使用 OpenCode **Plan 模式**进行全局代码分析和依赖识别
- 理解**多代理并行模式**的原理和工程价值
- 学会识别代码依赖关系，制定串行/并行执行策略
- 熟练使用三种并行操作方式：多终端窗口、后台进程、单终端顺序
- 掌握使用 OpenCode **Build 模式**进行 C# 代码重构
- 学会使用 `diff` 命令验证重构前后输出一致性

### 1.3 多代理并行模式核心概念

**什么是多代理并行模式？**

多代理并行模式是指在同一项目的**不同独立文件**上，同时启动多个 OpenCode 代理实例执行重构任务。其核心原理是：

```
如果任务 A 修改文件 X，任务 B 修改文件 Y，且 X ≠ Y（无交集）
→ 任务 A 和任务 B 可以并行执行
→ 总耗时 ≈ max(任务 A 耗时，任务 B 耗时)
→ 而非 任务 A 耗时 + 任务 B 耗时
```

**工程价值：**
1. **时间效率提升 40-50%**：原版串行约 30 分钟 → 多代理并行约 15-18 分钟
2. **资源充分利用**：现代开发机器多核 CPU，单代理模式无法充分利用
3. **模块化思维**：强制分析文件依赖关系，培养良好架构意识
4. **容错性**：单个 Agent 失败不影响其他并行任务

### 1.4 预计时间
**35-45 分钟**（包含多代理并行阶段）

### 1.5 前置条件
- 已完成 LAB-01 至 LAB-06，熟悉 OpenCode 基本操作
- 具备 C# 基础语法知识（类、方法、命名空间）
- 了解基本的 Git 操作
- 操作系统：Linux/macOS/Windows（本手册以 Linux 为例）

---

## 二、环境准备

### 2.1 验证 OpenCode 安装

```bash
opencode --version
```

确保版本号 ≥ 0.1.0。如未安装，请参考 LAB-01 安装指南。

### 2.2 验证 .NET SDK

```bash
dotnet --version
```

需要 .NET 8.0 或更高版本。**注意**：项目原始配置为 .NET 9.0，但服务器可能只有 .NET 8.0，后续需要修改。

### 2.3 下载并解压项目代码

```bash
# 进入实验目录
cd GHCopilotEx7LabApps

# 下载项目（如已存在可跳过）
# 项目结构：ECommerceOrderAndReturn（电商订单与退货系统）
```

### 2.4 修改 .csproj 目标框架

**重要**：项目使用 .NET 9，但服务器只有 .NET 8，需要先修改配置：

```bash
cd ECommerceOrderAndReturn
sed -i 's/net9.0/net8.0/' ECommerceOrderAndReturn.csproj
```

验证修改：
```bash
grep "TargetFramework" ECommerceOrderAndReturn.csproj
# 应输出：<TargetFramework>net8.0</TargetFramework>
```

### 2.5 初始运行验证

```bash
# 运行重构前的应用
dotnet run > ACTUAL_OUTPUT_BEFORE.txt 2>&1

# 查看输出
cat ACTUAL_OUTPUT_BEFORE.txt
```

正常输出应包含订单处理、退货处理、邮件发送、审计日志、库存操作等信息。保存此输出用于后续对比。

---

## 三、任务一：分析重复代码（Plan 模式）

### 3.1 启动 OpenCode，切换 Plan 模式

```bash
cd ECommerceOrderAndReturn
opencode
```

进入 OpenCode 后，切换至 **Plan 模式**：

```
/plan
```

### 3.2 使用 @explore 扫描全局重复代码

在 Plan 模式下输入以下 Prompt：

```
@explore 分析整个 ECommerceOrderAndReturn 项目，找出所有重复代码：
1. OrderProcessor.cs 和 ReturnProcessor.cs 中重复的方法
2. EmailService.cs 中重复的 helper 方法
3. AuditService.cs 中重复的逻辑
4. InventoryService.cs 中重复的逻辑
5. 分析哪些重构任务可以并行执行（文件无依赖）
输出到 .opencode/plans/refactor-plan.md
```

**说明**：
- `@explore` 是 OpenCode 的全局扫描指令，会遍历整个项目结构
- 第 5 点是关键：识别文件依赖关系，为并行执行做准备

### 3.3 分析文件依赖关系，制定并行策略

等待分析完成后，查看生成的计划文件：

```bash
cat .opencode/plans/refactor-plan.md
```

**依赖分析结果示例**：

```
## 重构任务分解

### Phase 1：创建 ValidationService
- 修改文件：OrderProcessor.cs, ReturnProcessor.cs, Services/ValidationService.cs（新建）

### Phase 2：创建 ShippingCalculationService
- 修改文件：OrderProcessor.cs, ReturnProcessor.cs, Services/ShippingCalculationService.cs（新建）

### Phase 3：重构 EmailService
- 修改文件：Services/EmailService.cs

### Phase 4：重构 AuditService
- 修改文件：Services/AuditService.cs

### Phase 5：重构 InventoryService
- 修改文件：Services/InventoryService.cs

## 并行策略

⚠️ Phase 1 和 Phase 2 都修改 OrderProcessor.cs 和 ReturnProcessor.cs → 必须串行执行

✓ Phase 3、Phase 4、Phase 5 各自修改独立文件 → 可以并行执行

推荐执行顺序：
1. 串行阶段：Agent-A 执行 Phase 1 → Phase 2
2. 等待 Phase 2 完成
3. 并行阶段：同时启动 Agent-B（Phase 3）、Agent-C（Phase 4）、Agent-D（Phase 5）
```

### 3.4 保存重构计划

确保计划已保存到 `.opencode/plans/refactor-plan.md`，后续任务将参照此计划执行。

```bash
# 确认文件存在
ls -la .opencode/plans/
```

---

## 四、任务二：串行阶段——重构处理器层（Build 模式）

**说明**：由于 Phase 1 和 Phase 2 都修改 `OrderProcessor.cs` 和 `ReturnProcessor.cs`，必须**串行执行**，避免文件冲突。

### 4.1 Phase 1：创建 ValidationService

在 OpenCode 中切换至 **Build 模式**：

```
/build
```

输入以下 Prompt：

```
@Services/ @OrderProcessor.cs @ReturnProcessor.cs
创建 Services/ValidationService.cs，合并 OrderProcessor 和 ReturnProcessor 中重复的 Validate() 方法：
1. 新建 ValidationService 类，包含通用的 ValidateId(string id, string type) 方法
2. 保留所有安全检查、业务规则和日志逻辑
3. 更新 OrderProcessor 和 ReturnProcessor 使用新服务
4. 删除两个类中的重复私有方法
5. 运行 dotnet build 确认编译通过
```

等待执行完成后，验证编译：

```
!dotnet build
```

### 4.2 Phase 2：创建 ShippingCalculationService

仍在 Build 模式下，输入以下 Prompt：

```
@Services/ @OrderProcessor.cs @ReturnProcessor.cs
创建 Services/ShippingCalculationService.cs，合并两个 CalculateShipping() 方法：
1. 新建 ShippingCalculationService 类
2. 保留订单（含免运费阈值）和退货（含处理费）的不同业务规则
3. 更新两个处理器类使用新服务
4. 运行 dotnet build 确认编译通过
```

等待执行完成后，再次验证编译：

```
!dotnet build
```

### 4.3 运行测试验证

```
!dotnet run
```

检查输出是否与 `ACTUAL_OUTPUT_BEFORE.txt` 一致（业务逻辑不应改变）。

### 4.4 Git 提交

```bash
git add Services/ValidationService.cs Services/ShippingCalculationService.cs
git add OrderProcessor.cs ReturnProcessor.cs
git commit -m "重构：合并 ValidationService 和 ShippingCalculationService（串行阶段完成）"
```

---

## 五、任务三：并行阶段——同时重构三个 Service（多代理模式）

### 5.1 多代理并行原理

**为什么这三个任务可以并行？**

| 任务 | 修改文件 | 与其他任务的文件交集 |
|------|----------|---------------------|
| Phase 3：EmailService | `Services/EmailService.cs` | 无 |
| Phase 4：AuditService | `Services/AuditService.cs` | 无 |
| Phase 5：InventoryService | `Services/InventoryService.cs` | 无 |

**结论**：三个任务修改完全不同的文件，互不干扰，可以安全并行。

**时间对比**：
- 串行执行：Phase 3（5 分钟）+ Phase 4（5 分钟）+ Phase 5（5 分钟）= **15 分钟**
- 并行执行：max(5, 5, 5) = **5-7 分钟**（含启动开销）
- **节省时间：约 50-60%**

### 5.2 并行操作方法（三种方式任选其一）

根据你当前的工作环境，选择以下三种方式之一：

---

#### **方法一：多终端窗口并行（推荐，最直观）**

打开**三个独立的终端窗口**，分别执行：

**终端 1（EmailService 重构）**：
```bash
cd ECommerceOrderAndReturn
opencode run "重构 EmailService 类，消除重复的 helper 方法：创建统一的模板构建方法和邮件发送 helper，保持公共方法签名不变"
```

**终端 2（AuditService 重构）**：
```bash
cd ECommerceOrderAndReturn
opencode run "重构 AuditService 类，合并 LogOrderActivity 和 LogReturnActivity：创建通用的 CreateAuditEntry 和 ValidateAndStore helper 方法"
```

**终端 3（InventoryService 重构）**：
```bash
cd ECommerceOrderAndReturn
opencode run "重构 InventoryService 类，合并 ReserveOrderInventory 和 RestoreReturnInventory：创建通用的 ValidateInventory 和 UpdateInventoryLevel helper 方法"
```

**等待全部完成**：观察三个终端，当所有进程都显示任务完成提示后，继续下一步。

---

#### **方法二：后台进程并行（单终端，适合远程 SSH）**

在**单个终端**中执行以下命令：

```bash
cd ECommerceOrderAndReturn

# 同时启动三个后台 OpenCode 进程
opencode run "重构 EmailService 类，消除重复的 helper 方法：创建统一的模板构建方法和邮件发送 helper，保持公共方法签名不变" > email-refactor.log 2>&1 &
PID1=$!

opencode run "重构 AuditService 类，合并 LogOrderActivity 和 LogReturnActivity：创建通用的 CreateAuditEntry 和 ValidateAndStore helper 方法" > audit-refactor.log 2>&1 &
PID2=$!

opencode run "重构 InventoryService 类，合并 ReserveOrderInventory 和 RestoreReturnInventory：创建通用的 ValidateInventory 和 UpdateInventoryLevel helper 方法" > inventory-refactor.log 2>&1 &
PID3=$!

echo "已启动三个并行任务：$PID1, $PID2, $PID3"

# 等待全部完成
wait $PID1 $PID2 $PID3

echo "所有并行任务完成，查看日志："
echo "=== EmailService 重构日志 ==="
cat email-refactor.log
echo "=== AuditService 重构日志 ==="
cat audit-refactor.log
echo "=== InventoryService 重构日志 ==="
cat inventory-refactor.log
```

**说明**：
- `> xxx.log 2>&1`：将标准输出和错误输出都重定向到日志文件
- `&`：后台运行
- `wait $PID1 $PID2 $PID3`：等待所有指定进程完成

---

#### **方法三：单终端顺序执行（备用方案，资源受限时使用）**

如果系统资源有限（如 CPU 核心数少、内存紧张），可以选择**单终端顺序执行**：

```bash
cd ECommerceOrderAndReturn

# 任务 1：EmailService
opencode run "重构 EmailService 类，消除重复的 helper 方法：创建统一的模板构建方法和邮件发送 helper，保持公共方法签名不变"
echo "EmailService 重构完成"

# 任务 2：AuditService
opencode run "重构 AuditService 类，合并 LogOrderActivity 和 LogReturnActivity：创建通用的 CreateAuditEntry 和 ValidateAndStore helper 方法"
echo "AuditService 重构完成"

# 任务 3：InventoryService
opencode run "重构 InventoryService 类，合并 ReserveOrderInventory 和 RestoreReturnInventory：创建通用的 ValidateInventory 和 UpdateInventoryLevel helper 方法"
echo "InventoryService 重构完成"
```

**注意**：此方式虽然不能节省时间，但**代码质量与并行方式相同**，适合资源受限环境。

---

### 5.3 等待三个任务全部完成

根据选择的方法，等待所有重构任务完成。成功标志：
- 每个任务都显示 "任务完成" 或类似提示
- `dotnet build` 编译通过
- 没有报错或冲突

### 5.4 ⚠️ 并行重构警告与回滚策略

**警告：并行重构可能引入以下问题**

| 问题类型 | 示例 | 原因 |
|---------|------|------|
| CustomerId 硬编码错误 | `CustomerId = ""` | Agent 未正确识别参数来源 |
| 日志格式简化过度 | 缺少 `Transaction ID` | Agent 过度简化主题行 |
| 验证逻辑被破坏 | 始终返回 `false` | 参数传递错误导致验证失败 |

**如果发现问题，使用以下步骤回滚和修复：**

```bash
# 1. 检查编译是否失败
dotnet build

# 2. 检查输出是否不一致
diff ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt

# 3. 如果有问题，使用 /undo 回滚有问题的 Agent
# 在对应的 OpenCode 会话中输入：
/undo

# 4. 重新运行该 Agent 并指定更精确的 prompt
# 例如修复 AuditService：
opencode run "重构 AuditService 类，合并 LogOrderActivity 和 LogReturnActivity：
1. 创建通用的 CreateAuditEntry 方法，参数包括 customerId，必须从 order.CustomerId 或 returnRequest.CustomerId 传入
2. CustomerId 字段必须使用传入的 customerId 参数，不能硬编码空字符串
3. 创建 ValidateAndStore helper 方法
4. 保持公共方法签名不变
5. 运行 dotnet build 确认编译通过"
```

**精确 Prompt 编写技巧：**
- 明确指出参数来源：`从 order.CustomerId 或 returnRequest.CustomerId 传入`
- 明确禁止错误模式：`不能硬编码空字符串`
- 明确要求保留格式：`保留原有邮件主题格式，包含 Transaction ID`

### 5.5 合并结果，运行测试验证

```bash
# 验证编译
dotnet build

# 运行应用
dotnet run > ACTUAL_OUTPUT_AFTER.txt 2>&1
```

### 5.6 Git 提交

```bash
git add Services/EmailService.cs Services/AuditService.cs Services/InventoryService.cs
git commit -m "重构：并行完成 EmailService/AuditService/InventoryService（多代理模式）"
```

---

## 六、任务四：验证重构结果

### 6.1 对比重构前后输出

```bash
diff ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt
```

**输出差异分析指南：**

#### 预期差异（正常，可接受）

| 差异类型 | 重构前示例 | 重构后示例 | 说明 |
|---------|-----------|-----------|------|
| **时间戳** | `2026-03-02 00:49:11` | `2026-03-02 00:55:08` | 正常，执行时间不同 |
| **日志参数名简化** | `Reason: RESERVED` | `Operation: RESERVE` | 重构后格式简化，语义不变 |

#### 问题差异（需要检查修复）

| 差异类型 | 重构前行为 | 重构后行为 | 严重性 |
|---------|-----------|-----------|--------|
| **验证失败** | `[AUDIT] Audit entry validation passed` | `[AUDIT] Validation failed: Customer ID is required` | 🔴 高 |
| **日志缺失** | 显示 `Storing:` 和 `Details:` 日志 | 缺少这些日志 | 🔴 高 |
| **功能改变** | 邮件主题包含 `Transaction ID: ORD12345` | 主题缺少 Transaction ID | 🟡 中 |
| **合规性检查缺失** | 显示 `[COMPLIANCE]` 日志 | 因验证失败被跳过 | 🔴 高 |

**判断标准：**
- 差异 ≤ 10 行：通常是预期差异（时间戳、格式简化）
- 差异 > 10 行：需要仔细检查，可能引入了 bug
- 出现 "Validation failed"：说明重构破坏了验证逻辑
- 出现大量日志缺失：说明功能被破坏

### 6.2 使用 OpenCode 进行最终代码审查

启动 OpenCode，切换至 Plan 模式：

```
/plan
```

输入以下 Prompt：

```
@EXPECTED_OUTPUT.md @ACTUAL_OUTPUT_AFTER.txt
对比重构前后的输出，确认业务逻辑没有改变。
如果有差异，指出具体差异并分析原因。
```

### 6.3 生成重构总结报告

继续在 Plan模式下输入：

```
@Services/
生成重构总结报告，包括：
1. 创建了哪些新文件（ValidationService.cs, ShippingCalculationService.cs）
2. 修改了哪些文件（OrderProcessor.cs, ReturnProcessor.cs, EmailService.cs, AuditService.cs, InventoryService.cs）
3. 消除了哪些重复代码
4. 代码行数变化统计
5. 重构后的代码质量评估（可维护性、可读性）
```

---

## 七、实验总结

### 7.1 并行重构效率对比

| 执行方式 | 预计耗时 | 实际耗时 | 时间节省 |
|----------|----------|----------|----------|
| 完全串行（5 个 Phase 顺序执行） | 30 分钟 | - | - |
| 多代理并行（Phase 1-2 串行 + Phase 3-5 并行） | 15-18 分钟 | ___ | **40-50%** |
| 单终端顺序（方法三） | 25-30 分钟 | ___ | 15-20% |

**你的实际数据**：
- 串行阶段（Phase 1 + Phase 2）耗时：______ 分钟
- 并行阶段（Phase 3 + Phase 4 + Phase 5）耗时：______ 分钟
- 总耗时：______ 分钟
- 时间节省：______ %

### 7.2 OpenCode 多代理工作流回顾

```
┌─────────────────────────────────────────────────────────────┐
│                    多代理并行工作流                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐                                          │
│   │  Plan 模式  │  ← 全局分析，识别依赖关系                 │
│   └──────┬──────┘                                          │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                          │
│   │  依赖分析   │  ← 确定哪些任务可并行                     │
│   └──────┬──────┘                                          │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                          │
│   │  串行阶段   │  Agent-A: Phase 1 → Phase 2              │
│   │  (文件冲突) │  （修改 OrderProcessor + ReturnProcessor）│
│   └──────┬──────┘                                          │
│          │                                                  │
│          ▼ 等待完成                                         │
│   ┌─────────────┐                                          │
│   │  并行阶段   │  Agent-B: Phase 3 (EmailService)         │
│   │  (文件独立) │  Agent-C: Phase 4 (AuditService)         │
│   │             │  Agent-D: Phase 5 (InventoryService)     │
│   └──────┬──────┘                                          │
│          │                                                  │
│          ▼ 全部完成                                         │
│   ┌─────────────┐                                          │
│   │  验证合并   │  dotnet run + diff 对比                  │
│   └─────────────┘                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 7.3 关键收获

1. **依赖分析是并行的前提**：只有准确识别文件依赖，才能安全并行
2. **多终端 vs 后台进程**：根据工作环境选择合适方式
3. **容错性**：单个 Agent 失败不影响其他任务，只需重跑失败的 Agent
4. **验证不可少**：重构后必须用 `diff` 验证业务逻辑未改变

### 7.4 拓展思考

1. 如果项目有 10 个独立文件需要重构，理论上最多可以启动多少个并行 Agent？
2. 什么情况下**不应该**使用多代理并行？（提示：文件有循环依赖时）
3. 如何将多代理并行模式应用到其他编程语言（如 Python、TypeScript）？

---

## 八、附录：快速参考

### 8.1 OpenCode 模式切换

| 命令 | 功能 |
|------|------|
| `/plan` | 切换至 Plan 模式（分析、规划） |
| `/build` | 切换至 Build 模式（代码修改、重构） |
| `/ask` | 切换至 Ask 模式（问答、解释） |

### 8.2 常用 Prompt 模板

**全局分析**：
```
@explore 分析整个项目，找出所有重复代码，并指出哪些任务可以并行执行。
```

**创建新服务**：
```
@Services/ @文件 1.cs @文件 2.cs
创建 Services/新服务.cs，合并两个文件中的重复方法：
1. 新建服务类
2. 保留业务规则
3. 更新原文件使用新服务
4. 运行 dotnet build 确认编译通过
```

**重构现有服务**：
```
@Services/服务名.cs
重构服务类，消除重复的 helper 方法：
1. 创建统一的 helper 方法
2. 保持公共方法签名不变
3. 运行 dotnet build 确认编译通过
```

### 8.3 并行操作命令速查

**多终端并行**：
```bash
# 三个终端分别执行
opencode run "任务描述..."
```

**后台进程并行**：
```bash
opencode run "任务 1" > log1.log 2>&1 &
opencode run "任务 2" > log2.log 2>&1 &
opencode run "任务 3" > log3.log 2>&1 &
wait
```

**单终端顺序**：
```bash
opencode run "任务 1"
opencode run "任务 2"
opencode run "任务 3"
```

### 8.4 .NET 版本兼容问题

```bash
# 检查当前版本
dotnet --version

# 修改 .csproj 目标框架（net9.0 → net8.0）
sed -i 's/net9.0/net8.0/' ECommerceOrderAndReturn.csproj

# 验证修改
grep "TargetFramework" ECommerceOrderAndReturn.csproj
```

### 8.5 输出验证命令

```bash
# 运行前保存输出
dotnet run > ACTUAL_OUTPUT_BEFORE.txt 2>&1

# 运行后保存输出
dotnet run > ACTUAL_OUTPUT_AFTER.txt 2>&1

# 对比差异
diff ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt

# 如有差异，查看具体内容
diff -u ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt
```

---

## 九、常见问题排查

### 问题 1：AuditService 的 CustomerId 为空字符串

**症状**：
```
[AUDIT] Validation failed: Customer ID is required
```
缺少 `Audit entry validation passed`、`Storing:`、`Details:` 日志

**原因**：重构后的 `CreateAuditEntry` 方法中，`CustomerId` 字段被硬编码为空字符串：
```csharp
// ❌ 错误代码
CustomerId = "",
```

**修复方法**：

1. 使用 `/undo` 回滚有问题的修改
2. 重新运行 Agent，使用精确的 prompt：

```
opencode run "重构 AuditService 类，合并 LogOrderActivity 和 LogReturnActivity：
1. 创建通用的 CreateAuditEntry 方法，参数包括 customerId，必须从 order.CustomerId 或 returnRequest.CustomerId 传入
2. CustomerId 字段必须使用传入的 customerId 参数，不能硬编码空字符串
3. 创建 ValidateAndStore helper 方法
4. 保持公共方法签名不变
5. 运行 dotnet build 确认编译通过"
```

**修复后的正确代码**：
```csharp
// ✅ 正确代码
private static AuditEntry CreateAuditEntry(string type, string id, string activity, string customerId, string details)
{
    return new AuditEntry
    {
        CustomerId = customerId,  // 使用传入的参数，而非硬编码
        // ...
    };
}
```

---

### 问题 2：EmailService 邮件主题缺少 Transaction ID

**症状**：
重构前：`[E-Commerce] Order Confirmation - Transaction ID: ORD12345`
重构后：`[E-Commerce] Order Confirmation`

**原因**：Agent 在合并重复代码时过度简化了邮件主题行，丢失了重要信息。

**修复方法**：

1. 使用 `/undo` 回滚有问题的修改
2. 重新运行 Agent，使用精确的 prompt：

```
opencode run "重构 EmailService 类，消除重复的 helper 方法：
1. 创建统一的 BuildEmailTemplate helper 方法
2. 创建通用的 SendEmail helper 方法
3. 保留原有邮件主题格式，必须包含 Transaction ID：
   - Order Confirmation 主题：$\"[E-Commerce] Order Confirmation - Transaction ID: {order.OrderId}\"
   - Return Confirmation 主题：$\"[E-Commerce] Return Confirmation - Transaction ID: {returnRequest.ReturnId}\"
4. 保持公共方法签名不变
5. 运行 dotnet build 确认编译通过"
```

**修复后的正确代码**：
```csharp
// ✅ 正确代码
public static void SendOrderConfirmation(Order order)
{
    var emailContent = BuildEmailTemplate("order", order.CustomerId, order.OrderId, order);
    SendEmail(order.CustomerId, $"[E-Commerce] Order Confirmation - Transaction ID: {order.OrderId}", emailContent, "OrderConfirmation");
}

public static void SendReturnConfirmation(Return returnRequest)
{
    var emailContent = BuildEmailTemplate("return", returnRequest.CustomerId, returnRequest.ReturnId, returnRequest);
    SendEmail(returnRequest.CustomerId, $"[E-Commerce] Return Confirmation - Transaction ID: {returnRequest.ReturnId}", emailContent, "ReturnConfirmation");
}
```

---

### 问题 3：dotnet build 编译失败

**常见错误**：
- `The name 'order' does not exist in the current context`
- `The name 'returnRequest' does not exist in the current context`

**原因**：重构后的方法参数名与原代码不匹配。

**修复方法**：
1. 检查方法签名是否正确
2. 确保传入的参数名在作用域内可用
3. 如有必要，手动修改代码修正参数名

---

### 问题 4：diff 显示超过 10 行差异

**排查步骤**：

1. 详细查看差异内容：
```bash
diff -u ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt | head -50
```

2. 分类差异类型：
   - 时间戳差异：正常，忽略
   - 日志格式简化（Reason → Operation）：正常，可接受
   - 验证失败信息：需要修复
   - 日志缺失：需要修复

3. 如果问题差异 > 5 处，考虑回滚重构：
```bash
# 在 OpenCode 中使用 /undo 回滚
# 或 git 回滚
git checkout HEAD -- Services/
```

4. 重新运行重构，使用更精确的 prompt

---

## 实验完成检查清单

- [ ] 环境准备完成（OpenCode、.NET 8.0+）
- [ ] 项目下载并修改 .csproj（net9.0 → net8.0）
- [ ] 保存重构前输出 `ACTUAL_OUTPUT_BEFORE.txt`
- [ ] Plan 模式完成全局分析，生成 `refactor-plan.md`
- [ ] 串行阶段完成（ValidationService + ShippingCalculationService）
- [ ] 并行阶段完成（EmailService + AuditService + InventoryService）
- [ ] 保存重构后输出 `ACTUAL_OUTPUT_AFTER.txt`
- [ ] 使用 `diff` 验证输出一致性
- [ ] Git 提交所有更改
- [ ] 填写实验总结中的时间统计数据

---

**实验完成！** 🎉

你现在已掌握 OpenCode 多代理并行模式的核心技能，可以将其应用到其他代码重构场景中。
