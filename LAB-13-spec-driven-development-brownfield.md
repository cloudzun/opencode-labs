# LAB-13：OpenCode SDD 进阶实验手册（Brownfield 项目）

## 1. 实验标题和概述

### 1.1 实验标题
**规格驱动开发进阶：在现有项目中添加新功能（Brownfield SDD）**

### 1.2 学习目标
- 掌握 **Brownfield SDD** 与 **Greenfield SDD** 的核心区别
- 学会在添加新功能前**分析现有代码库**，提取架构约束
- 使用 **@spec-writer Agent（Brownfield 版）** 生成四层文档
- 实现新功能与现有代码的**无缝集成**
- 理解接口抽象（如 `IFileStorageService`）的云迁移价值

### 1.3 预计时间
**70-80 分钟**

### 1.4 前置条件
- 已完成 **LAB-12（Greenfield SDD）**
- 安装 .NET 8.0+ SDK
- 安装 OpenCode
- 熟悉 Blazor Server、EF Core 基础

### 1.5 项目背景
- **现有项目**：ContosoDashboard（员工仪表盘，Blazor Server + ASP.NET Core + EF Core）
- **新功能**：文档上传与管理（Document Upload and Management）
- **stakeholder 文档**：`StakeholderDocs/document-upload-and-management-feature.md`
- **现有模型**：User、Project、ProjectMember、TaskItem、Notification

---

## 2. 核心概念：Brownfield SDD 与 Greenfield 的区别

### 2.1 Greenfield SDD（LAB-12）
```
从零开始 → 四层文档 → 架构设计 → 实现
```
- 无历史包袱，可自由设计架构
- 文档驱动从零创建代码
- @spec-writer 从零提取原则

### 2.2 Brownfield SDD（LAB-13）
```
现有项目 → 分析代码库 → 提取约束 → 四层文档 → 集成实现
```
- **额外步骤**：先分析现有代码，理解架构约束
- 新功能必须与现有代码**无缝集成**
- @spec-writer 需具备**代码库分析能力**

### 2.3 核心区别对比表

| 维度 | Greenfield（LAB-12） | Brownfield（LAB-13） |
|------|---------------------|---------------------|
| 起点 | 无代码 | 现有代码库 |
| 额外步骤 | 无 | **分析现有代码库** |
| 约束来源 | stakeholder 需求 | 现有架构 + stakeholder 需求 |
| @spec-writer | 从零创建原则 | **从现有代码提取原则** |
| 设计重点 | 自由架构设计 | 约束驱动集成设计 |
| 风险 | 需求理解偏差 | 现有代码理解错误 |

### 2.4 @spec-writer Agent 进阶版（Brownfield 专用）

Brownfield 版 @spec-writer 与 LAB-12 的区别：

1. **分析现有代码库**：先读取现有 Models/Services/Pages，理解架构约束
2. **constitution 基于现有代码**：从现有代码提取原则，而非从零创建
3. **spec 关注新功能**：只描述新增的文档管理功能，不重复现有功能
4. **plan 考虑集成点**：明确新功能如何与现有 User/Project/Notification 集成
5. **tasks 包含迁移步骤**：数据库迁移、现有页面修改等

---

## 3. 环境准备

### 3.1 Clone 项目
```bash
git clone <ContosoDashboard 仓库地址>
cd ContosoDashboard
```

### 3.2 SQLite 迁移（Linux 环境必须）

**原因**：原项目使用 SQL Server LocalDB，Linux 不支持，需改为 SQLite。

#### 步骤 1：修改 .csproj
```xml
<!-- 移除 -->
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />

<!-- 添加 -->
<PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
```

#### 步骤 2：修改 Program.cs
```csharp
// 移除
options.UseSqlServer(connectionString)

// 添加
options.UseSqlite(connectionString)
```

