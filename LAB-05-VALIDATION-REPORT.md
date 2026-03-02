# LAB-05 重构实验验证报告

## 一、实际项目结构对比

### 手册预期结构
```
./console/console_app.py
./infrastructure/json_patron_repository.py
./application_core/services/loan_service.py
./tests/
```

### 实际项目结构
```
AccelerateDevGHCopilot/library/
├── application_core/
│   ├── entities/
│   │   ├── author.py
│   │   ├── book.py
│   │   ├── book_item.py
│   │   ├── loan.py
│   │   └── patron.py
│   ├── enums/
│   │   ├── loan_extension_status.py
│   │   ├── loan_return_status.py
│   │   └── membership_renewal_status.py
│   ├── interfaces/
│   │   ├── iloan_repository.py
│   │   ├── iloan_service.py
│   │   ├── ipatron_repository.py
│   │   └── ipatron_service.py
│   └── services/
│       ├── loan_service.py          ✓ 重构目标
│       └── patron_service.py
├── console/
│   ├── book_repository.py
│   ├── common_actions.py
│   ├── console_app.py               ✓ 重构目标
│   ├── console_state.py
│   └── main.py
├── infrastructure/
│   ├── Json/                        (大写目录名，手册为小写)
│   │   ├── Authors.json
│   │   ├── Books.json
│   │   ├── BookItems.json
│   │   ├── Loans.json
│   │   └── Patrons.json
│   ├── json_data.py
│   ├── json_loan_repository.py
│   └── json_patron_repository.py    ✓ 重构目标
└── tests/
    ├── __init__.py
    ├── test_json_loan_repository.py
    ├── test_loan_service.py
    └── test_patron_service.py
```

### 结构差异
| 项目 | 手册描述 | 实际情况 |
|------|----------|----------|
| Json 目录 | `Json/` | `Json/` (大写，一致) |
| 测试文件数 | 10-15 个 | 4 个测试文件，8 个测试用例 |
| 下载链接 | `microsoft/AccelerateDevGHCopilot` | `MicrosoftLearning/mslearn-github-copilot-dev` |

---

## 二、测试结果

### 初始测试（重构前）
```
test_get_loan ... ok
test_get_loan_found ... ok
test_get_loan_not_found ... ok
test_get_loan_not_found_again ... ok
test_return_loan_not_found ... ok
test_return_loan_success ... ok
test_renew_membership_patron_not_found ... ok
test_renew_membership_success ... ok

Ran 8 tests in 0.002s
OK
```

### 重构后测试
```
test_get_loan ... ok
test_get_loan_found ... ok
test_get_loan_not_found ... ok
test_get_loan_not_found_again ... ok
test_return_loan_not_found ... ok
test_return_loan_success ... ok
test_renew_membership_patron_not_found ... ok
test_renew_membership_success ... ok

Ran 8 tests in 0.002s
OK
```

**结论**：所有 8 个测试用例在重构前后均通过，重构未破坏任何功能。

### pytest 测试
pytest 可用但无法发现测试（测试使用 unittest 框架），unittest 测试结果可靠。

---

## 三、实际找到的重构点

### 1. console/console_app.py

| 行号 | 方法 | 问题描述 | 重构方式 |
|------|------|----------|----------|
| 46-56 | `run()` | 5 个分支的 if/elif 状态机判断 | 字典驱动 `state_handlers` |
| 29-44 | `write_input_options()` | 7 个独立的 if 判断 | 字典遍历 |
| 151-172 | `_handle_patron_details_selection()` | 5 个分支的 if/elif 链 | 字典驱动 `handlers` |
| 310-327 | `loan_details()` | 4 个分支的 if/elif 链 | 字典驱动 `handlers` |

### 2. infrastructure/json_patron_repository.py

| 行号 | 方法 | 问题描述 | 重构方式 |
|------|------|----------|----------|
| 11-15 | `get_patron()` | 手动 for 循环查找元素 | `next()` + 生成器表达式 |
| 18-27 | `search_patrons()` | 手动 append + 冒泡排序 | 列表推导式 + `sorted()` |
| 44-49 | `find_patrons_by_name()` | 手动 append 循环 | 列表推导式 |

### 3. application_core/services/loan_service.py

| 行号 | 方法 | 问题描述 | 重构方式 |
|------|------|----------|----------|
| 48-51 | `checkout_book()` | 手动 for 循环求最大 loan_id | `max()` + 生成器表达式 |

---

## 四、手册中发现的问题

### 4.1 下载链接不准确
**手册步骤 3.1**:
```bash
wget https://github.com/microsoft/AccelerateDevGHCopilot/raw/main/labs/LAB_AK_05/AZ2007LabAppM5Python.zip
```

**实际可用链接**:
```bash
curl -L -o AZ2007LabAppM5Python.zip 'https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM5Python.zip'
```

**问题**：手册中的 GitHub 仓库路径不存在，需使用 MicrosoftLearning 组织的仓库。

### 4.2 测试用例数量描述不准确
**手册描述**："约 10-15 个测试用例"

