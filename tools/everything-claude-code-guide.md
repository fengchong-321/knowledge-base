# Everything Claude Code 使用指南

> 标签: #claude-code #ai #productivity #配置 #插件
> 创建时间: 2026-02-28
> 来源: [GitHub - affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

## 概述

`everything-claude-code` 是一个由 Anthropic 黑客马拉松获奖者创建的 **Claude Code 完整配置集合**，包含生产级别的 agents、skills、hooks、commands、rules 和 MCP 配置。这些配置经过 10 个月以上的高强度日常使用验证，用于构建真实产品。

---

## 一、整体架构

### 目录结构

```
everything-claude-code/
├── agents/           # 专用子代理（任务委托）
├── skills/           # 工作流定义和领域知识
├── commands/         # 斜杠命令（快速执行）
├── rules/            # 始终遵循的规则
├── hooks/            # 触发器自动化
├── scripts/          # 跨平台 Node.js 脚本
├── contexts/         # 动态系统提示注入
├── mcp-configs/      # MCP 服务器配置
└── examples/         # 示例配置
```

### 安装方式

**方式一：作为插件安装（推荐）**

```bash
# 添加市场
/plugin marketplace add affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code@everything-claude-code
```

**方式二：手动安装**

```bash
# 克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 复制组件到 Claude 配置目录
cp everything-claude-code/agents/*.md ~/.claude/agents/
cp everything-claude-code/rules/*.md ~/.claude/rules/
cp everything-claude-code/commands/*.md ~/.claude/commands/
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

---

## 二、Agents（子代理）

### 是什么

Agents 是主 Claude（编排器）可以委托任务的专用进程，具有有限的作用域。它们可以在后台或前台运行，释放主代理的上下文。

### 为什么用

| 好处 | 说明 |
|------|------|
| 上下文隔离 | 子代理有独立的上下文窗口，不消耗主代理的 token |
| 专业分工 | 每个代理专注于特定任务，提高质量 |
| 并行执行 | 多个代理可以同时工作 |
| 工具限定 | 可以限制代理只能使用特定工具，提高安全性 |

### 什么时候用

- 需要实现复杂功能时 → 使用 `planner`
- 代码写完后需要审查时 → 使用 `code-reviewer`
- 需要做安全审计时 → 使用 `security-reviewer`
- 需要写测试时 → 使用 `tdd-guide`
- 需要修复构建错误时 → 使用 `build-error-resolver`
- 需要运行 E2E 测试时 → 使用 `e2e-runner`
- 需要清理死代码时 → 使用 `refactor-cleaner`

### 怎么用

#### 1. Planner（规划代理）

**是什么**: 专门用于创建详细实现计划的专家

**为什么用**: 复杂功能需要先规划再实施，避免遗漏和返工

**什么时候用**:
- 用户请求功能实现时
- 需要架构变更时
- 需要复杂重构时

**怎么用**:

```markdown
# 代理配置示例
---
name: planner
description: Expert planning specialist for complex features and refactoring
tools: ["Read", "Grep", "Glob"]
model: opus
---
```

**输出格式**:
```markdown
# Implementation Plan: [功能名称]

## Overview
[2-3句话概述]

## Requirements
- [需求1]
- [需求2]

## Architecture Changes
- [变更1: 文件路径和描述]

## Implementation Steps
### Phase 1: [阶段名称]
1. **[步骤名]** (File: path/to/file.ts)
   - Action: 具体操作
   - Why: 原因
   - Dependencies: 无 / 依赖步骤X
   - Risk: 低/中/高

## Testing Strategy
- Unit tests: [测试文件]
- Integration tests: [测试流程]

## Risks & Mitigations
- **Risk**: [风险描述]
- Mitigation: [缓解措施]

## Success Criteria
- [ ] 标准1
- [ ] 标准2
```

#### 2. Code Reviewer（代码审查代理）

**是什么**: 确保代码质量和安全的资深代码审查专家

**为什么用**: 自动化代码审查，发现人工容易遗漏的问题

**什么时候用**:
- 写完或修改代码后立即使用
- 所有代码变更都应该审查

**怎么用**:

代理会自动：
1. 运行 `git diff --staged` 和 `git diff` 查看所有变更
2. 读取完整文件理解上下文
3. 按优先级应用审查清单

**审查优先级**:

| 级别 | 检查项 |
|------|--------|
| CRITICAL | 硬编码凭证、SQL注入、XSS、路径遍历、CSRF、认证绕过 |
| HIGH | 大函数(>50行)、深层嵌套(>4级)、缺失错误处理、console.log |
| MEDIUM | 低效算法、不必要的重渲染、缺失缓存 |
| LOW | TODO无工单、缺失JSDoc、命名不佳 |

**输出格式**:
```
[CRITICAL] 硬编码 API key
File: src/api/client.ts:42
Issue: API key "sk-abc..." 暴露在源代码中
Fix: 移到环境变量

## Review Summary
| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 0     | pass   |
| HIGH     | 2     | warn   |
| MEDIUM   | 3     | info   |
```

#### 3. 其他代理一览

| 代理 | 用途 | 工具 |
|------|------|------|
| `architect.md` | 系统设计决策 | Read, Grep, Glob |
| `tdd-guide.md` | 测试驱动开发 | Read, Grep, Glob, Bash |
| `security-reviewer.md` | 漏洞分析 | Read, Grep, Glob, Bash |
| `build-error-resolver.md` | 构建错误修复 | Read, Grep, Glob, Bash |
| `e2e-runner.md` | Playwright E2E 测试 | Read, Grep, Glob, Bash |
| `refactor-cleaner.md` | 死代码清理 | Read, Grep, Glob, Bash |
| `doc-updater.md` | 文档同步 | Read, Grep, Glob, Bash |

---

## 三、Skills（技能）

### 是什么

Skills 是工作流定义和领域知识，可以被命令或代理调用。它们是执行特定工作流时的提示词扩展。

### 为什么用

| 好处 | 说明 |
|------|------|
| 知识复用 | 将最佳实践编码为可重用的技能 |
| 一致性 | 确保每次都遵循相同的流程 |
| 效率 | 减少重复说明的时间 |

### 什么时候用

- 开始新项目时 → 加载 `coding-standards`
- 做后端开发时 → 加载 `backend-patterns`
- 做前端开发时 → 加载 `frontend-patterns`
- 需要持续学习时 → 使用 `continuous-learning`
- 做 TDD 时 → 使用 `tdd-workflow`
- 做安全审查时 → 使用 `security-review`

### 怎么用

#### 目录结构

```
~/.claude/skills/
├── coding-standards/       # 语言最佳实践
├── backend-patterns/       # API、数据库、缓存模式
├── frontend-patterns/      # React、Next.js 模式
├── continuous-learning/    # 自动从会话提取模式
├── tdd-workflow/           # TDD 方法论
├── security-review/        # 安全检查清单
├── eval-harness/           # 验证循环评估
└── verification-loop/      # 持续验证
```

#### 关键技能详解

**1. continuous-learning（持续学习）**

| 维度 | 说明 |
|------|------|
| 是什么 | 从当前会话中自动提取可复用模式 |
| 为什么用 | 让 Claude 从你的工作方式中学习 |
| 什么时候用 | 完成重要工作后，运行 `/learn` |
| 怎么用 | `/learn` 命令触发模式提取 |

**2. iterative-retrieval（迭代检索）**

| 维度 | 说明 |
|------|------|
| 是什么 | 为子代理提供的渐进式上下文精炼模式 |
| 为什么用 | 解决子代理上下文不足的问题 |
| 什么时候用 | 子代理需要探索代码库时 |
| 怎么用 | 在代理配置中引用此技能 |

**3. strategic-compact（战略压缩）**

| 维度 | 说明 |
|------|------|
| 是什么 | 手动压缩建议，保留重要信息 |
| 为什么用 | 上下文窗口有限，需要智能压缩 |
| 什么时候用 | 上下文接近上限时 |
| 怎么用 | Hook 会自动建议，也可手动触发 |

---

## 四、Commands（命令）

### 是什么

Commands 是通过斜杠快速执行的提示词简写。与 Skills 的区别是存储位置不同，Commands 存储在 `~/.claude/commands/`。

### 为什么用

| 好处 | 说明 |
|------|------|
| 快速执行 | 输入 `/tdd` 即可启动 TDD 工作流 |
| 可链式调用 | 多个命令可以在一个提示中链式调用 |
| 一致性 | 确保每次都遵循相同流程 |

### 什么时候用

| 命令 | 使用场景 |
|------|----------|
| `/tdd` | 实现新功能、修复 bug、重构代码 |
| `/plan` | 需要先规划再实施 |
| `/e2e` | 生成 E2E 测试 |
| `/code-review` | 审查代码质量 |
| `/build-fix` | 修复构建错误 |
| `/refactor-clean` | 清理死代码 |
| `/learn` | 从会话中提取模式 |
| `/checkpoint` | 保存验证状态 |
| `/verify` | 运行验证循环 |

### 怎么用

#### TDD 命令详解

```bash
# 启动 TDD 工作流
/tdd 我需要一个计算市场流动性分数的函数
```

**TDD 流程**:

```
RED → GREEN → REFACTOR → REPEAT

RED: 写失败的测试
GREEN: 写最少代码使测试通过
REFACTOR: 改进代码，保持测试通过
REPEAT: 下一个功能/场景
```

**完整示例**:

```typescript
// Step 1: 定义接口 (SCAFFOLD)
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  throw new Error('Not implemented')
}