#### 步骤 3：修改 appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=contoso.db"
  }
}
```

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

## 4. 任务一：分析现有代码库（Brownfield 特有步骤）

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

### 4.2 预期分析结果

#### 现有数据模型
| 实体 | 主键 | 关键字段 | 关系 |
|------|------|---------|------|
| User | `int UserId` | Name, Email, Role | 1:N Project |
| Project | `int ProjectId` | Name, Description | 1:N TaskItem |
| ProjectMember | `int ProjectMemberId` | UserId, ProjectId | N:M User-Project |
| TaskItem | `int TaskItemId` | ProjectId, AssignedToUserId | N:1 Project |
| Notification | `int NotificationId` | UserId, Message | N:1 User |

#### 技术约束
- **主键类型**：所有实体使用 `int` 主键
- **ORM**：EF Core 8.0
- **框架**：Blazor Server + ASP.NET Core 8.0
- **数据库**：SQLite（迁移后）
- **认证**：Mock 认证（`MockAuthStateProvider`）

#### 新功能集成点
- **User**：文档上传者（`UploadedByUserId` 外键）
- **Project**：文档可关联项目（`ProjectId` 可选外键）
- **Notification**：文档分享时发送通知

### 4.3 记录约束文档
在 `notes/existing-code-analysis.md` 记录：
```markdown
# 现有代码分析

## 架构约束
- 所有主键使用 int 类型
- EF Core + SQLite
- Blazor Server 模式
- Mock 认证系统

## 集成点
- Document.UploadedByUserId → User.UserId
- Document.ProjectId → Project.ProjectId (可选)
- DocumentShare 触发 Notification
```

---

## 5. 任务二：创建 @spec-writer Agent（Brownfield 版）

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

## 6. 任务三：生成 constitution.md（基于现有代码库）

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

### 6.3 预期 constitution.md 结构
```markdown
# 项目治理原则（Constitution）

## 1. 技术栈约束
- 后端：ASP.NET Core 8.0 + Blazor Server
- ORM：EF Core 8.0 + SQLite
- 主键：所有实体使用 int 类型

## 2. 架构原则
- 分层架构：Models / Services / Pages
- 依赖注入：所有服务通过 DI 注册
- 异步优先：所有 I/O 操作使用 async/await

## 3. 安全原则
- 基于角色的访问控制（RBAC）
- 文件上传大小限制：25MB
- 文件类型验证：白名单机制

## 4. 集成原则
- 新功能必须与现有 User/Project/Notification 无缝集成
- 文档上传者关联 User.UserId
- 文档可关联 Project.ProjectId（可选）
- 文档分享触发 Notification
```

---

## 7. 任务四：生成 spec.md（新功能规格）

### 7.1 Prompt
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

### 7.2 审查要点
- [ ] 是否只描述新增功能（不重复现有仪表盘功能）？
- [ ] MVP 范围是否清晰（US1：上传 + 列表）？
- [ ] 用户场景是否使用 Given-When-Then 格式？
- [ ] 需求是否使用 MUST/SHOULD/MAY 分级？
- [ ] 是否明确范围外功能？

### 7.3 预期 spec.md 结构
```markdown
# 文档管理功能规格（Specification）

## 1. 功能概述
在 ContosoDashboard 中添加文档上传与管理功能。

## 2. MVP 范围（Phase 1）
- US1：用户上传文档
- US2：查看我的文档列表

## 3. 用户场景（Given-When-Then）

### US1：文档上传
**Given** 用户已登录且有 Employee 或以上角色
**When** 用户选择文件（≤25MB）并点击上传
**Then** 文件保存到服务器，数据库记录文档信息

### US2：我的文档列表
**Given** 用户已登录
**When** 用户访问"文档"页面
**Then** 显示用户上传的所有文档列表

## 4. 需求分级

### MUST（必须有）
- 文件上传（≤25MB）
- 文件类型验证（白名单）
- 我的文档列表
- 文档下载

### SHOULD（应该有）
- 文档分类（Category）
- 关联项目（可选）

### MAY（可以有）
- 文档分享功能（Phase 2）
- 版本历史（Phase 3）

## 5. 范围外（Out of Scope）
- 版本历史
- 协作编辑
- Azure Blob Storage 集成
- 全文搜索
```

---

## 8. 任务五：生成 plan.md（集成技术计划）

### 8.1 Prompt
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

### 8.2 审查要点
- [ ] Document 模型是否使用 `int DocumentId`（与 User/Project 一致）？
- [ ] `IFileStorageService` 接口是否定义完整？
- [ ] 文件存储路径是否包含 userId/projectId 分层？
- [ ] 是否说明与现有 User/Project/Notification 的集成方式？
- [ ] 是否包含 SQLite 迁移策略？

### 8.3 预期 plan.md 结构
```markdown
# 技术实现计划（Plan）

## 1. 数据模型设计

