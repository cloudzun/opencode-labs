# LAB-04：使用 OpenCode 开发单元测试

## 1. 实验概述

### 1.1 学习目标

完成本实验后，你将能够：

- 使用 OpenCode 的 **Plan 模式** 分析现有测试架构
- 使用 OpenCode 的 **Build 模式** 快速生成单元测试代码
- 理解 `unittest` 框架的核心概念（`setUp`、`test_*` 方法、`MagicMock`）
- 为基础设施层（Infrastructure Layer）创建测试文件
- 同时使用 `unittest` 和 `pytest` 两种方式运行测试
- 掌握测试驱动开发（TDD）的基本流程

### 1.2 预计时间

- **总时长**：35-45 分钟
- 环境准备：5 分钟
- 任务一（理解测试架构）：10 分钟
- 任务二（创建测试文件）：15 分钟
- 任务三（运行验证）：5 分钟
- 任务四（扩展测试）：10 分钟（可选进阶）

### 1.3 前置条件

- 已完成 LAB-02（OpenCode 基础操作）和 LAB-03（代码生成与修改）
- 具备 Python 基础（类、方法、导入模块）
- 了解单元测试的基本概念

---

## 2. 环境准备

### 2.1 验证 OpenCode 安装

打开终端，运行：

```bash
opencode --version
```

确保版本号为最新。

### 2.2 验证 Python 环境

```bash
python3 --version
```

确保 Python 版本 ≥ 3.10。

### 2.3 安装 pytest

```bash
pip3 install pytest --break-system-packages
```

> 注意：在 Ubuntu/Debian 系统上需要添加 `--break-system-packages` 参数以绕过系统保护。

验证安装：

```bash
pytest --version
```

---

## 3. 项目配置

### 3.1 下载实验室项目

```bash
wget https://github.com/microsoft/AccelerateDevGHCopilot/raw/main/M4/LAB_AK_04/AZ2007LabAppM4Python.zip
unzip AZ2007LabAppM4Python.zip
cd AccelerateDevGHCopilot/library
```

### 3.2 查看项目结构

```bash
tree -L 3
```

项目结构应包含：

```
library/
├── application_core/      # 核心业务实体
├── console/               # 控制台应用（book_repository.py, main.py）
├── infrastructure/        # 基础设施层（JsonLoanRepository 等）
├── tests/                 # 测试目录
└── readme.md
```

### 3.3 运行初始测试验证

首先切换到 library 目录：

```bash
cd AccelerateDevGHCopilot/library
python3 -m unittest discover tests -v
```

预期输出：现有测试用例全部通过（4 个测试）。

---

## 4. 任务一：理解现有测试架构（Plan 模式）

> **目标**：使用 OpenCode 的 **Plan 模式** 分析现有测试文件，理解测试框架、mock 用法和测试组织方式。

### 4.1 启动 OpenCode 并切换到 Plan 模式

```bash
opencode
```

进入 OpenCode 后，默认进入 **Plan 模式**。如果当前是 Build 模式，按 `Esc` 或输入 `/plan` 切换。

### 4.2 使用 @explore 分析测试结构

输入以下 prompt：

```
@explore 分析 tests/ 目录的测试架构：
1. 使用了哪些测试框架和 mock 工具？
2. 测试类的组织方式（setUp、test_* 方法）
3. 如何 mock 依赖（MagicMock 的使用方式）
4. 如何扩展测试到 infrastructure/ 目录？
```

**预期输出示例**：

```
📊 测试架构分析

1. 测试框架：unittest (Python 内置)
2. Mock 工具：unittest.mock.MagicMock
3. 测试组织：
   - 每个测试类继承 unittest.TestCase
   - setUp 方法初始化测试数据
   - test_* 方法命名测试用例
4. infrastructure 层扩展建议：
   - 创建 test_json_loan_repository.py
   - 使用 DummyJsonData 替代真实 JSON 文件
   - 模拟 save_loans 和 load_data 方法
```

### 4.3 阅读示例测试文件

在 OpenCode 中打开现有测试文件参考：

```
@tests/test_loan_service.py 请展示这个文件的内容
```

**关键观察点**：

