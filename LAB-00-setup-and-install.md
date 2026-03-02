# LAB-00: OpenCode 安装与环境准备

> **本实验编号**：LAB-00  
> **系列**：OpenCode Labs 入门系列  
> **版本**：1.0  
> **最后更新**：2026-03-02

---

## 1. 实验概述

### 标题
OpenCode 安装与环境准备

### 学习目标
完成本实验后，你将能够：
1. 在 Windows/macOS/Linux 任一平台上安装 OpenCode CLI
2. 申请并配置阿里云百炼 Coding Plan API Key
3. 使用 Prompt 驱动 OpenCode 自动安装开发环境（.NET 8.0、Python 3.x、Git）
4. 验证所有工具安装成功， ready for LAB-01

### 预计时间
30-45 分钟

### 前置要求
- 一台全新或重置的计算机（Windows 10+/macOS 12+/Linux Ubuntu 20.04+）
- 稳定的网络连接
- 阿里云账号（用于申请百炼 API Key）

### 说明
> **重要**：完成本实验后即可开始 LAB-01。所有环境安装步骤请通过 Prompt 驱动 OpenCode 执行，无需手动安装。

---

## 2. 安装 OpenCode

### 2.1 Windows 安装

Windows 用户推荐使用 **Windows Terminal + PowerShell 7** 获得最佳体验。

#### 方式一：Scoop（推荐，国内网络友好）

```powershell
# 1. 安装 Scoop（如未安装）
iwr -useb get.scoop.sh | iex

# 2. 安装 OpenCode
scoop install opencode

# 3. 验证安装
opencode --version
```

#### 方式二：Chocolatey

```powershell
# 1. 以管理员身份运行 PowerShell
choco install opencode

# 2. 验证安装
opencode --version
```

#### 方式三：桌面版下载

1. 访问 https://opencode.ai/download
2. 下载 `opencode-desktop-windows-x64.exe`
3. 运行安装程序，跟随向导完成安装
4. 打开 PowerShell 验证：

```powershell
opencode --version
```

> **提示**：Scoop 方式最轻量且无需管理员权限，推荐首选。

---

### 2.2 macOS 安装

#### 方式一：Homebrew Tap（推荐，始终最新）

```bash
# 1. 安装 Homebrew（如未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. 添加 anomalyco tap 并安装
brew tap anomalyco/tap
brew install anomalyco/tap/opencode

# 3. 验证安装
opencode --version
```

#### 方式二：桌面版

```bash
# Apple Silicon (M1/M2/M3)
brew install --cask opencode-desktop

# Intel Mac
# 访问 https://opencode.ai/download 下载 opencode-desktop-darwin-x64.dmg
```

> **说明**：Apple Silicon 芯片使用 aarch64 版本，Intel 芯片使用 x64 版本。Homebrew tap 方式始终提供最新版本。

---

### 2.3 Linux 安装

#### 通用脚本方式（最简单，推荐）

```bash
# 1. 一键安装
curl -fsSL https://opencode.ai/install | bash

# 2. 验证安装
opencode --version
```

#### Arch Linux 包管理器方式

```bash
# 方式一：官方仓库
sudo pacman -S opencode

# 方式二：AUR（最新版本）
paru -S opencode-bin

# 验证安装
opencode --version
```

#### 安装路径说明

OpenCode 在 Linux/macOS 上的安装路径优先级：

| 优先级 | 环境变量 | 路径 |
|--------|----------|------|
| 1 | `$OPENCODE_INSTALL_DIR` | 自定义目录 |
| 2 | `$XDG_BIN_DIR` | `~/.local/bin` |
| 3 | - | `~/bin` |
| 4 | - | `~/.opencode/bin`（默认回退） |

> **提示**：如遇到 `command not found`，请将安装路径添加到 `$PATH`。

---

### 2.4 验证安装

```bash
opencode --version
```

**预期输出**：
```
1.2.x
```

> **要求**：版本号应为 1.2.15 或更高。如版本过低，请重新安装。