### Document 实体
```csharp
public class Document
{
    public int DocumentId { get; set; }          // int，与 User/Project 一致
    public string Category { get; set; }          // 文本，不用枚举
    public string FileType { get; set; }          // 255 字符，容纳 Office MIME 类型
    public string FilePath { get; set; }          // 相对路径，GUID 文件名
    public DateTime UploadedAt { get; set; }
    public int UploadedByUserId { get; set; }     // 外键 → User
    public int? ProjectId { get; set; }           // 可选外键 → Project
    
    // 导航属性
    public User UploadedByUser { get; set; }
    public Project? Project { get; set; }
}
```

### DocumentShare 实体
```csharp
public class DocumentShare
{
    public int DocumentShareId { get; set; }
    public int DocumentId { get; set; }
    public int SharedWithUserId { get; set; }
    public DateTime SharedAt { get; set; }
    
    // 导航属性
    public Document Document { get; set; }
    public User SharedWithUser { get; set; }
}
```

## 2. 服务层设计

### IFileStorageService 接口
```csharp
public interface IFileStorageService
{
    Task<string> UploadAsync(Stream fileStream, string fileName, string contentType, int userId, int? projectId);
    Task DeleteAsync(string filePath);
    Task<Stream> DownloadAsync(string filePath);
}
```

### LocalFileStorageService 实现
- 文件路径：`AppData/uploads/{userId}/{projectId}/{guid}.{ext}`
- GUID 文件名防止冲突
- 按用户/项目分层便于管理

### DocumentService
```csharp
public interface IDocumentService
{
    Task<Document> UploadAsync(int userId, IBrowserFile file, string category, int? projectId);
    Task<List<Document>> GetUserDocumentsAsync(int userId);
    Task<Stream> DownloadAsync(int documentId);
    Task DeleteAsync(int documentId, int userId);
}
```

## 3. API 端点设计
```csharp
// 文件下载（不能直接静态访问，需要权限验证）
GET /api/files/{documentId}
```

## 4. 与现有代码集成

### User 集成
- `Document.UploadedByUserId` → `User.UserId`
- 上传时自动关联当前用户

### Project 集成
- `Document.ProjectId` → `Project.ProjectId`（可选）
- 上传时可选择关联项目

### Notification 集成
- 文档分享时创建 Notification 记录
- 复现现有 NotificationService

## 5. 数据库迁移策略
1. 修改 .csproj：SqlServer → Sqlite
2. 修改 Program.cs：UseSqlite
3. 修改 appsettings.json：SQLite 连接字符串
4. 添加新实体：Documents、DocumentShares
5. 运行迁移：`dotnet ef migrations add AddDocumentManagement`
```

---

## 9. 任务六：生成 tasks.md（含集成任务）

### 9.1 Prompt
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

### 9.2 审查要点
- [ ] 是否包含 Phase 1 Setup（SQLite 迁移）？
- [ ] 是否包含 Phase 2 Foundation（模型 + 服务）？
- [ ] 是否包含 Phase 3 US1（页面实现）？
- [ ] 每个任务是否有验收标准？
- [ ] 任务编号是否连续？

### 9.3 预期 tasks.md 结构
```markdown
# 任务清单（Tasks）

## Implementation Strategy：MVP First

### Phase 1: Setup（T001-T010）
| 任务 ID | 任务描述 | 验收标准 |
|--------|---------|---------|
| T001 | 修改 .csproj：移除 SqlServer，添加 Sqlite | dotnet restore 成功 |
| T002 | 修改 Program.cs：UseSqlite 替代 UseSqlServer | 编译通过 |
| T003 | 修改 appsettings.json：SQLite 连接字符串 | 配置正确 |
| T004 | 安装 EF Core Design 包 | dotnet restore 成功 |
| T005-T010 | （其他 Setup 任务） | ... |

### Phase 2: Foundation（T011-T025）
| 任务 ID | 任务描述 | 验收标准 |
|--------|---------|---------|
| T011 | 创建 Models/Document.cs | 属性与 plan.md 一致 |
| T012 | 创建 Models/DocumentShare.cs | 属性与 plan.md 一致 |
| T013 | 更新 ApplicationDbContext（添加 DbSet） | 编译通过 |
| T014 | 创建 Services/IFileStorageService.cs | 接口定义完整 |
| T015 | 创建 Services/LocalFileStorageService.cs | 实现 Upload/Delete/Download |
| T016 | 创建 Services/DocumentService.cs | 业务逻辑完整 |
| T017 | 更新 Program.cs：注册新服务 | DI 注册成功 |
| T018-T025 | （其他 Foundation 任务） | ... |