- 导入路径如何设置（`sys.path` 添加项目根目录）
- `setUp` 方法如何初始化测试数据
- 如何使用 `MagicMock` 模拟依赖
- 测试方法的命名规范（`test_` 开头）

### 4.4 规划测试扩展

基于分析结果，规划下一步：

```
根据以上分析，我需要为 infrastructure/json_loan_repository.py 创建测试文件。
请列出需要测试的方法和测试场景。
```

**预期输出**：

```
需要测试的方法：
1. get_loan(loan_id) - 场景：找到/找不到
2. get_loans_by_patron_id(patron_id) - 场景：有贷款/无贷款
3. update_loan(loan) - 场景：更新成功

建议的测试文件：tests/test_json_loan_repository.py
```

---

## 5. 任务二：为 JsonLoanRepository 创建测试文件（Build 模式）

> **目标**：切换到 **Build 模式**，使用一个完整的 prompt 生成测试文件骨架、DummyJsonData 辅助类和所有测试方法。

### 5.1 切换到 Build 模式

在 OpenCode 中输入 `/build` 或按快捷键切换到 Build 模式。

> **Plan vs Build 模式对比**：
> | 模式 | 用途 | 适用场景 |
> |---|---|---|
> | Plan | 分析、规划、探索 | 理解代码结构、制定方案 |
> | Build | 生成、修改、创建文件 | 编写代码、创建测试 |

### 5.2 创建完整测试文件

输入以下 prompt（复制完整内容）：

```
@tests/test_loan_service.py @infrastructure/json_loan_repository.py @infrastructure/json_data.py

参考现有测试文件的风格，在 tests/ 目录创建 test_json_loan_repository.py，包含：
1. 正确的 import 路径（使用 sys.path 添加项目根目录）
2. DummyJsonData 辅助类（实现 save_loans 和 load_data 方法）
3. TestJsonLoanRepository 测试类
4. setUp 方法：创建测试用的 Patron、BookItem、Loan 对象，初始化 JsonLoanRepository
5. test_get_loan_found：测试 get_loan(1) 返回正确的 loan 对象
6. test_get_loan_not_found：测试 get_loan(999) 返回 None
7. if __name__ == "__main__": unittest.main()
```

### 5.3 理解生成的代码

OpenCode 会创建文件 `tests/test_json_loan_repository.py`。关键代码解析：

#### 5.3.1 sys.path 设置（解决 import 路径问题）

```python
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parent.parent))
```

**为什么需要**：测试文件在 `tests/` 子目录，需要添加项目根目录到导入路径。

#### 5.3.2 DummyJsonData 辅助类的设计思路

```python
class DummyJsonData:
    def __init__(self):
        self.loans = []
        self.save_loans_called = False
    
    def save_loans(self, loans):
        self.loans = loans
        self.save_loans_called = True
    
    def load_data(self):
        return self.loans
```

**设计思路**：

1. **隔离外部依赖**：不操作真实 JSON 文件，避免测试污染磁盘
2. **内存存储**：使用列表 `self.loans` 在内存中存储数据
3. **状态追踪**：`save_loans_called` 标记验证 `save_loans` 是否被调用
4. **接口一致**：实现与真实 `JsonData` 相同的方法签名

#### 5.3.3 setUp 方法中的测试数据构建

```python
from datetime import datetime, timedelta
from application_core.entities.patron import Patron
from application_core.entities.book_item import BookItem
from application_core.entities.loan import Loan

def setUp(self):
    # 创建测试 Patron
    self.test_patron = Patron(
        id=1, 
        name="Test Patron",
        membership_end=datetime.now() + timedelta(days=30),
        membership_start=datetime.now() - timedelta(days=365)
    )
    
    # 创建测试 BookItem
    self.test_book_item = BookItem(
        id=1, 
        book_id=1,
        acquisition_date=datetime.now() - timedelta(days=100)
    )
    
    # 创建测试 Loan
    self.test_loan = Loan(
        id=1, 
        book_item_id=1, 
        patron_id=1,
        patron=self.test_patron,
        loan_date=datetime.now() - timedelta(days=10),
        due_date=datetime.now() + timedelta(days=4),
        return_date=None, 
        book_item=self.test_book_item
    )
    
    # 初始化 Repository（使用 DummyJsonData）
    self.dummy_data = DummyJsonData()
    self.repository = JsonLoanRepository(self.dummy_data)
    self.repository.loans = [self.test_loan]
```

