# LAB-05：使用 OpenCode 重构改进代码质量

## 一、实验概述

### 学习目标
- 掌握使用 OpenCode 的 **Plan 模式** 分析代码质量问题
- 掌握使用 OpenCode 的 **Build 模式** 执行重构
- 学会用 Python 惯用写法改进代码：
  - 字典驱动替代长 if/elif 链
  - 列表推导式替代手动循环
  - 生成器表达式替代手动求最大值
- 养成"先分析 → 再重构 → 后验证"的工程习惯

### 预计时间
**40-50 分钟**

### 前置条件
- 已完成 LAB-01 至 LAB-04
- 熟悉 Python 基础语法（循环、列表、字典、函数）
- 已安装 OpenCode 并配置好 Python 环境

---

## 二、环境准备

### 2.1 验证 OpenCode 可用
```bash
opencode --version
```

### 2.2 验证 pytest/unittest 可用
```bash
python3 --version
python3 -c "import unittest; print('unittest OK')"
```

---

## 三、项目配置

### 3.1 下载并解压项目
```bash
# 下载图书馆管理系统
curl -L -o AZ2007LabAppM5Python.zip 'https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM5Python.zip'

# 解压
unzip AZ2007LabAppM5Python.zip

# 进入项目目录
cd AccelerateDevGHCopilot/library
```

### 3.2 查看项目结构
```bash
find . -name "*.py" | head -20
```

预期看到如下结构：
```
./console/console_app.py           # 控制台应用（重构重点）
./infrastructure/json_patron_repository.py  # 数据仓库（重构重点）
./application_core/services/loan_service.py # 贷款服务（重构重点）
./tests/                           # 测试目录
```

### 3.3 运行初始测试（建立基线）

> ⚠️ **注意**：确保当前目录为 `AccelerateDevGHCopilot/library`，再运行测试。

```bash
python3 -m unittest discover tests -v
```

> ✅ **验证点**：所有测试通过（8 个测试用例），确认重构前代码功能正常。

---

## 四、任务一：分析代码重构机会（Plan 模式）

### 4.1 用 @explore 全局扫描代码问题

启动 OpenCode，进入 **Plan 模式**：

```
@explore 分析整个项目，找出以下代码质量问题：
1. 使用反射（getattr）的地方
2. 超过 5 个分支的 if/elif 链
3. 可以用列表推导式替代的手动 for 循环
4. 可以用生成器表达式优化的地方
列出文件名和具体行号
```

**预期输出示例**：
```
发现以下重构机会：

1. console/console_app.py:
   - 第 85-120 行：run() 方法中有 6 个分支的 if/elif 状态判断
   - 第 245-280 行：write_input_options() 中有 5 个分支的 if/elif
   - 第 310-340 行：_handle_patron_details_selection() 中有 4 个分支的 if/elif

2. infrastructure/json_patron_repository.py:
   - 第 45-52 行：get_patron() 使用 for 循环查找，可用 next() + 生成器
   - 第 58-75 行：search_patrons() 手动 append + 冒泡排序
   - 第 80-90 行：find_patrons_by_name() 手动 append

3. application_core/services/loan_service.py:
   - 第 35-42 行：checkout_book() 手动 for 循环求最大 loan_id
```

### 4.2 理解重构目标

| 重构类型 | 重构前 | 重构后 | 好处 |
|---|---|---|---|
| 字典驱动 | `if x == 1: ... elif x == 2: ...` | `handlers[x]()` | 易扩展、易读 |
| 列表推导式 | `for x in xs: if cond: results.append(x)` | `[x for x in xs if cond]` | 简洁、Pythonic |
| 生成器表达式 | `for x in xs: if x.id == id: return x` | `next((x for x in xs if x.id == id), None)` | 惰性求值、简洁 |
| max() + 生成器 | 手动循环求最大值 | `max((x.id for x in xs), default=0)` | 一行代码 |

### 4.3 制定重构计划（输出到文件）

```
@console/console_app.py @infrastructure/json_patron_repository.py @application_core/services/loan_service.py
基于刚才的分析，制定重构计划：
1. 每个文件需要重构的具体方法
2. 重构方式（字典/列表推导式/生成器）
3. 预期效果
不要写代码，输出计划到 .opencode/plans/refactor-plan.md
```