### Phase 3: US1（T026-T045）
| 任务 ID | 任务描述 | 验收标准 |
|--------|---------|---------|
| T026 | 创建 Pages/Documents.razor（上传 + 列表） | 页面可访问 |
| T027 | 使用 MemoryStream 模式处理文件上传 | 无流释放错误 |
| T028 | 更新 Shared/NavMenu.razor（添加"文档"导航） | 导航项显示 |
| T029 | 运行数据库迁移 | 迁移成功 |
| T030 | dotnet build 验证 | 0 错误 |
| T031-T045 | （其他 US1 任务） | ... |
```

---

## 10. 任务七：Build 模式实现 MVP

### 10.1 SQLite 迁移

#### 修改 ContosoDashboard.csproj
```xml
<!-- 移除 -->
<!-- <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" /> -->

<!-- 添加 -->
<PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
```

#### 修改 Program.cs
```csharp
// 移除
// options.UseSqlServer(connectionString);

// 添加
options.UseSqlite(connectionString);
```

#### 修改 appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=contoso.db"
  }
}
```

### 10.2 创建 Document 模型和 DocumentShare 模型

#### Models/Document.cs
```csharp
public class Document
{
    public int DocumentId { get; set; }
    public string FileName { get; set; }
    public string Category { get; set; }
    public string FileType { get; set; }
    public long FileSize { get; set; }
    public string FilePath { get; set; }
    public DateTime UploadedAt { get; set; }
    public int UploadedByUserId { get; set; }
    public int? ProjectId { get; set; }
    
    // 导航属性
    public User UploadedByUser { get; set; }
    public Project? Project { get; set; }
}
```

#### Models/DocumentShare.cs
```csharp
public class DocumentShare
{
    public int DocumentShareId { get; set; }
    public int DocumentId { get; set; }
    public int SharedWithUserId { get; set; }
    public DateTime SharedAt { get; set; }
    
    // 导航属性
    public Document Document { get; set; }
    public User SharedWithUser { get; set; }
}
```

### 10.3 更新 ApplicationDbContext

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    // 现有 DbSet
    public DbSet<User> Users { get; set; }
    public DbSet<Project> Projects { get; set; }
    // ...
    
    // 新增 DbSet
    public DbSet<Document> Documents { get; set; }
    public DbSet<DocumentShare> DocumentShares { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // 现有配置
        // ...
        
        // 新增配置
        modelBuilder.Entity<Document>(entity =>
        {
            entity.HasKey(e => e.DocumentId);
            entity.HasIndex(e => e.UploadedByUserId);
            entity.HasIndex(e => e.ProjectId);
        });
        
        modelBuilder.Entity<DocumentShare>(entity =>
        {
            entity.HasKey(e => e.DocumentShareId);
        });
    }
}
```

### 10.4 实现 IFileStorageService + LocalFileStorageService

#### Services/IFileStorageService.cs
```csharp
public interface IFileStorageService
{
    Task<string> UploadAsync(Stream fileStream, string fileName, string contentType, int userId, int? projectId);
    Task DeleteAsync(string filePath);
    Task<Stream> DownloadAsync(string filePath);
}
```

#### Services/LocalFileStorageService.cs
```csharp
public class LocalFileStorageService : IFileStorageService
{
    private readonly string _basePath;
    
    public LocalFileStorageService(IWebHostEnvironment env)
    {
        _basePath = Path.Combine(env.ContentRootPath, "AppData", "uploads");
        Directory.CreateDirectory(_basePath);
    }
    
    public async Task<string> UploadAsync(Stream fileStream, string fileName, string contentType, int userId, int? projectId)
    {
        var userDir = Path.Combine(_basePath, userId.ToString());
        var projectDir = projectId.HasValue ? Path.Combine(userDir, projectId.ToString()) : userDir;
        Directory.CreateDirectory(projectDir);
        
        var ext = Path.GetExtension(fileName);
        var newFileName = $"{Guid.NewGuid()}{ext}";
        var filePath = Path.Combine(projectDir, newFileName);
        
        using (var outputStream = File.Create(filePath))
        {
            await fileStream.CopyToAsync(outputStream);
        }
        
        // 返回相对路径
        var relativePath = Path.Combine("uploads", userId.ToString(), projectId?.ToString() ?? "", newFileName);
        return relativePath.Replace("\\", "/");
    }
    
