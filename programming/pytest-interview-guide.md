# Pytest 面试完全指南

> 标签: #pytest #python #testing #面试
> 创建时间: 2026-02-24
> 来源: Claude Code 整理

## 概述

Pytest 是 Python 生态中最流行的单元测试框架，以简洁的 API、强大的 fixture 系统和丰富的插件生态著称。本文详细整理了 pytest 框架面试所需的知识点，按必要、掌握、了解三个优先级分类。

---

## 一、必要知识点（必须精通）

### 1.1 pytest 基本用法

#### 测试发现机制

pytest 自动发现测试的规则：
- 文件名以 `test_` 开头或 `_test` 结尾
- 函数/方法以 `test_` 开头
- 类名以 `Test` 开头（类内的方法也需以 `test_` 开头）

```python
# test_example.py
def test_add():
    assert 1 + 1 == 2

class TestMath:
    def test_multiply(self):
        assert 2 * 3 == 6
```

#### 常用命令行选项

| 选项 | 说明 |
|------|------|
| `-v` | 详细输出，显示每个测试的名称 |
| `-s` | 显示 print 输出（默认不显示） |
| `-x` | 遇到第一个失败就停止 |
| `-k` | 通过名称表达式过滤测试 |
| `-m` | 通过标记过滤测试 |
| `--tb` | 设置 traceback 格式（short/long/line/native） |
| `--lf` / `--last-failed` | 只运行上次失败的测试 |
| `-n` | 并行运行（需要 pytest-xdist） |

```bash
# 常用组合
pytest -v -s                    # 详细 + 显示输出
pytest -k "test_add"            # 只运行名称包含 test_add 的
pytest -m "slow"                # 只运行标记为 slow 的
pytest --tb=short               # 简短的错误信息
pytest -x --lf                  # 失败即停 + 只跑上次失败的
```

### 1.2 断言（Assert）

#### 基本断言

Pytest 使用原生 Python `assert` 语句，失败时自动提供详细的python
def test_basic():
    assert 1 + 1 == 2
    assert "hello断言信息。

```" in "hello world"
    assert [1, 2] == [1, 2]
```

#### 断言重写（Assertion Rewriting）

Pytest 内置的断言重写机制会在断言失败时提供更详细的错误信息：

```python
# 普通断言错误
assert [1, 2, 3] == [1, 2, 4]
# Pytest 输出: AssertionError: assert [1, 2, 3] == [1, 2, 4]

# 使用 pytest.raises 捕获异常
def test_raises():
    with pytest.raises(ValueError, match="invalid literal"):
        int("abc")
```

#### 常用断言函数

```python
import pytest

# 近似相等（浮点数）
assert pytest.approx(0.1 + 0.2, rel=1e-6) == pytest.approx(0.3)

# 异常捕获
with pytest.raises(ZeroDivisionError):
    1 / 0

# 警告捕获
with pytest.warns(UserWarning):
    warnings.warn("warning", UserWarning)

# 上下文管理器内的代码不抛出异常
def test_exception():
    with pytest.raises(Exception) as exc_info:
        raise ValueError("error")
    assert str(exc_info.value) == "error"
```

### 1.3 Fixture（核心重点）

Fixture 是 pytest 最强大的特性之一，用于提供测试所需的依赖。

#### 基本用法

```python
import pytest

@pytest.fixture
def db_connection():
    """返回一个数据库连接，测试后自动清理"""
    conn = create_connection()
    yield conn
    conn.close()

def test_query(db_connection):
    result = db_connection.query("SELECT * FROM users")
    assert len(result) > 0
```

#### Fixture 的 Scope（作用域）

| Scope | 说明 | 生命周期 |
|-------|------|----------|
| `function` | 默认 | 每个测试函数执行一次 |
| `class` | 类级别 | 每个测试类执行一次 |
| `module` | 模块级别 | 每个模块执行一次 |
| `session` | 会话级别 | 整个测试会话执行一次 |

```python
@pytest.fixture(scope="session")
def config():
    """整个测试会话只创建一次"""
    return load_config()

@pytest.fixture(scope="module")
def db():
    """每个模块共享一个数据库连接"""
    return Database()
```

#### autouse=True 自动使用

```python
@pytest.fixture(autouse=True)
def setup_logging():
    """每个测试自动启用日志"""
    logging.basicConfig(level=logging.DEBUG)
    yield
    # teardown
    logging.shutdown()
```

