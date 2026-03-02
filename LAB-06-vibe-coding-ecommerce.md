# LAB-06 Vibe Coding：用自然语言生成电商原型应用

## 1. 实验概述

### 1.1 什么是 Vibe Coding？

**Vibe Coding** 是一种全新的编程范式：用自然语言描述你的想法，让 AI 理解你的意图并生成代码。你不需要逐行编写代码，而是像"指挥"一样告诉 AI 你想要什么，然后快速迭代、测试、优化。

核心理念：
- **描述即代码**：用自然语言描述需求，AI 帮你实现
- **快速迭代**：不追求一次完美，生成 → 测试 → 改进
- **人类决策，AI 执行**：你把握方向，AI 处理细节

### 1.2 学习目标

完成本实验后，你将能够：
1. 使用 OpenCode 的 Plan 模式生成产品需求文档（PRD）
2. 使用 Build 模式基于 PRD 生成完整的 Web 应用
3. 通过多轮对话迭代优化应用功能
4. 掌握 OpenCode 在 Vibe Coding 中的独特优势

### 1.3 预计时间

**40-50 分钟**

### 1.4 前置条件

- 已完成前序 Lab，熟悉 OpenCode 基本操作
- 具备基础 Web 知识（HTML/CSS/JavaScript）
- 已安装 OpenCode

### 1.5 OpenCode 在 Vibe Coding 中的优势

| VSCode + Copilot | OpenCode | 优势说明 |
|---|---|---|
| `.github/copilot-instructions.md` | `AGENTS.md` | OpenCode 原生支持项目级规则文件，无需额外配置 |
| Ask 模式生成 PRD | Plan 模式 | 先分析需求再写文档，逻辑更清晰 |
| Agent 模式创建文件 | Build 模式 | 直接写入文件，不需要"生成→Apply→Keep"三步 |
| 拖拽文件到 Chat | `@文件路径` 引用 | 直接引用，更精确高效 |
| VSCode 内置浏览器预览 | `!python3 -m http.server 8080` | 一行命令启动，无需配置调试器 |
| 手动 Ctrl+Z 回滚 | `/undo` 命令 | OpenCode 专属回滚，更智能 |
| 无内置版本控制 | `!git commit` | 每次迭代后提交，完整保留每个版本 |

---

## 2. 环境准备

### 2.1 创建项目目录

```bash
mkdir -p ~/VibeCoding-ShoppingApp && cd ~/VibeCoding-ShoppingApp
```

### 2.2 初始化 Git 仓库

Git 用于版本控制，每次迭代后提交，方便回滚和追踪进度：

```bash
git init
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

### 2.3 验证 OpenCode

在终端中输入以下命令启动 OpenCode：

```bash
opencode
```

看到 OpenCode 提示符后，说明环境准备完成。

---

## 3. 任务一：定义产品需求（Plan 模式）

### 3.1 启动 Plan 模式

在 OpenCode 中，Plan 模式用于需求分析和文档生成。切换到 Plan 模式：

```
/plan
```

### 3.2 生成 PRD 文档

输入以下 Prompt，让 AI 基于高层描述生成完整的产品需求文档：

```
我要用 vibe coding 方式开发一个电商原型 App。
请帮我生成一份 PRD（产品需求文档），保存为 VibeCodingPRD.md。

高层需求：
1. 使用 HTML、CSS、JavaScript 开发纯前端 Web App
2. 包含以下页面：Products（商品列表）、ProductDetails（商品详情）、ShoppingCart（购物车）、Checkout（结账）
3. 页面间可以导航

详细需求：
- 动态响应式界面，自适应桌面和手机屏幕
- 10 种水果商品数据集（名称、描述、单价、emoji 图标）
- 左侧导航栏，宽度 <600px 时折叠仅显示 emoji 图标
- 基础样式，视觉美观但不需要完全响应式

页面说明：
- Products 页：显示商品列表（名称、价格、emoji），支持选择数量和加入购物车
- ProductDetails 页：显示商品详情（名称、描述、价格、emoji），支持返回商品列表
- ShoppingCart 页：显示购物车商品（名称、数量、小计），支持修改数量和删除
- Checkout 页：显示订单摘要（商品名、数量、价格、总价），有"处理订单"按钮

不包含：用户认证、支付处理、后端数据库