---

## 3. 配置阿里云百炼 Coding Plan（推荐）

### 3.1 为什么推荐百炼 Coding Plan

| 优势 | 说明 |
|------|------|
| 国内网络无需代理 | 直接访问，无需科学上网 |
| 价格友好 | Coding Plan 专为开发者优化，性价比高 |
| 模型质量高 | Qwen3.5 Plus 支持 thinking 模式 |
| 深度集成 | Anthropic SDK 兼容接口，与 OpenCode 无缝对接 |

---

### 3.2 申请 API Key

#### 步骤

1. **访问百炼控制台**
   - 打开 https://bailian.console.aliyun.com/

2. **注册/登录阿里云账号**
   - 使用阿里云账号登录（如无账号需先注册）

3. **开通百炼服务**
   - 首次访问需开通百炼服务（按提示完成）

4. **创建 API Key**
   - 进入「API Key 管理」页面
   - 点击「创建新的 API Key」
   - 复制生成的 API Key

5. **保存 API Key**
   - API Key 格式：`sk-sp-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
   - 妥善保存，后续配置需要

> **警告**：API Key 具有账户权限，请勿泄露或提交到代码仓库。

---

### 3.3 配置 opencode.json

#### 配置文件位置

```
~/.config/opencode/opencode.json
```

> 如目录不存在，请先创建：`mkdir -p ~/.config/opencode`

#### 完整配置

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "bailian-coding-plan": {
      "npm": "@ai-sdk/anthropic",
      "name": "Model Studio Coding Plan",
      "options": {
        "baseURL": "https://coding.dashscope.aliyuncs.com/apps/anthropic/v1",
        "apiKey": "YOUR_API_KEY_HERE"
      },
      "models": {
        "qwen3.5-plus": {
          "name": "Qwen3.5 Plus",
          "modalities": {
            "input": ["text", "image"],
            "output": ["text"]
          },
          "options": {
            "thinking": {
              "type": "enabled",
              "budgetTokens": 1024
            }
          }
        },
        "qwen3-coder-plus": {
          "name": "Qwen3 Coder Plus"
        }
      }
    }
  }
}
```

#### 字段说明

| 字段 | 含义 |
|------|------|
| `npm` | 使用 `@ai-sdk/anthropic` 兼容接口 |
| `baseURL` | 百炼 Coding Plan 的 API 端点 |
| `apiKey` | 替换为你的百炼 API Key（`sk-sp-...`） |
| `models` | 可用模型列表（至少包含 qwen3.5-plus） |

> **操作**：将 `YOUR_API_KEY_HERE` 替换为你从百炼控制台复制的 API Key。

---

### 3.4 验证配置

```bash
# 1. 启动 OpenCode TUI
opencode

# 2. 在 TUI 中输入以下命令
/models
```

**预期结果**：能看到 `qwen3.5-plus`、`qwen3-coder-plus` 等百炼模型。

---

## 4. 配置其他 Provider（可选）

### 4.1 通用配置方式（/connect 命令）

对于 OpenAI、Anthropic、Google 等主流 provider，可使用交互式配置：

```bash
# 1. 启动 OpenCode
opencode

# 2. 输入 /connect 命令
/connect

# 3. 选择 provider（如 OpenAI、Anthropic 等）

# 4. 按提示输入 API Key

# 5. 配置完成后验证
/models
```

> **说明**：API Key 存储在 `~/.local/share/opencode/auth.json`，安全加密。

---

### 4.2 OpenCode Zen（官方托管，国际用户推荐）

OpenCode Zen 是官方提供的托管模型服务，适合国际用户：

```bash
# 1. 启动 OpenCode
opencode

# 2. 连接到 Zen
/connect zen

# 3. 按提示完成配置
```

> **说明**：Zen 提供 OpenCode 优化的模型端点，无需单独配置 API Key。

---

## 5. 用 Prompt 准备实验环境

### 5.1 核心理念