// Step 2: 写失败的测试 (RED)
describe('calculateLiquidityScore', () => {
  it('should return high score for liquid market', () => {
    const market = { totalVolume: 100000, bidAskSpread: 0.01, activeTraders: 500, lastTradeTime: new Date() }
    const score = calculateLiquidityScore(market)
    expect(score).toBeGreaterThan(80)
  })

  it('should handle edge case: zero volume', () => {
    const market = { totalVolume: 0, bidAskSpread: 0, activeTraders: 0, lastTradeTime: new Date() }
    const score = calculateLiquidityScore(market)
    expect(score).toBe(0)
  })
})

// Step 3: 运行测试 - 验证失败
// npm test → FAIL ✅ 预期失败

// Step 4: 实现最小代码 (GREEN)
export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0
  // ... 实现逻辑
  return clamp(weightedScore, 0, 100)
}

// Step 5: 运行测试 - 验证通过
// npm test → PASS ✅

// Step 6: 重构 (REFACTOR)
// 提取常量、改进可读性...

// Step 7: 验证测试仍然通过
// npm test → PASS ✅

// Step 8: 检查覆盖率
// npm test -- --coverage → 100% ✅
```

#### 命令链式调用

```bash
# 可以在一个提示中链式调用多个命令
/plan 用户认证系统 && /tdd && /code-review
```

---

## 五、Rules（规则）

### 是什么

Rules 是 Claude 应该**始终遵循**的最佳实践指南，存储在 `~/.claude/rules/` 目录。

### 为什么用

| 好处 | 说明 |
|------|------|
| 强制执行 | 每次会话都会加载，确保一致性 |
| 模块化 | 按关注点分组，易于管理 |
| 可定制 | 根据项目需求调整 |

### 什么时候用

始终生效，无需手动触发。每次 Claude 启动时自动加载。

### 怎么用

#### 目录结构

```
~/.claude/rules/
├── security.md       # 强制安全检查
├── coding-style.md   # 不可变性、文件大小限制
├── testing.md        # TDD、80% 覆盖率要求
├── git-workflow.md   # 约定式提交
├── agents.md         # 子代理委托规则
├── performance.md    # 模型选择、上下文管理
├── patterns.md       # API 响应格式
└── hooks.md          # Hook 文档
```

#### 规则示例

**security.md**:
```markdown
# Security Rules

