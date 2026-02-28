# Playwright Python 测试框架精通指南

> 标签: #playwright #python #automation #testing #e2e #面试
> 创建时间: 2026-02-26
> 更新时间: 2026-02-28
> 来源: [Playwright Python官方文档](https://playwright.dev/python/) | [GitHub](https://github.com/microsoft/playwright-python)

## 概述

Playwright 是微软开发的现代端到端测试框架，支持 Chromium、Firefox、WebKit 三大浏览器引擎。Python 版本提供了与同步/异步 API，完美结合 Pytest 测试框架，是 2025 年 Python 自动化测试的首选方案。

---

## 一、知识体系总览

### 掌握程度分类说明

| 级别 | 说明 | 面试权重 |
|------|------|----------|
| 🔴 必须掌握 | 面试必问，项目必用，需熟练手写 | 40% |
| 🟠 重要 | 常见考点，需要理解原理并能应用 | 30% |
| 🟡 常用 | 项目中频繁使用，需要熟悉 | 20% |
| 🟢 了解 | 高级特性或边界场景，知道即可 | 10% |

---

## 二、核心知识点

### 🔴 必须掌握

#### 1. Playwright vs Selenium vs Cypress 对比

| 特性 | Playwright | Selenium | Cypress |
|------|------------|----------|---------|
| 浏览器支持 | Chromium, Firefox, WebKit (原生) | 所有主流浏览器 | 仅 Chromium 内核 |
| 自动等待 | ✅ 内置 | ❌ 需手动 | ✅ 内置 |
| 多语言支持 | Python, JS/TS, Java, .NET | Java, Python, C#, Ruby等 | 仅 JS/TS |
| 执行速度 | 快 | 较慢 | 快 |
| 移动端模拟 | ✅ 内置 | ✅ 需 Appium | ❌ 有限 |
| API 测试 | ✅ 内置 | ❌ 需额外工具 | ✅ 内置 |
| Shadow DOM | ✅ 支持 | ❌ 困难 | ✅ 支持 |
| 网络拦截 | ✅ 强大 | ❌ 有限 | ✅ 支持 |
| Pytest 集成 | ✅ 原生支持 | ✅ 需配置 | ❌ 不支持 |

**面试回答要点：**
- 选择 Playwright Python 的原因：自动等待、原生 Pytest 集成、跨浏览器、现代 API
- 相比 Selenium：更快、更稳定、API 更现代、内置等待机制
- 相比 Cypress：支持 Safari、支持 Python、网络控制更强

#### 2. 环境搭建

```bash
# 安装 Playwright
pip install playwright

# 安装浏览器（必须）
playwright install

# 安装特定浏览器
playwright install chromium
playwright install firefox
playwright install webkit

# 安装 Pytest 插件（推荐）
pip install pytest-playwright

# 验证安装
playwright --version
```

#### 3. 定位器 (Locators)

```python
from playwright.sync_api import Page

# ============ 推荐的定位方式（按优先级）============

# 1. Role 定位（最推荐，语义化）
page.get_by_role("button", name="提交")
page.get_by_role("textbox", name="用户名")
page.get_by_role("link", name="登录")
page.get_by_role("checkbox", name="记住我")

# 2. Test ID 定位（稳定，需开发配合）
page.get_by_test_id("submit-button")
page.get_by_test_id("login-form")

# 3. 文本定位
page.get_by_text("欢迎登录")
page.get_by_text("登录", exact=True)  # 精确匹配

# 4. Label 定位（表单元素）
page.get_by_label("密码")
page.get_by_label("用户名")

# 5. Placeholder 定位
page.get_by_placeholder("请输入用户名")

# 6. CSS 选择器
page.locator("#username")
page.locator(".btn-primary")
page.locator("form > button[type='submit']")

# 7. 组合定位
page.locator("article").filter(has_text="Playwright")
page.locator(".card").get_by_role("button")

# 8. XPath（不推荐，作为备选）
page.locator("//button[@type='submit']")

# 9. 链式定位
page.locator(".list").locator(".item").first
page.locator(".list").locator(".item").nth(2)
```

**面试考点：**
- 为什么优先使用 `get_by_role`？→ 语义化、可访问性、稳定性
- 为什么避免 XPath？→ 脆弱、难维护、性能差
- 如何定位 Shadow DOM 元素？→ Playwright 原生支持，直接定位

#### 4. 断言 (Assertions)

```python
import re
from playwright.sync_api import Page, expect

def test_assertions(page: Page):
    page.goto("https://example.com")

    # ----- 页面级断言 -----
    expect(page).to_have_url(re.compile(r".*dashboard.*"))
    expect(page).to_have_title(re.compile(r".*Playwright.*"))

    # ----- 元素可见性断言 -----
    locator = page.locator(".status")
    expect(locator).to_be_visible()
    expect(locator).to_be_hidden()
    expect(locator).to_be_enabled()
    expect(locator).to_be_disabled()
    expect(locator).to_be_editable()
    expect(locator).to_be_empty()
    expect(locator).to_be_focused()

    # ----- 文本内容断言 -----
    expect(locator).to_have_text("欢迎登录")
    expect(locator).to_have_text(re.compile(r"欢迎\s*登录"))
    expect(locator).to_contain_text("登录")

    # ----- 属性断言 -----
    expect(locator).to_have_attribute("href", "/docs")
    expect(locator).to_have_class("active")
    expect(locator).to_have_css("color", "rgb(255, 0, 0)")
    expect(locator).to_have_value("input value")

    # ----- 数量断言 -----
    expect(page.locator(".item")).to_have_count(5)

    # ----- 自定义超时 -----
    expect(locator).to_be_visible(timeout=10000)

    # ----- 否定断言 -----
    expect(locator).not_to_be_visible()

    # ----- 截图断言（视觉回归）-----
    expect(page).to_have_screenshot("homepage.png")
```

**面试考点：**
- Playwright 断言是自动重试的吗？→ 是，默认 5 秒内重试
- `to_have_text` vs `to_contain_text`？→ 完全匹配 vs 包含
- 如何处理异步断言？→ 使用 `expect`，Playwright 自动等待

#### 5. 自动等待机制

```python
from playwright.sync_api import Page

def test_auto_waiting(page: Page):
    # Playwright 自动等待，无需显式 sleep
    page.goto("https://example.com")

    # 这些操作都会自动等待元素可操作
    page.click("button")           # 等待元素可见、可点击
    page.fill("#input", "text")    # 等待元素可见、可编辑
    page.locator(".item").first.click()

    # 显式等待特定条件
    page.wait_for_selector(".loaded")
    page.wait_for_load_state("networkidle")  # 等待网络空闲
    page.wait_for_url("**/dashboard**")

    # 等待请求完成
    with page.expect_response("**/api/data**") as response:
        page.click("button")
    resp = response.value

    # 等待元素状态
    page.locator(".loading").wait_for(state="hidden")
```

**面试考点：**
- 为什么不需要 `time.sleep()`？→ Playwright 内置自动等待
- 什么时候需要显式等待？→ 复杂异步场景、网络请求、页面跳转
- `wait_for_load_state` 的三种状态？→ `domcontentloaded`、`load`、`networkidle`

#### 6. Page Object Model (POM)

```python
# ============ pages/login_page.py ============
from playwright.sync_api import Page, Locator

class LoginPage:
    def __init__(self, page: Page):
        self.page = page
        self.username_input: Locator = page.get_by_label("用户名")
        self.password_input: Locator = page.get_by_label("密码")
        self.login_button: Locator = page.get_by_role("button", name="登录")
        self.error_message: Locator = page.locator(".error-message")

    def goto(self):
        self.page.goto("/login")

    def login(self, username: str, password: str):
        self.username_input.fill(username)
        self.password_input.fill(password)
        self.login_button.click()

    def get_error(self) -> str:
        return self.error_message.text_content()


# ============ tests/test_login.py ============
from playwright.sync_api import Page, expect
from pages.login_page import LoginPage

def test_login_success(page: Page):
    login_page = LoginPage(page)
    login_page.goto()
    login_page.login("testuser", "password123")

    expect(page).to_have_url(re.compile(r".*dashboard.*"))


def test_login_failed(page: Page):
    login_page = LoginPage(page)
    login_page.goto()
    login_page.login("wrong", "wrong")

    expect(login_page.error_message).to_be_visible()
```

**面试考点：**
- 为什么要用 POM？→ 代码复用、易于维护、职责分离
- Page 类应该包含断言吗？→ 不应该，断言放在测试文件中
- 如何处理公共组件？→ 抽取 BasePage 或 Component 类

---

### 🟠 重要

#### 7. 多窗口/多标签页处理

```python
from playwright.sync_api import Page, BrowserContext

def test_multiple_pages(page: Page, context: BrowserContext):
    # 方式一：监听新页面
    with context.expect_page() as new_page_info:
        page.click("a[target='_blank']")
    new_page = new_page_info.value

    new_page.wait_for_load_state()
    expect(new_page).to_have_title(re.compile(r".*新页面.*"))
    new_page.close()

    # 方式二：获取所有页面
    all_pages = context.pages
    popup_page = next((p for p in all_pages if "popup" in p.url), None)
```

#### 8. iframe 处理

```python
from playwright.sync_api import Page

def test_iframe(page: Page):
    # 获取 frame
    frame = page.frame_locator("#myframe")

    # 在 frame 中操作
    frame.get_by_role("button", name="提交").click()
    frame.locator("#input").fill("text")

    # 嵌套 iframe
    nested_frame = frame.frame_locator(".inner-frame")
    nested_frame.get_by_text("内容").click()
```

#### 9. 网络拦截与 Mock

```python
from playwright.sync_api import Page, Route

def test_network_mock(page: Page):
    # 拦截并 Mock 响应
    def handle_route(route: Route):
        route.fulfill(
            status=200,
            content_type="application/json",
            body='{"name": "Mock User", "id": 123}'
        )

    page.route("**/api/user", handle_route)

    # 拦截并修改请求
    def modify_request(route: Route):
        headers = route.request.headers
        headers["Authorization"] = "Bearer token"
        route.continue_(headers=headers)

    page.route("**/api/login", modify_request)

    # 拦截并 abort
    page.route("**/analytics/**", lambda route: route.abort())

    # 模拟离线
    page.context.set_offline(True)

    page.goto("/")
```

#### 10. 文件上传与下载

```python
from playwright.sync_api import Page

def test_file_operations(page: Page):
    # ----- 文件上传 -----
    # 单文件
    page.set_input_files("input[type='file']", "tests/fixtures/test.pdf")

    # 多文件
    page.set_input_files("input[type='file']", ["file1.pdf", "file2.pdf"])

    # 清空文件
    page.set_input_files("input[type='file']", [])

    # ----- 文件下载 -----
    with page.expect_download() as download_info:
        page.click("a[download]")
    download = download_info.value

    path = download.path()
    file_name = download.suggested_filename
    download.save_as("downloads/" + file_name)
```

#### 11. API 测试

```python
from playwright.sync_api import APIRequestContext, expect

def test_api_get(api_request_context: APIRequestContext):
    response = api_request_context.get("/api/users")
    assert response.ok

    data = response.json()
    assert len(data) > 0


def test_api_post(api_request_context: APIRequestContext):
    response = api_request_context.post("/api/login", data={
        "username": "test",
        "password": "password123"
    })

    assert response.status == 200
    body = response.json()
    assert "token" in body


def test_api_with_auth(api_request_context: APIRequestContext):
    response = api_request_context.get("/api/profile", headers={
        "Authorization": "Bearer token123"
    })
    assert response.ok
```

#### 12. 设备模拟与响应式测试

```python
from playwright.sync_api import Page, BrowserContext
from playwright.sync_api import devices

def test_mobile(page: Page, context: BrowserContext):
    # 使用设备配置
    iphone = devices["iPhone 13 Pro"]
    # 在 conftest.py 中配置

    page.goto("/")

    # 模拟地理位置
    page.set_geolocation({"latitude": 39.9042, "longitude": 116.4074})

    # 模拟语言
    context.set_locale("zh-CN")

    # 模拟深色模式
    page.emulate_media(color_scheme="dark")

    # 自定义视口
    page.set_viewport_size({"width": 375, "height": 667})
```

#### 13. 测试标记与分组

```python
import pytest
from playwright.sync_api import Page

@pytest.mark.slow
def test_slow_operation(page: Page):
    """慢速测试"""
    pass

@pytest.mark.skip(reason="功能待开发")
def test_skip_example(page: Page):
    pass

@pytest.mark.only_browser("chromium")
def test_chromium_only(page: Page):
    """仅在 Chromium 运行"""
    pass

@pytest.mark.skip_browser("firefox")
def test_skip_firefox(page: Page):
    """跳过 Firefox"""
    pass

# 运行方式：
# pytest -m slow                    # 只运行 slow 标记
# pytest -m "not slow"              # 跳过 slow 标记
# pytest --browser=chromium         # 指定浏览器
```

---

### 🟡 常用

#### 14. playwright.config.py 配置详解

```python
# playwright.config.py
from playwright.sync_api import BrowserType
from typing import List

# 测试目录
test_dir = "./tests"

# 并行执行
fully_parallel = True

# CI 上禁止 only
forbid_only = True if os.getenv("CI") else False

# 重试次数
retries = 2 if os.getenv("CI") else 0

# 并行数
workers = 1 if os.getenv("CI") else 4

# 超时设置
timeout = 30000

# Reporter
reporter = [
    ["html", {"outputFolder": "playwright-report"}],
    ["list"]
]

# 全局设置
use = {
    "base_url": "http://localhost:3000",
    "trace": "retain-on-failure",
    "screenshot": "only-on-failure",
    "video": "retain-on-failure",
    "action_timeout": 10000,
    "navigation_timeout": 30000,
}

# 浏览器项目配置
projects = [
    {
        "name": "chromium",
        "use": {"browser_name": "chromium"}
    },
    {
        "name": "firefox",
        "use": {"browser_name": "firefox"}
    },
    {
        "name": "webkit",
        "use": {"browser_name": "webkit"}
    },
    {
        "name": "Mobile Chrome",
        "use": devices["Pixel 5"]
    },
]
```

#### 15. Pytest Fixtures

```python
# conftest.py
import pytest
from playwright.sync_api import Browser, BrowserContext, Page

@pytest.fixture
def authenticated_page(page: Page):
    """已登录的页面"""
    page.goto("/login")
    page.fill("#username", "testuser")
    page.fill("#password", "password")
    page.click("button[type='submit']")
    page.wait_for_url("**/dashboard**")

    yield page

    # Teardown: 登出
    page.click("#logout")


@pytest.fixture
def api_context(browser: Browser):
    """API 请求上下文"""
    context = browser.new_context()
    request_context = context.request
    yield request_context
    context.close()


# 使用
def test_with_auth(authenticated_page: Page):
    authenticated_page.click(".profile")
```

#### 16. 参数化测试

```python
import pytest
from playwright.sync_api import Page, expect

# 数据驱动测试
login_data = [
    {"username": "user1", "password": "pass1", "expected": "success"},
    {"username": "user2", "password": "wrong", "expected": "error"},
    {"username": "", "password": "pass1", "expected": "error"},
]

@pytest.mark.parametrize("data", login_data)
def test_login(data, page: Page):
    page.goto("/login")
    page.fill("#username", data["username"])
    page.fill("#password", data["password"])
    page.click("button[type='submit']")

    if data["expected"] == "success":
        expect(page).to_have_url(re.compile(r".*dashboard.*"))
    else:
        expect(page.locator(".error")).to_be_visible()
```

#### 17. Trace Viewer 与调试

```bash
# 调试命令
pytest --ui                          # UI 模式
pytest --debug                       # 调试模式
pytest --tracing on                  # 开启 trace
playwright show-trace trace.zip      # 查看 trace

# 代码生成
playwright codegen https://example.com

# 查看报告
playwright show-report

# 指定浏览器
pytest --browser=chromium
pytest --browser=firefox --browser=webkit

# 有头模式
pytest --headed

# 慢速执行
pytest --slowmo=1000
```

---

### 🟢 了解

#### 18. 高级特性

```python
from playwright.sync_api import Page, expect

def test_visual_regression(page: Page):
    """视觉回归测试"""
    page.goto("/")
    expect(page).to_have_screenshot("homepage.png", max_diff_pixels=100)


def test_performance(page: Page):
    """性能测试"""
    page.goto("/")

    # 获取性能指标
    timing = page.evaluate("""() => {
        const nav = performance.getEntriesByType('navigation')[0];
        return {
            domContentLoaded: nav.domContentLoadedEventEnd,
            load: nav.loadEventEnd
        }
    }""")

    assert timing["load"] < 3000


def test_console_errors(page: Page):
    """捕获控制台错误"""
    errors = []

    page.on("console", lambda msg: errors.append(msg) if msg.type == "error" else None)

    page.goto("/")

    assert len(errors) == 0, f"发现控制台错误: {errors}"
```

#### 19. 测试分片 (Sharding)

```bash
# CI 中分片执行
pytest --shard=1/3  # 第一片
pytest --shard=2/3  # 第二片
pytest --shard=3/3  # 第三片
```

#### 20. 同步与异步 API

```python
# 同步 API（推荐入门）
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())
    browser.close()


# 异步 API（高性能场景）
import asyncio
from playwright.async_api import async_playwright

async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        await page.goto("https://example.com")
        print(await page.title())
        await browser.close()

asyncio.run(main())
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 核心答案 |
|------|----------|
| 什么是 Playwright？ | 微软开发的现代 E2E 测试框架，支持跨浏览器、跨语言 |
| 为什么选择 Playwright Python？ | 自动等待、原生 Pytest 集成、跨浏览器、API 现代 |
| Playwright 支持哪些浏览器？ | Chromium、Firefox、WebKit |
| 什么是自动等待？ | 操作前自动等待元素可交互，无需显式 sleep |
| 如何安装 Playwright？ | `pip install playwright` + `playwright install` |

### 进阶篇

| 问题 | 核心答案 |
|------|----------|
| 如何处理动态元素？ | 使用稳定的定位策略（role、testId），利用自动等待 |
| POM 的核心原则？ | 分离页面逻辑与测试逻辑、单一职责、复用性 |
| 如何处理 iframe？ | `page.frame_locator()` |
| 如何模拟 API 响应？ | `page.route()` + `route.fulfill()` |
| 如何优化测试速度？ | 并行执行、减少等待、复用 context、分片 |

### 高级篇

| 问题 | 核心答案 |
|------|----------|
| 如何集成 CI/CD？ | GitHub Actions、Jenkins、Docker 镜像 |
| 如何处理认证状态？ | `storage_state` 保存登录状态复用 |
| Trace Viewer 是什么？ | 测试执行的完整记录，可回放调试 |
| 如何做视觉回归测试？ | `expect(page).to_have_screenshot()` |
| 同步和异步 API 区别？ | sync_api 简单易用，async_api 性能更高 |

---

## 四、项目实战经验（面试话术）

### 项目结构

```
playwright-python-project/
├── tests/
│   ├── e2e/                     # 端到端测试
│   ├── api/                     # API 测试
│   └── visual/                  # 视觉回归测试
├── pages/                       # Page Object
├── fixtures/                    # Pytest fixtures
├── test_data/                   # 测试数据
├── utils/                       # 工具函数
├── conftest.py                  # Pytest 配置
├── playwright.config.py         # Playwright 配置
├── pytest.ini                   # Pytest 配置
└── requirements.txt
```

### 面试项目描述模板

> "在我们的电商项目中，我使用 Playwright + Pytest 搭建了完整的自动化测试框架：
>
> 1. **框架设计**：采用 Page Object Model，分离业务逻辑和测试脚本
> 2. **跨浏览器**：配置 Chrome、Firefox、Safari 三浏览器并行测试
> 3. **CI 集成**：GitHub Actions 自动触发，失败自动截图和录像
> 4. **测试覆盖**：包括 UI 测试、API 测试、视觉回归测试
> 5. **优化成果**：通过并行执行和分片，将测试时间从 30 分钟降到 5 分钟"

### 常见问题解决方案

| 场景 | 解决方案 |
|------|----------|
| 元素定位不稳定 | 使用 `get_by_role`、`get_by_test_id`，避免 XPath |
| 测试偶发失败 | 检查自动等待、增加重试、分析 Trace |
| 测试执行慢 | 并行执行、复用登录状态、优化选择器 |
| 多环境测试 | 使用 `base_url` 配置 + 环境变量 |
| 测试数据管理 | 使用 fixtures 或外部数据文件 |

---

## 五、CI/CD 集成

### GitHub Actions

```yaml
name: Playwright Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          playwright install --with-deps
      - name: Run tests
        run: pytest
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

### Jenkins Pipeline

```groovy
pipeline {
  agent any
  stages {
    stage('Install') {
      steps {
        sh 'pip install -r requirements.txt'
        sh 'playwright install --with-deps'
      }
    }
    stage('Test') {
      steps {
        sh 'pytest --html=report.html'
      }
      post {
        always {
          archiveArtifacts artifacts: 'playwright-report/**/*'
          publishHTML target: [
            reportDir: '.',
            reportFiles: 'report.html',
            reportName: 'Playwright Report'
          ]
        }
      }
    }
  }
}
```

### Docker 集成

```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.40.0-jammy

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

CMD ["pytest"]
```

---

## 六、常用命令速查

```bash
# 安装
pip install playwright pytest-playwright
playwright install

# 运行测试
pytest                                # 运行所有
pytest tests/test_login.py            # 运行指定文件
pytest -k "login"                     # 按名称过滤
pytest -m "smoke"                     # 按标记运行

# 浏览器选项
pytest --browser=chromium             # 指定浏览器
pytest --browser=firefox --browser=webkit  # 多浏览器
pytest --headed                       # 有头模式
pytest --slowmo=1000                  # 慢速执行

# 调试
pytest --ui                           # UI 模式
pytest --debug                        # 调试模式
pytest --tracing on                   # 开启 trace

# 代码生成
playwright codegen https://example.com

# 报告
pytest --html=report.html             # HTML 报告
playwright show-report                # 查看报告
playwright show-trace trace.zip       # 查看 trace

# 安装浏览器
playwright install
playwright install chromium
```

---

## 七、Pytest + Playwright 常用 Fixtures

| Fixture | 说明 |
|---------|------|
| `page` | Page 对象，每个测试独立 |
| `context` | BrowserContext 对象 |
| `browser` | Browser 对象 |
| `browser_name` | 当前浏览器名称 |
| `browser_type` | BrowserType 对象 |
| `api_request_context` | API 请求上下文 |
| `request` | Pytest request 对象 |

---

## 八、学习路径建议

```
Week 1: 基础入门
├── 环境搭建（pip install + playwright install）
├── 定位器 (get_by_role, get_by_test_id)
├── 断言 (expect)
└── 基本操作 (click, fill, goto)

Week 2: 进阶技能
├── Page Object Model
├── Pytest fixtures
├── 多窗口/iframe 处理
└── 文件上传/下载

Week 3: 高级特性
├── 网络拦截与 Mock
├── API 测试
├── 视觉回归测试
└── 设备模拟

Week 4: 工程化
├── playwright.config.py 配置
├── CI/CD 集成
├── 测试报告
└── 框架优化
```

---

## 相关知识点

- [[Pytest 面试完全指南]]
- [[Docker 常用命令速查]]
- [[Python Requests 精通指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [Playwright Python 官方文档](https://playwright.dev/python/)
- [Playwright Python GitHub](https://github.com/microsoft/playwright-python)
- [Pytest-Playwright 插件](https://github.com/microsoft/playwright-pytest)
