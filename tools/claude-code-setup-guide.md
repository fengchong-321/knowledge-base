# Claude Code 完整配置指南

> 标签: #claude-code #ai #productivity #配置 #实战
> 创建时间: 2026-02-28
> 来源: [Twitter @affaanmustafa](https://x.com/affaanmustafa/status/2012378465664745795)

## 概述

这是作者经过 10 个月日常使用后的完整 Claude Code 配置，包括 skills、hooks、subagents、MCPs、plugins 以及真正有效的实践。作者从 2 月份实验性推出开始就是 Claude Code 的忠实用户，并使用 Claude Code 完全构建了 PMFProbe 项目，赢得了 Anthropic x Forum Ventures 黑客马拉松。

---

## 一、Skills（技能）

Skills 类似规则，限定在特定范围和工作流中。它们是执行特定工作流时的提示词简写。

### 基本用法

```bash
# 长时间编码后，想清理死代码和零散的 .md 文件？
/refactor-clean

# 需要测试？
/tdd
/e2e
/test-coverage
```

**Skills 和 commands 可以在单个提示中链式调用**

### 示例结构

```bash
~/.claude/skills/
├── pmx-guidelines.md      # 项目特定模式
├── coding-standards.md    # 语言最佳实践
├── tdd-workflow/          # 多文件 skill（含 README.md）
└── security-review/       # 基于检查清单的 skill
```

### Skills vs Commands

- **Skills**: `~/.claude/skills` - 更广泛的工作流定义
- **Commands**: `~/.claude/commands` - 快速可执行的提示

### Codemap Updater 示例

可以创建一个在检查点更新 codemaps 的 skill - 让 Claude 快速导航代码库而不用在探索上消耗上下文。

```bash
~/.claude/skills/codemap-updater.md
```

---

## 二、Hooks（钩子）

Hooks 是基于触发器的自动化，在特定事件发生时触发。与 skills 不同，它们限定在工具调用和生命周期事件。

### Hook 类型

| 类型 | 触发时机 | 用途 |
|------|----------|------|
| PreToolUse | 工具执行前 | 验证、提醒 |
| PostToolUse | 工具完成后 | 格式化、反馈循环 |
| UserPromptSubmit | 发送消息时 | 预处理 |
| Stop | Claude 完成响应时 | 清理、审计 |
| PreCompact | 上下文压缩前 | 保存重要信息 |
| Notification | 权限请求时 | 自定义通知 |

### 示例：长时间运行命令前的 tmux 提醒

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] Consider tmux for session persistence' >&2; fi"
        }
      ]
    }
  ]
}
```

### Pro Tip

使用 `hookify` 插件通过对话创建 hooks，而不是手动编写 JSON。运行 `/hookify` 并描述你想要的功能。

---

## 三、Subagents（子代理）

Subagents 是你的编排器（主 Claude）可以委托任务的进程，具有有限的范围。它们可以在后台或前台运行，释放主代理的上下文。

### 与 Skills 配合

Subagent 与 skills 配合良好 - 能够执行你的 skills 子集的 subagent 可以被委托任务并自主使用这些 skills。它们也可以通过特定的工具权限进行沙箱化。

### 示例结构

```bash
~/.claude/agents/
├── planner.md              # 功能实现规划
├── architect.md            # 系统设计决策
├── tdd-guide.md            # 测试驱动开发
├── code-reviewer.md        # 质量/安全审查
├── security-reviewer.md    # 漏洞分析
├── build-error-resolver.md
├── e2e-runner.md
└── refactor-cleaner.md
```

### 配置要点

为每个 subagent 配置允许的工具、MCPs 和权限，以实现正确的作用域限定。

---

## 四、Rules（规则）

`.rules` 文件夹包含 Claude 应该**始终遵循**的最佳实践的 `.md` 文件。

### 两种方法

1. **单文件** - 所有内容在一个文件中（用户或项目级别）
2. **Rules 文件夹** - 按关注点分组的模块化 `.md` 文件

### 示例结构

```bash
~/.claude/rules/
├── security.md       # 不硬编码密钥，验证输入
├── coding-style.md   # 不可变性，文件组织
├── testing.md        # TDD 工作流，80% 覆盖率
├── git-workflow.md   # 提交格式，PR 流程
├── agents.md         # 何时委托给 subagents
└── performance.md    # 模型选择，上下文管理
```

### 示例规则

- 代码库中不使用表情符号
- 前端避免使用紫色调
- 部署前始终测试代码
- 优先使用模块化代码而非巨型文件
- 永远不要提交 console.logs

---

## 五、MCPs（模型上下文协议）

MCPs 将 Claude 直接连接到外部服务。它不是 API 的替代品 - 而是围绕它们的提示驱动包装器，允许更灵活地导航信息。

### 示例：Supabase MCP

让 Claude 直接提取特定数据，运行 SQL 而无需复制粘贴。同样适用于数据库、部署平台等。

### 关键警告：上下文窗口管理

**要对 MCPs 挑剔**。将所有 MCPs 保留在用户配置中，但禁用所有未使用的。导航到 `/plugins` 向下滚动或运行 `/mcp`。

- 压缩前的 200k 上下文窗口可能只有 70k（如果有太多工具启用）
- 性能会显著下降

**经验法则**：配置中有 20-30 个 MCPs，但保持少于 10 个启用 / 少于 80 个工具活跃。

### Chrome in Claude

内置的插件 MCP，让 Claude 自主控制你的浏览器 - 点击查看事物如何工作。

---

## 六、Plugins（插件）

Plugins 将工具打包以便于安装，而不是繁琐的手动设置。插件可以是 skill + MCP 组合，或 hooks/tools 打包在一起。

### 安装插件

```bash
# 添加市场
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# 打开 Claude，运行 /plugins，找到新市场，从那里安装
```

### LSP Plugins

如果你经常在编辑器外运行 Claude Code，LSP 插件特别有用。语言服务器协议为 Claude 提供实时类型检查、跳转到定义和智能补全，而无需打开 IDE。

```bash
# 已启用的插件示例
typescript-lsp@claude-plugins-official    # TypeScript 智能
pyright-lsp@claude-plugins-official       # Python 类型检查
hookify@claude-plugins-official           # 通过对话创建 hooks
mgrep@Mixedbread-Grep                     # 比 ripgrep 更好的搜索
```

**与 MCPs 相同的警告** - 注意你的上下文窗口。

---

## 七、键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+U` | 删除整行（比退格键快） |
| `!` | 快速 bash 命令前缀 |
| `@` | 搜索文件 |
| `/` | 启动斜杠命令 |
| `Shift+Enter` | 多行输入 |
| `Tab` | 切换思考显示 |
| `Esc Esc` | 中断 Claude / 恢复代码 |

