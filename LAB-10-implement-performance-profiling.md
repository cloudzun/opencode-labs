# LAB-10：实现性能分析与优化

## 实验概述

**学习目标**
- 掌握使用 OpenCode 进行 .NET 性能瓶颈分析的方法
- 学会创建和使用自定义 `@performance-auditor` Agent
- 理解量化性能优化的完整流程：基准测量 → 分析瓶颈 → 优化代码 → 验证提升
- 体验"用数据证明快了"与"感觉快了"的本质区别

**预计时间**：40-50 分钟

**前置条件**
- 已安装 .NET 8.0+ SDK
- 已完成 LAB-09（了解 OpenCode 基础操作）
- 已下载并解压实验项目代码

---

## 一、环境准备

### 1.1 验证环境

```bash
# 验证 OpenCode 可用
opencode --version

# 验证 .NET SDK
dotnet --version
```

### 1.2 准备项目代码

1. 解压提供的 `GHCopilotEx10LabApps.zip` 文件
2. 进入项目目录 `ContosoOnlineStore`
3. 调整目标框架（如需要）：
   ```bash
   # 将 net9.0 改为 net8.0
   find . -name "*.csproj" -exec sed -i 's/net9.0/net8.0/g' {} \;
   ```

### 1.3 验证初始状态

```bash
# 运行单元测试，确保 16/16 通过
dotnet test
```

---

## 二、核心概念：量化性能优化

### 2.1 为什么要量化？

性能优化最忌讳"拍脑袋"：
- ❌ **感觉快了**：主观判断，无法复现，无法证明
- ✅ **数据证明快了**：有基准测量、有对比数据、有提升百分比

### 2.2 @performance-auditor Agent 的价值

| 对比项 | 传统方式 | @performance-auditor |
|--------|----------|---------------------|
| 分析标准 | 每次重复说明要求 | 一次创建，永久复用 |
| 分析深度 | 依赖临时 Prompt | 内置性能检查清单 |
| 输出格式 | 不统一 | 结构化风险矩阵 + 优先级 |

### 2.3 模拟延迟说明

项目代码中包含 `Thread.Sleep` / `Task.Delay`，这些是**模拟真实世界的外部依赖**（数据库响应、网络延迟）：
- **不应该被优化掉**：保留用于前后对比
- **真实的性能提升**来自：
  - 算法优化（O(n) → O(1)）
  - 并行化（串行 → 并发）
  - 缓存策略

---

## 三、任务一：建立基准（Baseline）

### 3.1 启动 OpenCode

```bash
cd ContosoOnlineStore
opencode
```

### 3.2 运行应用并保存基准数据

在 OpenCode 中执行：

```
!dotnet run > baseline_metrics.txt
```

### 3.3 记录关键指标

查看保存的基准文件：

```
!cat baseline_metrics.txt
```

典型输出示例：
```
=== 产品目录性能测试 ===
GetProductById: 6.30ms/次 (800 次共 5042ms)
SearchProducts: 12.45ms/次

=== 订单处理性能测试 ===
CalculateOrderTotal: 2430ms/次
FinalizeOrderAsync: 3150ms/次

=== 库存更新性能测试 ===
UpdateStockLevels: 890ms/次

=== 邮件发送性能测试 ===
SendOrderConfirmation: 1200ms/次

单元测试：16/16 通过
```

> 💡 **提示**：将关键数据记录到你的实验笔记中，后续用于对比。

---

## 四、任务二：创建 @performance-auditor Agent

### 4.1 创建 Agent 配置文件

在项目根目录创建文件 `.opencode/agent/performance-auditor.md`：

```markdown
---
description: 性能瓶颈分析专家
mode: subagent
---

# Performance Auditor Agent

你是一位专注于 .NET 性能优化的代码审查专家。

## 审查重点

### 算法复杂度
- [ ] 是否有 O(n) 可以优化为 O(1) 的查找？（线性搜索→字典）
- [ ] 是否有 N+1 查询模式？（循环内单条查询→批量查询）
- [ ] 是否有重复计算可以缓存？

### 异步/并发
- [ ] 是否有可以并行的独立操作？（Task.WhenAll）
- [ ] 是否有不必要的 await 串行化？
- [ ] 是否有阻塞操作（Thread.Sleep）可以改为 Task.Delay？

### 内存分配
- [ ] 是否有频繁的字符串拼接？（StringBuilder）
- [ ] 是否有可以复用的对象被重复创建？
- [ ] 是否有 LINQ 链式操作可以优化？

### 缓存策略
- [ ] 是否有热点数据没有缓存？
- [ ] 缓存 key 生成是否高效？
- [ ] 缓存失效策略是否合理？

## 输出格式

对于每个问题：
- **位置**：类名。方法名
- **问题类型**：算法/异步/内存/缓存
- **当前复杂度**：O(?) 或描述
- **优化方案**：具体建议（含代码示例）
- **预期提升**：定性描述（如"查找从 O(n)→O(1)"）

最后给出**性能风险矩阵**（影响度 × 修复难度）和**优化优先级列表**。
```

