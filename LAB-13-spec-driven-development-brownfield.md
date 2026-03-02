# LAB-13：OpenCode SDD 进阶实验手册（Brownfield 项目）

## 1. 实验概述

### 1.1 实验标题
**规格驱动开发进阶：在现有项目中添加新功能（Brownfield SDD）**

### 1.2 学习目标
- 掌握 **Brownfield SDD** 与 **Greenfield SDD** 的核心区别
- 学会在添加新功能前**分析现有代码库**，提取架构约束
- 使用 **@spec-writer Agent（Brownfield 版）** 生成四层文档
- 实现新功能与现有代码的**无缝集成**

### 1.3 预计时间
**70-80 分钟**

### 1.4 前置条件
- 已完成 **LAB-12（Greenfield SDD）**
- 安装 .NET 8.0+ SDK
- 安装 OpenCode
- 熟悉 Blazor Server、EF Core 基础

### 1.5 Greenfield vs Brownfield
- **Greenfield**：从零开始，无历史包袱，自由设计架构
- **Brownfield**：现有项目，先分析代码库，理解约束后再集成新功能

---

## 2. 核心概念

### 2.1 Brownfield SDD 流程（文字版）
现有项目 → 分析代码库 → 提取约束 → 生成 constitution → 生成 spec → 生成 plan → 生成 tasks → 集成实现

### 2.2 @spec-writer Brownfield 版的 5 个额外职责
1. **分析现有代码库**：读取现有 Models/Services/Pages，理解架构约束
2. **从现有代码提取原则**：constitution 基于现有代码，而非从零创建
3. **新功能规格设计**：spec 只描述新增功能，不重复现有功能
4. **集成计划**：plan 明确新功能如何与现有 User/Project/Notification 集成
5. **任务分解**：tasks 包含数据库迁移、现有页面修改等步骤

### 2.3 Greenfield vs Brownfield 对比

| 维度 | Greenfield（LAB-12） | Brownfield（LAB-13） |
|------|---------------------|---------------------|
| 起点 | 无代码 | 现有代码库 |
| 额外步骤 | 无 | 分析现有代码库 |
| 约束来源 | stakeholder 需求 | 现有架构 + stakeholder 需求 |
| @spec-writer | 从零创建原则 | 从现有代码提取原则 |
| 设计重点 | 自由架构设计 | 约束驱动集成设计 |

---

## 3. 环境准备

### 3.1 Clone 项目
```bash
git clone <ContosoDashboard 仓库地址>
cd ContosoDashboard
```

### 3.2 SQLite 迁移（Linux 环境必须）

**原因**：原项目使用 SQL Server LocalDB，Linux 不支持，需改为 SQLite。迁移将在任务五 Phase 1 中通过 Prompt 完成，此处无需手动操作。

### 3.3 验证编译
```bash
dotnet build
```
预期：**7 个警告，0 错误**

### 3.4 创建 OpenCode Agent 目录
```bash
mkdir -p .opencode/agent
mkdir -p specs
```

---

## 4. 任务一：分析现有代码库

### 4.1 使用 OpenCode Explore 模式分析项目结构

```prompt
请分析 ContosoDashboard 项目的现有架构：
@ContosoDashboard/Models/
@ContosoDashboard/Services/
@ContosoDashboard/Data/ApplicationDbContext.cs

输出：
1. 现有数据模型（实体及关系）
2. 现有服务层架构
3. 认证系统（mock auth）
4. 新功能集成点（User、Project、Notification）
5. 技术约束（主键类型、ORM、框架版本）
```

### 4.2 审查要点
- [ ] 所有实体的主键类型是否一致（应为 int）？
- [ ] 现有服务层是否使用依赖注入？
- [ ] 认证系统如何实现（MockAuthStateProvider）？
- [ ] 新功能需要与哪些现有实体集成（User、Project、Notification）？
- [ ] 数据库 ORM 是什么（EF Core 8.0）？

---

## 5. 任务二：创建 @spec-writer Agent

### 5.1 创建 Agent 定义文件

创建 `.opencode/agent/spec-writer.md`：

```markdown
# @spec-writer Agent（Brownfield 版）

## 角色
规格驱动开发专家，专注于在现有项目中添加新功能。

## 核心能力
1. **分析现有代码库**：读取现有 Models/Services/Pages，理解架构约束
2. **从现有代码提取原则**：constitution 基于现有代码，而非从零创建
3. **新功能规格设计**：spec 只描述新增功能，不重复现有功能
4. **集成计划**：plan 明确新功能如何与现有代码集成
5. **任务分解**：tasks 包含数据库迁移、现有页面修改等步骤

## 工作流程
1. 分析现有代码 → 2. 生成 constitution → 3. 生成 spec → 4. 生成 plan → 5. 生成 tasks

## 输出规范
- constitution.md：从现有代码提取的架构原则
- spec.md：新功能规格（Given-When-Then 格式）
- plan.md：技术实现计划（含集成点）
- tasks.md：可执行任务清单（含验收标准）
```