> 📄 **查看生成的计划**：`.opencode/plans/refactor-plan.md`

---

## 五、任务二：重构 console_app.py（Build 模式）

切换到 **Build 模式**，执行第一个文件的重构。

### 5.1 重构目标

`console_app.py` 是控制台应用的核心，包含一个状态机。重构目标：
1. 用 `state_handlers` 字典替代 `run()` 方法中的 if/elif 状态判断
2. 用字典替代 `write_input_options()` 中的 if/elif 链
3. 用字典替代 `_handle_patron_details_selection()` 中的 if/elif 链

### 5.2 执行重构

```
@console/console_app.py
重构 ConsoleApp 类，完成以下改进：
1. 在 run() 方法中，用 state_handlers 字典替代 if/elif 状态判断：
   state_handlers = {
       ConsoleState.PATRON_SEARCH: self.patron_search,
       ConsoleState.PATRON_SEARCH_RESULTS: self.patron_search_results,
       ConsoleState.PATRON_DETAILS: self.patron_details,
       ConsoleState.LOAN_DETAILS: self.loan_details,
   }
2. 在 write_input_options() 中，用字典替代 if/elif 链
3. 在 _handle_patron_details_selection() 中，用字典替代 if/elif 链
4. 为未知状态添加明确的错误处理
保持所有现有功能不变，不要修改方法签名
```

### 5.3 重构前后对比

#### run() 方法重构对比

**重构前**（长 if/elif 链）：
```python
def run(self) -> None:
    while True:
        if self._current_state == ConsoleState.PATRON_SEARCH:
            next_state = self.patron_search()
        elif self._current_state == ConsoleState.PATRON_SEARCH_RESULTS:
            next_state = self.patron_search_results()
        elif self._current_state == ConsoleState.PATRON_DETAILS:
            next_state = self.patron_details()
        elif self._current_state == ConsoleState.LOAN_DETAILS:
            next_state = self.loan_details()
        elif self._current_state == ConsoleState.QUIT:
            print("Exiting application.")
            break
        else:
            print(f"Unknown state: {self._current_state}")
            break
        self._current_state = next_state
```

**重构后**（字典驱动）：
```python
def run(self) -> None:
    state_handlers = {
        ConsoleState.PATRON_SEARCH: self.patron_search,
        ConsoleState.PATRON_SEARCH_RESULTS: self.patron_search_results,
        ConsoleState.PATRON_DETAILS: self.patron_details,
        ConsoleState.LOAN_DETAILS: self.loan_details,
        ConsoleState.QUIT: lambda: ConsoleState.QUIT
    }
    while True:
        handler = state_handlers.get(self._current_state)
        if handler is None:
            print(f"Unknown state: {self._current_state}")
            break
        next_state = handler()
        if next_state == ConsoleState.QUIT:
            print("Exiting application.")
            break
        self._current_state = next_state
```

**改进点**：
- 消除 5 个 elif 分支
- 添加新状态只需在字典中加一行
- 未知状态有统一的错误处理

### 5.4 运行测试验证

```bash
!python3 -m unittest discover tests -v
```

> ✅ **验证点**：所有测试仍通过。如果失败，使用 `/undo` 回滚重构。

---

## 六、任务三：重构 json_patron_repository.py（Build 模式）

### 6.1 重构目标

`json_patron_repository.py` 负责读者数据的增删改查。重构目标：
1. `get_patron()`：用 `next()` + 生成器表达式替代 for 循环查找
2. `search_patrons()`：用列表推导式替代手动 append，用 `sorted()` 替代冒泡排序
3. `find_patrons_by_name()`：用列表推导式替代手动 append

### 6.2 执行重构

```
@infrastructure/json_patron_repository.py
重构 JsonPatronRepository 类，使用 Python 惯用写法：
1. get_patron()：用 next() + 生成器表达式替代 for 循环
   改为：return next((p for p in self._json_data.patrons if p.id == patron_id), None)
2. search_patrons()：用列表推导式替代手动 append，用 sorted() 替代冒泡排序
3. find_patrons_by_name()：用列表推导式替代手动 append
保持方法签名和返回值类型不变
```

