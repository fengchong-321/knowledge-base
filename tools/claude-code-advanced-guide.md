# Claude Code 高级使用指南：详解篇

> 标签: #claude-code #ai #productivity #workflow #advanced
> 原文作者: affaanmustafa
> 来源: [Twitter/X](https://x.com/affaanmustafa/status/2014040193557471352)
> 翻译时间: 2026-02-28

## 概述

本文是《Claude Code 全面指南》的详解篇，涵盖将高效会话与低效会话区分开来的高级技术。假设你已经配置好了 skills、agents、hooks 和 MCPs。核心主题包括：Token 经济学、记忆持久化、验证模式、并行化策略，以及构建可复用工作流的复合效应。

**GitHub 仓库**: https://github.com/affaan-m/everything-claude-code

---

## 一、跨会话记忆共享

### 会话存储模式

创建一个 skill 或 command，用于总结进度并保存到 `.claude` 文件夹中的 `.tmp` 文件，在会话期间持续追加内容。第二天可以使用它作为上下文，从上次离开的地方继续。为每个会话创建新文件，避免将旧上下文污染到新工作中。

**会话文件应包含：**
- 哪些方法有效（有证据可验证）
- 尝试过但失败的方法
- 尚未尝试的方法
- 还有什么需要完成

**示例会话存储结构：**
```
~/.claude/sessions/YYYY-MM-DD-topic.tmp
```

### 策略性清理上下文

一旦计划确定并清理了上下文（Claude Code 计划模式中的默认选项），你就可以从计划开始工作。当你积累了大量与执行不再相关的探索上下文时，这很有用。

**策略性压缩建议：** 禁用自动压缩。在逻辑间隔手动压缩，或创建一个 skill 为你在特定条件下建议或执行压缩。

```bash
#!/bin/bash
# Strategic Compact Suggester
# 在 PreToolUse 时运行，在逻辑间隔建议手动压缩
#
# 为什么选择手动而非自动压缩：
# - 自动压缩发生在任意时间点，通常在任务中间
# - 策略性压缩通过逻辑阶段保留上下文
# - 在探索后、执行前压缩
# - 在完成里程碑后、开始下一个前压缩

COUNTER_FILE="/tmp/claude-tool-count-$$"
THRESHOLD=${COMPACT_THRESHOLD:-50}

# 初始化或增加计数器
if [ -f "$COUNTER_FILE" ]; then
  count=$(cat "$COUNTER_FILE")
  count=$((count + 1))
  echo "$count" > "$COUNTER_FILE"
else
  echo "1" > "$COUNTER_FILE"
  count=1
fi

# 在达到阈值工具调用后建议压缩
if [ "$count" -eq "$THRESHOLD" ]; then
  echo "[StrategicCompact] $THRESHOLD tool calls reached - consider /compact if transitioning phases" >&2
fi
```

将其挂载到 Edit/Write 操作的 PreToolUse - 当你积累了足够的上下文时，它会提醒你压缩可能有帮助。

---

## 二、高级：动态系统提示注入

### 模式说明

与其仅仅将所有内容放在 `~/.claude.json`（用户范围）或 `.claude/rules/`（项目范围）中每次会话都加载，不如使用 CLI 标志动态注入上下文：

```bash
claude --system-prompt "$(cat memory.md)"
```

这让你可以更精确地控制何时加载什么上下文。你可以根据正在处理的内容为每个会话注入不同的上下文。

### 为什么这很重要

当你使用 `@file.md` 或将内容放在 `.claude/rules/` 中时，Claude 通过 Read 工具在对话期间读取它 - 它作为工具输出进入。当你使用 `--system-prompt` 时，内容在对话开始之前被注入到实际的系统提示中。

**区别在于指令层次结构：**
- 系统提示内容 > 用户消息 > 工具结果

对于大多数日常工作，这种差异是边缘的。但对于严格的行为规则、项目特定约束或你绝对需要 Claude 优先考虑的上下文 - 系统提示注入确保它被适当加权。

### 实践设置

一个有效的方法是利用 `.claude/rules/` 作为基线项目规则，然后为场景特定上下文创建 CLI 别名：

```bash
# 日常开发
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'

# PR 审查模式
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'

# 研究/探索模式
alias claude-research='claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'
```

- `dev.md` 专注于实现
- `review.md` 专注于代码质量/安全
- `research.md` 专注于行动前的探索

同样，对于大多数事情，使用 `.claude/rules/context1.md` 和直接将 `--system-prompt` 追加到系统提示的区别是边缘的。CLI 方法更快（无工具调用）、更可靠（系统级权威）、略微更节省 token。但这是一个小优化，对许多人来说可能得不偿失。

---

## 三、高级：记忆持久化 Hooks

### 会话生命周期 Hooks

```
SESSION 1                    SESSION 2
─────────                    ─────────
[Start]                      [Start]
   │                            │
   ▼                            ▼
┌──────────────┐         ┌──────────────┐
│ SessionStart │ ◄───    │ SessionStart │◄── 加载之前的
│     Hook     │  reads  │     Hook     │    上下文
└──────┬───────┘  nothing └──────┬───────┘
       │        yet             │
       ▼                         ▼
[Working]                   [Working]
   │                      (informed)
   ▼                         │
┌──────────────┐             ▼
│  PreCompact  │──► saves   [Continue...]
│     Hook     │   state
└──────┬───────┘   before
       │          summary
       ▼
[Compacted]
       │
       ▼
┌──────────────┐
│   Stop Hook  │──► persists to ──────────►
│(session-end) │   ~/.claude/sessions/
└──────────────┘
```

**Hook 类型：**
- **PreCompact Hook**: 在上下文压缩发生之前，将重要状态保存到文件
- **SessionComplete Hook**: 会话结束时，将学习内容持久化到文件
- **SessionStart Hook**: 新会话时，自动加载之前的上下文

### 配置示例

```json
{
  "hooks": {
    "PreCompact": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/memory-persistence/pre-compact.sh"
      }]
    }],
    "SessionStart": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/memory-persistence/session-start.sh"
      }]
    }],
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/memory-persistence/session-end.sh"
      }]
    }]
  }
}
```

**各脚本功能：**
- `pre-compact.sh`: 记录压缩事件，用压缩时间戳更新活动会话文件
- `session-start.sh`: 检查最近的会话文件（最近7天），通知可用的上下文和已学习的 skills
- `session-end.sh`: 创建/更新每日会话文件，跟踪开始/结束时间

将这些链接在一起，实现跨会话的连续记忆，无需手动干预。

---

## 四、持续学习系统

### 问题与解决方案

**问题：** 浪费 tokens、浪费上下文、浪费时间，当你沮丧地对 Claude 大喊不要做它在之前的会话中已经被告知不要做的事情时，你的皮质醇飙升。

**解决方案：** 当 Claude Code 发现非平凡的内容 - 调试技术、变通方法、某些项目特定模式 - 它将该知识保存为新的 skill。下次出现类似问题时，skill 会自动加载。

### 为什么使用 Stop Hook

为什么使用 Stop hook 而不是 UserPromptSubmit？
- UserPromptSubmit 在你发送的每条消息上运行 - 开销大，给每个提示增加延迟
- Stop 在会话结束时运行一次 - 轻量级，不会在会话期间减慢你的速度
- 评估完整会话而不是零碎的

### 安装

```bash
# 克隆到 skills 文件夹
git clone https://github.com/affaan-m/everything-claude-code.git ~/.claude/skills/everything-claude-code

# 或只获取 continuous-learning skill
mkdir -p ~/.claude/skills/continuous-learning
curl -sL https://raw.githubusercontent.com/affaan-m/everything-claude-code/main/skills/continuous-learning/evaluate-session.sh > ~/.claude/skills/continuous-learning/evaluate-session.sh
chmod +x ~/.claude/skills/continuous-learning/evaluate-session.sh
```

### Hook 配置

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
          }
        ]
      }
    ]
  }
}
```

Stop hook 在会话结束时触发 - 脚本分析会话中值得提取的模式（错误解决、调试技术、变通方法、项目特定模式等），并将它们保存为 `~/.claude/skills/learned/` 中可复用的 skills。

### 手动提取 /learn

你不必等到会话结束。仓库还包括一个 `/learn` 命令，你可以在会话中刚刚解决了非平凡问题时运行。它会提示你立即提取模式，起草 skill 文件，并在保存前请求确认。

### 其他自我改进记忆模式

一种方法涉及对会话日志进行反思以提炼用户偏好 - 本质上是建立关于什么有效、什么无效的"日记"。每次会话后，反思 agent 提取什么顺利、什么失败、你做了什么修正。这些学习更新一个在后续会话中加载的记忆文件。

另一种方法让系统每15分钟主动建议改进，而不是等待你注意模式。agent 审查最近的交互，提议记忆更新，你批准或拒绝。随着时间的推移，它从你的批准模式中学习。

---

## 五、Token 优化策略

### 主要策略：子 Agent 架构

主要优化你使用的工具和子 agent 架构，旨在委托给能够完成任务的最便宜的模型以减少浪费。

**模型选择快速参考：**

| 任务类型 | 推荐模型 | 原因 |
|---------|---------|------|
| 90% 编码任务 | Sonnet | 平衡性能和成本 |
| 首次尝试失败 | Opus | 复杂任务需要更强能力 |
| 跨5+文件的任务 | Opus | 需要更强的上下文理解 |
| 架构决策 | Opus | 关键决策需要最佳推理 |
| 安全关键代码 | Opus | 不能妥协代码质量 |
| 重复性任务 | Haiku | 指令清晰，执行简单 |
| 多 agent 设置中的"工作者" | Haiku | 成本敏感的批量操作 |

**成本对比：**
- Sonnet 4.5: $3/百万输入 tokens, $15/百万输出 tokens
- 相比 Opus 节省约 66.7%
- Haiku vs Opus: 5x 成本差异
- Haiku vs Sonnet: 1.67x 价格差异

**Haiku + Opus 组合最合理。**

### 在 Agent 定义中指定模型

```yaml
---
name: quick-search
description: Fast file search
tools: Glob, Grep
model: haiku # Cheap and fast
---
```

### 工具特定优化

考虑 Claude 最频繁调用的工具。例如，用 mgrep 替换 grep - 在各种任务上平均有效减少约一半的 tokens，相比传统 grep 或 ripgrep。

### 后台进程

适用时，在 Claude 之外运行后台进程，如果你不需要 Claude 处理整个输出并实时流式传输。这可以通过 tmux 轻松实现。获取终端输出并仅总结或复制你需要的部分。这将节省大量输入 tokens - Opus 4.5 为 $5/百万 tokens，输出为 $25/百万 tokens。

### 模块化代码库的好处

拥有更模块化的代码库，包含可复用的实用程序、函数、hooks 等 - 主文件在数百行而不是数千行 - 有助于 token 优化成本和一次性正确完成任务，这两者相关。如果你必须多次提示 Claude，你正在消耗 tokens，特别是当它反复阅读非常长的文件时。

**模块化代码库示例：**
```
root/
├── docs/                    # 全局文档
├── scripts/                 # CI/CD 和构建脚本
├── src/
│   ├── apps/                # 入口点（API、CLI、Workers）
│   │   ├── api-gateway/     # 将请求路由到模块
│   │   └── cron-jobs/
│   │
│   ├── modules/             # 系统核心
│   │   ├── ordering/        # 自包含的"订单"模块
│   │   │   ├── api/         # 其他模块的公共接口
│   │   │   ├── domain/      # 业务逻辑和实体（纯）
│   │   │   ├── infrastructure/  # DB、外部客户端、仓库
│   │   │   ├── use-cases/   # 应用逻辑（编排）
│   │   │   └── tests/       # 单元和集成测试
│   │   │
│   │   ├── catalog/         # 自包含的"目录"模块
│   │   │   ├── domain/
│   │   │   └── ...
│   │   │
│   │   └── identity/        # 自包含的"认证/用户"模块
│   │       ├── domain/
│   │       └── ...
│   │
│   ├── shared/              # 每个模块都使用的代码
│   │   ├── kernel/          # 基类（Entity、ValueObject）
│   │   ├── events/          # 全局事件总线定义
│   │   └── utils/           # 深度通用的助手
│   │
│   └── main.ts              # 应用引导
├── tests/                   # 端到端（E2E）全局测试
├── package.json
└── README.md
```

### 精简代码库 = 更便宜的 Tokens

这可能是显而易见的，但代码库越精简，token 成本就越低。通过使用 skills 持续清理代码库，重构使用 skills 和命令来识别死代码至关重要。此外，在某些时候，我喜欢浏览整个代码库，寻找突出或看起来重复的内容，手动拼凑该上下文，然后将其与重构 skill 和死代码 skill 一起提供给 Claude。

### 系统提示瘦身（高级）

对于真正关注成本的人：Claude Code 的系统提示占用约 18k tokens（约 200k 上下文的 9%）。这可以通过补丁减少到约 10k tokens，节省约 7,300 tokens（静态开销的 41%）。如果你想要这条路线，请参阅 YK 的指南。作者个人不这样做。

---

## 六、评估和 Harness 调优

### 可观察性方法

一种方法是让 tmux 进程挂钩到跟踪思考流，并在触发 skill 时输出。另一种方法是使用 PostToolUse hook 记录 Claude 具体执行了什么以及确切的更改和输出是什么。

### 基准测试工作流

将使用 skill 的结果与不使用 skill 请求相同内容并检查输出差异进行比较，以基准测试相对性能：

```
[Same Task]
     │
     ┌────────┴────────┐
     ▼                 ▼