---

## 八、并行工作流

### /fork - 分叉对话

分叉对话以并行执行非重叠任务，而不是在队列中堆积消息。

### Git Worktrees

用于重叠的并行 Claude 实例而不产生冲突。每个 worktree 是一个独立的检出。

```bash
git worktree add ../feature-branch feature-branch
# 现在在每个 worktree 中运行独立的 Claude 实例
```

### tmux 用于长时间运行的命令

流式传输和监视 Claude 运行的 logs/bash 进程。

```bash
tmux new -s dev
# Claude 在这里运行命令，你可以分离和重新附加
tmux attach -t dev
```

### mgrep > grep

`mgrep` 是对 ripgrep/grep 的重大改进。通过插件市场安装，然后使用 `/mgrep` skill。支持本地搜索和网页搜索。

```bash
mgrep "function handleSubmit"           # 本地搜索
mgrep --web "Next.js 15 app router changes"  # 网页搜索
```

---

## 九、其他有用命令

| 命令 | 功能 |
|------|------|
| `/rewind` | 回到之前的状态 |
| `/statusline` | 自定义显示分支、上下文 %、todos |
| `/checkpoints` | 文件级别的撤销点 |
| `/compact` | 手动触发上下文压缩 |

---

## 十、GitHub Actions CI/CD

