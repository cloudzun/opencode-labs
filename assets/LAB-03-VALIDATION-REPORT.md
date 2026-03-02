# LAB-03 实验手册验证报告

## 1. 实际项目结构（与手册对比）

### 手册描述的项目结构
```
library/
├── console/
│   ├── common_actions.py   ← 添加枚举值
│   ├── console_app.py      ← 主要修改（菜单、处理、方法）
│   └── main.py
├── infrastructure/
│   ├── Json/
│   │   ├── Books.json      ← 查询数据源
│   │   ├── BookItems.json  ← 查询数据源
│   │   └── Loans.json      ← 查询数据源
│   └── json_data.py
└── tests/
```

### 实际项目结构
```
AccelerateDevGHCopilot/library/
├── application_core/           # 手册未详细说明
│   ├── entities/              # 实体类 (book.py, loan.py, patron.py 等)
│   ├── enums/                 # 枚举类型
│   ├── interfaces/            # 接口定义
│   └── services/              # 服务层 (loan_service.py, patron_service.py)
├── console/
│   ├── common_actions.py
│   ├── console_app.py
│   ├── console_state.py
│   └── main.py                # main.py 实际在 console_app.py 文件末尾，非独立文件
├── infrastructure/
│   ├── Json/
│   │   ├── Authors.json
│   │   ├── Books.json
│   │   ├── BookItems.json
│   │   ├── Loans.json
│   │   └── Patrons.json
│   ├── json_data.py
│   ├── json_loan_repository.py
│   └── json_patron_repository.py
└── tests/
    ├── test_loan_service.py
    └── test_patron_service.py
```

### 差异说明
1. **main.py 位置**：手册提到 `console/main.py`，但实际项目中 `main()` 函数定义在 `console/console_app.py` 文件末尾（第 257-268 行）
2. **额外文件**：实际项目包含更多文件，如 `console_state.py`、`json_loan_repository.py`、`json_patron_repository.py` 等
3. **application_core 目录**：手册未详细说明此目录结构，但它是项目的核心业务逻辑层

---

## 2. 测试结果

### 初始测试（修改前）
```
$ python3 -m unittest discover tests
....
----------------------------------------------------------------------
Ran 4 tests in 0.002s

OK
```

### 修改后测试
```
$ python3 -m unittest discover tests
....
----------------------------------------------------------------------
Ran 4 tests in 0.002s

OK
```

**结论**：所有 4 个测试通过，修改未破坏现有功能。

---

## 3. 手册中发现的问题

### 3.1 步骤描述问题

#### 问题 1：main.py 文件位置不准确
- **手册描述**：`console/main.py` 是独立文件
- **实际情况**：`main()` 函数定义在 `console/console_app.py` 文件末尾
- **建议改进**：手册应明确说明 `main()` 函数在 `console_app.py` 中

#### 问题 2：ConsoleApp 构造函数参数不完整
- **手册描述**：未提及需要传递 `patron_repository` 和 `loan_repository` 参数
- **实际情况**：ConsoleApp 的 `__init__` 方法需要 4 个参数：
  ```python
  def __init__(
      self,
      loan_service: ILoanService,
      patron_service: IPatronService,
      patron_repository: IPatronRepository,
      loan_repository: ILoanRepository,
  )
  ```
- **建议改进**：在实现 search_books 方法时，需要同时修改 ConsoleApp 构造函数和 main 函数

#### 问题 3：数据访问方式不清晰
- **手册描述**：使用 `JsonBookRepository` 和 `JsonBookItemRepository`
- **实际情况**：项目中只有 `JsonData` 类统一管理所有 JSON 数据，没有单独的 `JsonBookRepository`
- **建议改进**：手册应说明通过 `JsonData` 的 `books`、`book_items`、`loans` 属性访问数据

#### 问题 4：输出格式描述不准确
- **手册描述**：
  - 可借：`"{book.title} is available for loan"`
  - 已借出：`"{book.title} is on loan to another patron. The return due date is {loan.due_date}"`
- **实际情况**：`loan.due_date` 是 datetime 对象，需要格式化为字符串
- **建议改进**：手册应提示需要格式化日期，如 `loan.due_date.strftime("%Y-%m-%d")`

### 3.2 Prompt 示例改进建议