┌───────────────┐ ┌───────────────┐
│  Worktree A   │ │  Worktree B   │
│  WITH skill   │ │  WITHOUT skill│
└───────┬───────┘ └───────┬───────┘
        │                 │
        ▼                 ▼
   [Output A]        [Output B]
        │                 │
        └──────────┬──────┘
                   ▼
              [git diff]
                   │
                   ▼
          ┌────────────────┐
          │ Compare logs,  │
          │ token usage,   │
          │ output quality │
          └────────────────┘
```

分叉对话，在其中一个创建没有 skill 的新 worktree，最后拉出 diff，查看记录了什么。这与持续学习和记忆部分相关。

### 评估模式类型

**检查点式 vs 持续式：**

```
CHECKPOINT-BASED          CONTINUOUS
─────────────────         ──────────
[Task 1]                  [Work]
   │                         │
   ▼                         ▼
┌─────────┐            ┌─────────┐
│Checkpoint│◄── verify │ Timer/  │
│   #1    │   criteria │ Change  │
└────┬────┘            └────┬────┘
     │ pass?                │
  ┌──┴──┐                   ▼
yes│   no──► fix──┐    ┌──────────┐
  │               │    │Run Tests │
  ▼               └────┤  + Lint  │
[Task 2]               └────┬─────┘
   │                        │
   ▼                   ┌────┴────┐
