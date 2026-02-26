# Playwright 测试框架精通指南

> 标签: #playwright #automation #testing #e2e #面试
> 创建时间: 2026-02-26
> 来源: [Playwright官方文档](https://playwright.dev/) | [GitHub](https://github.com/microsoft/playwright)

## 概述

Playwright 是微软开发的现代端到端测试框架，支持 Chromium、Firefox、WebKit 三大浏览器引擎，提供跨浏览器、跨语言的自动化测试能力。以其自动等待、并行执行、网络拦截等特性成为 2025 年最热门的测试框架之一。

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
| 多语言支持 | JS/TS, Python, Java, .NET | Java, Python, C#, Ruby等 | 仅 JS/TS |
| 执行速度 | 快 | 较慢 | 快 |
| 移动端模拟 | ✅ 内置 | ✅ 需 Appium | ❌ 有限 |
| API 测试 | ✅ 内置 | ❌ 需额外工具 | ✅ 内置 |
| Shadow DOM | ✅ 支持 | ❌ 困难 | ✅ 支持 |
| 网络拦截 | ✅ 强大 | ❌ 有限 | ✅ 支持 |

**面试回答要点：**
- 选择 Playwright 的原因：跨浏览器原生支持、自动等待机制、现代 Web 特性支持、微软维护
- 相比 Cypress：支持 Safari、支持多语言、网络控制更强
- 相比 Selenium：更快、更稳定、API 更现代

#### 2. 定位器 (Locators)

```javascript
// ============ 推荐的定位方式（按优先级）============

// 1. Role 定位（最推荐，语义化）
page.getByRole('button', { name: '提交' })
page.getByRole('textbox', { name: '用户名' })
page.getByRole('link', { name: '登录' })

// 2. Test ID 定位（稳定，需开发配合）
page.getByTestId('submit-button')
page.getByTestId('login-form')

// 3. 文本定位
page.getByText('欢迎登录')
page.getByText(/欢迎\s*登录/)  // 正则

// 4. Label 定位（表单元素）
page.getByLabel('密码')

// 5. Placeholder 定位
page.getByPlaceholder('请输入用户名')

// 6. CSS 选择器
page.locator('#username')
page.locator('.btn-primary')
page.locator('form > button[type="submit"]')

// 7. 组合定位
page.locator('article').filter({ hasText: 'Playwright' })
page.locator('.card').getByRole('button')

// 8. XPath（不推荐，作为备选）
page.locator('//button[@type="submit"]')
```

**面试考点：**
- 为什么优先使用 `getByRole`？→ 语义化、可访问性、稳定性
- 为什么避免 XPath？→ 脆弱、难维护、性能差
- 如何定位 Shadow DOM 元素？→ Playwright 原生支持，直接定位

#### 3. 断言 (Assertions)

```javascript
import { test, expect } from '@playwright/test';

test('断言示例', async ({ page }) => {
  // ----- 页面级断言 -----
  await expect(page).toHaveURL(/dashboard/);
  await expect(page).toHaveTitle(/Playwright/);
  await expect(page).toHaveScreenshot('homepage.png');  // 视觉回归

  // ----- 元素可见性断言 -----
  await expect(locator).toBeVisible();
  await expect(locator).toBeHidden();
  await expect(locator).toBeEnabled();
  await expect(locator).toBeDisabled();
  await expect(locator).toBeEditable();

  // ----- 文本内容断言 -----
  await expect(locator).toHaveText('欢迎登录');
  await expect(locator).toHaveText(/欢迎\s*登录/);  // 正则
  await expect(locator).toContainText('登录');

  // ----- 属性断言 -----
  await expect(locator).toHaveAttribute('href', '/docs');
  await expect(locator).toHaveClass(/active/);
  await expect(locator).toHaveCSS('color', 'rgb(255, 0, 0)');
  await expect(locator).toHaveValue('input value');

  // ----- 数量断言 -----
  await expect(locator).toHaveCount(5);

  // ----- 自定义超时 -----
  await expect(locator).toBeVisible({ timeout: 10000 });

  // ----- 否定断言 -----
  await expect(locator).not.toBeVisible();
});
```

**面试考点：**
- Playwright 断言是自动重试的吗？→ 是，默认 5 秒内重试
- `toHaveText` vs `toContainText`？→ 完全匹配 vs 包含
- 如何处理异步断言？→ 使用 `await`，Playwright 自动等待

#### 4. 自动等待机制

```javascript
// Playwright 自动等待，无需显式 sleep
test('自动等待示例', async ({ page }) => {
  // 这些操作都会自动等待元素可操作
  await page.click('button');          // 等待元素可见、可点击
  await page.fill('#input', 'text');   // 等待元素可见、可编辑
  await page.locator('.item').first().click();

  // 显式等待特定条件
  await page.waitForSelector('.loaded');
  await page.waitForLoadState('networkidle');  // 等待网络空闲
  await page.waitForURL(/dashboard/);
  await page.waitForResponse(resp => resp.url().includes('/api/'));

  // 等待元素状态
  await expect(page.locator('.loading')).toBeHidden();
});
```

**面试考点：**
- 为什么不需要 `sleep()`？→ Playwright 内置自动等待
- 什么时候需要显式等待？→ 复杂异步场景、网络请求、页面跳转
- `waitForLoadState` 的三种状态？→ `domcontentloaded`、`load`、`networkidle`

#### 5. Page Object Model (POM)

```javascript
// ============ pages/LoginPage.ts ============
import { Locator, Page } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly usernameInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.usernameInput = page.getByLabel('用户名');
    this.passwordInput = page.getByLabel('密码');
    this.loginButton = page.getByRole('button', { name: '登录' });
    this.errorMessage = page.locator('.error-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(username: string, password: string) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}

// ============ tests/login.spec.ts ============
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('登录成功', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('testuser', 'password123');

  await expect(page).toHaveURL(/dashboard/);
});
```

**面试考点：**
- 为什么要用 POM？→ 代码复用、易于维护、职责分离
- Page 类应该包含断言吗？→ 不应该，断言放在测试文件中
- 如何处理公共组件？→ 抽取 BasePage 或 Component 类

---

### 🟠 重要

#### 6. 多窗口/多标签页处理

```javascript
test('多窗口处理', async ({ page, context }) => {
  // 方式一：监听新页面
  const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.click('a[target="_blank"]')
  ]);
  await newPage.waitForLoadState();
  await expect(newPage).toHaveTitle(/新页面/);
  await newPage.close();

  // 方式二：获取所有页面
  const allPages = context.pages();
  const popupPage = allPages.find(p => p.url().includes('popup'));
});
```

#### 7. iframe 处理

```javascript
test('iframe 处理', async ({ page }) => {
  // 获取 frame
  const frame = page.frameLocator('#myframe');

  // 在 frame 中操作
  await frame.getByRole('button', { name: '提交' }).click();
  await frame.locator('#input').fill('text');

  // 嵌套 iframe
  const nestedFrame = frame.frameLocator('.inner-frame');
  await nestedFrame.getByText('内容').click();
});
```

#### 8. 网络拦截与 Mock

```javascript
test('网络拦截', async ({ page }) => {
  // 拦截并 Mock 响应
  await page.route('**/api/user', async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({ name: 'Mock User', id: 123 })
    });
  });

  // 拦截并修改请求
  await page.route('**/api/login', async route => {
    const request = route.request();
    await route.continue({
      headers: { ...request.headers(), 'Authorization': 'Bearer token' }
    });
  });

  // 拦截并 abort
  await page.route('**/analytics/**', route => route.abort());

  // 模拟离线
  await context.setOffline(true);

  await page.goto('/');
});
```

#### 9. 文件上传与下载

```javascript
test('文件操作', async ({ page }) => {
  // ----- 文件上传 -----
  // 单文件
  await page.setInputFiles('input[type="file"]', 'tests/fixtures/test.pdf');

  // 多文件
  await page.setInputFiles('input[type="file"]', ['file1.pdf', 'file2.pdf']);

  // 清空文件
  await page.setInputFiles('input[type="file"]', []);

  // ----- 文件下载 -----
  const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.click('a[download]')
  ]);

  const path = await download.path();
  const fileName = download.suggestedFilename();
  await download.saveAs('downloads/' + fileName);
});
```

#### 10. API 测试

```javascript
import { test, expect } from '@playwright/test';

test.describe('API 测试', () => {
  test('GET 请求', async ({ request }) => {
    const response = await request.get('/api/users');
    expect(response.ok()).toBeTruthy();

    const data = await response.json();
    expect(data.length).toBeGreaterThan(0);
  });

  test('POST 请求', async ({ request }) => {
    const response = await request.post('/api/login', {
      data: {
        username: 'test',
        password: 'password123'
      }
    });

    expect(response.status()).toBe(200);
    const body = await response.json();
    expect(body.token).toBeTruthy();
  });

  test('带认证的请求', async ({ request }) => {
    const response = await request.get('/api/profile', {
      headers: {
        'Authorization': 'Bearer token123'
      }
    });
  });
});
```

#### 11. 设备模拟与响应式测试

```javascript
import { test, devices } from '@playwright/test';

test.use({ ...devices['iPhone 13 Pro'] });

test('移动端测试', async ({ page }) => {
  await page.goto('/');

  // 模拟地理位置
  await page.setGeolocation({ latitude: 39.9042, longitude: 116.4074 });

  // 模拟语言
  await page.context().setLocale('zh-CN');

  // 模拟深色模式
  await page.emulateMedia({ colorScheme: 'dark' });

  // 自定义视口
  await page.setViewportSize({ width: 375, height: 667 });
});
```

#### 12. 测试注解与分组

```javascript
import { test, expect } from '@playwright/test';

test.describe('用户模块', () => {
  test.describe('登录功能', () => {
    test('登录成功', async ({ page }) => {});

    test.skip('跳过此测试', async ({ page }) => {});

    test.only('只运行此测试', async ({ page }) => {});

    test.fixme('待修复的测试', async ({ page }) => {});

    test.fail('预期失败的测试', async ({ page }) => {});

    test.slow('慢速测试，超时 3 倍', async ({ page }) => {});
  });

  // 条件跳过
  test('仅 Chrome 运行', async ({ page, browserName }) => {
    test.skip(browserName !== 'chromium', '仅 Chrome 支持');
  });

  // 标签分组
  test('冒烟测试 @smoke', async ({ page }) => {});
});
```

---

### 🟡 常用

#### 13. playwright.config.ts 配置详解

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // 测试目录
  testDir: './tests',

  // 完全并行
  fullyParallel: true,

  // CI 上失败时禁止 test.only
  forbidOnly: !!process.env.CI,

  // CI 上重试
  retries: process.env.CI ? 2 : 0,

  // CI 上减少并行
  workers: process.env.CI ? 1 : undefined,

  // Reporter 配置
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['junit', { outputFile: 'results.xml' }],
    ['list']
  ],

  // 全局设置
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'retain-on-failure',      // 失败时保留 trace
    screenshot: 'only-on-failure',   // 失败时截图
    video: 'retain-on-failure',      // 失败时录像
    actionTimeout: 10000,            // 操作超时
    navigationTimeout: 30000,        // 导航超时
  },

  // 项目配置（多浏览器）
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'Mobile Chrome', use: { ...devices['Pixel 5'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 12'] } },
  ],

  // 本地启动服务
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

#### 14. 测试钩子 (Fixtures)

```javascript
import { test as base, expect } from '@playwright/test';

// 自定义 fixture
type MyFixtures = {
  loginPage: LoginPage;
  authenticatedPage: Page;
};

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  authenticatedPage: async ({ page }, use) => {
    // Setup: 登录
    await page.goto('/login');
    await page.fill('#username', 'testuser');
    await page.fill('#password', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL(/dashboard/);

    await use(page);

    // Teardown: 登出
    await page.click('#logout');
  },
});

test('已登录状态测试', async ({ authenticatedPage }) => {
  await authenticatedPage.click('.profile');
});
```

#### 15. 参数化测试

```javascript
// 数据驱动测试
const loginData = [
  { username: 'user1', password: 'pass1', expected: 'success' },
  { username: 'user2', password: 'wrong', expected: 'error' },
  { username: '', password: 'pass1', expected: 'error' },
];

for (const data of loginData) {
  test(`登录测试: ${data.username}`, async ({ page }) => {
    await page.goto('/login');
    await page.fill('#username', data.username);
    await page.fill('#password', data.password);
    await page.click('button[type="submit"]');

    if (data.expected === 'success') {
      await expect(page).toHaveURL(/dashboard/);
    } else {
      await expect(page.locator('.error')).toBeVisible();
    }
  });
}
```

#### 16. Trace Viewer 与调试

```bash
# 调试命令
npx playwright test --ui              # UI 模式
npx playwright test --debug           # 调试模式
npx playwright test --trace on        # 开启 trace
npx playwright show-trace trace.zip   # 查看 trace

# 代码生成
npx playwright codegen https://example.com

# 查看报告
npx playwright show-report
```

---

### 🟢 了解

#### 17. 高级特性

```javascript
// ----- Visual Regression Testing -----
await expect(page).toHaveScreenshot('homepage.png', {
  maxDiffPixels: 100,
  animations: 'disabled'
});

// ----- 组件测试 (React/Vue/Svelte) -----
import { test, expect } from '@playwright/experimental-ct-react';
import Button from './Button';

test('组件测试', async ({ mount }) => {
  const component = await mount(<Button>Click me</Button>);
  await expect(component).toContainText('Click me');
});

// ----- 电子邮件测试 -----
test('邮件验证', async ({ page }) => {
  // 使用 mailosaur 或类似服务
});

// ----- 性能测试 -----
test('性能指标', async ({ page }) => {
  await page.goto('/');
  const timing = await page.evaluate(() => {
    const nav = performance.getEntriesByType('navigation')[0];
    return {
      domContentLoaded: nav.domContentLoadedEventEnd,
      load: nav.loadEventEnd,
    };
  });
  expect(timing.load).toBeLessThan(3000);
});
```

#### 18. 测试分片 (Sharding)

```bash
# CI 中分片执行
npx playwright test --shard=1/3  # 第一片
npx playwright test --shard=2/3  # 第二片
npx playwright test --shard=3/3  # 第三片
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 核心答案 |
|------|----------|
| 什么是 Playwright？ | 微软开发的现代 E2E 测试框架，支持跨浏览器、跨语言 |
| 为什么选择 Playwright？ | 自动等待、跨浏览器原生支持、现代 Web 特性、微软维护 |
| Playwright 支持哪些浏览器？ | Chromium、Firefox、WebKit |
| 什么是自动等待？ | 操作前自动等待元素可交互，无需显式 sleep |

### 进阶篇

| 问题 | 核心答案 |
|------|----------|
| 如何处理动态元素？ | 使用稳定的定位策略（role、testId），利用自动等待 |
| POM 的核心原则？ | 分离页面逻辑与测试逻辑、单一职责、复用性 |
| 如何处理 iframe？ | `page.frameLocator()` |
| 如何模拟 API 响应？ | `page.route()` + `route.fulfill()` |
| 如何优化测试速度？ | 并行执行、减少等待、复用 context、分片 |

### 高级篇

| 问题 | 核心答案 |
|------|----------|
| 如何集成 CI/CD？ | GitHub Actions、Jenkins、Docker 镜像 |
| 如何处理认证状态？ | `storageState` 保存登录状态复用 |
| Trace Viewer 是什么？ | 测试执行的完整记录，可回放调试 |
| 如何做视觉回归测试？ | `expect(page).toHaveScreenshot()` |

---

## 四、项目实战经验（面试话术）

### 项目结构

```
playwright-project/
├── tests/
│   ├── e2e/                 # 端到端测试
│   ├── api/                 # API 测试
│   └── visual/              # 视觉回归测试
├── pages/                   # Page Object
├── fixtures/                # 自定义 fixtures
├── test-data/               # 测试数据
├── utils/                   # 工具函数
├── playwright.config.ts     # 配置文件
└── package.json
```

### 面试项目描述模板

> "在我们的电商项目中，我使用 Playwright 搭建了完整的自动化测试框架：
>
> 1. **框架设计**：采用 Page Object Model，分离业务逻辑和测试脚本
> 2. **跨浏览器**：配置 Chrome、Firefox、Safari 三浏览器并行测试
> 3. **CI 集成**：GitHub Actions 自动触发，失败自动截图和录像
> 4. **测试覆盖**：包括 UI 测试、API 测试、视觉回归测试
> 5. **优化成果**：通过并行执行和分片，将测试时间从 30 分钟降到 5 分钟"

### 常见问题解决方案

| 场景 | 解决方案 |
|------|----------|
| 元素定位不稳定 | 使用 `getByRole`、`getByTestId`，避免 XPath |
| 测试偶发失败 | 检查自动等待、增加重试、分析 Trace |
| 测试执行慢 | 并行执行、复用登录状态、优化选择器 |
| 多环境测试 | 使用环境变量配置 `baseURL` |
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
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
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
        sh 'npm ci'
        sh 'npx playwright install --with-deps'
      }
    }
    stage('Test') {
      steps {
        sh 'npx playwright test'
      }
      post {
        always {
          archiveArtifacts artifacts: 'playwright-report/**/*'
        }
      }
    }
  }
}
```

### Docker 集成

```dockerfile
FROM mcr.microsoft.com/playwright:v1.40.0-jammy
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npx", "playwright", "test"]
```

---

## 六、常用命令速查

```bash
# 安装
npm init playwright@latest

# 运行测试
npx playwright test                    # 运行所有
npx playwright test example.spec.ts    # 运行指定文件
npx playwright test --project=chromium # 指定浏览器
npx playwright test --headed           # 有头模式
npx playwright test --ui               # UI 模式
npx playwright test --debug            # 调试模式

# 代码生成
npx playwright codegen https://example.com

# 报告
npx playwright show-report

# Trace
npx playwright test --trace on
npx playwright show-trace trace.zip

# 安装浏览器
npx playwright install
npx playwright install chromium
```

---

## 七、学习路径建议

```
Week 1: 基础入门
├── 环境搭建
├── 定位器 (Locators)
├── 断言 (Assertions)
└── 基本操作 (click, fill, navigate)

Week 2: 进阶技能
├── Page Object Model
├── 测试钩子 (beforeEach, fixtures)
├── 多窗口/iframe 处理
└── 文件上传/下载

Week 3: 高级特性
├── 网络拦截与 Mock
├── API 测试
├── 视觉回归测试
└── 设备模拟

Week 4: 工程化
├── playwright.config.ts 配置
├── CI/CD 集成
├── 测试报告
└── 框架优化
```

---

## 相关知识点

- [[Pytest 面试完全指南]]
- [[Docker 常用命令速查]]
- [[Claude Code 使用指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [Playwright 官方文档](https://playwright.dev/)
- [Playwright GitHub](https://github.com/microsoft/playwright)
- [Coursera Playwright Course](https://www.coursera.org/)
- [BrowserStack Playwright Guide](https://www.browserstack.com/guide/playwright-with-javascript)