    public Task DeleteAsync(string filePath)
    {
        var fullPath = Path.Combine(_basePath, filePath.Replace("/", "\\"));
        if (File.Exists(fullPath))
        {
            File.Delete(fullPath);
        }
        return Task.CompletedTask;
    }
    
    public Task<Stream> DownloadAsync(string filePath)
    {
        var fullPath = Path.Combine(_basePath, filePath.Replace("/", "\\"));
        return Task.FromResult<Stream>(File.OpenRead(fullPath));
    }
}
```

### 10.5 实现 DocumentService

#### Services/DocumentService.cs
```csharp
public class DocumentService : IDocumentService
{
    private readonly ApplicationDbContext _context;
    private readonly IFileStorageService _fileStorage;
    
    public DocumentService(ApplicationDbContext context, IFileStorageService fileStorage)
    {
        _context = context;
        _fileStorage = fileStorage;
    }
    
    public async Task<Document> UploadAsync(int userId, IBrowserFile file, string category, int? projectId)
    {
        // 使用 MemoryStream 模式防止流提前释放
        using var memoryStream = new MemoryStream();
        using (var fileStream = file.OpenReadStream(maxAllowedSize: 25 * 1024 * 1024))
        {
            await fileStream.CopyToAsync(memoryStream);
        }
        memoryStream.Position = 0;
        
        // 上传到文件存储
        var filePath = await _fileStorage.UploadAsync(memoryStream, file.Name, file.ContentType, userId, projectId);
        
        // 创建数据库记录
        var document = new Document
        {
            FileName = file.Name,
            Category = category,
            FileType = file.ContentType,
            FileSize = file.Size,
            FilePath = filePath,
            UploadedAt = DateTime.UtcNow,
            UploadedByUserId = userId,
            ProjectId = projectId
        };
        
        _context.Documents.Add(document);
        await _context.SaveChangesAsync();
        
        return document;
    }
    
    public async Task<List<Document>> GetUserDocumentsAsync(int userId)
    {
        return await _context.Documents
            .Include(d => d.Project)
            .Where(d => d.UploadedByUserId == userId)
            .OrderByDescending(d => d.UploadedAt)
            .ToListAsync();
    }
    
    public async Task<Stream> DownloadAsync(int documentId)
    {
        var document = await _context.Documents.FindAsync(documentId);
        if (document == null) throw new NotFoundException();
        return await _fileStorage.DownloadAsync(document.FilePath);
    }
    
    public async Task DeleteAsync(int documentId, int userId)
    {
        var document = await _context.Documents.FindAsync(documentId);
        if (document == null || document.UploadedByUserId != userId)
            throw new NotFoundException();
        
        await _fileStorage.DeleteAsync(document.FilePath);
        _context.Documents.Remove(document);
        await _context.SaveChangesAsync();
    }
}
```

### 10.6 创建 Pages/Documents.razor

```razor
@page "/documents"
@using Microsoft.AspNetCore.Components.Forms
@inject IDocumentService DocumentService
@inject NavigationManager Navigation

<PageTitle>文档管理</PageTitle>

<h3>我的文档</h3>

@if (_documents == null)
{
    <p><em>加载中...</em></p>
}
else if (!_documents.Any())
{
    <p>暂无文档，请上传。</p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>文件名</th>
                <th>分类</th>
                <th>大小</th>
                <th>上传时间</th>
                <th>操作</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var doc in _documents)
            {
                <tr>
                    <td>@doc.FileName</td>
                    <td>@doc.Category</td>
                    <td>@FormatSize(doc.FileSize)</td>
                    <td>@doc.UploadedAt.ToLocalTime()</td>
                    <td>
                        <a href="/api/files/@doc.DocumentId" download>下载</a>
                        <button @onclick="() => DeleteDocument(doc.DocumentId)">删除</button>
                    </td>
                </tr>
            }
        </tbody>
    </table>
}

<h4>上传文档</h4>

<EditForm Model="@_uploadModel" OnValidSubmit="HandleUpload">
    <InputSelect @bind-Value="_uploadModel.Category">
        <option value="general">通用</option>
        <option value="report">报告</option>
        <option value="spec">规格</option>
    </InputSelect>
    
    <InputFile @ref="_fileElement" OnChange="HandleFileSelected" accept=".pdf,.doc,.docx,.xls,.xlsx,.txt" />
    
    <button type="submit" disabled="@(_selectedFile == null)">上传</button>