### 5.2 重启 OpenCode 使 Agent 生效
```bash
# 重启 OpenCode 或重新加载窗口
```

---

## 6. 任务三：生成 constitution.md

### 6.1 Prompt

```prompt
@spec-writer
分析现有 ContosoDashboard 代码库，生成项目治理原则：
@ContosoDashboard/Models/User.cs
@ContosoDashboard/Models/Project.cs
@ContosoDashboard/Data/ApplicationDbContext.cs
@ContosoDashboard/Services/DashboardService.cs
@StakeholderDocs/document-upload-and-management-feature.md

要求：
- 从现有代码提取架构原则（不是凭空创建）
- 包含技术约束：整数主键、EF Core、Blazor Server、SQLite
- 包含安全原则：基于角色的访问控制（Employee/TeamLead/ProjectManager/Admin）
- 包含集成原则：新功能必须与现有 User/Project/Notification 无缝集成
- 保存到 specs/constitution.md
```

### 6.2 审查要点
- [ ] 原则是否反映了现有架构约束？
- [ ] 是否包含 EF Core、Blazor Server、SQLite 技术栈约束？
- [ ] 是否定义了角色权限（Employee/TeamLead/ProjectManager/Admin）？
- [ ] 是否明确新功能与现有 User/Project/Notification 的集成方式？

---

## 7. 任务四：生成 spec → plan → tasks（顺序生成）

### 7.1 生成 spec.md

```prompt
@spec-writer
基于 constitution.md 和 stakeholder 文档，生成文档管理功能规格：
@specs/constitution.md
@StakeholderDocs/document-upload-and-management-feature.md

要求：
- 只描述新增的文档管理功能（不重复现有功能）
- MVP 范围：文档上传 + 我的文档列表（US1）
- 用户场景使用 Given-When-Then 格式
- 需求使用 MUST/SHOULD/MAY 分级
- 明确范围外：版本历史、协作编辑、Azure 集成等
- 保存到 specs/spec.md
```

**审查要点**：
- [ ] 是否只描述新增功能（不重复现有仪表盘功能）？
- [ ] MVP 范围是否清晰（US1：上传 + 列表）？
- [ ] 用户场景是否使用 Given-When-Then 格式？
- [ ] 是否明确范围外功能？

### 7.2 生成 plan.md

```prompt
@spec-writer
基于规格说明和现有代码，生成技术实现计划：
@specs/spec.md
@specs/constitution.md
@ContosoDashboard/Data/ApplicationDbContext.cs
@ContosoDashboard/Models/User.cs
@ContosoDashboard/Models/Project.cs
@StakeholderDocs/document-upload-and-management-feature.md

要求：
- 定义 Document 数据模型（int DocumentId，Category 存文本，FileType 255 字符）
- 定义 IFileStorageService 接口（UploadAsync/DeleteAsync/DownloadAsync）
- 定义 LocalFileStorageService 实现（AppData/uploads/{userId}/{projectId}/{guid}.ext）
- 定义 DocumentService 方法（UploadAsync/GetUserDocumentsAsync/DownloadAsync）
- 定义 API 端点（文件下载需要 controller，不能直接静态访问）
- 说明与现有 User/Project/Notification 的集成方式
- 保存到 specs/plan.md
```

**审查要点**：
- [ ] Document 模型是否使用 `int DocumentId`（与 User/Project 一致）？
- [ ] `IFileStorageService` 接口是否定义完整？
- [ ] 文件存储路径是否包含 userId/projectId 分层？
- [ ] 是否说明与现有 User/Project/Notification 的集成方式？

### 7.3 生成 tasks.md

```prompt
@spec-writer
基于规格说明和技术计划，生成可执行任务清单：
@specs/spec.md
@specs/plan.md
@specs/constitution.md

要求：
- Phase 1: Setup（T001-T010：SQLite 迁移、包安装）
- Phase 2: Foundation（T011-T025：Document 模型、IFileStorageService、DocumentService）
- Phase 3: US1（T026-T045：Documents.razor 页面、NavMenu 更新）
- Implementation Strategy：MVP First（T001-T045）
- 每个任务包含验收标准
- 保存到 specs/tasks.md
```

**审查要点**：
- [ ] 是否包含 Phase 1 Setup（SQLite 迁移）？
- [ ] 是否包含 Phase 2 Foundation（模型 + 服务）？
- [ ] 是否包含 Phase 3 US1（页面实现）？
- [ ] 每个任务是否有验收标准？

---

## 8. 任务五：Build 模式实现 MVP

**注意：本节所有代码操作均通过 Prompt 让 OpenCode 完成，不要手写代码！**

### 8.1 Phase 1 Setup：SQLite 迁移 + 项目配置

```prompt
请完成 Phase 1 Setup：
1. 修改 .csproj：移除 Microsoft.EntityFrameworkCore.SqlServer，添加 Microsoft.EntityFrameworkCore.Sqlite 和 Microsoft.EntityFrameworkCore.Design（版本 8.0.0）
2. 运行 dotnet restore 确保依赖安装成功
```

```bash
dotnet build
```

