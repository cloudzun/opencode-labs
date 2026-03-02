# LAB-08：大型函数重构 - 子代理分析模式

## 1. 实验标题和概述

### 实验名称
大型函数重构：子代理分析 + 主代理执行模式

### 学习目标
- 理解大型函数重构的核心原则（单一职责、可测试性、可维护性）
- 掌握 OpenCode 的「子代理分析模式」（Plan 模式）
- 学会区分何时使用并行模式 vs 子代理分析模式
- 实践 C# Clean Architecture 中的服务层重构技巧

### 预计时间
**35-45 分钟**

### 前置条件
- .NET 8.0+ SDK 已安装
- OpenCode CLI 工具已配置
- 已完成 LAB-07（基础重构练习）
- 具备 C# 基础语法知识

---

## 2. 环境准备

### 2.1 验证开发环境

```bash
# 验证 .NET SDK 版本
dotnet --version

# 验证 OpenCode 可用
opencode --version
```

### 2.2 下载并解压实验项目

```bash
# 确认 GHCopilotEx8LabApps.zip 存在
ls -lh GHCopilotEx8LabApps.zip

# 解压（如未解压）
unzip -o GHCopilotEx8LabApps.zip
```

### 2.3 批量修改项目文件（.NET 9.0 → 8.0）

实验项目默认使用 .NET 9.0，需统一修改为 8.0：

```bash
# 查找所有 .csproj 文件并替换 net9.0 为 net8.0
find . -name '*.csproj' | xargs sed -i 's/net9.0/net8.0/g'

# 验证修改结果
grep -r "net8.0" --include="*.csproj" .
```

**命令说明**：
- `find . -name '*.csproj'`：递归查找所有 .csproj 文件
- `xargs sed -i 's/net9.0/net8.0/g'`：对每个文件执行就地替换

### 2.4 保存基准输出

```bash
# 进入项目目录
cd ECommerceOrderProcessing

# 运行原始版本，保存输出
dotnet run > ../ACTUAL_OUTPUT_BEFORE.txt 2>&1

# 返回上级目录
cd ..

# 查看基准输出（确认 Test 1 显示 FAILED - 这是已知问题）
cat ACTUAL_OUTPUT_BEFORE.txt
```

**重要说明**：
- Test 1 在原始代码中即为 FAILED（非重构引入的问题）
- 这是实验设计的一部分，用于验证重构不改变原有行为

---

## 3. 项目结构介绍

### 3.1 Clean Architecture 三层结构

```
ECommerceOrderProcessing/
├── src/
│   ├── ECommerce.ApplicationCore/    # 核心业务逻辑层
│   │   ├── Services/
│   │   │   └── OrderProcessor.cs     # ⭐ 本次重构目标
│   │   ├── Entities/                  # 业务实体
│   │   ├── Interfaces/                # 接口定义
│   │   └── Exceptions/                # 自定义异常
│   │
│   ├── ECommerce.Console/             # 表现层（CLI）
│   │   └── Program.cs                 # 入口程序
│   │
│   └── ECommerce.Infrastructure/      # 基础设施层
│       └── Services/                  # 外部服务实现
```

### 3.2 阅读 OrderProcessor.cs

打开文件 `ECommerceOrderProcessing/src/ECommerce.ApplicationCore/Services/OrderProcessor.cs`

**关键观察**：
- 原始 `ProcessOrder` 方法约 **246 行**
- 包含 7 个明显的逻辑职责块（见代码注释）
- 每个职责块都有独立的输入验证、处理逻辑、错误处理

### 3.3 ProcessOrder 的 7 个职责

| 序号 | 职责 | 原始代码行范围 | 核心操作 |
|------|------|---------------|---------|
| 1 | 输入验证与安全检测 | 54-101 | 验证订单数据、邮箱、地址、风险评分 |
| 2 | 库存管理 | 105-135 | 检查库存、预留商品 |
| 3 | 支付处理 | 137-162 | 安全验证、扣款、异常回滚 |
| 4 | 物流调度 | 164-185 | 安排配送、生成运单号 |
| 5 | 客户通知 | 187-208 | 发送邮件通知（失败不阻断） |
| 6 | 订单完成 | 210-222 | 更新状态、记录完成时间 |
| 7 | 异常处理 | 226-245 | 意外异常捕获、库存清理 |