</EditForm>

@code {
    private List<Document> _documents = new();
    private UploadModel _uploadModel = new();
    private InputFile _fileElement;
    private IBrowserFile _selectedFile;
    
    protected override async Task OnInitializedAsync()
    {
        // 模拟当前用户 ID（实际应从认证状态获取）
        var currentUserId = 1;
        _documents = await DocumentService.GetUserDocumentsAsync(currentUserId);
    }
    
    private void HandleFileSelected(InputFileChangeEventArgs e)
    {
        _selectedFile = e.File;
    }
    
    private async Task HandleUpload()
    {
        if (_selectedFile == null) return;
        
        var fileName = _selectedFile.Name;
        var fileSize = _selectedFile.Size;
        var contentType = _selectedFile.ContentType;
        
        // MemoryStream 模式
        using var memoryStream = new MemoryStream();
        using (var fileStream = _selectedFile.OpenReadStream(maxAllowedSize: 25 * 1024 * 1024))
        {
            await fileStream.CopyToAsync(memoryStream);
        }
        memoryStream.Position = 0;
        
        // 调用服务上传
        var currentUserId = 1;
        await DocumentService.UploadAsync(currentUserId, _selectedFile, _uploadModel.Category, null);
        
        // 刷新列表
        _documents = await DocumentService.GetUserDocumentsAsync(currentUserId);
        _selectedFile = null;
        StateHasChanged();
    }
    
    private async Task DeleteDocument(int documentId)
    {
        var currentUserId = 1;
        await DocumentService.DeleteAsync(documentId, currentUserId);
        _documents = await DocumentService.GetUserDocumentsAsync(currentUserId);
        StateHasChanged();
    }
    
    private string FormatSize(long bytes)
    {
        string[] sizes = { "B", "KB", "MB", "GB" };
        int order = 0;
        double size = bytes;
        while (size >= 1024 && order < sizes.Length - 1)
        {
            order++;
            size /= 1024;
        }
        return $"{size:0.##} {sizes[order]}";
    }
    
    public class UploadModel
    {
        public string Category { get; set; } = "general";
    }
}
```

### 10.7 更新 NavMenu.razor

```razor
<div class="nav-item px-3">
    <NavLink class="nav-link" href="documents">
        <span class="bi bi-file-earmark" aria-hidden="true"></span> 文档
    </NavLink>
</div>
```

### 10.8 运行数据库迁移

```bash
dotnet ef migrations add AddDocumentManagement
dotnet ef database update
```

### 10.9 验证编译

```bash
dotnet build
```
预期：**0 错误**

---

## 11. 任务八：验证 MVP

### 11.1 启动应用
```bash
dotnet run
```

### 11.2 测试验收场景

#### US1：文档上传
1. 访问 `/documents` 页面
2. 选择文件（≤25MB）
3. 选择分类
4. 点击上传
5. **验证**：文件出现在列表中

#### US2：我的文档列表
1. 访问 `/documents` 页面
2. **验证**：显示已上传的所有文档

#### US3：文档下载
1. 点击文档列表中的"下载"链接
2. **验证**：文件成功下载

#### US4：文档删除
1. 点击文档列表中的"删除"按钮
2. **验证**：文档从列表中消失，文件被删除

---

## 12. 实验总结

### 12.1 Brownfield SDD 的核心价值

**约束驱动设计**：Brownfield SDD 的核心在于先理解现有代码的约束，再设计新功能。这与 Greenfield 的"从零设计"有本质区别。

### 12.2 与 LAB-12 Greenfield 的对比

| 方面 | LAB-12（Greenfield） | LAB-13（Brownfield） |
|------|---------------------|---------------------|
| 起点 | 无代码 | 现有代码库 |
| 额外步骤 | 无 | **分析现有代码库** |
| constitution | 从零创建 | **从现有代码提取** |
| spec | 完整系统规格 | **新功能规格** |
| plan | 完整架构计划 | **集成计划** |
| 风险 | 需求理解偏差 | **现有代码理解错误** |

### 12.3 IFileStorageService 接口的云迁移价值

通过抽象 `IFileStorageService` 接口，未来可以轻松迁移到云存储：

```csharp
// 未来迁移到 Azure Blob Storage
public class AzureBlobStorageService : IFileStorageService
{
    // 实现 Azure Blob 存储
}