PRD 需要包含以下章节：
1. 应用概述
2. 目标用户
3. 核心功能
4. 页面说明
5. 导航设计
6. 数据集定义
7. 技术要求
8. 样式规范
9. 用户用例
10. 范围外功能
```

OpenCode 会分析需求并生成 `VibeCodingPRD.md` 文件。

### 3.3 生成线框图

线框图用于描述每个页面的布局和元素。输入以下 Prompt：

```
@VibeCodingPRD.md
请为电商原型 App 的每个页面生成文字线框图，保存到 wireframes/ 目录：
- wireframes/products.txt
- wireframes/product-details.txt
- wireframes/shopping-cart.txt
- wireframes/checkout.txt
- wireframes/navigation.txt（展开和折叠两种状态）
```

### 3.4 创建 AGENTS.md（持久上下文）

**AGENTS.md 是 OpenCode 的项目级指令文件**，类似于 VSCode 的 `.github/copilot-instructions.md`。OpenCode 在每次对话时都会自动读取它，无需手动引用。

输入以下 Prompt：

```
@VibeCodingPRD.md @wireframes/
请创建 AGENTS.md 文件，将 PRD 内容和线框图内容整合进去。
AGENTS.md 将作为本项目的持久上下文，OpenCode 在每次对话中都会自动读取它。
```

### 3.5 审查并调整 PRD

阅读生成的 `VibeCodingPRD.md` 和 `AGENTS.md`，如有需要调整的地方：
- 直接编辑 `AGENTS.md` 文件修改需求
- 或在对话中告诉 OpenCode 需要调整的内容

---

## 4. 任务二：生成初始原型（Build 模式）

### 4.1 切换到 Build 模式

Build 模式用于代码生成和文件创建。切换到 Build 模式：

```
/build
```

### 4.2 生成完整 App

基于 PRD 和线框图，生成完整的电商原型应用：

```
@VibeCodingPRD.md @wireframes/products.txt @wireframes/product-details.txt @wireframes/shopping-cart.txt @wireframes/checkout.txt @wireframes/navigation.txt

请基于 PRD 和线框图，创建完整的电商原型 App，包含以下文件：
- index.html：App 入口，包含基本结构和导航
- styles.css：响应式样式，左侧导航栏（<600px 时折叠）
- app.js：JavaScript 逻辑（商品数据、购物车、页面导航、UI 渲染）

要求：
1. 10 种水果商品数据（含 emoji）
2. 左侧导航栏，600px 断点折叠
3. 四个页面的完整功能
4. 无需后端，纯前端实现
```

OpenCode 会直接写入文件，不需要额外确认步骤。

### 4.3 启动本地服务器预览

在 OpenCode 中直接运行 Python 内置服务器：

```
!python3 -m http.server 8080 &
```

**说明**：
- `!` 前缀表示在终端执行 shell 命令
- `&` 表示后台运行，不阻塞 OpenCode

然后在浏览器中打开：

```
http://localhost:8080
```

**备选方案**：
- 如果 `python3` 不可用，可以用 `npx serve .` 或直接用浏览器打开 `index.html` 文件

### 4.4 测试用例

在浏览器中验证以下功能：

| 序号 | 测试用例 | 预期结果 |
|---|---|---|
| 1 | 浏览商品列表（Products 页） | 显示 10 种水果商品 |
| 2 | 点击商品查看详情 | 跳转到 ProductDetails 页 |
| 3 | 添加商品到购物车 | 购物车数量增加 |
| 4 | 在购物车修改数量 | 小计金额更新 |
| 5 | 删除购物车商品 | 商品从购物车移除 |
| 6 | 进入结账页面 | 显示订单摘要和总价 |
| 7 | 调整浏览器宽度 <600px | 导航栏折叠显示缩写字母 |

### 4.5 提交初始版本

初始原型生成完成后，使用 Git 提交保存：

```
!git add . && git commit -m "feat: initial ecommerce prototype"
```

---

## 5. 任务三：迭代优化（Build 模式）

Vibe Coding 的核心理念是**快速迭代**：不追求一次完美，而是生成 → 测试 → 改进。

### 5.1 修复导航栏折叠断点

如果测试发现导航栏折叠断点不正确（如 300px 而非 600px），输入以下 Prompt 修复：

```
@index.html @styles.css @app.js
请将导航栏折叠的媒体查询断点从 300px 改为 600px。
```

**注意**：如果初始生成的 App 断点已经是 600px，可以跳过这步；如果是 300px，则需要修改。

### 5.2 为导航栏添加 emoji 图标

为导航项添加视觉化的 emoji 图标：

```
@index.html @styles.css @app.js
请为导航栏的每个菜单项添加 emoji 图标：
- Products 页：🛒
- ShoppingCart 页：🛍️
- Checkout 页：💳
```

### 5.3 用 AI 自动发现改进点（Plan 模式审查）

切换到 Plan 模式，让 AI 审查代码并提出改进建议：

```
/plan
@index.html @styles.css @app.js
请审查当前代码，列出 3-5 个可以改进的地方（用户体验、代码质量、功能完整性）。
```

AI 可能会建议：
- 用 toast 通知替代 alert() 弹窗
- 添加订单确认页面
- 导航栏添加购物车数量徽章

### 5.4 实现改进功能

切换回 Build 模式，实现 AI 建议的改进：

```
/build
@index.html @styles.css @app.js
实现以下改进：
1. 用 toast 通知替代 alert() 弹窗（加入购物车时显示绿色 toast）
2. 处理订单后显示确认页面（感谢信息 + 订单号）
3. 导航栏购物车图标旁添加数量徽章（红色圆形，显示购物车商品总数）