---

## 4. 核心概念：子代理分析模式

### 4.1 为什么 LAB-08 不适合并行模式？

**并行模式适用场景**：
- 多个 **独立文件** 的重构
- 各文件之间 **无数据依赖**
- 可以安全地同时修改多个位置

**LAB-08 的特点**：
- 所有修改集中在 **同一个文件**（OrderProcessor.cs）
- 7 个方法之间存在 **严格的数据流依赖**（如支付失败需回滚库存）
- 方法提取顺序 **不能颠倒**（必须先有验证，才有后续处理）

**结论**：同一文件 + 数据依赖 → 不适合并行

### 4.2 「子代理分析 + 主代理执行」两阶段模式

```
┌─────────────────────────────────────────────────────────┐
│  阶段一：子代理分析（Plan 模式）                          │
│  - 专注理解代码结构                                      │
│  - 识别 7 个职责边界                                     │
│  - 输出精确重构计划（方法签名、数据流、回滚依赖）          │
│  - 生成 REFACTOR-PLAN.md                                │
└─────────────────────────────────────────────────────────┘
                          ↓
                  （人工审查计划）
                          ↓
┌─────────────────────────────────────────────────────────┐
│  阶段二：主代理执行（Build 模式）                         │
│  - 按照计划逐步提取方法                                  │
│  - 每步验证编译通过                                      │
│  - 每步验证运行结果一致                                  │
└─────────────────────────────────────────────────────────┘
```

### 4.3 两阶段模式的优势

| 维度 | 单代理（又分析又执行） | 两阶段模式 |
|------|---------------------|-----------|
| **分析深度** | 可能跳过细节 | 专注分析，输出完整计划 |
| **执行稳定性** | 边分析边改，易出错 | 按计划执行，减少出错率 |
| **可审查性** | 难以预审 | 计划可人工审查后再执行 |
| **回滚依赖** | 容易遗漏 | 计划中明确标注 |
| **适用场景** | 简单重构 | 复杂、有依赖的重构 |

### 4.4 模式选择标准

```
是否涉及同一文件？
    ├─ 否 → 可使用并行模式（多个 Agent 同时改不同文件）
    └─ 是 → 是否步骤间有数据依赖？
            ├─ 否 → 可用单代理顺序执行
            └─ 是 → 必须用子代理分析 + 主代理执行
```

---

## 5. 任务一：子代理深度分析（Plan 模式）

### 5.1 启动 OpenCode 并切换 Plan 模式

```bash
# 进入项目根目录
cd /home/chengzh/clawd/opencode-labs/lab08-refactor-functions

# 启动 OpenCode（Plan 模式）
opencode --mode plan
```

### 5.2 使用 @explore 分析 ProcessOrder

**Prompt 示例 1 - 初步分析**：
```
@explore 分析 ECommerceOrderProcessing/src/ECommerce.ApplicationCore/Services/OrderProcessor.cs 中的 ProcessOrder 方法

请识别：
1. 方法包含哪 7 个逻辑职责块
2. 每个职责块的起始和结束行号
3. 各职责块之间的数据依赖关系
4. 哪些步骤失败时需要回滚前序步骤的资源

输出格式：请按职责列出分析结果
```

**Prompt 示例 2 - 深入数据流**：
```
继续分析 ProcessOrder 方法的数据流：

1. 每个职责块的输入参数是什么
2. 每个职责块的输出（返回值）是什么
3. 哪些成员变量被每个职责块使用（如 _inventoryService, _paymentGateway）
4. 如果某个步骤失败，需要调用哪些方法进行回滚

特别关注：
- ProcessPayment 失败时如何释放库存
- ScheduleShipping 失败时是否需要回滚支付和库存
- HandleUnexpectedError 的清理逻辑
```

### 5.3 生成重构计划到 REFACTOR-PLAN.md