### 6.3 重构前后对比

#### get_patron() 方法对比

**重构前**（手动循环）：
```python
def get_patron(self, patron_id: int) -> Patron | None:
    for patron in self._json_data.patrons:
        if patron.id == patron_id:
            return patron
    return None
```

**重构后**（生成器表达式）：
```python
def get_patron(self, patron_id: int) -> Patron | None:
    return next((p for p in self._json_data.patrons if p.id == patron_id), None)
```

**改进点**：
- 5 行 → 1 行
- 惰性求值，找到即返回

#### search_patrons() 方法对比

**重构前**（手动 append + 冒泡排序）：
```python
def search_patrons(self, search_input: str) -> list[Patron]:
    results = []
    for patron in self._json_data.patrons:
        if search_input.lower() in patron.name.lower():
            results.append(patron)
    
    # 冒泡排序
    n = len(results)
    for i in range(n):
        for j in range(0, n - i - 1):
            if results[j].name > results[j + 1].name:
                results[j], results[j + 1] = results[j + 1], results[j]
    
    return results
```

**重构后**（列表推导式 + sorted()）：
```python
def search_patrons(self, search_input: str) -> list[Patron]:
    results = [p for p in self._json_data.patrons if search_input.lower() in p.name.lower()]
    results.sort(key=lambda p: p.name)
    return results
```

**改进点**：
- 15 行 → 3 行
- 使用内置 `sorted()`/`.sort()`，性能更好
- 代码意图清晰

### 6.4 运行测试验证

```bash
!python3 -m unittest discover tests -v
```

> ✅ **验证点**：所有测试仍通过。

---

## 七、任务四：重构 loan_service.py（Build 模式）

### 7.1 重构目标

`loan_service.py` 中的 `checkout_book()` 方法需要生成新的 loan_id。重构目标：
- 用 `max()` + 生成器表达式替代手动 for 循环求最大值

### 7.2 执行重构

```
@application_core/services/loan_service.py
重构 checkout_book() 方法中的 loan_id 生成逻辑：
将手动 for 循环求最大值：
    max_id = 0
    for l in all_loans:
        if l.id > max_id:
            max_id = l.id
    loan_id = max_id + 1 if all_loans else 1
改为生成器表达式：
    loan_id = max((l.id for l in all_loans), default=0) + 1
```

### 7.3 重构前后对比

**重构前**（手动循环）：
```python
def checkout_book(self, patron_id: int, book_id: int, due_date: date) -> Loan:
    all_loans = self._loan_repository.get_all_loans()
    
    max_id = 0
    for l in all_loans:
        if l.id > max_id:
            max_id = l.id
    loan_id = max_id + 1 if all_loans else 1
    
    # ... 创建 loan 对象
```

**重构后**（max() + 生成器）：
```python
def checkout_book(self, patron_id: int, book_id: int, due_date: date) -> Loan:
    all_loans = self._loan_repository.get_all_loans()
    
    loan_id = max((l.id for l in all_loans), default=0) + 1
    
    # ... 创建 loan 对象
```

**改进点**：
- 6 行 → 1 行
- 使用内置 `max()`，更可靠
- `default=0` 参数可处理空列表情况，不需要额外的 `if all_loans` 判断

### 7.4 运行测试验证

```bash
!python3 -m unittest discover tests -v
```

> ✅ **验证点**：所有测试仍通过。

---

## 八、任务五：全量测试和代码审查

### 8.1 运行完整测试套件

```bash
# unittest 测试
!python3 -m unittest discover tests -v

# 如果有 pytest，也运行
!python3 -m pytest tests/ -v
```

> ✅ **验证点**：所有测试通过，重构没有破坏任何功能。

### 8.2 用 Plan 模式审查重构结果

切换到 **Plan 模式**：

```
@console/console_app.py @infrastructure/json_patron_repository.py @application_core/services/loan_service.py
审查刚才完成的重构，检查：
1. 是否还有遗漏的 if/elif 链可以优化？
2. 重构后的代码是否更易读？
3. 有没有引入新的问题？
```