#### 改进 1：Step 8 的 prompt 需要调整
原 prompt 提到使用 `JsonBookRepository` 和 `JsonBookItemRepository`，但项目中不存在这些类。

**建议修改为**：
```
@console/console_app.py @infrastructure/json_data.py
@infrastructure/Json/Books.json @infrastructure/Json/BookItems.json @infrastructure/Json/Loans.json

请实现 search_books 方法，步骤如下：

1. 提示用户输入书名："Enter a book title to search for: "
2. 使用 self._json_data.books 加载书籍数据
3. 在 books 中查找匹配的书籍（不区分大小写，使用 title 字段）
4. 如果找到书籍，使用 self._json_data.book_items 找到对应的物理副本
5. 使用 self._json_data.loans 检查是否有未归还的借阅记录
6. 判断逻辑：
   - 如果所有副本都已借出（return_date 为 null）：显示已借出状态
   - 否则：显示可借阅状态
7. 输出格式：
   - 可借："{book.title} is available for loan"
   - 已借出："{book.title} is on loan to another patron. The return due date is {due_date}"
   （注意：due_date 需要格式化为 "YYYY-MM-DD" 格式）

注意：需要在 ConsoleApp 的 __init__ 中添加 json_data 参数，并在 main 函数中传递。

直接修改 console_app.py 文件，添加 search_books 方法。
```

#### 改进 2：缺少 ConsoleApp 构造函数修改的明确指示
手册 Step 7 只提到"检查 patron_details 方法中的 options 定义"，但没有明确说明需要修改 ConsoleApp 构造函数和 main 函数。

**建议添加**：
```
### 6.6 更新 ConsoleApp 构造函数和 main 函数

**Step 8.5:** 由于 search_books 方法需要访问 JSON 数据，需要更新 ConsoleApp 的构造函数：

```
@console/console_app.py
在 ConsoleApp 的 __init__ 方法中添加 json_data 参数，并存储为 self._json_data。
同时更新 main 函数，在创建 ConsoleApp 实例时传递 json_data 参数。
```
```

---

## 4. 实际实现 search_books 的代码（参考答案）

### 4.1 common_actions.py 修改
```python
from enum import Flag, auto

class CommonActions(Flag):
    REPEAT = 0
    SELECT = auto()
    QUIT = auto()
    SEARCH_PATRONS = auto()
    SEARCH_BOOKS = auto()  # 新增
    RENEW_PATRON_MEMBERSHIP = auto()
    RETURN_LOANED_BOOK = auto()
    EXTEND_LOANED_BOOK = auto()
```

### 4.2 console_app.py 修改

#### write_input_options 方法
```python
def write_input_options(self, options):
    print("Input Options:")
    if options & CommonActions.RETURN_LOANED_BOOK:
        print(' - "r" to mark as returned')
    if options & CommonActions.EXTEND_LOANED_BOOK:
        print(' - "e" to extend the book loan')
    if options & CommonActions.RENEW_PATRON_MEMBERSHIP:
        print(' - "m" to extend patron\'s membership')
    if options & CommonActions.SEARCH_PATRONS:
        print(' - "s" for new search')
    if options & CommonActions.SEARCH_BOOKS:  # 新增
        print(' - "b" to search book availability')
    if options & CommonActions.QUIT:
        print(' - "q" to quit')
    if options & CommonActions.SELECT:
        print(" - type a number to select a list item.")
```

#### patron_details 方法
```python
def patron_details(self) -> ConsoleState:
    patron = self.selected_patron_details
    print(f"\nName: {patron.name}")
    print(f"Membership Expiration: {patron.membership_end}")
    loans = self._loan_repository.get_loans_by_patron_id(patron.id)
    print("\nBook Loans History:")

    valid_loans = self._print_loans(loans)

    if valid_loans:
        options = (
            CommonActions.RENEW_PATRON_MEMBERSHIP
            | CommonActions.SEARCH_PATRONS
            | CommonActions.SEARCH_BOOKS  # 新增
            | CommonActions.QUIT
            | CommonActions.SELECT
        )
        selection = self._get_patron_details_input(options)
        return self._handle_patron_details_selection(selection, patron, valid_loans)
    else:
        print("No valid loans for this patron.")
        options = (
            CommonActions.SEARCH_PATRONS
            | CommonActions.SEARCH_BOOKS  # 新增
            | CommonActions.QUIT
        )
        selection = self._get_patron_details_input(options)
        return self._handle_no_loans_selection(selection)
```