**Prompt 示例 3 - 生成计划**：
```
根据以上分析，生成完整的重构计划文件 REFACTOR-PLAN.md

要求包含：
1. 每个提取方法的完整签名（含 out 参数和元组返回值）
2. 每个方法的代码行范围
3. 每个方法的数据流（输入/输出表格）
4. 每个方法依赖的成员变量
5. 回滚依赖关系（哪些方法失败时需要调用前序步骤的回滚）
6. 整体数据流图（ASCII 流程图）
7. 重构优先级与依赖关系表

请确保计划足够详细，可以让另一个 Agent 直接按步骤执行而无需再次分析。
```

### 5.4 验证 REFACTOR-PLAN.md

检查生成的计划文件是否包含：
- ✅ 7 个方法的完整签名
- ✅ 精确的行号范围
- ✅ 数据流表格
- ✅ 回滚依赖说明（特别是 ProcessPayment 和 HandleUnexpectedError）
- ✅ ScheduleShipping 返回类型使用 `ShippingDetails?` 处理 null 安全

**关键检查点**：
```markdown
# 方法 3：ProcessPayment 的回滚依赖
支付失败 → 必须释放已预留的库存
调用：_inventoryService.ReleaseStock(order.Items)

# 方法 4：ScheduleShipping 的返回类型
private (bool Success, ShippingDetails? Details, string ErrorMessage) ScheduleShipping(Order order)
                                   ↑
                            使用 ? 处理 null 安全警告
```

---

## 6. 任务二：主代理创建 stub 方法（Build 模式）

### 6.1 切换 Build 模式

```bash
# 重启 OpenCode（Build 模式）
opencode --mode build
```

### 6.2 添加 7 个 stub 方法

**Prompt 示例**：
```
在 OrderProcessor.cs 中，ProcessOrder 方法结束后、类结束前，添加以下 7 个 stub 方法：

1. ValidateOrderAndSecurity
2. CheckAndReserveInventory
3. ProcessPayment
4. ScheduleShipping
5. SendNotifications
6. FinalizeOrder
7. HandleUnexpectedError

每个 stub 方法要求：
- 使用 REFACTOR-PLAN.md 中的完整签名
- 方法体只包含：return 默认值;（如 return (true, "");）
- 保持方法为 private
- 添加 XML 注释说明方法职责

注意：
- ScheduleShipping 的 ShippingDetails 参数使用 ? 标记可为 null
- ProcessPayment 返回三元组 (bool, string, string)
- HandleUnexpectedError 返回 OrderResult

添加后运行 dotnet build 验证编译通过。
```

### 6.3 7 个方法的签名说明

```csharp
// 方法 1：验证订单和安全
private (bool IsValid, string ErrorMessage) ValidateOrderAndSecurity(Order order)

// 方法 2：库存检查与预留
private (bool Success, string ErrorMessage) CheckAndReserveInventory(Order order)

// 方法 3：支付处理（含支付参考号）
private (bool Success, string PaymentReference, string ErrorMessage) ProcessPayment(Order order)

// 方法 4：物流调度（Details 可为 null）
private (bool Success, ShippingDetails? Details, string ErrorMessage) ScheduleShipping(Order order)

// 方法 5：发送通知（无返回值）
private void SendNotifications(Order order)

// 方法 6：完成订单（无返回值）
private void FinalizeOrder(Order order)

// 方法 7：处理意外异常
private OrderResult HandleUnexpectedError(Order order, Exception ex)
```

### 6.4 验证编译

```bash
cd ECommerceOrderProcessing
dotnet build
```

**预期输出**：
```
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

---

## 7. 任务三：逐步提取代码（Build 模式，7 步）

**重要原则**：每提取一个方法，立即运行 `dotnet run` 验证输出与基准一致。

### 步骤 1：提取 ValidateOrderAndSecurity

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 1 个职责块到 ValidateOrderAndSecurity 方法。

源代码位置：ProcessOrder 方法内的行 54-101（验证逻辑部分）
目标：将验证逻辑移动到 ValidateOrderAndSecurity stub 方法中

要求：
1. 将行 54-101 的验证代码移动到 ValidateOrderAndSecurity 方法体
2. ProcessOrder 方法中保留调用：var (isValid, errorMessage) = ValidateOrderAndSecurity(order);
3. 确保所有 _auditLogger 和 _securityValidator 的调用正确迁移
4. 验证逻辑包括：null 检查、空订单检查、邮箱验证、地址验证、支付信息验证、风险评分、金额验证

完成后运行 dotnet run 验证输出与 ACTUAL_OUTPUT_BEFORE.txt 一致。
```