### 4.2 重启 OpenCode 使 Agent 生效

退出 OpenCode 后重新启动：
```
/exit
```

然后重新进入项目目录启动 OpenCode。

### 4.3 验证 Agent 可用

```
@performance-auditor 你好，请介绍你的分析能力
```

---

## 五、任务三：@performance-auditor 分析瓶颈

### 5.1 执行性能分析

在 OpenCode 中输入：

```
@performance-auditor @ProductCatalog.cs @OrderProcessor.cs @InventoryManager.cs @Services/EmailService.cs
请分析这四个文件的性能瓶颈，输出性能风险矩阵和优化优先级列表
```

### 5.2 解读分析报告

Agent 将输出类似以下的结构化分析：

#### 发现的问题清单

| 位置 | 问题类型 | 当前复杂度 | 优化方案 | 预期提升 |
|------|----------|------------|----------|----------|
| ProductCatalog.GetProductById | 算法 | O(n) | 使用 Dictionary | O(1) 查找 |
| OrderProcessor.CalculateOrderTotal | 算法 | N+1 查询 | 批量查询 | 减少 N-1 次调用 |
| OrderProcessor.FinalizeOrderAsync | 异步 | 串行执行 | Task.WhenAll | 并行执行 |
| InventoryManager.UpdateStockLevels | 异步 | 逐条更新 | 批量更新 | 减少数据库往返 |
| EmailService.SendEmails | 异步 | 串行发送 | 并行发送 | 时间≈单次最长 |

#### 性能风险矩阵

| 问题 | 影响度 | 修复难度 | 优先级 |
|------|--------|----------|--------|
| GetProductById 线性查找 | 高 | 低 | ⭐⭐⭐ |
| CalculateOrderTotal N+1 | 高 | 中 | ⭐⭐⭐ |
| FinalizeOrderAsync 串行 | 中 | 低 | ⭐⭐ |
| UpdateStockLevels 逐条 | 中 | 低 | ⭐⭐ |
| EmailService 串行发送 | 低 | 低 | ⭐ |

### 5.3 记录优化优先级

按照 Agent 给出的优先级列表，依次进行优化。

---

## 六、任务四：Build 模式执行优化

### 6.1 优化 ProductCatalog（最高优先级）

在 Build 模式下输入：

```
@ProductCatalog.cs
优化 GetProductById 和 SearchProducts 方法：
1. GetProductById：用 Dictionary<string, Product> 替代 List 线性查找，在构造函数中初始化字典
2. SearchProducts：优化缓存 key 生成（用 string.Join 替代字符串拼接），保留模拟延迟不变
3. 保持所有公共方法签名不变
完成后运行 dotnet build 确认编译通过
```

等待编译完成后，验证：
```
!dotnet build
```

### 6.2 优化 OrderProcessor

```
@OrderProcessor.cs @ProductCatalog.cs
优化 CalculateOrderTotal 和 FinalizeOrderAsync：
1. CalculateOrderTotal：批量获取所有产品（一次查询），避免循环内单条查询（N+1 模式）
2. FinalizeOrderAsync：用 Task.WhenAll 并行执行独立操作（库存更新、邮件发送）
3. 保留所有模拟延迟（Thread.Sleep/Task.Delay），用于前后对比
完成后运行 dotnet build 确认编译通过
```

### 6.3 优化 InventoryManager

```
@InventoryManager.cs
优化 UpdateStockLevels 方法：
1. 将逐条更新改为批量更新（一次数据库调用）
2. 保留模拟延迟不变
3. 保持公共方法签名不变
完成后运行 dotnet build 确认编译通过
```

### 6.4 优化 EmailService

```
@Services/EmailService.cs
优化邮件发送方法：
1. 将串行发送改为并行发送（Task.WhenAll）
2. 保留模拟延迟不变
3. 保持公共方法签名不变
完成后运行 dotnet build 确认编译通过
```

### 6.5 验证所有编译通过

```
!dotnet build
```

确保无错误、无警告。

---

## 七、任务五：量化验证

### 7.1 运行优化后应用

```
!dotnet run > optimized_metrics.txt
```

### 7.2 对比基准数据

```
!diff baseline_metrics.txt optimized_metrics.txt
```

查看关键指标变化：

| 指标 | 优化前 | 优化后 | 提升幅度 |
|------|--------|--------|----------|
| GetProductById (800 次) | ~5042ms | ~XXms | XX% |
| SearchProducts | ~XXms | ~XXms | XX% |
| CalculateOrderTotal | ~2430ms | ~XXms | XX% |
| FinalizeOrderAsync | ~3150ms | ~XXms | XX% |
| UpdateStockLevels | ~890ms | ~XXms | XX% |
| SendOrderConfirmation | ~1200ms | ~XXms | XX% |