┌─────────┐           pass    fail
│Checkpoint│            │        │
│   #2    │            ▼        ▼
└────┬────┘       [Continue][Stop & Fix]
   │                        │
  ...                      └────
```

**检查点式评估：**
- 在工作流中设置明确的检查点
- 在每个检查点验证定义的标准
- 如果验证失败，Claude 必须在继续之前修复
- 适用于有明确里程碑的线性工作流

**持续式评估：**
- 每 N 分钟或重大更改后运行
- 完整测试套件、构建状态、lint
- 立即报告回归
- 停止并在继续之前修复
- 适用于长时间运行的会话

### 评分器类型

| 类型 | 特点 |
|------|------|
| **基于代码的评分器** | 字符串匹配、二进制测试、静态分析、结果验证。快速、便宜、客观，但对有效变化脆弱 |
| **基于模型的评分器** | 评分标准、自然语言断言、成对比较。灵活且处理细微差别，但不确定且更昂贵 |
| **人工评分器** | SME 审查、众包判断、抽查采样。黄金标准质量，但昂贵且慢 |

### 关键指标

```
pass@k: k 次尝试中至少一次成功
┌─────────────────────────────────────┐
│ k=1: 70%   k=3: 91%   k=5: 97%     │
│ 更高的 k = 更高的成功几率            │
└─────────────────────────────────────┘