**验证命令**：
```bash
dotnet run > /tmp/step1_output.txt 2>&1
diff -u ../ACTUAL_OUTPUT_BEFORE.txt /tmp/step1_output.txt
# 预期：仅时间戳差异
```

---

### 步骤 2：提取 CheckAndReserveInventory

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 2 个职责块到 CheckAndReserveInventory 方法。

源代码位置：ProcessOrder 方法内的库存管理部分（行 105-135 范围）
目标：将库存检查与预留逻辑移动到 CheckAndReserveInventory 方法中

要求：
1. 移动库存检查循环和 ReserveStock 调用
2. ProcessOrder 保留调用：var (inventorySuccess, inventoryError) = CheckAndReserveInventory(order);
3. 确保 _inventoryService.CheckStock, _inventoryService.ReserveStock 调用正确
4. 确保 _securityValidator.ValidateOrderItem 调用正确
5. 确保审计日志 LogInventoryIssue, LogInventoryReserved 正确迁移

注意：此方法无回滚逻辑，只负责预留。

完成后运行 dotnet run 验证。
```

---

### 步骤 3：提取 ProcessPayment（关键：回滚逻辑）

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 3 个职责块到 ProcessPayment 方法。

⚠️ 关键：此方法包含回滚逻辑，必须正确处理支付失败时的库存释放。

源代码位置：ProcessOrder 方法内的支付处理部分（行 137-162 范围）
目标：将支付处理逻辑移动到 ProcessPayment 方法中

要求：
1. 移动支付处理代码，包含 try-catch 块
2. ProcessOrder 保留调用：var (paymentSuccess, paymentReference, paymentError) = ProcessPayment(order);
3. 确保支付成功后将 paymentReference 写回 order.PaymentReference
4. 回滚逻辑必须保留：
   - 安全验证失败 → 调用 _inventoryService.ReleaseStock(order.Items)
   - PaymentException 异常 → 捕获后调用 _inventoryService.ReleaseStock(order.Items)
5. 审计日志 LogSecurityEvent, LogPaymentProcessed, LogPaymentFailure 正确迁移

重点检查：
- 两个失败分支都调用了 ReleaseStock 回滚库存
- 返回三元组 (false, "", errorMsg) 格式正确

完成后运行 dotnet run 验证。
```

**回滚逻辑示意图**：
```
ProcessPayment
    │
    ├─ 安全验证失败
    │     └─→ ReleaseStock → 返回 Failure
    │
    ├─ PaymentException
    │     └─→ ReleaseStock → 返回 Failure
    │
    └─ 成功
          └─→ 返回 (true, paymentReference, "")
```

---

### 步骤 4：提取 ScheduleShipping

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 4 个职责块到 ScheduleShipping 方法。

源代码位置：ProcessOrder 方法内的物流调度部分（行 164-185 范围）
目标：将物流调度逻辑移动到 ScheduleShipping 方法中

要求：
1. 移动物流调度代码，包含 try-catch 块
2. ProcessOrder 保留调用：var (shippingSuccess, shippingDetails, shippingError) = ScheduleShipping(order);
3. 确保成功后将 TrackingNumber 和 EstimatedDeliveryDate 写回 order 对象
4. ShippingDetails 返回类型使用 ? 标记可为 null（处理验证失败情况）
5. 审计日志 LogSecurityEvent, LogShippingScheduled, LogShippingFailure 正确迁移

注意：
- 当前实现失败时不回滚已完成的支付（这是已知设计，非 BUG）
- 返回类型必须是 (bool, ShippingDetails?, string)