### 8.2 Phase 2 Foundation：模型 + 服务层创建

**关键约束说明**（在 Prompt 中已包含，此处供学员理解背景）：
- 所有实体使用 int 主键，与现有 User/Project 保持一致
- Category 字段存储文本字符串，不用枚举（便于扩展分类）
- FileType 字段需要 255 字符，因为 Office 文档的 MIME 类型较长
- IFileStorageService 是文件存储的抽象接口，便于未来迁移到云存储
- 文件上传使用 MemoryStream 模式，详见附录 A

```prompt
请完成 Phase 2 Foundation：
1. 创建 Models/Document.cs 实体：int DocumentId 主键，FileName，Category（文本），FileType（255 字符），FileSize，FilePath，UploadedAt，UploadedByUserId（外键→User），ProjectId（可选外键→Project），导航属性 UploadedByUser 和 Project
2. 创建 Models/DocumentShare.cs 实体：int DocumentShareId 主键，DocumentId，SharedWithUserId，SharedAt，导航属性 Document 和 SharedWithUser
3. 更新 Data/ApplicationDbContext.cs：添加 DbSet<Document> 和 DbSet<DocumentShare>，在 OnModelCreating 中配置主键和索引
4. 创建 Services/IFileStorageService.cs 接口：UploadAsync(Stream, string, string, int, int?) 返回 Task<string>，DeleteAsync(string) 返回 Task，DownloadAsync(string) 返回 Task<Stream>
5. 创建 Services/LocalFileStorageService.cs 实现：文件存储在 AppData/uploads/{userId}/{projectId}/{guid}.{ext}，UploadAsync 创建目录并复制流，DeleteAsync 删除文件，DownloadAsync 返回文件流
6. 创建 Services/DocumentService.cs：注入 ApplicationDbContext 和 IFileStorageService，实现 UploadAsync（使用 MemoryStream 模式）/GetUserDocumentsAsync/DownloadAsync/DeleteAsync
7. 更新 Program.cs：注册 IFileStorageService 和 IDocumentService 服务
```

```bash
dotnet build
```

### 8.3 Phase 3 US1：Documents.razor 页面 + NavMenu 更新

```prompt
请完成 Phase 3 US1：
1. 创建 Pages/Documents.razor 页面：包含文档列表表格（文件名、分类、大小、上传时间、操作列）、上传表单（InputSelect 分类、InputFile 文件选择、上传按钮）、使用 MemoryStream 模式处理文件上传、注入 IDocumentService
2. 更新 Shared/NavMenu.razor：在导航菜单中添加"文档"导航项，链接到 /documents，使用 bi-file-earmark 图标
3. 确保所有异步操作使用 async/await
```

```bash
dotnet build
```

---

## 9. 任务六：验证 MVP

### 9.1 启动应用
```bash
dotnet run
```

### 9.2 验收场景（Given-When-Then）

**US1：文档上传**
- **Given** 用户已登录且有 Employee 或以上角色
- **When** 用户在 /documents 页面选择文件（≤25MB）并点击上传
- **Then** 文件出现在我的文档列表中

**US2：我的文档列表**
- **Given** 用户已登录
- **When** 用户访问 /documents 页面
- **Then** 显示用户上传的所有文档列表

**US3：文档下载**
- **Given** 文档列表中存在文档
- **When** 用户点击下载链接
- **Then** 文件成功下载

**US4：文档删除**
- **Given** 文档列表中存在文档
- **When** 用户点击删除按钮
- **Then** 文档从列表中消失，文件被删除

---

## 10. 实验总结 + 附录

### 10.1 核心收获
1. **Brownfield SDD 核心**：先分析现有代码库，理解约束后再设计新功能
2. **@spec-writer 进阶**：从现有代码提取原则，而非从零创建
3. **接口抽象价值**：IFileStorageService 使云迁移无需修改业务逻辑

### 10.2 附录 A：MemoryStream 模式原理

**为什么需要**：`IBrowserFile.OpenReadStream()` 返回的流在 using 块结束后会被释放，无法在后续操作中使用。

**解决方案**：先复制到 `MemoryStream`，再传递给服务层。

**代码示例**（5 行核心逻辑）：
```csharp
using var memoryStream = new MemoryStream();
using (var fileStream = file.OpenReadStream(maxAllowedSize: 25 * 1024 * 1024))
{
    await fileStream.CopyToAsync(memoryStream);
}
memoryStream.Position = 0; // 重置读取位置
```

### 10.3 附录 B：IFileStorageService 云迁移价值

**为什么用接口**：通过抽象 `IFileStorageService` 接口，未来迁移到云存储（如 Azure Blob Storage）时，只需更换实现类，无需修改业务逻辑。

**迁移示例**：
```csharp
// Program.cs 只需修改注册
services.AddScoped<IFileStorageService, AzureBlobStorageService>();
```

---

**实验手册版本**：2.0（重写版）  
**适用课程**：OpenCode SDD 进阶（LAB-13）  
**前置实验**：LAB-12（Greenfield SDD）