**关键点**：

- 使用 `self.` 存储为实例变量，供测试方法使用
- 设置合理的日期（借书日期在过去，应还日期在未来）
- `return_date=None` 表示尚未归还

### 5.4 验证生成的文件

```bash
cat tests/test_json_loan_repository.py
```

检查文件是否包含：

- [ ] `DummyJsonData` 类
- [ ] `TestJsonLoanRepository` 类
- [ ] `setUp` 方法
- [ ] `test_get_loan_found` 方法
- [ ] `test_get_loan_not_found` 方法

---

## 6. 任务三：运行测试验证

### 6.1 使用 unittest 运行测试

首先切换到 library 目录：

```bash
cd AccelerateDevGHCopilot/library
```

在 OpenCode 中直接运行（使用 `!` 前缀）：

```
!python3 -m unittest discover tests -v
```

或在终端运行：

```bash
python3 -m unittest discover tests -v
```

**预期输出**：

```
test_get_loan_found (tests.test_json_loan_repository.TestJsonLoanRepository) ... ok
test_get_loan_not_found (tests.test_json_loan_repository.TestJsonLoanRepository) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
```

### 6.2 使用 pytest 运行测试

确保当前在 library 目录下：

```bash
cd AccelerateDevGHCopilot/library
```

在 OpenCode 中直接运行（使用 `!` 前缀）：

```
!pytest tests -v
```

或：

```bash
pytest tests -v
```

**预期输出**：

```
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loan_found PASSED
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loan_not_found PASSED

2 passed in 0.02s
```

### 6.3 unittest vs pytest 对比

| 特性 | unittest | pytest |
|---|---|---|
| **运行命令** | `python3 -m unittest discover -v` | `pytest -v` |
| **输出格式** | 传统 xUnit 风格 | 更现代、彩色输出 |
| **断言方式** | `self.assertEqual()` | `assert` 关键字 |
| **参数化测试** | 复杂（需要 `@parameterized`） | 简单（`@pytest.mark.parametrize`） |
| ** fixtures** | `setUp`/`tearDown` | `@pytest.fixture` |
| **插件生态** | 有限 | 丰富（coverage、mock 等） |
| **学习曲线** | 较陡（面向对象） | 平缓（函数式） |

**推荐使用场景**：

- **unittest**：Python 标准库，无需额外安装，适合简单项目
- **pytest**：功能强大，插件丰富，适合中大型项目

---

## 7. 任务四：扩展测试（进阶）

> **目标**：为 `JsonLoanRepository` 的其他方法添加更多测试用例。

### 7.1 切换到 Build 模式

确保当前处于 Build 模式。

### 7.2 生成扩展测试

输入以下 prompt：

```
@tests/test_json_loan_repository.py @infrastructure/json_loan_repository.py

为 JsonLoanRepository 的以下方法添加测试：
1. test_get_loans_by_patron_id：验证按 patron_id 查询返回正确的 loan 列表
2. test_update_loan：验证更新 loan 后数据正确保存（检查 save_loans_called）
每个测试方法要有清晰的 Arrange-Act-Assert 注释
```

### 7.3 理解生成的测试代码

#### 7.3.1 test_get_loans_by_patron_id

```python
def test_get_loans_by_patron_id(self):
    # Arrange - 准备测试数据
    patron_id = 1
    
    # Act - 执行被测试的方法
    loans = self.repository.get_loans_by_patron_id(patron_id)
    
    # Assert - 验证结果
    self.assertEqual(len(loans), 1)
    self.assertEqual(loans[0].patron_id, patron_id)
```

#### 7.3.2 test_update_loan

```python
def test_update_loan(self):
    # Arrange - 准备测试数据
    original_due_date = self.test_loan.due_date
    
    # Act - 执行被测试的方法
    from datetime import timedelta
    self.test_loan.due_date = self.test_loan.due_date + timedelta(days=7)
    self.repository.update_loan(self.test_loan)
    
    # Assert - 验证结果
    self.assertTrue(self.dummy_data.save_loans_called)
    updated_loan = self.repository.get_loan(1)
    self.assertEqual(updated_loan.due_date, self.test_loan.due_date)
```