> **重要**：OpenCode 不只是写代码的工具，也可以帮你配置开发环境。通过自然语言描述需求，OpenCode 会自动检测系统、安装依赖、验证结果。

以下所有环境安装步骤，**请直接把 Prompt 粘贴给 OpenCode 执行**，不需要手动安装。

---

### 5.2 安装 Git

```
请检查我的系统是否已安装 Git。
如果没有安装，请根据我的操作系统（Windows/macOS/Linux）选择合适的安装方式安装 Git。
安装完成后验证：git --version 应该输出版本号。
```

---

### 5.3 安装 .NET 8.0 SDK

```
请检查我的系统是否已安装 .NET 8.0 SDK。
如果没有安装，请根据我的操作系统安装 .NET 8.0 SDK：
- Windows：使用 winget 或下载官方安装包
- macOS：使用 Homebrew（brew install dotnet@8）
- Linux（Ubuntu/Debian）：使用 apt 安装 dotnet-sdk-8.0
安装完成后验证：dotnet --version 应该输出 8.x.x
```

---

### 5.4 安装 Python 3.x

```
请检查我的系统是否已安装 Python 3.8 或更高版本。
如果没有安装，请根据我的操作系统安装 Python：
- Windows：使用 winget install Python.Python.3 或 Microsoft Store
- macOS：使用 Homebrew（brew install python3）
- Linux：使用系统包管理器（apt/yum/pacman）
安装完成后验证：python3 --version 应该输出 3.x.x
同时确保 pip3 可用。
```

---

### 5.5 安装 Node.js（可选，部分实验需要）

```
请检查我的系统是否已安装 Node.js 18 或更高版本。
如果没有安装，请根据我的操作系统安装 Node.js LTS 版本。
安装完成后验证：node --version 和 npm --version 均应输出版本号。
```

---

### 5.6 一键检查所有环境

```
请检查以下开发环境是否已正确安装，并输出检查报告：
1. Git（git --version）
2. .NET 8.0 SDK（dotnet --version）
3. Python 3.x（python3 --version）
4. Node.js（node --version，可选）
5. OpenCode（opencode --version）

对于未安装的工具，请说明安装命令。
```

---

## 6. 验证完整安装

### 6.1 最终验收清单

完成所有步骤后，请逐项检查：

- [ ] `opencode --version` 输出版本号（1.2.15+）
- [ ] `/models` 命令能看到百炼模型（或其他已配置的模型）
- [ ] `git --version` 输出版本号
- [ ] `dotnet --version` 输出 8.x.x
- [ ] `python3 --version` 输出 3.x.x（3.8+）
- [ ] `node --version` 输出版本号（可选，18+）
- [ ] `pip3 --version` 可用

---

### 6.2 常见问题排查

#### 问题 1：`opencode: command not found`

**原因**：PATH 未配置

**解决**：
```bash
# Linux/macOS：添加安装路径到 PATH
export PATH="$HOME/.opencode/bin:$PATH"
# 或添加到 ~/.bashrc / ~/.zshrc 永久生效

# Windows：在系统环境变量中添加
# %USERPROFILE%\.opencode\bin
```

---

#### 问题 2：模型列表为空

**原因**：API Key 未配置或配置错误

**解决**：
1. 检查 `~/.config/opencode/opencode.json` 是否存在
2. 确认 `apiKey` 字段正确填写（`sk-sp-...` 格式）
3. 确认 `baseURL` 为 `https://coding.dashscope.aliyuncs.com/apps/anthropic/v1`
4. 重启 OpenCode：`opencode`

---

#### 问题 3：`dotnet: command not found`

**原因**：.NET SDK 未安装或 PATH 问题

**解决**：
```bash
# 重新安装 .NET 8.0 SDK
# macOS
brew install dotnet@8

# Ubuntu/Debian
sudo apt install dotnet-sdk-8.0

# Windows（PowerShell）
winget install Microsoft.DotNet.SDK.8
```

---