pass^k: 所有 k 次尝试都必须成功
┌─────────────────────────────────────┐
│ k=1: 70%   k=3: 34%   k=5: 17%     │
│ 更高的 k = 更难（一致性）            │
└─────────────────────────────────────┘
```

- 当你只需要它工作且任何验证反馈就足够时使用 `pass@k`
- 当一致性至关重要且你需要接近确定性的输出一致性（结果/质量/风格）时使用 `pass^k`

### 构建评估路线图

1. 尽早开始 - 从真实失败中获取 20-50 个简单任务
2. 将用户报告的失败转换为测试用例
3. 编写明确的任务 - 两个专家应该达成相同的结论
4. 构建平衡的问题集 - 测试行为应该和不应该发生的情况
5. 构建健壮的 harness - 每次试验从干净环境开始
6. 评分 agent 产生的内容，而不是它采取的路径
7. 阅读许多试验的记录
8. 监控饱和 - 100% 通过率意味着添加更多测试

---

## 七、并行化策略

### 分叉会话

在多 Claude 终端设置中分叉会话时，确保分叉和原始会话中操作的范围定义明确。在代码更改方面以最小重叠为目标。选择相互正交的任务以防止干扰的可能性。

### 首选模式

作者个人更喜欢主聊天进行代码更改，分叉用于关于代码库及其当前状态的问题，或研究外部服务，如拉取文档、搜索 GitHub 寻找适用的开源仓库，或其他有帮助的一般研究。

### 关于任意终端数量

Boris（Claude Code 创建者）建议在本地运行 5 个 Claude 实例，5 个上游。作者建议不要设置这样的任意终端数量。终端和实例的添加应该出于真正的必要和目的。

如果你可以使用脚本完成任务，使用脚本。如果你可以在主聊天中让 Claude 在 tmux 中启动实例并以那种方式在单独的终端中流式传输，这样做。

**你的目标应该是：用最小可行的并行化量完成尽可能多的事情。**

对于大多数新手，作者甚至建议远离并行化，直到你掌握运行单个实例并在其中管理一切的技巧。大多数时候，即使作者也只使用约 4 个终端。作者发现通常只用 2 或 3 个 Claude 实例就能完成大多数事情。

### 扩展实例时

如果你要开始扩展实例并且你有多个 Claude 实例处理相互重叠的代码，必须使用 git worktrees 并为每个制定非常明确的计划。此外，为了在恢复会话时不混淆或迷失哪个 git worktree 是为了什么（除了树的名称），使用 `/rename <name>` 命名所有聊天。

### Git Worktrees 用于并行实例

```bash
# 为并行工作创建 worktrees
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-refactor refactor-branch

