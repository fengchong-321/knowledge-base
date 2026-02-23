# Add Knowledge Skill

> 标签: #claude-code #skill #自动化
> 创建时间: 2026-02-23
> 来源: 自定义 Skill

## 概述

用于快速添加知识点到个人知识库的 Claude Code Skill，支持全局调用。

## 安装位置

```
~/.claude/commands/add-knowledge.md
```

## 使用方法

```bash
# 基本用法
/add-knowledge {主题}

# 指定分类
/add-knowledge {主题} -c {分类}
```

**分类选项**：
- `programming` - 编程语言、框架、算法
- `tools` - 开发工具、CLI、配置
- `concepts` - 概念、原理、设计模式
- `snippets` - 代码片段、常用模板

## 执行流程

```
1. 确认主题和分类
      ↓
2. WebSearch 搜索相关信息
      ↓
3. 按模板整理 Markdown
      ↓
4. 写入 ~/knowledge-base/{分类}/
      ↓
5. 更新 README.md 索引
      ↓
6. Git commit & push
      ↓
7. 告知用户记录结果
```

## 示例

```bash
/add-knowledge React Hooks 最佳实践

/add-knowledge 数据库索引原理 -c concepts

/add-knowledge git rebase 工作流 -c tools
```

## 知识点模板

```markdown
# {标题}

> 标签: #{tag1} #{tag2}
> 创建时间: {date}
> 来源: {url}

## 概述
{摘要}

## 详细内容
{详细说明}

## 相关知识点
- [[{相关项}]]

---
*采集自 Claude Code 对话*
```

## 源码

完整源码见: `~/.claude/commands/add-knowledge.md`

## 相关知识点

- [[Claude Code 使用指南]]
- [[Skill vs MCP 选择指南]]