**预期输出**：
```
重构审查结果：

✅ console_app.py：
   - state_handlers 字典已正确实现
   - write_input_options() 已优化
   - 无遗漏的 if/elif 链

✅ json_patron_repository.py：
   - 所有 for 循环已转换为列表推导式/生成器
   - 排序已使用内置 sorted()

✅ loan_service.py：
   - loan_id 生成已优化

建议：重构质量良好，代码更简洁易读。
```

### 8.3 Git 提交重构

```bash
# 查看变更
git status
git diff

# 提交
git add -A
git commit -m "refactor: 用字典驱动和生成器表达式改进代码质量

- console_app.py: 用 state_handlers 字典替代 if/elif 状态机
- json_patron_repository.py: 用列表推导式和 sorted() 简化代码
- loan_service.py: 用 max() + 生成器替代手动求最大值
- 所有测试通过"
```

---

## 九、实验总结

### 9.1 重构前后对比

| 文件 | 重构前代码行数 | 重构后代码行数 | 减少行数 |
|---|---|---|---|
| console_app.py | ~350 | ~320 | -30 |
| json_patron_repository.py | ~100 | ~70 | -30 |
| loan_service.py | ~60 | ~55 | -5 |

### 9.2 学到的重构模式

1. **字典驱动状态机**：适合有多个状态/分支的场景
2. **列表推导式**：适合过滤 + 收集的场景
3. **生成器表达式**：适合惰性求值、一次遍历的场景
4. **max() + default**：适合求最值的场景

### 9.3 OpenCode 重构工作流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Plan 模式   │ ──▶ │  Build 模式  │ ──▶ │   测试验证   │ ──▶ │   git commit│
│  @explore   │     │  @文件重构   │     │  unittest   │     │             │
│  分析问题   │     │  执行重构   │     │  验证功能   │     │  提交变更   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
       │                                       │
       │              如果测试失败              │
       └───────────────────◀───────────────────┘
                      使用 /undo 回滚
```

### 9.4 /undo 的使用场景

如果重构后测试失败，立即使用：
```
/undo
```
这会撤销 OpenCode 对文件的所有修改，恢复到重构前的状态。

---

## 十、附录：快速参考

### A. 常用 Prompt 模板

**全局分析**：
```
@explore 分析整个项目，找出以下代码质量问题：
1. 使用反射（getattr）的地方
2. 超过 5 个分支的 if/elif 链
3. 可以用列表推导式替代的手动 for 循环
4. 可以用生成器表达式优化的地方
列出文件名和具体行号
```

**制定计划**：
```
@文件 1 @文件 2 @文件 3
基于分析结果，制定重构计划：
1. 每个文件需要重构的具体方法
2. 重构方式（字典/列表推导式/生成器）
3. 预期效果
不要写代码，输出计划到 .opencode/plans/xxx.md
```

**执行重构**：
```
@目标文件.py
重构 XXX 类，完成以下改进：
1. 具体改进点 1
2. 具体改进点 2
保持所有现有功能不变，不要修改方法签名
```

**审查结果**：
```
@文件 1 @文件 2
审查刚才完成的重构，检查：
1. 是否还有遗漏的优化机会？
2. 重构后的代码是否更易读？
3. 有没有引入新的问题？
```

### B. Python 惯用写法速查

| 场景 | 重构前 | 重构后 |
|---|---|---|
| 查找元素 | `for x in xs: if x.id == id: return x` | `next((x for x in xs if x.id == id), None)` |
| 过滤列表 | `for x in xs: if cond: results.append(x)` | `[x for x in xs if cond]` |
| 求最大值 | `max_val = 0; for x in xs: if x > max_val: max_val = x` | `max((x for x in xs), default=0)` |
| 多分支判断 | `if x == 1: ... elif x == 2: ...` | `handlers = {1: f1, 2: f2}; handlers[x]()` |

### C. 命令速查

```bash
# 运行测试
python3 -m unittest discover tests -v
python3 -m pytest tests/ -v

# Git 操作
git status
git diff
git add -A
git commit -m "message"

# OpenCode 命令
/undo        # 撤销上一步修改
/plan        # 切换到 Plan 模式
/build       # 切换到 Build 模式
```

---

**实验完成！** 🎉

你现在掌握了使用 OpenCode 进行代码重构的完整工作流。继续探索其他重构模式，如提取函数、引入策略模式等。
