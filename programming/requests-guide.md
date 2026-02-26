# Python Requests 测试工程师精通指南

> 标签: #python #requests #api-testing #http #面试
> 创建时间: 2026-02-26
> 来源: [Requests官方文档](https://docs.python-requests.org/) | [Real Python](https://realpython.com/python-requests/)

## 概述

Requests 是 Python 最流行的 HTTP 客户端库，是 API 测试的核心工具。简洁优雅的 API 设计使其成为测试工程师必备技能。本文整理测试面试高频 Requests 知识点，按重要程度分类。

---

## 一、知识体系总览

### 掌握程度分类

| 级别 | 说明 | 面试权重 |
|------|------|----------|
| 🔴 必须掌握 | 面试必问，日常必用 | 40% |
| 🟠 重要 | 常见考点，需要熟练 | 30% |
| 🟡 常用 | 工作中频繁使用 | 20% |
| 🟢 了解 | 高级场景，知道即可 | 10% |

---

## 二、核心知识点

### 🔴 必须掌握

#### 1. 基础请求

```python
import requests

# ========== GET 请求 ==========
# 基础 GET
response = requests.get('https://api.example.com/users')

# 带参数
params = {'page': 1, 'size': 10}
response = requests.get('https://api.example.com/users', params=params)

# 带 Headers
headers = {'User-Agent': 'MyApp/1.0'}
response = requests.get('https://api.example.com/users', headers=headers)

# ========== POST 请求 ==========
# JSON 数据
data = {'name': '张三', 'age': 25}
response = requests.post('https://api.example.com/users', json=data)

# Form 表单
data = {'username': 'admin', 'password': '123456'}
response = requests.post('https://api.example.com/login', data=data)

# ========== 其他方法 ==========
response = requests.put('https://api.example.com/users/1', json=data)
response = requests.patch('https://api.example.com/users/1', json=data)
response = requests.delete('https://api.example.com/users/1')
response = requests.head('https://api.example.com')
response = requests.options('https://api.example.com')
```

#### 2. 响应处理

```python
# ========== 响应属性 ==========
response.status_code          # 状态码：200, 404, 500 等
response.ok                   # 状态码是否为 2xx
response.reason               # 状态描述：OK, Not Found 等
response.headers              # 响应头（字典）
response.cookies              # Cookies
response.url                  # 最终 URL（处理重定向后）
response.elapsed              # 请求耗时

# ========== 响应内容 ==========
response.text                 # 文本内容（自动解码）
response.content              # 字节内容（二进制）
response.json()               # JSON 解析（常用！）

# 示例
response = requests.get('https://api.example.com/users/1')
if response.status_code == 200:
    data = response.json()
    print(data['name'])

# ========== 状态码判断 ==========
if response.status_code == 200:
    print('成功')
elif response.status_code == 404:
    print('资源不存在')
elif response.status_code == 500:
    print('服务器错误')

# 推荐方式
if response.ok:
    print('请求成功')

# 状态码抛出异常
response.raise_for_status()   # 非 2xx 抛出 HTTPError
```

#### 3. 请求头与认证

```python
# ========== 常用 Headers ==========
headers = {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'User-Agent': 'MyTestClient/1.0',
    'Accept-Language': 'zh-CN,zh;q=0.9',
}

response = requests.get('https://api.example.com', headers=headers)

# ========== Basic Auth ==========
from requests.auth import HTTPBasicAuth

# 方式1
response = requests.get('https://api.example.com',
                        auth=HTTPBasicAuth('user', 'pass'))

# 方式2
response = requests.get('https://api.example.com',
                        auth=('user', 'pass'))

# ========== Bearer Token ==========
headers = {'Authorization': 'Bearer your_token_here'}
response = requests.get('https://api.example.com', headers=headers)

# ========== API Key ==========
# Header 方式
headers = {'X-API-Key': 'your_api_key'}
response = requests.get('https://api.example.com', headers=headers)

# Query 参数方式
params = {'api_key': 'your_api_key'}
response = requests.get('https://api.example.com', params=params)
```

#### 4. 异常处理

```python
import requests
from requests.exceptions import (
    RequestException,
    HTTPError,
    ConnectionError,
    Timeout,
    TooManyRedirects
)

try:
    response = requests.get('https://api.example.com', timeout=5)
    response.raise_for_status()  # 非 2xx 抛出异常
    data = response.json()

except HTTPError as e:
    print(f'HTTP 错误: {e}')
except ConnectionError:
    print('连接错误')
except Timeout:
    print('请求超时')
except RequestException as e:
    print(f'请求异常: {e}')

# 推荐的完整模式
def safe_request(url, method='GET', **kwargs):
    try:
        response = requests.request(method, url, **kwargs)
        response.raise_for_status()
        return response.json()
    except HTTPError as e:
        print(f'HTTP 错误: {e.response.status_code}')
        return None
    except Exception as e:
        print(f'请求失败: {e}')
        return None
```

#### 5. 超时与重试

```python
# ========== 超时设置 ==========
# 连接超时 + 读取超时
response = requests.get(url, timeout=5)

# 分别设置
response = requests.get(url, timeout=(3, 10))  # 连接3秒，读取10秒

# ========== 重试机制 ==========
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# 创建 session 并配置重试
session = requests.Session()

retry_strategy = Retry(
    total=3,                    # 总重试次数
    backoff_factor=1,           # 重试间隔因子
    status_forcelist=[500, 502, 503, 504],  # 触发重试的状态码
    allowed_methods=["GET", "POST"]
)

adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("http://", adapter)
session.mount("https://", adapter)

response = session.get(url)
```

---

### 🟠 重要

#### 6. Session 管理

```python
# ========== Session 简介 ==========
# Session 可以跨请求保持某些参数（如 Cookies）
# 也可以复用 TCP 连接，提高性能

session = requests.Session()

# 设置全局 headers
session.headers.update({
    'User-Agent': 'MyApp/1.0',
    'Authorization': 'Bearer token'
})

# 登录获取 cookie
login_resp = session.post('https://api.example.com/login',
                          json={'username': 'admin', 'password': '123'})
# 后续请求自动携带 cookie
profile_resp = session.get('https://api.example.com/profile')

# ========== 手动管理 Cookies ==========
session.cookies.set('session_id', 'abc123')
session.cookies.get('session_id')

# ========== 上下文管理 ==========
with requests.Session() as session:
    session.post(login_url, json=credentials)
    response = session.get(profile_url)

# ========== 测试登录态复用 ==========
class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url
        self.session = requests.Session()

    def login(self, username, password):
        resp = self.session.post(f'{self.base_url}/login',
                                  json={'username': username, 'password': password})
        return resp.ok

    def get_profile(self):
        return self.session.get(f'{self.base_url}/profile').json()
```

#### 7. 文件操作

```python
# ========== 文件上传 ==========
# 单文件
files = {'file': open('report.pdf', 'rb')}
response = requests.post('https://api.example.com/upload', files=files)

# 指定文件名和类型
files = {'file': ('report.pdf', open('report.pdf', 'rb'), 'application/pdf')}
response = requests.post(url, files=files)

# 多文件
files = [
    ('files', ('file1.txt', open('file1.txt', 'rb'))),
    ('files', ('file2.txt', open('file2.txt', 'rb')))
]
response = requests.post(url, files=files)

# 上传 + 其他字段
files = {'file': open('data.csv', 'rb')}
data = {'description': '测试数据'}
response = requests.post(url, files=files, data=data)

# ========== 文件下载 ==========
# 小文件
response = requests.get(file_url)
with open('downloaded_file.zip', 'wb') as f:
    f.write(response.content)

# 大文件（流式下载）
response = requests.get(file_url, stream=True)
with open('large_file.zip', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)

# 带进度条
from tqdm import tqdm

response = requests.get(file_url, stream=True)
total_size = int(response.headers.get('content-length', 0))

with open('file.zip', 'wb') as f:
    with tqdm(total=total_size, unit='B', unit_scale=True) as pbar:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
            pbar.update(len(chunk))
```

#### 8. Cookie 处理

```python
# ========== 获取 Cookies ==========
response = requests.get(url)

# 获取所有 cookies
cookies = response.cookies
cookies_dict = response.cookies.get_dict()

# 获取单个 cookie
session_id = response.cookies.get('session_id')

# ========== 发送 Cookies ==========
# 方式1：字典
cookies = {'session_id': 'abc123', 'user_id': '1'}
response = requests.get(url, cookies=cookies)

# 方式2：Session 自动管理
session = requests.Session()
session.get(login_url)
# 后续请求自动携带 cookies

# ========== CookieJar ==========
from http.cookiejar import CookieJar

jar = requests.cookies.RequestsCookieJar()
jar.set('name', 'value', domain='.example.com', path='/')
response = requests.get(url, cookies=jar)
```

#### 9. 代理与 SSL

```python
# ========== 代理设置 ==========
proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'http://proxy.example.com:8080',
}
response = requests.get(url, proxies=proxies)

# 带认证的代理
proxies = {
    'http': 'http://user:pass@proxy.example.com:8080',
    'https': 'http://user:pass@proxy.example.com:8080',
}

# ========== SSL 验证 ==========
# 跳过 SSL 验证（不推荐生产使用）
response = requests.get(url, verify=False)

# 指定证书
response = requests.get(url, verify='/path/to/certfile')

# 客户端证书
response = requests.get(url, cert=('/path/client.cert', '/path/client.key'))
```

---

### 🟡 常用

#### 10. 高级特性

```python
# ========== 流式请求 ==========
# 流式响应（大响应体）
response = requests.get(url, stream=True)
for line in response.iter_lines():
    print(line.decode('utf-8'))

# 流式上传
with open('large_file.dat', 'rb') as f:
    response = requests.post(url, data=f)

# ========== 钩子 ==========
def log_request(response, *args, **kwargs):
    print(f'请求 URL: {response.request.url}')
    print(f'状态码: {response.status_code}')
    return response

response = requests.get(url, hooks={'response': log_request})

# ========== 准备请求 ==========
from requests import Request, Session

req = Request('GET', url, headers={'User-Agent': 'MyApp'})
prepared = req.prepare()
response = Session().send(prepared)
```

#### 11. 响应时间与性能

```python
# ========== 测量响应时间 ==========
import time

start = time.time()
response = requests.get(url)
elapsed = time.time() - start
print(f'请求耗时: {elapsed:.2f}s')

# 使用内置属性
print(f'请求耗时: {response.elapsed.total_seconds():.2f}s')

# ========== 性能测试示例 ==========
def measure_api_performance(url, n=10):
    times = []
    for _ in range(n):
        start = time.time()
        response = requests.get(url)
        times.append(time.time() - start)

    return {
        'avg': sum(times) / len(times),
        'min': min(times),
        'max': max(times),
    }
```

#### 12. 与 Pytest 集成

```python
import pytest
import requests

# ========== 基础 API 测试 ==========
class TestUserAPI:
    BASE_URL = 'https://api.example.com'

    def test_get_users(self):
        response = requests.get(f'{self.BASE_URL}/users')
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    def test_create_user(self):
        data = {'name': '测试用户', 'email': 'test@example.com'}
        response = requests.post(f'{self.BASE_URL}/users', json=data)
        assert response.status_code == 201
        assert response.json()['name'] == '测试用户'

# ========== 参数化测试 ==========
@pytest.mark.parametrize('user_id,expected_status', [
    (1, 200),
    (999, 404),
    (-1, 400),
])
def test_get_user_by_id(user_id, expected_status):
    response = requests.get(f'https://api.example.com/users/{user_id}')
    assert response.status_code == expected_status

# ========== Fixtures ==========
@pytest.fixture
def auth_token():
    response = requests.post('https://api.example.com/login',
                             json={'username': 'test', 'password': 'test'})
    return response.json()['token']

def test_with_auth(auth_token):
    headers = {'Authorization': f'Bearer {auth_token}'}
    response = requests.get('https://api.example.com/profile', headers=headers)
    assert response.ok
```

---

### 🟢 了解

#### 13. 高级主题

```python
# ========== 异步请求 ==========
# 使用 aiohttp 或 httpx
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# ========== 并发请求 ==========
import concurrent.futures

urls = ['url1', 'url2', 'url3']

with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(requests.get, url) for url in urls]
    results = [f.result() for f in futures]

# ========== Mock 测试 ==========
import responses

@responses.activate
def test_mock_api():
    responses.add(
        responses.GET,
        'https://api.example.com/users',
        json={'users': []},
        status=200
    )

    response = requests.get('https://api.example.com/users')
    assert response.status_code == 200
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 答案 |
|------|------|
| requests 如何发送 GET 请求？ | `requests.get(url, params=params)` |
| 如何发送 JSON 数据？ | `requests.post(url, json=data)` |
| 如何获取响应 JSON？ | `response.json()` |
| 如何处理请求异常？ | `try-except` + `raise_for_status()` |
| 如何设置超时？ | `requests.get(url, timeout=5)` |

### 进阶篇

| 问题 | 答案 |
|------|------|
| Session 的作用？ | 保持会话状态（Cookies）、复用连接 |
| 如何上传文件？ | `files={'file': open(path, 'rb')}` |
| 如何设置代理？ | `proxies={'http': 'http://proxy:port'}` |
| json= 和 data= 区别？ | json 发送 JSON，data 发送表单 |
| 如何实现重试？ | `HTTPAdapter` + `Retry` |

### 高级篇

| 问题 | 答案 |
|------|------|
| 如何处理大文件下载？ | `stream=True` + `iter_content()` |
| 如何实现并发请求？ | `ThreadPoolExecutor` 或 `aiohttp` |
| 如何测试需要登录的接口？ | Session 管理登录态 |
| 与 Postman 相比的优缺点？ | 代码化、可版本控制、易集成 CI |

---

## 四、实战场景

### 场景1：API 测试框架

```python
import requests
import pytest

class APIClient:
    """API 测试客户端"""

    def __init__(self, base_url):
        self.base_url = base_url
        self.session = requests.Session()
        self.token = None

    def set_auth(self, token):
        """设置认证 Token"""
        self.token = token
        self.session.headers.update({
            'Authorization': f'Bearer {token}'
        })

    def login(self, username, password):
        """登录"""
        resp = self.session.post(
            f'{self.base_url}/login',
            json={'username': username, 'password': password}
        )
        if resp.ok:
            self.set_auth(resp.json()['token'])
        return resp

    def get(self, endpoint, **kwargs):
        """GET 请求"""
        return self.session.get(f'{self.base_url}{endpoint}', **kwargs)

    def post(self, endpoint, **kwargs):
        """POST 请求"""
        return self.session.post(f'{self.base_url}{endpoint}', **kwargs)

# 使用
client = APIClient('https://api.example.com')
client.login('testuser', 'password')
response = client.get('/users')
```

### 场景2：接口响应断言

```python
def assert_response(response, status_code=200, json_schema=None):
    """统一响应断言"""
    # 状态码断言
    assert response.status_code == status_code, \
        f'状态码错误: 期望 {status_code}, 实际 {response.status_code}'

    # JSON 解析断言
    try:
        data = response.json()
    except Exception:
        raise AssertionError('响应不是有效的 JSON')

    # 响应时间断言
    assert response.elapsed.total_seconds() < 5, '响应超时'

    return data

# 使用
response = requests.get(url)
data = assert_response(response, status_code=200)
assert 'users' in data
```

### 场景3：数据驱动测试

```python
import pytest
import requests

# 测试数据
test_cases = [
    {
        'name': '正常登录',
        'data': {'username': 'admin', 'password': '123456'},
        'expected_status': 200
    },
    {
        'name': '密码错误',
        'data': {'username': 'admin', 'password': 'wrong'},
        'expected_status': 401
    },
    {
        'name': '用户名为空',
        'data': {'username': '', 'password': '123456'},
        'expected_status': 400
    },
]

@pytest.mark.parametrize('case', test_cases, ids=[c['name'] for c in test_cases])
def test_login(case):
    response = requests.post('https://api.example.com/login', json=case['data'])
    assert response.status_code == case['expected_status']
```

### 场景4：接口性能监控

```python
import time
import statistics

def monitor_api_performance(url, iterations=100):
    """API 性能监控"""
    response_times = []
    success_count = 0

    for _ in range(iterations):
        try:
            start = time.time()
            response = requests.get(url, timeout=10)
            elapsed = time.time() - start

            response_times.append(elapsed)
            if response.ok:
                success_count += 1

        except Exception as e:
            print(f'请求失败: {e}')

    return {
        'total_requests': iterations,
        'success_rate': success_count / iterations * 100,
        'avg_time': statistics.mean(response_times),
        'min_time': min(response_times),
        'max_time': max(response_times),
        'p95': sorted(response_times)[int(len(response_times) * 0.95)]
    }
```

---

## 五、Requests 速查表

### 请求方法

| 方法 | 代码 |
|------|------|
| GET | `requests.get(url, params={})` |
| POST | `requests.post(url, json={})` |
| PUT | `requests.put(url, json={})` |
| DELETE | `requests.delete(url)` |

### 常用参数

| 参数 | 说明 |
|------|------|
| `params` | URL 查询参数 |
| `json` | JSON 请求体 |
| `data` | 表单数据 |
| `headers` | 请求头 |
| `cookies` | Cookies |
| `timeout` | 超时时间 |
| `files` | 上传文件 |
| `auth` | 认证信息 |

### 响应属性

| 属性 | 说明 |
|------|------|
| `status_code` | 状态码 |
| `ok` | 是否成功 |
| `text` | 文本内容 |
| `content` | 字节内容 |
| `json()` | JSON 解析 |
| `headers` | 响应头 |
| `cookies` | Cookies |
| `elapsed` | 耗时 |

---

## 相关知识点

- [[Python Pandas 精通指南]]
- [[Pytest 面试完全指南]]
- [[SQL 命令测试工程师精通指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [Requests 官方文档](https://docs.python-requests.org/)
- [Real Python - Python Requests](https://realpython.com/python-requests/)
- [API接口测试Python框架全攻略](https://blog.csdn.net/)