- 不硬编码密钥、密码、Token
- 验证所有用户输入
- 使用参数化查询防止 SQL 注入
- 对用户内容进行 XSS 过滤
- 敏感数据不记录到日志
```

**coding-style.md**:
```markdown
# Coding Style Rules

- 代码库中不使用表情符号
- 文件大小限制：200-400 行（典型），800 行（最大）
- 优先使用不可变操作（spread 而非 mutation）
- 避免巨型文件，保持模块化
- 永远不要提交 console.log
```

**testing.md**:
```markdown
# Testing Rules

- 遵循 TDD 工作流
- 最低覆盖率要求：80%
- 关键代码覆盖率要求：100%
- 测试行为，而非实现细节
```

---

## 六、Hooks（钩子）

### 是什么

Hooks 是基于触发器的自动化，在特定事件发生时触发。与 Skills 不同，它们限定在工具调用和生命周期事件。

### 为什么用

| 好处 | 说明 |
|------|------|
| 自动化 | 无需手动触发，自动执行 |
| 一致性 | 确保每次都执行相同的检查 |
| 效率 | 减少重复性工作 |
| 安全 | 在危险操作前提醒或阻止 |

### 什么时候用

| Hook 类型 | 触发时机 | 用途 |
|-----------|----------|------|
| PreToolUse | 工具执行前 | 验证、阻止、提醒 |
| PostToolUse | 工具完成后 | 格式化、检查、反馈 |
| SessionStart | 会话开始时 | 加载上下文 |
| SessionEnd | 会话结束时 | 保存状态 |
| PreCompact | 上下文压缩前 | 保存重要信息 |
| Stop | Claude 响应完成时 | 清理、审计 |

### 怎么用

#### 实用 Hook 配置

**1. 阻止在 tmux 外运行开发服务器**

```json
{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "检查是否是 dev 命令且不在 tmux 中，如果是则阻止"
  }],
  "description": "开发服务器必须在 tmux 中运行以便访问日志"
}
```

**为什么**: 如果开发服务器在主终端运行，Claude 会话结束后服务器也会停止，且无法查看日志。

**2. 阻止创建随机 .md 文件**

```json
{
  "matcher": "Write",
  "hooks": [{
    "type": "command",
    "command": "检查是否是 .md 文件且不是 README/CLAUDE/AGENTS，如果是则阻止"
  }],
  "description": "阻止创建不必要的文档文件，保持文档集中"
}
```

**为什么**: Claude 有时会创建很多临时 .md 文件，导致项目混乱。文档应该集中在 README.md 中。

**3. git push 前提醒审查**

```json
{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "检测 git push 命令，提醒审查变更"
  }],
  "description": "git push 前提醒审查变更"
}
```

**为什么**: 防止不经审查就推送代码。

**4. 编辑后自动格式化**

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "command",
    "command": "对 .ts/.tsx/.js/.jsx 文件运行 prettier --write"
  }],
  "description": "编辑 JS/TS 文件后自动格式化"
}
```