#### _handle_patron_details_selection 方法
```python
def _handle_patron_details_selection(self, selection, patron, valid_loans):
    if selection == 'q':
        return ConsoleState.QUIT
    elif selection == 's':
        return ConsoleState.PATRON_SEARCH
    elif selection == 'b':  # 新增
        return self.search_books()
    elif selection == 'm':
        status = self._patron_service.renew_membership(patron.id)
        print(status)
        self.selected_patron_details = self._patron_repository.get_patron(patron.id)
        return ConsoleState.PATRON_DETAILS
    elif selection.isdigit():
        idx = int(selection)
        if 1 <= idx <= len(valid_loans):
            self.selected_loan_details = valid_loans[idx - 1][1]
            return ConsoleState.LOAN_DETAILS
        print("Invalid selection. Please enter a number shown in the list above.")
        return ConsoleState.PATRON_DETAILS
    else:
        print("Invalid input. Please enter a number, 'm', 's', 'b', or 'q'.")
        return ConsoleState.PATRON_DETAILS
```

#### search_books 方法（新增）
```python
def search_books(self) -> ConsoleState:
    title = input("Enter a book title to search for: ").strip()
    if not title:
        print("No input provided.")
        return ConsoleState.PATRON_DETAILS

    books = self._json_data.books
    book_items = self._json_data.book_items
    loans = self._json_data.loans

    matching_books = [b for b in books if title.lower() in b.title.lower()]

    if not matching_books:
        print(f"No books found with title containing '{title}'.")
        return ConsoleState.PATRON_DETAILS

    for book in matching_books:
        book_item_ids = [bi.id for bi in book_items if bi.book_id == book.id]
        active_loans = [
            l
            for l in loans
            if l.book_item_id in book_item_ids and l.return_date is None
        ]

        if active_loans:
            due_date = active_loans[0].due_date.strftime("%Y-%m-%d")
            print(
                f"{book.title} is on loan to another patron. The return due date is {due_date}"
            )
        else:
            print(f"{book.title} is available for loan")

    return ConsoleState.PATRON_DETAILS
```

#### ConsoleApp 构造函数修改
```python
def __init__(
    self,
    loan_service: ILoanService,
    patron_service: IPatronService,
    patron_repository: IPatronRepository,
    loan_repository: ILoanRepository,
    json_data: JsonData = None,  # 新增参数
):
    self._current_state: ConsoleState = ConsoleState.PATRON_SEARCH
    self.matching_patrons = []
    self.selected_patron_details = None
    self.selected_loan_details = None
    self._patron_repository = patron_repository
    self._loan_repository = loan_repository
    self._loan_service = loan_service
    self._patron_service = patron_service
    self._json_data = json_data  # 新增
```

#### main 函数修改
```python
def main():
    json_data = JsonData()
    patron_repo = JsonPatronRepository(json_data)
    loan_repo = JsonLoanRepository(json_data)
    loan_service = LoanService(loan_repo)
    patron_service = PatronService(patron_repo)

    app = ConsoleApp(
        loan_service=loan_service,
        patron_service=patron_service,
        patron_repository=patron_repo,      # 新增
        loan_repository=loan_repo,          # 新增
        json_data=json_data,                # 新增
    )
    app.run()
```

---

## 5. 总结

### 实验手册整体评价
- **优点**：
  - 提供了清晰的 Plan→Build→验证→Commit 工作流
  - Prompt 模板设计合理，引导用户分步骤实现功能
  - 测试验证环节确保代码质量

- **需要改进**：
  - 项目结构描述与实际有出入
  - 部分类名和 API 与实际代码不符
  - 缺少对 ConsoleApp 构造函数修改的说明
  - 日期格式化等细节需要提示

### 建议
1. 更新手册中的项目结构图，反映实际文件布局
2. 修正 prompt 示例中的类名引用
3. 添加 ConsoleApp 构造函数修改的明确步骤
4. 在输出格式说明中提示日期格式化需求

---

**验证完成时间**：2026-03-01
**验证者**：实验手册验证员