在你的 PR 上设置代码审查与 GitHub Actions。配置后 Claude 可以自动审查 PR。

---

## 十一、沙箱模式

对风险操作使用沙箱模式 - Claude 在受限环境中运行，不影响你的实际系统。

**注意**：使用 `--dangerously-skip-permissions` 做相反的事情，让 Claude 自由漫游，如果不小心可能会造成破坏。

---

## 十二、编辑器集成

虽然不需要编辑器，但它可以积极或消极地影响你的 Claude Code 工作流。

### Zed（作者推荐）

基于 Rust 的编辑器，轻量、快速、高度可定制。

**Zed 与 Claude Code 配合良好的原因**：

1. **Agent Panel 集成** - Zed 的 Claude 集成让你实时跟踪 Claude 编辑的文件变化
2. **性能** - 用 Rust 编写，瞬间打开，处理大型代码库无延迟
3. **CMD+Shift+R 命令面板** - 快速访问所有自定义斜杠命令、调试器和工具
4. **最小资源使用** - 在繁重操作期间不会与 Claude 竞争系统资源
5. **Vim 模式** - 完整的 vim 键绑定（如果这是你的风格）

**Zed 使用技巧**：

1. 分屏 - 一边是带 Claude Code 的终端，另一边是编辑器
2. `Ctrl + G` - 快速在 Zed 中打开 Claude 当前正在处理的文件
3. 自动保存 - 启用自动保存，让 Claude 的文件读取始终是最新的
4. Git 集成 - 使用编辑器的 git 功能在提交前审查 Claude 的更改
5. 文件监视器 - 大多数编辑器自动重新加载更改的文件，验证这是启用的

### VSCode / Cursor

也是一个可行的选择，与 Claude Code 配合良好。可以在终端格式中使用，使用 `\ide` 与编辑器自动同步启用 LSP 功能（现在与插件有些冗余）。或者选择扩展，它与编辑器更集成并有匹配的 UI。

---

## 十三、实际配置示例

### 已安装的 Plugins

（作者通常一次只启用 4-5 个）

```markdown
ralph-wiggum@claude-code-plugins          # 循环自动化
frontend-design@claude-code-plugins       # UI/UX 模式
commit-commands@claude-code-plugins       # Git 工作流
security-guidance@claude-code-plugins     # 安全检查
pr-review-toolkit@claude-code-plugins     # PR 自动化
typescript-lsp@claude-plugins-official    # TS 智能
hookify@claude-plugins-official           # Hook 创建
code-simplifier@claude-plugins-official
feature-dev@claude-code-plugins
explanatory-output-style@claude-code-plugins
code-review@claude-code-plugins
context7@claude-plugins-official          # 实时文档
pyright-lsp@claude-plugins-official       # Python 类型
mgrep@Mixedbread-Grep                     # 更好的搜索
```

### 配置的 MCP Servers（用户级别）

```json
{
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"]
  },
  "firecrawl": {
    "command": "npx",
    "args": ["-y", "firecrawl-mcp"]
  },
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"]
  },
  "memory": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-memory"]
  },
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  },
  "vercel": {
    "type": "http",
    "url": "https://mcp.vercel.com"
  },
  "railway": {
    "command": "npx",
    "args": ["-y", "@railway/mcp-server"]
  },
  "cloudflare-docs": {
    "type": "http",
    "url": "https://docs.mcp.cloudflare.com/mcp"
  },
  "cloudflare-workers-bindings": {
    "type": "http",
    "url": "https://bindings.mcp.cloudflare.com/mcp"
  },
  "cloudflare-workers-builds": {
    "type": "http",
    "url": "https://builds.mcp.cloudflare.com/mcp"
  },
  "cloudflare-observability": {
    "type": "http",
    "url": "https://observability.mcp.cloudflare.com/mcp"
  },
  "clickhouse": {
    "type": "http",
    "url": "https://mcp.clickhouse.cloud/mcp"
  },
  "AbletonMCP": {
    "command": "uvx",
    "args": ["ableton-mcp"]
  },
  "magic": {
    "command": "npx",
    "args": ["-y", "@magicuidesign/mcp@latest"]
  }
}
```