同时更新 AGENTS.md，记录这些新增功能。
```

### 5.5 使用 /undo 回滚

如果某次迭代结果不理想，使用 `/undo` 命令回滚：

```
/undo
```

**/undo 的优势**：
- 智能回滚到上一次对话前的状态
- 比 Ctrl+Z 更可靠，专门针对 OpenCode 的修改
- 可以连续使用，回滚多步

### 5.6 每次迭代后提交 Git

每次成功迭代后，使用 Git 提交保存进度：

```
!git add . && git commit -m "feat: add toast notifications and order confirmation"
```

**版本控制的好处**：
- 完整保留每个迭代版本
- 可以随时回退到任意历史版本
- 清晰追踪功能演进过程

---

## 6. 实验总结

### 6.1 Vibe Coding 工作流回顾

本实验完整体验了 Vibe Coding 的核心工作流：

```
1. 定义需求（Plan 模式）
   ↓
2. 生成原型（Build 模式）
   ↓
3. 测试验证（浏览器预览）
   ↓
4. 迭代优化（多轮对话）
   ↓
5. 版本控制（Git 提交）
```

### 6.2 OpenCode 优势总结

| 特性 | 说明 |
|---|---|
| **AGENTS.md** | 原生支持项目级规则文件，持久上下文自动加载 |
| **Plan 模式** | 先分析再执行，PRD 生成更结构化 |
| **Build 模式** | 直接写入文件，无需"生成→Apply→Keep"三步 |
| **终端预览** | `!python3 -m http.server 8080` 一行命令启动服务 |
| **/undo 回滚** | 智能回滚到对话前状态，比 Ctrl+Z 更可靠 |
| **@文件引用** | 直接引用任意文件作为上下文，精确高效 |
| **Git 集成** | 每次迭代后提交，完整保留版本历史 |

### 6.3 关键收获

1. **自然语言即代码**：用描述性语言驱动 AI 生成，而非逐行编写
2. **迭代优于完美**：快速生成原型，通过测试发现问题，逐步改进
3. **上下文管理**：AGENTS.md 作为持久上下文，@引用作为临时上下文
4. **版本控制意识**：每次迭代后 Git 提交，保留完整演进历史

---

## 7. 附录：快速参考

### 7.1 常用命令

| 命令 | 说明 |
|---|---|
| `/plan` | 切换到 Plan 模式（需求分析、文档生成） |
| `/build` | 切换到 Build 模式（代码生成、文件创建） |
| `/undo` | 回滚到上一次对话前的状态 |
| `!命令` | 在终端执行 shell 命令 |
| `@文件路径` | 引用文件作为上下文 |

### 7.2 本地预览方法

```bash
# 方法一：Python 内置服务器（推荐）
python3 -m http.server 8080
# 然后在浏览器打开 http://localhost:8080

# 方法二：在 OpenCode 内直接运行
!python3 -m http.server 8080 &
```

### 7.3 AGENTS.md vs copilot-instructions.md

| 特性 | VSCode + Copilot | OpenCode |
|---|---|---|
| 文件名 | `.github/copilot-instructions.md` | `AGENTS.md` |
| 位置 | `.github/` 子目录 | 项目根目录 |
| 加载方式 | 自动加载 | 自动加载 |
| 用途 | 项目级指令、编码规范 | 项目级指令、编码规范 |

### 7.4 迭代版本控制

```bash
# 每次迭代后提交
!git add . && git commit -m "feat: 描述本次改进"

# 如果迭代失败，回滚
/undo

# 查看历史版本
!git log --oneline

# 回退到指定版本
!git reset --hard <commit-hash>
```

### 7.5 测试检查清单

完成实验后，确保以下用例全部通过：

- [ ] 浏览商品列表（Products 页）
- [ ] 点击商品查看详情（ProductDetails 页）
- [ ] 添加商品到购物车
- [ ] 在购物车修改数量和删除商品
- [ ] 进入结账页面查看订单摘要
- [ ] 点击"处理订单"显示确认页面
- [ ] 调整浏览器宽度测试导航栏折叠（<600px）
- [ ] 加入购物车时显示 toast 通知
- [ ] 导航栏显示购物车数量徽章

---

**实验完成！你已成功使用 Vibe Coding 方式从零生成一个完整的电商原型应用。**