#### Fixture 依赖注入

```python
@pytest.fixture
def user():
    return {"name": "test", "email": "test@example.com"}

@pytest.fixture
def db_with_user(user):  # 注入 user fixture
    db = Database()
    db.create_user(user)
    yield db
    db.delete_user(user["email"])
```

#### yield vs return

- `return`: 简单返回，teardown 在 fixture 函数结束后执行
- `yield`: 支持清理代码，yield 后的代码作为 teardown 执行

```python
@pytest.fixture
def temp_file():
    file = open("temp.txt", "w")
    yield file
    file.close()  # teardown

@pytest.fixture
def transaction():
    db.begin()
    yield db
    db.rollback()  # 测试后回滚
```

### 1.4 参数化（@pytest.mark.parametrize）

参数化是避免重复测试代码的核心手段。

#### 基本参数化

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert a + b == expected
```

#### 多参数组合

```python
@pytest.mark.parametrize("x", [1, 2, 3])
@pytest.mark.parametrize("y", [10, 20])
def test_combination(x, y):
    # 会运行 3 * 2 = 6 次
    assert x * y > 0
```

#### 名称参数化

```python
@pytest.mark.parametrize("a,b,expected", [
    pytest.param(1, 2, 3, id="positive"),
    pytest.param(-1, -2, -3, id="negative"),
    pytest.param(0, 0, 0, id="zero"),
])
def test_add(a, b, expected):
    assert a + b == expected
```

#### 间接参数化

```python
@pytest.fixture
def db_config(request):
    return request.param

@pytest.mark.parametrize("db_config", ["mysql", "postgres"], indirect=True)
def test_db_connection(db_config):
    assert connect(db_config)
```

### 1.5 标记（Markers）

#### 内置标记

| 标记 | 说明 |
|------|------|
| `@pytest.mark.skip` | 跳过测试 |
| `@pytest.mark.skipif(condition, reason)` | 条件跳过 |
| `@pytest.mark.xfail(reason, strict)` | 预期失败 |
| `@pytest.mark.parametrize` | 参数化 |
| `@pytest.mark.fixture` | 标记为 fixture |

#### 自定义标记

```python
# pytest.ini 或 pyproject.toml
[tool.pytest.ini_options]
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    unit: marks tests as unit tests
```

```python
@pytest.mark.slow
@pytest.mark.integration
def test_full_workflow():
    """集成测试"""
    pass

@pytest.mark.unit
def test_unit():
    """单元测试"""
    pass
```

#### 使用标记过滤

```bash
pytest -m "slow"              # 只运行 slow 标记
pytest -m "not slow"          # 排除 slow 标记
pytest -m "slow or integration"  # 或条件
pytest -m "slow and not integration"  # 组合条件
```

### 1.6 配置文件

#### pytest.ini

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short
markers =
    slow: slow running tests
    integration: integration tests
```

#### pyproject.toml（推荐）

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"
markers = [
    "slow: marks tests as slow",
    "integration: integration tests"
]

[tool.coverage.run]
source = ["src"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
]
```

#### conftest.py

`conftest.py` 用于共享 fixtures 和 hooks，pytest 会自动加载同目录及子目录的 `conftest.py`。

```
project/
├── tests/
│   ├── conftest.py      # 共享 fixtures
│   ├── unit/
│   │   └── test_math.py
│   └── integration/
│       └── test_api.py
```

```python
# conftest.py
import pytest

@pytest.fixture
def mock_api():
    return MockAPI()

# pytest_collection_modifyitems hook
def pytest_collection_modifyitems(config, items):
    for item in items:
        if "integration" in item.nodeid:
            item.add_marker(pytest.mark.integration)
```

### 1.7 常用 pytest 插件

| 插件 | 用途 |
|------|------|
| pytest-cov | 代码覆盖率 |
| pytest-xdist | 并行测试 |
| pytest-mock | Mock 支持 |
| pytest-asyncio | 异步测试 |
| pytest-timeout | 超时控制 |
| pytest-randomly | 随机顺序 |
| pytest-html | HTML 报告 |
| pytest-markers | 标记管理 |

---

## 二、掌握知识点

### 2.1 测试收集与修改（pytest_collection_modifyitems）

```python
# conftest.py
def pytest_collection_modifyitems(config, items):
    """在收集完成后修改测试项"""
    # 按标记排序
    items.sort(key=lambda item: (
        "integration" in item.nodeid,
        "slow" in [m.name for m in item.iter_markers()]
    ))

    # 为特定测试添加标记
    for item in items:
        if "test_db" in item.nodeid:
            item.add_marker(pytest.mark.slow)