// Program.cs 只需修改注册
services.AddScoped<IFileStorageService, AzureBlobStorageService>();
```

**无需修改业务逻辑**，只需更换实现类。

---

## 13. 附录

### 13.1 SQLite 迁移完整步骤

#### 步骤 1：修改 .csproj
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  
  <ItemGroup>
    <!-- 移除 SqlServer -->
    <!-- <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" /> -->
    
    <!-- 添加 Sqlite -->
    <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="8.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
  </ItemGroup>
</Project>
```

#### 步骤 2：修改 Program.cs
```csharp
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(connectionString));  // 改为 UseSqlite
```

#### 步骤 3：修改 appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=contoso.db"
  }
}
```

#### 步骤 4：运行迁移
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

### 13.2 IFileStorageService 接口定义

```csharp
public interface IFileStorageService
{
    /// <summary>
    /// 上传文件
    /// </summary>
    /// <param name="fileStream">文件流</param>
    /// <param name="fileName">原始文件名</param>
    /// <param name="contentType">MIME 类型</param>
    /// <param name="userId">用户 ID</param>
    /// <param name="projectId">项目 ID（可选）</param>
    /// <returns>文件存储路径（相对路径）</returns>
    Task<string> UploadAsync(Stream fileStream, string fileName, string contentType, int userId, int? projectId);
    
    /// <summary>
    /// 删除文件
    /// </summary>
    Task DeleteAsync(string filePath);
    
    /// <summary>
    /// 下载文件
    /// </summary>
    /// <returns>文件流</returns>
    Task<Stream> DownloadAsync(string filePath);
}
```

### 13.3 Blazor MemoryStream 文件上传模式

**问题**：`IBrowserFile.OpenReadStream()` 返回的流在 using 块结束后会被释放，无法在后续操作中使用。

**解决方案**：先复制到 `MemoryStream`，再传递：

```csharp
@inject IDocumentService DocumentService

<InputFile @ref="_fileElement" OnChange="HandleFileSelected" />

@code {
    private IBrowserFile _selectedFile;
    
    private void HandleFileSelected(InputFileChangeEventArgs e)
    {
        _selectedFile = e.File;
    }
    
    private async Task HandleUpload()
    {
        if (_selectedFile == null) return;
        
        var fileName = _selectedFile.Name;
        var fileSize = _selectedFile.Size;
        var contentType = _selectedFile.ContentType;
        
        // MemoryStream 模式
        using var memoryStream = new MemoryStream();
        using (var fileStream = _selectedFile.OpenReadStream(maxAllowedSize: 25 * 1024 * 1024))
        {
            await fileStream.CopyToAsync(memoryStream);
        }
        memoryStream.Position = 0;
        
        // 现在 memoryStream 可以安全使用
        var currentUserId = 1;
        await DocumentService.UploadAsync(currentUserId, _selectedFile, "general", null);
        
        _selectedFile = null; // 清除引用防止重用错误
        StateHasChanged();
    }
}
```

**关键点**：
1. 用 `using` 块包裹 `OpenReadStream()`，确保及时释放浏览器端流
2. 复制到 `MemoryStream` 后，设置 `Position = 0` 重置读取位置
3. 上传完成后清除 `_selectedFile` 引用，防止意外重用

---

## 实验完成检查清单

- [ ] 完成 SQLite 迁移（.csproj、Program.cs、appsettings.json）
- [ ] 创建 `.opencode/agent/spec-writer.md`
- [ ] 生成 `specs/constitution.md`（从现有代码提取原则）
- [ ] 生成 `specs/spec.md`（新功能规格，Given-When-Then）
- [ ] 生成 `specs/plan.md`（集成技术计划）
- [ ] 生成 `specs/tasks.md`（可执行任务清单）
- [ ] 实现 Document 和 DocumentShare 模型
- [ ] 实现 IFileStorageService + LocalFileStorageService
- [ ] 实现 DocumentService
- [ ] 创建 Documents.razor 页面（使用 MemoryStream 模式）
- [ ] 更新 NavMenu.razor
- [ ] 运行数据库迁移
- [ ] dotnet build 验证（0 错误）
- [ ] dotnet run 测试 MVP 功能

---

**实验手册版本**：1.0  
**适用课程**：OpenCode SDD 进阶（LAB-13）  
**前置实验**：LAB-12（Greenfield SDD）
