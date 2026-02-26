# Airtest 框架测试工程师精通指南

> 标签: #airtest #automation #mobile-testing #game-testing #面试
> 创建时间: 2026-02-26
> 来源: [Airtest官方文档](https://airtest.doc.io.netease.com/) | [Airtest GitHub](https://github.com/AirtestProject/Airtest)

## 概述

Airtest 是网易开源的跨平台 UI 自动化测试框架，基于图像识别技术，支持 Android、iOS、Windows、Web 和游戏测试。与 Poco 框架配合可实现更精准的控件操作。本文整理测试面试高频 Airtest 知识点，按重要程度分类。

---

## 一、知识体系总览

### 掌握程度分类

| 级别 | 说明 | 面试权重 |
|------|------|----------|
| 🔴 必须掌握 | 面试必问，项目必用 | 40% |
| 🟠 重要 | 常见考点，需要熟练 | 30% |
| 🟡 常用 | 工作中频繁使用 | 20% |
| 🟢 了解 | 高级场景，知道即可 | 10% |

---

## 二、核心知识点

### 🔴 必须掌握

#### 1. 框架简介与对比

```markdown
## Airtest 是什么？
- 网易开源的 UI 自动化测试框架
- 基于 **图像识别** 技术
- 支持 **跨平台**：Android、iOS、Windows、Web、游戏

## 核心特点
1. 图像识别定位元素（无需控件 ID）
2. 生成 HTML 测试报告
3. 与 Poco 配合支持控件树操作
4. AirtestIDE 可视化录制

## vs Appium 对比

| 特性 | Airtest | Appium |
|------|---------|--------|
| 定位方式 | 图像识别 | 控件属性 |
| 学习曲线 | 低 | 较高 |
| 跨平台 | ✅ 全面 | ✅ 移动端 |
| 游戏测试 | ✅ 支持 | ❌ 困难 |
| 稳定性 | 依赖图像匹配 | 依赖控件属性 |
| 维护成本 | 图像需更新 | 选择器需更新 |
```

#### 2. 环境搭建

```bash
# ========== 安装 ==========
pip install airtest
pip install pocoui        # Poco 框架

# ========== 验证安装 ==========
python -c "import airtest; print(airtest.__version__)"

# ========== Android 调试桥 ==========
# 确保已安装 ADB
adb devices               # 查看连接设备

# Mac/Linux 权限问题
chmod +x airtest/core/android/static/adb/*/adb

# ========== AirtestIDE ==========
# 下载地址: http://airtest.netease.com/changelog.html
# 可视化开发环境，支持录制和调试
```

#### 3. 基础操作

```python
from airtest.core.api import *

# ========== 设备连接 ==========
# 连接 Android
connect_device("Android:///")                    # 默认设备
connect_device("Android://127.0.0.1:5037/设备序列号")

# 连接 iOS（需要 iOS-Tagent）
connect_device("iOS:///127.0.0.1:8100")

# 连接 Windows
connect_device("Windows:///")

# ========== 初始化设备 ==========
init_device(platform="Android", uuid="设备序列号")

# ========== 触摸操作 ==========
# 基于图像
touch(Template("button.png"))

# 基于坐标
touch((100, 200))

# 带参数
touch(Template("button.png"), times=2)          # 双击
touch(Template("button.png"), duration=2)       # 长按

# ========== 滑动操作 ==========
swipe((100, 500), (100, 100))                   # 坐标滑动
swipe(Template("start.png"), Template("end.png"))  # 图像滑动
swipe((100, 500), vector=[0, -0.5])             # 向量滑动

# ========== 文本输入 ==========
text("Hello World")
text("中文输入", enter=True)                     # 输入后回车

# ========== 等待 ==========
wait(Template("loading.png"))                   # 等待图像出现
wait(Template("loading.png"), timeout=30)       # 超时设置

# ========== 断言 ==========
assert_exists(Template("success.png"), "成功图标存在")
assert_not_exists(Template("error.png"), "错误图标不存在")

# ========== 截图 ==========
snapshot(filename="screenshot.png")
snapshot(msg="登录成功后的截图")

# ========== 关键点 ==========
keyevent("HOME")                                # Android 按键
keyevent("BACK")
keyevent("MENU")

# ========== 应用操作 ==========
start_app("com.example.app")                    # 启动应用
stop_app("com.example.app")                     # 停止应用
clear_app("com.example.app")                    # 清除数据
install("path/to/app.apk")                      # 安装应用
uninstall("com.example.app")                    # 卸载应用
```

#### 4. 图像识别与模板

```python
from airtest.core.api import *

# ========== Template 参数 ==========
touch(Template(
    "button.png",                    # 图片路径
    threshold=0.8,                   # 匹配阈值（0-1）
    target_pos=5,                    # 点击位置（九宫格 1-9）
    record_pos=(0, 0),               # 录制时的相对位置
    resolution=(1080, 1920)          # 目标分辨率
))

# ========== target_pos 九宫格 ==========
# 1 2 3
# 4 5 6
# 7 8 9
# 5 = 中心点（默认）

# ========== 常用场景 ==========
# 点击按钮中心
touch(Template("btn.png", target_pos=5))

# 点击按钮右侧
touch(Template("btn.png", target_pos=6))

# ========== exists 检查 ==========
# 检查元素是否存在（不抛异常）
pos = exists(Template("element.png"))
if pos:
    touch(pos)

# ========== 循环等待 ==========
# 等待元素出现后操作
wait(Template("element.png"), timeout=60, interval=1)
```

#### 5. Poco 框架基础

```python
from poco.drivers.android.uiautomation import AndroidUiautomationPoco

# ========== 初始化 Poco ==========
poco = AndroidUiautomationPoco()

# ========== 控件定位 ==========
# 基本选择器
poco("button")                        # 按 name
poco(text="登录")                      # 按 text 属性
poco(resourceName="com.app:id/btn")   # 按 resourceId

# 属性选择
poco(textMatches=".*登录.*")          # 正则匹配
poco(type="android.widget.Button")    # 按类型

# 层级选择
poco("parent").child("child")         # 子元素
poco("list").offspring("item")        # 后代元素
poco("item").sibling("next")          # 兄弟元素

# ========== 控件操作 ==========
poco("button").click()                # 点击
poco("input").set_text("Hello")       # 输入文本
poco("input").get_text()              # 获取文本
poco("switch").swipe([0, 0.5])        # 滑动开关

# ========== 断言 ==========
assert poco("element").exists()
assert poco("text").get_text() == "期望值"

# ========== 等待 ==========
poco("element").wait_for_appearance(timeout=10)
poco("element").wait_for_disappearance(timeout=10)

# ========== 列表操作 ==========
items = poco("ListView").children()
for item in items:
    print(item.get_text())

# ========== 获取属性 ==========
element = poco("button")
element.attr('text')                  # 获取 text 属性
element.attr('visible')               # 是否可见
element.attr('enabled')               # 是否可用
element.get_position()                # 获取位置
element.get_size()                    # 获取大小
```

---

### 🟠 重要

#### 6. 高级操作

```python
from airtest.core.api import *
from poco.drivers.android.uiautomation import AndroidUiautomationPoco

poco = AndroidUiautomationPoco()

# ========== 手势操作 ==========
# 双指缩放
pinch(in_or_out='in', center=(500, 500))   # 放大
pinch(in_or_out='out', center=(500, 500))  # 缩小

# 长按
long_press = (100, 200)
touch(long_press, duration=2000)

# 多点触控
multitouch_event = [
    TouchEvent((100, 100), 'down'),
    TouchEvent((200, 200), 'down'),
    TouchEvent((100, 100), 'up'),
    TouchEvent((200, 200), 'up'),
]
device.minitouch.perform(multitouch_event)

# ========== 滚动 ==========
# 列表滚动
poco("ListView").swipe([0, -0.5])      # 向上滑动
poco("ListView").swipe([0, 0.5])       # 向下滑动

# 滚动到指定元素
poco.scroll_until(poco("target_element"))

# ========== 坐标转换 ==========
# 屏幕坐标 (0-1) 转 像素坐标
pos = poco("element").get_position()  # 返回 (0.5, 0.3)
pixel_pos = (pos[0] * screen_width, pos[1] * screen_height)

# ========== 截图与日志 ==========
# 带描述的截图
snapshot(filename="step1.png", msg="第一步完成")

# 日志
from airtest.core.helper import G
G.LOGGER.info("自定义日志信息")
```

#### 7. 多设备管理

```python
from airtest.core.api import *
from airtest.core.android.android import Android

# ========== 连接多设备 ==========
dev1 = connect_device("Android:///设备1序列号")
dev2 = connect_device("Android:///设备2序列号")

# ========== 切换设备 ==========
set_current("设备1序列号")
touch(Template("btn.png"))

set_current("设备2序列号")
touch(Template("btn.png"))

# ========== 获取当前设备 ==========
current = device()
print(current.serial)

# ========== 获取所有设备 ==========
from airtest.core.api import G
for dev in G.DEVICE_LIST:
    print(dev.serial)
```

#### 8. 测试报告

```python
from airtest.core.api import *
from airtest.report.report import simple_report

# ========== 生成报告 ==========
# 简单报告
simple_report(__file__, logpath=True, output="report.html")

# 自定义报告
from airtest.report.report import LogToHtml

h = LogToHtml(script_root=".", log_root="./log")
h.report(output_file="custom_report.html")

# ========== 报告内容 ==========
# 自动包含：
# - 截图
# - 操作步骤
# - 断言结果
# - 执行时间
# - 错误信息

# ========== 运行命令 ==========
# airtest run test.py --log logs/
# airtest report test.py --log_root logs/ --output report.html
```

#### 9. 异常处理

```python
from airtest.core.api import *
from airtest.core.error import *

try:
    touch(Template("button.png"))

except TargetNotFoundError as e:
    print(f"元素未找到: {e}")
    snapshot("error_not_found.png")

except AirtestError as e:
    print(f"Airtest 错误: {e}")

except Exception as e:
    print(f"未知错误: {e}")
    raise

# ========== 超时处理 ==========
try:
    wait(Template("element.png"), timeout=30)
except TargetNotFoundError:
    print("等待超时，元素未出现")
```

---

### 🟡 常用

#### 10. 游戏测试

```python
from airtest.core.api import *
from poco.drivers.unity3d import UnityPoco

# ========== Unity 游戏测试 ==========
# 连接 Unity 游戏
poco = UnityPoco()

# 游戏控件操作
poco("StartButton").click()
poco("Player").attr("hp")
poco("Enemy").wait_for_appearance()

# ========== Cocos2d 游戏 ==========
from poco.drivers.cocosjs import CocosJsPoco
poco = CocosJsPoco()

# ========== 图像识别优先 ==========
# 游戏中常用图像识别
touch(Template("skill_1.png"))
wait(Template("boss_appear.png"))
assert_exists(Template("victory.png"))

# ========== 性能监控 ==========
# 获取 FPS
fps = device().get_fps()
print(f"当前 FPS: {fps}")

# 获取内存
mem = device().get_memory()
print(f"内存占用: {mem}")
```

#### 11. iOS 测试

```python
from airtest.core.api import *
from airtest.core.ios import IOS

# ========== 连接 iOS ==========
# 需要 iOS-Tagent 和 WebDriverAgent
connect_device("iOS:///127.0.0.1:8100")

# ========== iOS 操作 ==========
# 基本操作与 Android 相同
touch(Template("button.png"))
swipe((100, 500), (100, 100))
text("Hello")

# ========== iOS 特有 ==========
# Home 键
home()

# Siri
device().siri_activate("打开设置")

# ========== Poco iOS ==========
from poco.drivers.ios import IOSPoco
poco = IOSPoco()
poco("按钮").click()
```

#### 12. Web 测试

```python
from airtest.core.api import *
from airtest.core.browser import Browser

# ========== 连接浏览器 ==========
# 需要 ChromeDriver
connect_device("browser:chrome")

# ========== Web 操作 ==========
start_app("https://example.com")
touch(Template("login_btn.png"))
text("username", enter=False)
```

---

### 🟢 了解

#### 13. 高级特性

```python
# ========== 性能测试 ==========
# 内存监控
mem_info = device().get_memory_info()
print(mem_info)

# CPU 使用
cpu_info = device().get_cpu_usage()
print(cpu_info)

# ========== 录屏 ==========
# 开始录屏
device().start_recording()

# 停止录屏
device().stop_recording("test.mp4")

# ========== 图像识别阈值 ==========
# 全局阈值设置
from airtest.core.setting import Settings
Settings.THRESHOLD = 0.7
Settings.THRESHOLD_STRICT = 0.9

# ========== 并行执行 ==========
# 使用 pytest-xdist
# pytest -n 4 tests/
```

#### 14. CI/CD 集成

```yaml
# ========== Jenkins Pipeline ==========
pipeline {
    agent any
    stages {
        stage('Airtest Test') {
            steps {
                sh 'pip install airtest pocoui'
                sh 'airtest run tests/ --log logs/'
                sh 'airtest report tests/ --log_root logs/ --output report.html'
            }
            post {
                always {
                    publishHTML([
                        reportDir: '.',
                        reportFiles: 'report.html',
                        reportName: 'Airtest Report'
                    ])
                }
            }
        }
    }
}

# ========== GitHub Actions ==========
name: Airtest Tests
on: [push]
jobs:
  test:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - run: pip install airtest pocoui
      - run: airtest run tests/ --log logs/
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 答案 |
|------|------|
| Airtest 是什么？ | 网易开源的跨平台 UI 自动化框架，基于图像识别 |
| Airtest 和 Appium 区别？ | Airtest 基于图像，Appium 基于控件属性 |
| 如何连接 Android 设备？ | `connect_device("Android:///")` |
| 如何点击图像？ | `touch(Template("image.png"))` |
| 如何输入文本？ | `text("内容")` |

### 进阶篇

| 问题 | 答案 |
|------|------|
| Poco 和 Airtest 的关系？ | Poco 基于控件树，Airtest 基于图像，可配合使用 |
| 如何提高图像识别准确率？ | 调整 threshold、使用合适的分辨率截图 |
| 如何处理动态元素？ | 使用 Poco 控件定位或相对定位 |
| 如何生成测试报告？ | `simple_report()` 或命令行 |
| 如何处理弹窗？ | `exists()` 检查 + `touch()` 处理 |

### 高级篇

| 问题 | 答案 |
|------|------|
| 如何测试游戏？ | 图像识别 + UnityPoco/CocosPoco |
| 如何处理多设备？ | `connect_device()` + `set_current()` |
| 如何集成 CI/CD？ | 命令行执行 + 报告发布 |
| 图像识别失败怎么办？ | 调整阈值、优化截图、使用 Poco |

---

## 四、实战场景

### 场景1：App 登录测试

```python
from airtest.core.api import *
from poco.drivers.android.uiautomation import AndroidUiautomationPoco

class LoginTest:
    def __init__(self):
        connect_device("Android:///")
        self.poco = AndroidUiautomationPoco()

    def login(self, username, password):
        """登录操作"""
        # 启动应用
        start_app("com.example.app")

        # 等待登录页面
        wait(Template("login_page.png"))

        # 输入用户名密码
        self.poco("username_input").set_text(username)
        self.poco("password_input").set_text(password)

        # 点击登录
        self.poco("login_button").click()

        # 验证登录成功
        assert_exists(Template("home_page.png"), "登录成功")

    def logout(self):
        """退出登录"""
        self.poco("profile").click()
        self.poco("logout").click()
        assert_exists(Template("login_page.png"), "退出成功")

# 使用
test = LoginTest()
test.login("testuser", "password123")
test.logout()
```

### 场景2：列表滑动测试

```python
from airtest.core.api import *
from poco.drivers.android.uiautomation import AndroidUiautomationPoco

poco = AndroidUiautomationPoco()

def scroll_and_find(target_text, max_swipes=10):
    """滑动查找元素"""
    for i in range(max_swipes):
        # 检查是否在当前屏幕
        target = poco(text=target_text)
        if target.exists():
            return target

        # 向上滑动
        poco("RecyclerView").swipe([0, -0.5])
        sleep(1)

    return None

# 使用
element = scroll_and_find("目标项")
if element:
    element.click()
else:
    print("未找到目标元素")
```

### 场景3：异常处理封装

```python
from airtest.core.api import *
from airtest.core.error import TargetNotFoundError

def safe_touch(image, retry=3, timeout=10):
    """安全点击，带重试"""
    for i in range(retry):
        try:
            wait(image, timeout=timeout)
            touch(image)
            return True
        except TargetNotFoundError:
            print(f"第 {i+1} 次尝试失败")
            if i < retry - 1:
                sleep(2)
    return False

def safe_input(poco_element, text, retry=3):
    """安全输入，带重试"""
    for i in range(retry):
        try:
            if poco_element.exists():
                poco_element.set_text(text)
                return True
        except Exception as e:
            print(f"输入失败: {e}")
    return False

# 使用
if safe_touch(Template("submit.png")):
    print("点击成功")
else:
    print("点击失败")
```

### 场景4：测试数据驱动

```python
import pytest
from airtest.core.api import *
from poco.drivers.android.uiautomation import AndroidUiautomationPoco

# 测试数据
test_data = [
    {"username": "user1", "password": "pass1", "expected": "success"},
    {"username": "user2", "password": "wrong", "expected": "fail"},
    {"username": "", "password": "pass1", "expected": "fail"},
]

class TestLogin:
    @pytest.fixture(autouse=True)
    def setup(self):
        connect_device("Android:///")
        self.poco = AndroidUiautomationPoco()
        start_app("com.example.app")

    @pytest.mark.parametrize("data", test_data)
    def test_login(self, data):
        self.poco("username").set_text(data["username"])
        self.poco("password").set_text(data["password"])
        self.poco("login").click()

        if data["expected"] == "success":
            assert_exists(Template("home.png"))
        else:
            assert_exists(Template("error.png"))
```

---

## 五、Airtest 速查表

### 基础操作

| 操作 | 代码 |
|------|------|
| 连接设备 | `connect_device("Android:///")` |
| 点击图像 | `touch(Template("img.png"))` |
| 点击坐标 | `touch((x, y))` |
| 滑动 | `swipe((x1,y1), (x2,y2))` |
| 输入文本 | `text("内容")` |
| 等待 | `wait(Template("img.png"))` |
| 截图 | `snapshot()` |

### Poco 操作

| 操作 | 代码 |
|------|------|
| 定位 | `poco("name")` |
| 点击 | `poco("btn").click()` |
| 输入 | `poco("input").set_text("text")` |
| 获取文本 | `poco("label").get_text()` |
| 等待出现 | `poco("el").wait_for_appearance()` |
| 检查存在 | `poco("el").exists()` |

### 断言

| 操作 | 代码 |
|------|------|
| 存在 | `assert_exists(Template())` |
| 不存在 | `assert_not_exists(Template())` |
| 相等 | `assert actual == expected` |

### 应用操作

| 操作 | 代码 |
|------|------|
| 启动 | `start_app("package")` |
| 停止 | `stop_app("package")` |
| 安装 | `install("path.apk")` |
| 卸载 | `uninstall("package")` |

---

## 相关知识点

- [[Playwright 测试框架精通指南]]
- [[Pytest 面试完全指南]]
- [[Linux 命令测试工程师精通指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [Airtest 官方文档](https://airtest.doc.io.netease.com/)
- [Airtest GitHub](https://github.com/AirtestProject/Airtest)
- [Poco GitHub](https://github.com/AirtestProject/Poco)