```

### 2.2 MonkeyPatch

用于在测试中动态替换属性、方法、环境变量等。

```python
def test_monkeypatch(monkeypatch):
    # 替换函数
    monkeypatch.setattr("os.path.exists", lambda x: True)

    # 替换属性
    class Foo:
        def __init__(self):
            self.value = 42

    monkeypatch.setattr(Foo, "value", 100)

    # 环境变量
    monkeypatch.setenv("DATABASE_URL", "test://localhost")

    # 字典操作
    monkeypatch.setitem({"a": 1}, "b", 2)
```

### 2.3 临时目录（tmp_path）

```python
def test_tmp_path(tmp_path):
    # 创建临时文件
    file = tmp_path / "test.txt"
    file.write_text("hello")

    assert file.read_text() == "hello"

def test_tmp_path_factory(tmp_path_factory):
    # 创建命名临时目录
    tmp_dir = tmp_path_factory.mktemp("data")
    (tmp_dir / "file.txt").write_text("data")
```

### 2.4 pytest Hooks

#### 常用 Hooks

```python
# conftest.py
def pytest_configure(config):
    """pytest 启动时调用"""
    config.addinivalue_line("markers", "custom: custom marker")

def pytest_collection_finish(session):
    """收集完成后调用"""
    print(f"收集到 {len(session.items)} 个测试")

def pytest_runtest_setup(item):
    """每个测试 setup 阶段调用"""
    pass

def pytest_runtest_teardown(item, nextitem):
    """每个测试 teardown 阶段调用"""
    pass

def pytest_runtest_makereport(item, call):
    """测试结果生成后调用"""
    if call.excinfo is not None:
        print(f"测试失败: {item.name}")
```

### 2.5 跳过测试（Skip/XFail）

```python
import pytest
import sys

# 简单跳过
@pytest.mark.skip(reason="not ready")
def test_not_ready():
    pass

# 条件跳过
@pytest.mark.skipif(sys.version_info < (3, 8), reason="requires python3.8+")
def test_new_feature():
    pass

# 预期失败
@pytest.mark.xfail(reason="known bug")
def test_known_bug():
    assert False

# 严格模式 - 预期失败但实际通过会报错
@pytest.mark.xfail(strict=True)
def test_should_fail():
    assert False
```

### 2.6 异步测试（pytest-asyncio）

```python
import pytest

@pytest.mark.asyncio
async def test_async():
    result = await async_function()
    assert result == expected

# 可选 - 标记为 fixture
@pytest.fixture
async def async_fixture():
    await setup()
    yield
    await teardown()
```

### 2.7 报告生成

```bash
# JUnit XML 报告
pytest --junitxml=report.xml

# HTML 报告（需要 pytest-html）
pytest --html=report.html --self-contained-html

# 覆盖率报告
pytest --cov=src --cov-report=html
```

---

## 三、了解知识点

### 3.1 pytest 插件开发

```python
# myplugin.py
import pytest

def pytest_addoption(parser):
    parser.addoption("--my-option", action="store", default="default")

def pytest_configure(config):
    config.addinivalue_line("markers", "myplugin: my plugin marker")

@pytest.hookimpl(tryfirst=True)
def pytest_runtest_makereport(item, call):
    # 自定义报告逻辑
    pass
```

### 3.2 pytest 内部机制

- **断言重写**: 在字节码级别修改 assert 语句
- **fixture 缓存**: 相同 scope 的 fixture 只执行一次
- **插件系统**: 通过 setuptools entry points 加载

### 3.3 性能优化

```bash
# 并行执行
pytest -n auto

# 只运行失败的测试
pytest --lf