### 按项目禁用的 MCPs（上下文窗口管理）

```yaml
# 在 ~/.claude.json 中的 projects.[path].disabledMcpServers
disabledMcpServers: [
  "playwright",
  "cloudflare-workers-builds",
  "cloudflare-workers-bindings",
  "cloudflare-observability",
  "cloudflare-docs",
  "clickhouse",
  "AbletonMCP",
  "context7",
  "magic"
]
```

**这是关键** - 配置了 14 个 MCPs 但每个项目只启用 ~5-6 个。保持上下文窗口健康。

### 关键 Hooks 配置

```json
{
  "PreToolUse": [
    // 长时间运行命令的 tmux 提醒
    { "matcher": "npm|pnpm|yarn|cargo|pytest", "hooks": ["tmux reminder"] },
    // 阻止不必要的 .md 文件创建
    { "matcher": "Write && .md file", "hooks": ["block unless README/CLAUDE"] },
    // git push 前审查
    { "matcher": "git push", "hooks": ["open editor for review"] }
  ],
  "PostToolUse": [
    // 用 Prettier 自动格式化 JS/TS
    { "matcher": "Edit && .ts/.tsx/.js/.jsx", "hooks": ["prettier --write"] },
    // 编辑后 TypeScript 检查
    { "matcher": "Edit && .ts/.tsx", "hooks": ["tsc --noEmit"] },
    // 关于 console.log 的警告
    { "matcher": "Edit", "hooks": ["grep console.log warning"] }
  ],
  "Stop": [
    // 会话结束前审计 console.logs
    { "matcher": "*", "hooks": ["check modified files for console.log"] }
  ]
}
```

### 自定义状态栏

显示用户、目录、git 分支（带 dirty 指示器）、剩余上下文 %、模型、时间和 todo 数量。

### Rules 结构

```bash
~/.claude/rules/
├── security.md       # 强制安全检查
├── coding-style.md   # 不可变性，文件大小限制
├── testing.md        # TDD，80% 覆盖率
├── git-workflow.md   # 约定式提交
├── agents.md         # Subagent 委托规则
├── patterns.md       # API 响应格式
├── performance.md    # 模型选择（Haiku vs Sonnet vs Opus）
└── hooks.md          # Hook 文档
```

### Subagents 结构

```bash
~/.claude/agents/
├── planner.md              # 分解功能
├── architect.md            # 系统设计
├── tdd-guide.md            # 先写测试
├── code-reviewer.md        # 质量审查
├── security-reviewer.md    # 漏洞扫描
├── build-error-resolver.md
├── e2e-runner.md           # Playwright 测试
├── refactor-cleaner.md     # 死代码删除
└── doc-updater.md          # 保持文档同步
```

---

## 十四、核心原则

1. **不要过度复杂化** - 把配置当作微调，而不是架构
2. **上下文窗口很宝贵** - 禁用未使用的 MCPs 和 plugins
3. **并行执行** - 分叉对话，使用 git worktrees
4. **自动化重复工作** - 用 hooks 处理格式化、linting、提醒
5. **限定你的 subagents** - 有限的工具 = 专注的执行

---

## 相关知识点

- [[Claude Code 高级使用指南]]
- [[Claude Code 使用指南]]

---

*翻译自 Twitter @affaanmustafa 的实战分享*

**Sources:**
- [Twitter - Complete Claude Code Setup Guide](https://x.com/affaanmustafa/status/2012378465664745795)
