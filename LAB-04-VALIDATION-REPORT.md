# LAB-04 实验手册验证报告

## 1. 实际项目结构（与手册对比）

### 手册描述的结构
```
library/
├── application_core/      # 核心业务实体
├── infrastructure/        # 基础设施层（JsonLoanRepository 等）
├── services/              # 服务层
├── tests/                 # 测试目录
└── requirements.txt
```

### 实际项目结构
```
AccelerateDevGHCopilot/library/
├── application_core/
│   ├── entities/          # 实体类（Author, Book, BookItem, Loan, Patron）
│   ├── enums/             # 枚举类型
│   ├── interfaces/        # 接口定义
│   └── services/          # 服务层（LoanService, PatronService）
├── console/               # 控制台应用
├── infrastructure/
│   ├── Json/              # JSON 数据文件目录
│   ├── json_data.py
│   ├── json_loan_repository.py
│   └── json_patron_repository.py
├── tests/
│   ├── __init__.py
│   ├── test_loan_service.py
│   ├── test_patron_service.py
│   └── test_json_loan_repository.py（新增）
└── readme.md
```

### 差异说明
- 实际结构中 `services` 目录位于 `application_core/services/` 而非独立的 `services/`
- 实际项目包含 `console/` 目录用于控制台应用
- 实际项目没有 `requirements.txt` 文件

---

## 2. 初始测试结果

### 运行命令
```bash
cd AccelerateDevGHCopilot/library
python3 -m unittest discover tests -v
```

### 初始测试输出（4 个测试）
```
test_return_loan_not_found (test_loan_service.TestLoanService.test_return_loan_not_found) ... ok
test_return_loan_success (test_loan_service.TestLoanService.test_return_loan_success) ... ok
test_renew_membership_patron_not_found (test_patron_service.PatronServiceTest.test_renew_membership_patron_not_found) ... ok
test_renew_membership_success (test_patron_service.PatronServiceTest.test_renew_membership_success) ... ok

----------------------------------------------------------------------
Ran 4 tests in 0.002s

OK
```

---

## 3. 新增测试后的测试结果

### unittest 运行结果（6 个测试）
```
test_get_loan_found (test_json_loan_repository.TestJsonLoanRepository.test_get_loan_found) ... ok
test_get_loan_not_found (test_json_loan_repository.TestJsonLoanRepository.test_get_loan_not_found) ... ok
test_return_loan_not_found (test_loan_service.TestLoanService.test_return_loan_not_found) ... ok
test_return_loan_success (test_loan_service.TestLoanService.test_return_loan_success) ... ok
test_renew_membership_patron_not_found (test_patron_service.PatronServiceTest.test_renew_membership_patron_not_found) ... ok
test_renew_membership_success (test_patron_service.PatronServiceTest.test_renew_membership_success) ... ok

----------------------------------------------------------------------
Ran 6 tests in 0.002s

OK
```

### pytest 运行结果（6 个测试）
```
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loan_found PASSED [ 16%]
tests/test_json_loan_repository.py::TestJsonLoanRepository::test_get_loan_not_found PASSED [ 33%]
tests/test_loan_service.py::TestLoanService::test_return_loan_not_found PASSED [ 50%]
tests/test_loan_service.py::TestLoanService::test_return_loan_success PASSED [ 66%]
tests/test_patron_service.py::PatronServiceTest::test_renew_membership_patron_not_found PASSED [ 83%]
tests/test_patron_service.py::PatronServiceTest::test_renew_membership_success PASSED [100%]

============================== 6 passed in 0.04s ===============================
```

---

## 4. 手册中发现的问题

### 4.1 步骤问题

| 问题编号 | 手册描述 | 实际情况 | 建议修正 |
|---------|---------|---------|---------|
| P1 | 3.1 节下载 URL 为 `https://github.com/microsoft/AccelerateDevGHCopilot/raw/main/M4/LAB_AK_04/AZ2007LabAppM4Python.zip` | 该 URL 返回 404 错误 | 使用实验者提供的 URL：`https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM4Python.zip` |
| P2 | 3.3 节预期输出"现有测试用例全部通过（约 5-8 个测试）" | 实际只有 4 个测试 | 更新为"4 个测试" |
| P3 | 6.1 节运行 unittest 命令在 `tests/` 目录 | 需要在 `library/` 目录运行 | 添加说明：`cd AccelerateDevGHCopilot/library` |