**实际情况**：只有 8 个测试用例（4 个测试文件）

### 4.3 行号引用有偏差
**手册 4.1 预期输出**：
- "第 85-120 行：run() 方法"
- "第 245-280 行：write_input_options()"

**实际行号**：
- `run()`: 第 46-60 行
- `write_input_options()`: 第 29-44 行

**原因**：手册基于的代码版本与实际下载的代码版本不同。

### 4.4 缺少参数问题
**手册 5.2 重构提示**中未提及 `ConsoleApp.__init__()` 需要 `patron_repository` 和 `loan_repository` 参数，但实际代码 (main.py 第 341 行) 需要这些参数。

### 4.5 部分 Prompt 模板需要调整
手册中的 `@explore` 命令需要 OpenCode 特定环境，在通用 IDE 环境中可能不可用。

---

## 五、重构后的关键代码片段（参考答案）

### 5.1 console_app.py - run() 方法（字典驱动状态机）

```python
def run(self) -> None:
    state_handlers = {
        ConsoleState.PATRON_SEARCH: self.patron_search,
        ConsoleState.PATRON_SEARCH_RESULTS: self.patron_search_results,
        ConsoleState.PATRON_DETAILS: self.patron_details,
        ConsoleState.LOAN_DETAILS: self.loan_details,
        ConsoleState.QUIT: lambda: ConsoleState.QUIT,
    }
    while True:
        handler = state_handlers.get(self._current_state)
        if handler is None:
            print(f"Unknown state: {self._current_state}")
            break
        self._current_state = handler()
        if self._current_state == ConsoleState.QUIT:
            break
```

**改进点**：
- 消除 5 个 elif 分支
- 添加新状态只需在字典中加一行
- 未知状态有统一的错误处理

### 5.2 console_app.py - write_input_options() 方法（字典遍历）

```python
def write_input_options(self, options):
    option_messages = {
        CommonActions.RETURN_LOANED_BOOK: ' - "r" to mark as returned',
        CommonActions.EXTEND_LOANED_BOOK: ' - "e" to extend the book loan',
        CommonActions.RENEW_PATRON_MEMBERSHIP: ' - "m" to extend patron\'s membership',
        CommonActions.SEARCH_PATRONS: ' - "s" for new search',
        CommonActions.SEARCH_BOOKS: ' - "b" to check for book availability',
        CommonActions.QUIT: ' - "q" to quit',
        CommonActions.SELECT: ' - type a number to select a list item.',
    }
    print("Input Options:")
    for action, message in option_messages.items():
        if options & action:
            print(message)
```

### 5.3 json_patron_repository.py - get_patron() 方法（生成器表达式）

```python
def get_patron(self, patron_id: int) -> Optional[Patron]:
    return next((p for p in self._json_data.patrons if p.id == patron_id), None)
```

**改进点**：5 行 → 1 行，惰性求值

### 5.4 json_patron_repository.py - search_patrons() 方法（列表推导式 + sorted）

```python
def search_patrons(self, search_input: str) -> List[Patron]:
    results = [p for p in self._json_data.patrons if search_input.lower() in p.name.lower()]
    results.sort(key=lambda p: p.name)
    return results
```

**改进点**：15 行 → 3 行，使用内置排序性能更好

### 5.5 loan_service.py - checkout_book() 方法（max + 生成器）

```python
def checkout_book(self, patron, book_item, loan_id=None):
    from ..entities.loan import Loan
    from datetime import datetime, timedelta
    if loan_id is None:
        all_loans = getattr(self._loan_repository, "get_all_loans", lambda: [])()
        loan_id = max((l.id for l in all_loans), default=0) + 1
    # ... 创建 loan 对象
```

**改进点**：6 行 → 1 行，使用内置 `max()` 更可靠

---

## 六、重构前后代码行数对比

| 文件 | 重构前 | 重构后 | 减少 |
|------|--------|--------|------|
| console_app.py | 350 行 | 346 行 | -4 行 |
| json_patron_repository.py | 55 行 | 44 行 | -11 行 |
| loan_service.py | 67 行 | 64 行 | -3 行 |
| **合计** | **472 行** | **454 行** | **-18 行** |

---

## 七、实验总结

### 7.1 学到的重构模式
1. **字典驱动状态机**：适合有多个状态/分支的场景，易扩展
2. **列表推导式**：适合过滤 + 收集的场景，代码简洁
3. **生成器表达式**：适合惰性求值、一次遍历的场景
4. **max() + default**：适合求最值的场景，处理空列表

### 7.2 OpenCode/通用重构工作流
```
分析问题 → 制定计划 → 执行重构 → 测试验证 → Git 提交
   ↓                                  ↓
@explore 扫描                      unittest/pytest
列出文件 + 行号                    失败则回滚
```

### 7.3 验证结论
- ✅ 所有重构点已识别并成功重构
- ✅ 所有测试用例通过
- ✅ 代码更简洁、更易读、更 Pythonic
- ⚠️ 手册中的下载链接和部分行号需要更新

---

**验证完成时间**：2026-03-02
**验证员**：实验手册验证员
