# Claude Code 使用指南

> 标签: #claude-code #cli #ai #开发工具
> 创建时间: 2026-02-23
> 来源: [Claude Code CLI 参考](https://m.runoob.com/claude-code/claude-code-cli-ref.html) | [掘金命令大全](https://juejin.cn/post/7585484081435770931)

## 概述

Claude Code 是 Anthropic 官方的 AI 编程助手 CLI 工具，支持代码生成、重构、调试、Git 操作等功能。

## 安装

```bash
npm install -g @anthropic-ai/claude-code
```

## 常用 CLI 命令

| 命令 | 说明 |
|------|------|
| `claude` | 启动交互式 REPL |
| `claude "提示词"` | 带初始提示启动 |
| `claude -p "提示词"` | 单次查询后退出 |
| `cat file \| claude -p "提示词"` | 处理管道输入 |
| `claude -c` | 继续最近对话 |
| `claude update` | 更新到最新版本 |
| `claude mcp` | 配置 MCP 服务器 |

## 会话内斜杠命令

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助 |
| `/clear` | 清空对话历史 |
| `/compact` | 压缩上下文节省 token |
| `/resume` | 恢复之前的会话 |
| `/export` | 导出当前对话 |
| `/init` | 初始化 CLAUDE.md 文件 |
| `/add-dir` | 添加工作目录 |
| `/skills` | 列举可用技能 |
| `/todos` | 列举当前待办项 |

## 自定义斜杠命令

在以下目录创建 `.md` 文件：
- **项目级别**: `.claude/commands/`
- **全局级别**: `~/.claude/commands/`

支持 `$ARGUMENTS` 接收参数、bash 预处理、文件引用等。

## 高级功能

- **CLAUDE.md** - 项目上下文记忆文件
- **MCP 集成** - 连接外部服务（数据库、API 等）
- **Hooks** - 代码操作时自动触发脚本
- **图像支持** - 拖放/粘贴图像分析

## 相关知识点

- [[MCP 服务器配置]]
- [[Git 工作流最佳实践]]

---
*采集自 Claude Code 对话*