# 每个 worktree 获得自己的 Claude 实例
cd ../project-feature-a && claude
```

**好处：**
- 实例之间没有 git 冲突
- 每个都有干净的工作目录
- 易于比较输出
- 可以跨不同方法对相同任务进行基准测试

### 级联方法

运行多个 Claude Code 实例时，使用"级联"模式组织：
- 在右侧的新标签页中打开新任务
- 从左到右扫描，从旧到新
- 保持一致的方向流程
- 根据需要检查特定任务
- 一次最多关注 3-4 个任务 - 超过这个数量，心理开销比生产力增长得更快

---

## 八、项目启动模式

### 双实例启动模式

对于工作流管理（不是必要但有帮助），作者喜欢用 2 个打开的 Claude 实例启动一个空仓库。

**实例 1：脚手架 Agent**
- 铺设脚手架和基础
- 创建项目结构
- 设置配置（`.claude.json`、rules、agents - 简明指南中的所有内容）
- 建立约定
- 将骨架放到位

**实例 2：深度研究 Agent**
- 连接到所有服务、网络搜索等
- 创建详细的 PRD
- 创建架构 mermaid 图
- 使用实际文档中的实际片段编译参考

**启动设置：** 左终端用于编码，右终端用于问题 - 使用 `/rename` 和 `/fork`。

### llms.txt 模式

如果可用，你可以在许多文档参考上找到 llms.txt，方法是在到达文档页面后执行 `/llms.txt`。这为你提供了可以直接提供给 Claude 的干净、LLM 优化的文档版本。

---

## 九、构建可复用模式

### 理念

一个关键见解："早期，我花时间构建可复用的工作流/模式。构建起来很乏味，但随着模型和 agent harness 的改进，这产生了疯狂的复合效应。"

**值得投资的：**
- Subagents（简明指南）
- Skills（简明指南）
- Commands（简明指南）
- 规划模式
- MCP 工具（简明指南）
- 上下文工程模式

**为什么它会复合：** "最好的部分是所有这些工作流都可以转移到其他 agent 如 Codex。" 一旦构建，它们就可以跨模型升级工作。对模式的投资 > 对特定模型技巧的投资。

---

## 十、子 Agent 上下文问题

### 问题描述

子 agent 通过返回摘要而不是倾倒所有内容来节省上下文。但编排器拥有子 agent 缺乏的语义上下文。子 agent 只知道字面查询，不知道请求背后的目的/推理。摘要经常遗漏关键细节。

**类比：** "你的老板派你去开会并要求摘要。你回来给他汇报。十有八九，他会有后续问题。你的摘要不会包含他需要的一切，因为你没有他拥有的隐式上下文。"

### 迭代检索模式

```
┌─────────────────┐
│   ORCHESTRATOR  │
│  (has context)  │
└────────┬────────┘
         │ dispatch with query + objective
         ▼