> 💡 **注意**：包含模拟延迟的指标提升有限是正常的，真实代码优化效果需要排除延迟影响分析。

### 7.3 运行单元测试验证

```
!dotnet test
```

确保 **16/16 测试仍然通过**，优化没有破坏功能。

### 7.4 调用 @performance-auditor 分析提升

```
@performance-auditor @baseline_metrics.txt @optimized_metrics.txt
对比优化前后的性能数据：
1. 计算各指标的提升幅度（%）
2. 排除模拟延迟的影响，分析真实代码优化效果
3. 给出优化总结报告
```

---

## 八、实验总结

### 8.1 量化对比表

将你的实验结果填入下表：

| 优化项 | 优化前 | 优化后 | 提升幅度 | 优化手段 |
|--------|--------|--------|----------|----------|
| GetProductById | ms | ms | % | 字典替代线性查找 |
| SearchProducts | ms | ms | % | 缓存 key 优化 |
| CalculateOrderTotal | ms | ms | % | 批量查询消除 N+1 |
| FinalizeOrderAsync | ms | ms | % | Task.WhenAll 并行 |
| UpdateStockLevels | ms | ms | % | 批量更新 |
| SendOrderConfirmation | ms | ms | % | 并行发送 |

### 8.2 @performance-auditor Agent 的复用价值

本次实验创建的 Agent 可以复用于：
- 后续 .NET 项目的性能审查
- Code Review 流程中的性能检查环节
- 性能回归测试的自动化分析

与 `@code-reviewer` 的区别：
| Agent | 关注点 | 输出 |
|-------|--------|------|
| @code-reviewer | 代码可读性、规范性、安全性 | 通用代码质量报告 |
| @performance-auditor | 执行效率、资源消耗、扩展性 | 性能风险矩阵 + 量化提升 |

### 8.3 性能优化的工程原则

通过本实验，掌握完整性能优化流程：

```
量化基准 → 分析瓶颈 → 优化代码 → 验证提升
    ↓           ↓           ↓           ↓
dotnet run  @performance  Build 模式  diff 对比
            -auditor      执行优化    + 单元测试
```

**核心原则**：
1. **先测量，后优化**：没有基准就没有优化
2. **数据驱动决策**：用数字说话，不用感觉
3. **验证功能完整**：优化后必须跑测试
4. **保留对比证据**：baseline 和 optimized 文件存档

---

## 九、附录：快速参考

### 9.1 常用命令速查

```bash
# 保存基准数据
dotnet run > baseline_metrics.txt

# 保存优化后数据
dotnet run > optimized_metrics.txt

# 对比差异
diff baseline_metrics.txt optimized_metrics.txt

# 运行测试
dotnet test

# 编译验证
dotnet build
```

### 9.2 N+1 查询模式示例

```csharp
// ❌ 优化前（N+1：每个 item 单独查询）
foreach (var item in order.Items)
{
    var product = await _catalog.GetProductByIdAsync(item.ProductId);
    total += product.Price * item.Quantity;
}

// ✅ 优化后（批量查询）
var productIds = order.Items.Select(i => i.ProductId).ToList();
var products = await _catalog.GetProductsByIdsAsync(productIds);
var productMap = products.ToDictionary(p => p.Id);
total = order.Items.Sum(i => productMap[i.ProductId].Price * i.Quantity);
```

### 9.3 并行化示例

```csharp
// ❌ 优化前（串行执行）
await inventoryService.UpdateStockAsync(order);
await emailService.SendConfirmationAsync(order);
await notificationService.NotifyAsync(order);

// ✅ 优化后（并行执行）
await Task.WhenAll(
    inventoryService.UpdateStockAsync(order),
    emailService.SendConfirmationAsync(order),
    notificationService.NotifyAsync(order)
);
```

### 9.4 字典查找示例

```csharp
// ❌ 优化前（O(n) 线性查找）
public Product GetProductById(string id)
{
    return _products.FirstOrDefault(p => p.Id == id);
}

// ✅ 优化后（O(1) 字典查找）
private readonly Dictionary<string, Product> _productMap;

public ProductCatalog()
{
    _productMap = _products.ToDictionary(p => p.Id);
}

public Product GetProductById(string id)
{
    return _productMap.GetValueOrDefault(id);
}
```

---

**实验完成标志**
- [ ] baseline_metrics.txt 和 optimized_metrics.txt 已保存
- [ ] 所有 16 个单元测试通过
- [ ] 量化对比表已填写
- [ ] @performance-auditor Agent 文件已创建并可复用

🎉 恭喜完成 LAB-10 性能分析与优化实验！