# 减少导入开销
pytest --p no:cacheprovider
```

### 3.4 conftest.py 的加载顺序

- 按文件路径字母顺序
- 父目录的 conftest 先于子目录加载
- `pytest_plugins` 可以在代码中加载插件

---

## 四、面试常见问题

### Q1: pytest 相比 unittest 的优势？

1. **简洁的 API**: 不需要继承任何类
2. **强大的 fixture**: 自动管理依赖和清理
3. **丰富的标记系统**: 灵活控制测试执行
4. **参数化支持**: 避免重复代码
5. **优秀的插件生态**: 覆盖各种测试场景

### Q2: fixture 的 scope 如何选择？

- **function**: 默认，适合大多数情况
- **class**: 测试类共享资源时使用
- **module**: 模块级别的资源（如数据库连接）
- **session**: 整个测试会话共享（如配置加载）

### Q3: 如何实现测试间的数据共享？

1. **session/module scope fixtures**: 通过 fixture 共享
2. **class scope fixtures**: 类内共享
3. **pytest 的缓存**: `pytest_cache` 目录
4. **外部文件**: 临时文件或数据库

### Q4: pytest 如何处理异步代码？

使用 `pytest-asyncio` 插件：
- 安装: `pip install pytest-asyncio`
- 标记: `@pytest.mark.asyncio`
- 配置: `asyncio_mode = "auto"`

### Q5: 如何 mock 第三方库？

使用 `pytest-mock` 或标准库 `unittest.mock`：

```python
def test_with_mock(monkeypatch):
    from module import external_api

    def mock_api():
        return "mocked"

    monkeypatch.setattr(external_api, "call", mock_api)
```

---

## 五、实战代码示例

### 完整的测试文件示例

```python
# tests/test_user.py
import pytest
from user import UserService

# ==================== Fixtures ====================
@pytest.fixture
def user_service():
    return UserService()

@pytest.fixture
def sample_user():
    return {"name": "test", "email": "test@example.com"}

# ==================== 单元测试 ====================
class TestUserService:
    """用户服务单元测试"""

    def test_create_user(self, user_service, sample_user):
        """测试创建用户"""
        result = user_service.create_user(sample_user)
        assert result["name"] == sample_user["name"]
        assert "id" in result

    def test_create_duplicate_user(self, user_service, sample_user):
        """测试重复创建用户"""
        user_service.create_user(sample_user)

        with pytest.raises(ValueError, match="already exists"):
            user_service.create_user(sample_user)

    @pytest.mark.parametrize("email,valid", [
        ("test@example.com", True),
        ("invalid", False),
        ("test@", False),
    ])
    def test_email_validation(self, user_service, email, valid):
        """参数化测试邮箱验证"""
        if not valid:
            with pytest.raises(ValueError):
                user_service.validate_email(email)
        else:
            assert user_service.validate_email(email) is None

# ==================== 集成测试 ====================
@pytest.mark.integration
class TestUserServiceIntegration:
    """集成测试"""

    @pytest.fixture(autouse=True)
    def setup_db(self, db_connection):
        db_connection.clear()
        yield
        db_connection.clear()

    def test_full_workflow(self, user_service):
        """完整工作流测试"""
        user = user_service.create_user({"name": "test", "email": "test@example.com"})
        fetched = user_service.get_user(user["id"])
        assert fetched["email"] == user["email"]
```

### conftest.py 示例

```python
# tests/conftest.py
import pytest
import logging

# ==================== Fixtures ====================
@pytest.fixture(scope="session")
def db_connection():
    """数据库连接 - session 级别"""
    conn = create_test_db()
    yield conn
    conn.close()

@pytest.fixture
def db(db_connection):
    """每个测试的干净数据库"""
    db_connection.clear()
    return db_connection

@pytest.fixture
def mock_logger():
    """Mock 日志"""
    with pytest.mock.patch("logging.Logger") as mock:
        yield mock

# ==================== Hooks ====================
def pytest_configure(config):
    """配置 pytest"""
    config.addinivalue_line("markers", "integration: integration tests")
    config.addinivalue_line("markers", "slow: slow tests")

def pytest_collection_modifyitems(config, items):
    """自动为集成测试添加标记"""
    for item in items:
        if "integration" in item.nodeid:
            item.add_marker(pytest.mark.integration)

def pytest_runtest_makereport(item, call):
    """自定义测试报告"""
    if call.when == "call":
        if call.excinfo is not None:
            # 记录失败
            logging.error(f"FAILED: {item.nodeid}")
```

---

## 相关知识点

- [[Python Unittest 框架]]
- [[Mock 单元测试]]
- [[测试驱动开发 TDD]]

---
*采集自 Claude Code 对话*