完成后运行 dotnet run 验证。
```

---

### 步骤 5：提取 SendNotifications

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 5 个职责块到 SendNotifications 方法。

源代码位置：ProcessOrder 方法内的通知发送部分（行 187-208 范围）
目标：将通知发送逻辑移动到 SendNotifications 方法中

要求：
1. 移动通知发送代码，包含 try-catch 块
2. ProcessOrder 保留调用：SendNotifications(order);
3. 此方法是唯一一个「失败不阻断流程」的方法
4. 内部异常捕获后仅记录日志，不抛出，不影响订单状态
5. 高价值订单（TotalAmount > 1000）发送额外警报
6. 审计日志 LogNotificationSent, LogNotificationFailure 正确迁移
7. 需要 _securityValidator.MaskEmail 用于日志脱敏

重点检查：
- 外部 try-catch 包裹，异常被吞掉
- ProcessOrder 调用后无返回值检查（因为失败不影响流程）

完成后运行 dotnet run 验证。
```

---

### 步骤 6：提取 FinalizeOrder

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 6 个职责块到 FinalizeOrder 方法。

源代码位置：ProcessOrder 方法内的订单完成部分（行 210-222 范围）
目标：将订单状态更新逻辑移动到 FinalizeOrder 方法中

要求：
1. 移动状态更新代码
2. ProcessOrder 保留调用：FinalizeOrder(order);
3. 状态更新包括：
   - order.Status = OrderStatus.Completed
   - order.CompletionDate = DateTime.UtcNow
   - order.ProcessingDuration = DateTime.UtcNow - order.OrderDate
4. 审计日志 LogOrderCompleted 正确迁移
5. 无外部依赖调用，纯状态更新操作

完成后运行 dotnet run 验证。
```

---

### 步骤 7：提取 HandleUnexpectedError

**Prompt 示例**：
```
从 ProcessOrder 方法中提取第 7 个职责块到 HandleUnexpectedError 方法。

源代码位置：ProcessOrder 方法内的 catch 块部分（行 226-245 范围）
目标：将意外异常处理逻辑移动到 HandleUnexpectedError 方法中

要求：
1. 移动异常处理代码到 HandleUnexpectedError 方法
2. ProcessOrder 的 catch 块保留调用：return HandleUnexpectedError(order, ex);
3. 必须处理 order 为 null 的情况（使用 order?.Id ?? "UNKNOWN"）
4. 清理逻辑必须保留：
   if (order?.Id != null)
       try
           _inventoryService.ReleaseStock(order.Items)
       catch (Exception cleanupEx)
           _auditLogger.LogUnexpectedError(order.Id, $"Cleanup failed: {cleanupEx.Message}")
5. 返回统一的失败消息，不暴露内部异常细节
6. 审计日志 LogUnexpectedError 正确迁移

重点检查：
- order 为 null 时不崩溃
- 清理操作本身也可能失败，有嵌套 try-catch
- 返回 OrderResult.Failure("...")

完成后运行 dotnet run 验证最终结果。
```

---

## 8. 任务四：验证重构结果

### 8.1 运行最终测试

```bash
cd ECommerceOrderProcessing
dotnet run > ../ACTUAL_OUTPUT_AFTER.txt 2>&1
cd ..
```

### 8.2 diff 对比

```bash
# 对比重构前后的输出
diff -u ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt
```

**预期结果**：
- 仅时间戳差异（如 ProcessingDuration）
- 所有 Test 结果一致（包括 Test 1 的 FAILED）
- 无功能行为变化

**示例 diff 输出**：
```
@@ -15,7 +15,7 @@
 Order ORD-003 completed successfully in 0.05 seconds.
 Test 3 (High-value Order): PASSED

-Order ORD-004 completed successfully in 0.03 seconds.
+Order ORD-004 completed successfully in 0.04 seconds.
 Test 4 (Normal Order): PASSED
 ...