### 4.2 Prompt 问题

| 问题编号 | 手册描述 | 实际情况 | 建议修正 |
|---------|---------|---------|---------|
| PP1 | 4.2 节使用 `@explore` 命令 | OpenCode 中可能没有 `@explore` 内置命令 | 说明这需要使用 Task 工具或 Explore Agent |
| PP2 | 4.3 节使用 `@tests/test_loan_service.py` 语法 | OpenCode 中文件引用语法可能不同 | 根据实际 OpenCode 版本调整语法 |

### 4.3 技术细节问题

| 问题编号 | 手册描述 | 实际情况 | 建议修正 |
|---------|---------|---------|---------|
| PT1 | 5.3.3 节 `setUp` 方法中直接设置 `self.repository.loans = [self.test_loan]` | `JsonLoanRepository` 没有 `loans` 属性，应通过 `DummyJsonData` 设置 | 修正为 `self.dummy_data.loans = [self.test_loan]` |
| PT2 | 手册假设项目根目录包含 `requirements.txt` | 实际项目没有此文件 | 删除相关要求或说明项目无依赖 |
| PT3 | 6.2 节 pytest 安装 | 在某些系统（如 Debian/Ubuntu）需要 `--break-system-packages` 参数或使用虚拟环境 | 添加环境注意事项说明 |

### 4.4 代码类型问题

现有代码存在类型注解不匹配问题（不影响测试运行）：
- `json_data.py:45` - `acquisition_date` 可能为 `None` 但 `BookItem` 要求 `datetime`
- `json_data.py:48` - `membership_end/membership_start` 可能为 `None` 但 `Patron` 要求 `datetime`
- `json_data.py:51` - `loan_date/due_date` 可能为 `None` 但 `Loan` 要求 `datetime`

---

## 5. 实际可运行的 test_json_loan_repository.py 完整代码

```python
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parent.parent))

import unittest
from datetime import datetime, timedelta
from infrastructure.json_loan_repository import JsonLoanRepository
from infrastructure.json_data import JsonData
from application_core.entities.patron import Patron
from application_core.entities.book_item import BookItem
from application_core.entities.loan import Loan


class DummyJsonData:
    def __init__(self):
        self.loans = []
        self.save_loans_called = False

    def save_loans(self, loans):
        self.loans = loans
        self.save_loans_called = True

    def load_data(self):
        return self.loans


class TestJsonLoanRepository(unittest.TestCase):
    def setUp(self):
        self.test_patron = Patron(
            id=1,
            name="Test Patron",
            membership_end=datetime.now() + timedelta(days=30),
            membership_start=datetime.now() - timedelta(days=365)
        )

        self.test_book_item = BookItem(
            id=1,
            book_id=1,
            acquisition_date=datetime.now() - timedelta(days=100)
        )

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

        self.dummy_data = DummyJsonData()
        self.repository = JsonLoanRepository(self.dummy_data)
        self.dummy_data.loans = [self.test_loan]

    def test_get_loan_found(self):
        result = self.repository.get_loan(1)
        self.assertIsNotNone(result)
        self.assertEqual(result.id, 1)
        self.assertEqual(result.patron_id, 1)
        self.assertEqual(result.book_item_id, 1)

    def test_get_loan_not_found(self):
        result = self.repository.get_loan(999)
        self.assertIsNone(result)


if __name__ == "__main__":
    unittest.main()
```

### 关键修正点

1. **sys.path 设置**：正确添加项目根目录到导入路径
2. **DummyJsonData 实现**：简化实现，只存储 loans 列表和调用标志
3. **setUp 方法修正**：通过 `self.dummy_data.loans` 设置测试数据，而非 `self.repository.loans`
4. **测试断言**：使用 `assertIsNone` 和 `assertIsNotNone` 进行空值验证

---

## 6. 验证结论

- ✅ 项目下载和解压成功
- ✅ 初始 4 个测试全部通过
- ✅ 新增 2 个测试全部通过
- ✅ unittest 和 pytest 两种方式均可运行
- ⚠️ 手册存在多处 URL、目录路径和技术细节不准确
- ⚠️ 现有代码存在类型注解问题（不影响功能）

**总体评价**：实验手册核心流程可行，但需要修正多处技术细节以确保学员能够顺利执行。

---

*验证日期：2026-03-01*
*验证环境：Python 3.12.3, pytest 9.0.2, Linux*