┌─────────────────┐
│   SUB-AGENT     │
│ (lacks context) │
└────────┬────────┘
         │ returns summary
         ▼
┌─────────────────┐    ┌─────────────┐
│    EVALUATE     │──no►│  FOLLOW-UP  │
│   Sufficient?   │     │  QUESTIONS  │
└────────┬────────┘     └──────┬──────┘
         │ yes                 │
         ▼                     │ sub-agent
      [ACCEPT]                 │ fetches answers
         │                     │
         ◄─────────────────────┘
           (max 3 cycles)
```

**解决方案：**
- 让编排器评估每个子 agent 返回
- 在接受之前提出后续问题
- 子 agent 返回源，获取答案，返回
- 循环直到足够（最多 3 个循环以防止无限循环）

传递目标上下文，而不仅仅是查询。在分派子 agent 时，包括特定查询和更广泛的目标。这有助于子 agent 优先考虑在其摘要中包含什么。

---

## 十一、编排器与顺序阶段模式

```markdown
Phase 1: RESEARCH (use Explore agent)
- 收集上下文
- 识别模式
- 输出: research-summary.md

Phase 2: PLAN (use planner agent)
- 阅读 research-summary.md
- 创建实施计划
- 输出: plan.md

Phase 3: IMPLEMENT (use tdd-guide agent)
- 阅读 plan.md
- 先写测试
- 实现代码
- 输出: 代码更改

Phase 4: REVIEW (use code-reviewer agent)
- 审查所有更改
- 输出: review-comments.md