**为什么**: 确保代码风格一致，无需手动格式化。

**5. TypeScript 类型检查**

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "command",
    "command": "对 .ts/.tsx 文件运行 tsc --noEmit"
  }],
  "description": "编辑 TypeScript 文件后进行类型检查"
}
```

**为什么**: 及时发现类型错误，而不是等到构建时。

**6. console.log 警告**

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "command",
    "command": "检查编辑的文件是否包含 console.log"
  }],
  "description": "警告 console.log 语句"
}
```

**为什么**: 防止调试代码被提交。

**7. 会话结束时检查 console.log**

```json
{
  "matcher": "*",
  "hooks": [{
    "type": "command",
    "command": "检查所有修改的文件是否包含 console.log"
  }],
  "description": "会话结束前审计 console.log"
}
```

**为什么**: 最后的防线，确保没有遗漏。

#### Hook 工作流程

```
用户发送消息
    ↓
[SessionStart] 加载之前的上下文
    ↓
Claude 准备执行工具
    ↓
[PreToolUse] 检查、验证、阻止
    ↓
工具执行
    ↓
[PostToolUse] 格式化、检查、反馈
    ↓
Claude 完成响应
    ↓
[Stop] 清理、审计
    ↓
会话结束
    ↓
[SessionEnd] 保存状态、评估会话
```

---

## 七、上下文窗口管理（关键）

### 是什么

Claude Code 有 200k token 的上下文窗口限制。启用的 MCPs、Plugins、Rules 都会占用这个空间。

### 为什么重要

| 问题 | 影响 |
|------|------|
| 启用太多 MCPs | 200k 可能缩减到 70k |
| 启用太多工具 | 性能显著下降 |
| 上下文不足 | Claude 无法理解完整上下文 |

### 最佳实践

**经验法则**:
- 配置 20-30 个 MCPs
- 每个项目启用 <10 个
- 活跃工具 <80 个