#### 问题 4：Windows 上 `curl` 命令不可用

**原因**：Windows 默认不带 curl

**解决**：
```powershell
# 使用 PowerShell 原生命令替代
iwr -useb https://opencode.ai/install | iex

# 或安装 curl
choco install curl
```

---

#### 问题 5：macOS 上 Homebrew 安装慢

**原因**：默认源在国外

**解决**：
```bash
# 配置国内镜像（如清华源）
export HOMEBREW_BOTTLE_HOST=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles

# 或编辑 ~/.zshrc / ~/.bashrc
echo 'export HOMEBREW_BOTTLE_HOST=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

---

## 7. 附录

### 附录 A：百炼 Coding Plan 完整模型列表

| 模型名称 | 特点 | 推荐用途 |
|---------|------|---------|
| qwen3.5-plus | thinking + 多模态，通用最强 | 大多数任务 |
| qwen3-coder-plus | 编码专用优化 | 代码生成/重构 |
| qwen3-max-2026-01-23 | 最强推理能力 | 复杂分析 |
| glm-5 | 快速响应，thinking 支持 | 简单任务 |
| glm-4.7 | 快速响应 | 快速查询 |
| kimi-k2.5 | 多模态 + thinking | 图像分析 |
| MiniMax-M2.5 | thinking 支持 | 推理任务 |

---

### 附录 B：opencode.json 完整配置参考

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "bailian-coding-plan": {
      "npm": "@ai-sdk/anthropic",
      "name": "Model Studio Coding Plan",
      "options": {
        "baseURL": "https://coding.dashscope.aliyuncs.com/apps/anthropic/v1",
        "apiKey": "sk-sp-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      },
      "models": {
        "qwen3.5-plus": {
          "name": "Qwen3.5 Plus",
          "modalities": {
            "input": ["text", "image"],
            "output": ["text"]
          },
          "options": {
            "thinking": {
              "type": "enabled",
              "budgetTokens": 1024
            }
          }
        },
        "qwen3-coder-plus": {
          "name": "Qwen3 Coder Plus"
        },
        "qwen3-max-2026-01-23": {
          "name": "Qwen3 Max"
        },
        "glm-5": {
          "name": "GLM-5"
        },
        "glm-4.7": {
          "name": "GLM-4.7"
        },
        "kimi-k2.5": {
          "name": "Kimi K2.5"
        },
        "minimax-m2.5": {
          "name": "MiniMax M2.5"
        }
      }
    }
  }
}
```

> **说明**：可根据需要添加或删除模型。至少保留 `qwen3.5-plus` 作为默认模型。

---

### 附录 C：安装路径速查

| 平台 | 默认安装路径 | 配置文件路径 |
|------|------------|------------|
| Linux/macOS | `~/.opencode/bin/opencode` | `~/.config/opencode/opencode.json` |
| Windows | `%USERPROFILE%\.opencode\bin\opencode.exe` | `%APPDATA%\opencode\opencode.json` |
| Arch Linux | `/usr/bin/opencode` | `~/.config/opencode/opencode.json` |

---

### 附录 D：官方资源

| 资源 | 链接 |
|------|------|
| OpenCode 官网 | https://opencode.ai |
| OpenCode GitHub | https://github.com/anomalyco/opencode |
| OpenCode 下载 | https://opencode.ai/download |
| 阿里云百炼控制台 | https://bailian.console.aliyun.com/ |
| 百炼 API 文档 | https://help.aliyun.com/zh/model-studio |

---

### 附录 E：下一步

完成本实验后，请继续：

1. **LAB-01**：OpenCode 基础操作与 TUI 使用
2. **LAB-02**：Prompt 工程与上下文管理
3. **LAB-03**：使用 OpenCode 开发 .NET 应用
4. **LAB-04**：使用 OpenCode 开发 Python 应用

> **祝贺**：你已完成 OpenCode 安装与环境准备！现在可以开始 LAB-01 了。

---

**文档结束**