Phase 5: VERIFY (use build-error-resolver if needed)
- 运行测试
- 修复问题
- 输出: 完成或循环返回
```

**关键规则：**
1. 每个 agent 获得一个明确的输入并产生一个明确的输出
2. 输出成为下一阶段的输入
3. 永不跳过阶段 - 每个都增加价值
4. 在 agent 之间使用 `/clear` 保持上下文新鲜
5. 将中间输出存储在文件中（不仅仅是内存）

---

## 十二、Agent 抽象层级

### Tier 1: 直接增强（易于使用）

| 模式 | 说明 |
|------|------|
| **Subagents** | 防止上下文腐烂和临时专业化的直接增强。不如多 agent 有用，但复杂度低得多 |
| **元提示** | "我花 3 分钟提示一个 20 分钟的任务。" 直接增强 - 提高稳定性并健全检查假设 |
| **开始时多问用户** | 通常是增强，虽然你必须在计划模式下回答问题 |

### Tier 2: 高技能门槛（更难用好）

| 模式 | 说明 |
|------|------|
| **长时间运行的 agent** | 需要理解 15 分钟任务 vs 1.5 小时 vs 4 小时任务的形状和权衡。需要一些调整，显然是长时间的试错 |
| **并行多 agent** | 非常高的方差，只对高度复杂或分割良好的任务有用 |
| **基于角色的多 agent** | "模型演进太快，无法进行硬编码启发式，除非套利非常高。" 难以测试 |
| **计算机使用 agent** | 非常早期的范式，需要管理 |

**要点：** 从 Tier 1 模式开始。只有在掌握基础知识并有真正需求时才升级到 Tier 2。

---

## 十三、用 Skills/Commands 替代 MCP

### 原理

对于版本控制（GitHub）、数据库（Supabase）、部署（Vercel、Railway）等 MCP - 这些平台大多数已经有健壮的 CLI，MCP 本质上只是在包装它们。MCP 是一个不错的包装器，但它有成本。

要让 CLI 功能更像 MCP 而不实际使用 MCP（以及随之而来的上下文窗口减少），考虑将功能捆绑到 skills 和 commands 中。剥离 MCP 暴露的使其易于使用的工具，将它们转换为命令。

**示例：** 与其让 GitHub MCP 始终加载，不如创建一个 `/gh-pr` 命令，用你喜欢的选项包装 `gh pr create`。与其让 Supabase MCP 消耗上下文，不如创建直接使用 Supabase CLI 的 skills。功能相同，便利性相似，但你的上下文窗口为实际工作释放了。

### 懒加载更新

自原作者发布原始文章以来的过去几天里，Boris 和 Claude Code 团队在内存管理和优化方面取得了很大进展，主要是 MCP 的懒加载，使它们不再从一开始就占用你的窗口。以前作者会建议在可能的情况下将 MCP 转换为 skills，通过以下两种方式之一启用 MCP 功能：在当时启用（不太理想，因为需要离开并恢复会话）或使用 skills 使用 CLI 类似物（如果存在）并让 skill 成为它的包装器 - 本质上让它充当伪 MCP。

**有了懒加载，上下文窗口问题基本解决了。但 token 使用和成本没有以同样的方式解决。** CLI + skills 方法仍然是一种 token 优化方法，其效果可能与使用 MCP 相当或接近。此外，你可以通过 CLI 而不是在上下文中运行 MCP 操作，这显着减少了 token 使用，对于繁重的 MCP 操作（如数据库查询或部署）特别有用。

---

## 参考资料

- [Anthropic: Demystifying evals for AI agents](https://anthropic.com) (Jan 2026)
- Anthropic: "Claude Code Best Practices" (Apr 2025)
- Fireworks AI: "Eval Driven Development with Claude Code" (Aug 2025)
- [YK: 32 Claude Code Tips](https://yk.do) (Dec 2025)
- Addy Osmani: "My LLM coding workflow going into 2026"
- Sub-Agent Context Negotiation
- Agent Abstractions Tierlist
- Compound Effects Philosophy
- [RLanceMartin: Session Reflection Pattern](https://github.com/rlancemartin)
- Self-Improving Memory System

---

## 相关知识点

- [[Claude Code 使用指南]]
- [[Docker 常用命令速查]]

---
*翻译自 Claude Code 对话*