```

### 8.3 审查重构后的 ProcessOrder 方法

打开 `OrderProcessor.cs`，检查 ProcessOrder 方法应简化为：

```csharp
public OrderResult ProcessOrder(Order order)
{
    _auditLogger.LogOrderProcessingStarted(order.Id, order.CustomerEmail);

    try
    {
        var (isValid, errorMessage) = ValidateOrderAndSecurity(order);
        if (!isValid) return OrderResult.Failure(errorMessage);

        var (inventorySuccess, inventoryError) = CheckAndReserveInventory(order);
        if (!inventorySuccess) return OrderResult.Failure(inventoryError);

        var (paymentSuccess, paymentReference, paymentError) = ProcessPayment(order);
        if (!paymentSuccess) return OrderResult.Failure(paymentError);
        order.PaymentReference = paymentReference;

        var (shippingSuccess, shippingDetails, shippingError) = ScheduleShipping(order);
        if (!shippingSuccess) return OrderResult.Failure(shippingError);
        order.TrackingNumber = shippingDetails.TrackingNumber;
        order.EstimatedDeliveryDate = shippingDetails.EstimatedDelivery;

        SendNotifications(order);
        FinalizeOrder(order);

        return OrderResult.Success(order.Id, order.TrackingNumber ?? "");
    }
    catch (Exception ex)
    {
        return HandleUnexpectedError(order, ex);
    }
}
```

**验证要点**：
- ✅ 约 30 行（从 246 行减少到约 30 行）
- ✅ 只剩 7 个方法调用
- ✅ 无具体业务逻辑实现
- ✅ 流程控制清晰可见

### 8.4 统计重构效果

```bash
# 统计 ProcessOrder 方法行数（重构后）
grep -A 100 "public OrderResult ProcessOrder" ECommerceOrderProcessing/src/ECommerce.ApplicationCore/Services/OrderProcessor.cs | grep -n "^    }" | head -1
```

**对比表**：

| 指标 | 重构前 | 重构后 | 改进 |
|------|--------|--------|------|
| ProcessOrder 行数 | ~246 行 | ~30 行 | **-88%** |
| 最大方法圈复杂度 | 15+ | 2 | **降低 87%** |
| 可单独测试方法数 | 1 | 7 | **提升 7 倍** |
| 代码可读性 | 低 | 高 | **显著提升** |

---

## 9. 实验总结

### 9.1 重构前后对比

**ProcessOrder 方法变化**：
```
重构前：246 行巨型方法，包含 7 个职责
  ├── 难以测试（需构造完整场景）
  ├── 难以维护（修改一处可能影响全局）
  ├── 难以理解（需要跟踪大量局部变量）
  └── 难以复用（逻辑耦合在一起）

重构后：30 行流程控制方法 + 7 个单职责方法
  ├── 每个方法可单独测试
  ├── 修改局部逻辑不影响其他方法
  ├── 方法名即文档，职责清晰
  └── 可独立复用（如单独调用 ValidateOrderAndSecurity）
```

### 9.2 子代理分析模式的价值

**本次实验的核心收获**：

1. **分析深度**：子代理专注分析，输出了精确到行号的重构计划
2. **执行稳定性**：主代理按计划执行，避免了「边分析边改」的错误
3. **可审查性**：REFACTOR-PLAN.md 可在执行前人工审查
4. **回滚安全**：计划中明确标注了每个方法的回滚依赖

**如果没有子代理分析**：
- 可能遗漏 ProcessPayment 的回滚逻辑
- 可能搞错方法提取顺序
- 可能忘记 ScheduleShipping 的 null 安全处理

### 9.3 何时用并行 vs 子代理分析

**决策树**：

```
开始重构任务
    │
    ↓
是否修改多个文件？
    ├─ 否 → 进入单文件判断
    │         │
    │         ↓
    │     步骤间有数据依赖？
    │         ├─ 否 → 单代理顺序执行即可
    │         └─ 是 → ⭐ 使用子代理分析 + 主代理执行
    │
    └─ 是 → 文件间有依赖？
              ├─ 否 → ⭐ 使用并行模式（多个 Agent 同时处理）
              └─ 是 → 子代理分析依赖关系后顺序执行