**项目级禁用**:

```json
// ~/.claude.json
{
  "projects": {
    "/path/to/project": {
      "disabledMcpServers": [
        "playwright",
        "cloudflare-docs",
        "clickhouse",
        "AbletonMCP",
        "context7",
        "magic"
      ]
    }
  }
}
```

---

## 八、跨平台支持

### 是什么

所有 hooks 和脚本都用 Node.js 重写，支持 Windows、macOS 和 Linux。

### 包管理器检测

自动检测优先级：
1. 环境变量: `CLAUDE_PACKAGE_MANAGER`
2. 项目配置: `.claude/package-manager.json`
3. package.json: `packageManager` 字段
4. 锁文件: package-lock.json, yarn.lock, pnpm-lock.yaml, bun.lockb
5. 全局配置: `~/.claude/package-manager.json`
6. 回退: 第一个可用的包管理器

### 设置包管理器

```bash
# 通过环境变量
export CLAUDE_PACKAGE_MANAGER=pnpm

# 通过全局配置
node scripts/setup-package-manager.js --global pnpm

# 通过项目配置
node scripts/setup-package-manager.js --project bun

# 检测当前设置
node scripts/setup-package-manager.js --detect

# 或在 Claude Code 中使用
/setup-pm
```

---

## 九、实战场景

### 场景1：新功能开发

```bash
# 1. 先规划
/plan 用户认证系统

# 2. TDD 实现
/tdd 实现登录功能

# 3. 代码审查
/code-review

# 4. 运行 E2E 测试
/e2e 登录流程
```

### 场景2：Bug 修复

```bash
# 1. 先写复现测试
/tdd 修复登录失败的 bug

# 2. 代码审查
/code-review

# 3. 构建检查
/build-fix
```

### 场景3：代码清理

```bash
# 清理死代码和零散文件
/refactor-clean
```

### 场景4：会话学习

```bash
# 完成工作后提取模式
/learn

# 会话结束时会自动：
# - 保存状态
# - 评估可提取的模式
# - 更新技能库
```

---

## 十、生态系统工具

### ecc.tools - Skill Creator

自动从你的仓库生成 Claude Code 技能：
- 生成 SKILL.md 文件
- 创建 instinct 集合
- 从提交历史提取模式

安装 GitHub App 后，技能会出现在：
```
~/.claude/skills/generated/
```

---

## 十一、快速参考卡

### 常用命令

| 命令 | 用途 |
|------|------|
| `/tdd` | 测试驱动开发 |
| `/plan` | 实现规划 |
| `/code-review` | 代码审查 |
| `/e2e` | E2E 测试 |
| `/build-fix` | 修复构建错误 |
| `/refactor-clean` | 清理死代码 |
| `/learn` | 提取模式 |
| `/checkpoint` | 保存状态 |
| `/verify` | 验证循环 |
| `/setup-pm` | 配置包管理器 |

### 代理一览

| 代理 | 用途 | 模型 |
|------|------|------|
| planner | 功能规划 | opus |
| architect | 系统设计 | opus |
| code-reviewer | 代码审查 | sonnet |
| security-reviewer | 安全审计 | sonnet |
| tdd-guide | TDD 指导 | sonnet |
| build-error-resolver | 构建修复 | sonnet |
| e2e-runner | E2E 测试 | sonnet |
| refactor-cleaner | 代码清理 | sonnet |

### Hook 类型

| 类型 | 时机 | 用途 |
|------|------|------|
| PreToolUse | 工具前 | 验证/阻止 |
| PostToolUse | 工具后 | 格式化/检查 |
| SessionStart | 会话开始 | 加载上下文 |
| SessionEnd | 会话结束 | 保存状态 |
| PreCompact | 压缩前 | 保存重要信息 |
| Stop | 响应完成 | 清理/审计 |

---

## 相关知识点

- [[Claude Code 高级使用指南]]
- [[Claude Code 完整配置指南]]
- [[Claude Code 使用指南]]

---

*整理自 GitHub - affaan-m/everything-claude-code 仓库*

**Sources:**
- [GitHub - everything-claude-code](https://github.com/affaan-m/everything-claude-code)
- [Twitter @affaanmustafa](https://x.com/affaanmustafa)