### 7.4 运行完整测试套件

确保当前在 library 目录下：

```bash
cd AccelerateDevGHCopilot/library
pytest tests/test_json_loan_repository.py -v
```

**预期输出**：

```
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loan_found PASSED
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loan_not_found PASSED
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loans_by_patron_id PASSED
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_update_loan PASSED

4 passed in 0.03s
```

### 7.5 （可选）添加边界条件测试

```
@tests/test_json_loan_repository.py

添加以下边界条件测试：
1. test_get_loans_by_patron_id_no_loans：查询不存在的 patron_id 返回空列表
2. test_update_loan_not_found：更新不存在的 loan 抛出异常或返回错误
```

---

## 8. 实验总结

### 8.1 关键收获

1. **Plan → Build 模式切换**：
   - Plan 模式：分析现有代码、理解架构、制定方案
   - Build 模式：生成代码、创建文件、执行修改
   - 两种模式配合使用，效率最高

2. **DummyJsonData 设计思路**：
   - 隔离外部依赖（不操作真实文件）
   - 内存存储测试数据
   - 状态追踪验证方法调用

3. **unittest 和 pytest 对比**：
   - unittest：标准库，无需安装，适合简单项目
   - pytest：功能强大，输出友好，推荐用于新项目

4. **测试命名规范**：
   - 测试类：`Test` + 被测类名
   - 测试方法：`test_` + 场景描述（`found`/`not_found`）

### 8.2 OpenCode 效率提升

| 传统方式 | OpenCode 方式 |
|---|---|
| 手动创建文件 → 编写 import → 写 setUp → 写测试方法 | 一个 prompt 生成完整测试文件 |
| 查阅文档了解 mock 用法 | `@explore` 自动分析现有模式 |
| 切换终端运行测试 | `!pytest` 直接在 OpenCode 内运行 |
| 手动修改多个测试方法 | 描述需求自动扩展测试 |

### 8.3 下一步

- 尝试为其他 Repository 类创建测试（如 `JsonPatronRepository`）
- 学习使用 `pytest` 的参数化测试（`@pytest.mark.parametrize`）
- 了解测试覆盖率工具（`pytest-cov`）

---

## 9. 附录：快速参考

### 9.1 OpenCode 模式切换

```
/plan   # 切换到 Plan 模式（分析、探索）
/build  # 切换到 Build 模式（生成、修改）
```

### 9.2 常用 Prompt 模板

**分析测试架构**：
```
@explore 分析 tests/ 目录的测试架构：
1. 使用了哪些测试框架和 mock 工具？
2. 测试类的组织方式
3. 如何扩展测试到新模块？
```

**创建测试文件**：
```
@参考文件路径 @被测文件路径

参考现有测试风格，创建 test_xxx.py，包含：
1. import 路径设置
2. Dummy 辅助类
3. setUp 方法
4. test_* 方法
```

**扩展测试**：
```
@测试文件 @被测文件

为以下方法添加测试：
1. method_one：场景描述
2. method_two：场景描述
使用 Arrange-Act-Assert 注释
```

### 9.3 运行测试命令

首先切换到 library 目录：

```bash
cd AccelerateDevGHCopilot/library
```

然后运行：

```bash
# unittest 方式
python3 -m unittest discover tests -v
python3 -m unittest tests.test_json_loan_repository -v

# pytest 方式
pytest tests -v
pytest tests/test_json_loan_repository.py -v
pytest tests -v --tb=short  # 简短错误输出
```

### 9.4 常见问题排查

**问题 1：import 错误（ModuleNotFoundError）**

```python
# 解决方法：添加 sys.path 设置
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parent.parent))
```

**问题 2：测试未被发现**

- 确保文件名以 `test_` 开头
- 确保测试类继承 `unittest.TestCase`
- 确保测试方法以 `test_` 开头

**问题 3：pytest 未找到**

```bash
pip3 install pytest
```

---

**实验完成！** 🎉

你现在掌握了使用 OpenCode 开发单元测试的完整流程：Plan 模式分析 → Build 模式生成 → 运行验证 → 扩展测试。