```

**LAB-08 的分类**：
- 单文件（OrderProcessor.cs）✓
- 步骤间有数据依赖（支付依赖库存、物流依赖支付）✓
- **结论**：必须使用子代理分析模式

### 9.4 关键经验教训

1. **每步验证**：每提取一个方法后立即 `dotnet run`，不要等全部完成再验证
2. **回滚逻辑**：涉及资源占用的步骤（如库存预留）必须有对应的回滚方法
3. **null 安全**：C# 中使用 `?` 标记可为 null 的返回类型，避免运行时异常
4. **测试一致性**：重构不改变行为，diff 应仅显示时间戳等非功能差异
5. **计划先行**：复杂重构前生成详细计划，比直接动手更稳定

---

## 10. 附录：快速参考

### A. 常用命令速查

```bash
# 批量修改 .csproj 的 TFM
find . -name '*.csproj' | xargs sed -i 's/net9.0/net8.0/g'

# 保存基准输出
dotnet run > ACTUAL_OUTPUT_BEFORE.txt 2>&1

# 验证重构结果
dotnet run > ACTUAL_OUTPUT_AFTER.txt 2>&1
diff -u ACTUAL_OUTPUT_BEFORE.txt ACTUAL_OUTPUT_AFTER.txt

# 编译检查
dotnet build

# 统计方法行数
grep -c "^" ECommerceOrderProcessing/src/ECommerce.ApplicationCore/Services/OrderProcessor.cs
```

### B. 7 个方法签名速查表

| 方法 | 签名 | 返回值说明 |
|------|------|-----------|
| ValidateOrderAndSecurity | `(bool IsValid, string ErrorMessage)` | 验证是否通过 |
| CheckAndReserveInventory | `(bool Success, string ErrorMessage)` | 库存预留是否成功 |
| ProcessPayment | `(bool Success, string PaymentReference, string ErrorMessage)` | 支付结果 + 参考号 |
| ScheduleShipping | `(bool Success, ShippingDetails? Details, string ErrorMessage)` | 物流详情（可为 null） |
| SendNotifications | `void` | 无返回值（失败不阻断） |
| FinalizeOrder | `void` | 无返回值（状态更新） |
| HandleUnexpectedError | `OrderResult` | 统一的失败结果 |

### C. 回滚依赖关系速查

| 方法 | 失败时回滚 | 回滚操作 |
|------|-----------|---------|
| ValidateOrderAndSecurity | 无 | 无资源占用 |
| CheckAndReserveInventory | 无 | 尚未占用资源 |
| ProcessPayment | 库存 | `_inventoryService.ReleaseStock(order.Items)` |
| ScheduleShipping | 当前未实现 | 应考虑：退款 + 释放库存 |
| SendNotifications | 无 | 非关键路径 |
| FinalizeOrder | 无 | 最后一步 |
| HandleUnexpectedError | 库存 | `_inventoryService.ReleaseStock(order.Items)` |

### D. OpenCode 模式切换

```bash
# Plan 模式（分析）
opencode --mode plan

# Build 模式（执行）
opencode --mode build

# 查看当前模式
opencode --status
```

### E. 已知问题说明

**Test 1 FAILED**：
- 这是原始代码中的已知问题，非重构引入
- 用于验证重构不改变原有行为（包括原有 BUG）
- 无需修复，保持与基准输出一致即可

---

## 实验完成检查清单

- [ ] 环境准备完成（.NET 8.0、OpenCode 可用）
- [ ] .csproj 批量修改为 net8.0
- [ ] 基准输出 ACTUAL_OUTPUT_BEFORE.txt 已保存
- [ ] REFACTOR-PLAN.md 已生成（含 7 个方法签名、数据流、回滚依赖）
- [ ] 7 个 stub 方法已创建，编译通过
- [ ] 7 个方法按顺序提取完成
- [ ] 每步提取后运行 dotnet run 验证
- [ ] 最终输出 ACTUAL_OUTPUT_AFTER.txt 与基准对比一致
- [ ] ProcessOrder 方法简化为约 30 行
- [ ] 理解子代理分析模式与并行模式的选择标准

---

**实验完成时间**：预计 35-45 分钟

**下一步建议**：
- 尝试为 7 个提取的方法编写单元测试
- 考虑为 ScheduleShipping 添加回滚逻辑（退款 + 释放库存）
- 实践 LAB-09（并行模式重构多文件项目）
