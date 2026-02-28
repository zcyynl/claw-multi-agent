# claw-multi-agent 🐝

**OpenClaw 多智能体并行编排 Skill**

---

## 为什么需要多智能体？

OpenClaw 默认是**单 Agent 串行**工作：一个问题问完等回答，再问下一个。

这带来三个问题：

- ⏱ **慢**：3 个调研任务要等 3 倍时间
- 🧠 **上下文污染**：所有中间过程都堆在同一个会话里，越跑越慢越贵
- 🎯 **视角单一**：一个 AI 做所有事，容易有盲区

**claw-multi-agent 解决这三个问题：**

| 问题 | 解决方式 | 效果 |
|------|----------|------|
| 慢 | 多个 Agent 同时跑 | **节省 50-67% 时间** ⚡ |
| 上下文污染 | 每个子 Agent 独立会话，只返回摘要 | **主线程 token 消耗降低 60-80%** 💰 |
| 视角单一 | 不同 Agent 专注不同子任务 | **结果更全面、更专业** 🎯 |

---

## 能做什么？

```
你说：帮我调研 LangChain、CrewAI、AutoGen 三个框架

普通 OpenClaw（串行）：
  搜索 LangChain... 等待 25s
  搜索 CrewAI...    等待 25s   → 共 75s，上下文越来越长
  搜索 AutoGen...   等待 25s

claw-multi-agent（并行）：
  ┌── 🔍 Agent-1 搜索 LangChain ──┐
  ├── 🔍 Agent-2 搜索 CrewAI    ──┤ → 同时跑，共 25s
  └── 🔍 Agent-3 搜索 AutoGen  ──┘
  主 Agent 整合 → 写完整报告
```

**实测数据**：

| 场景 | 串行 | 并行 | 节省 |
|------|------|------|------|
| 3 个主题同时调研 | ~75s | ~25s | **67%** ⚡ |
| 5 个 Agent 对比分析 | ~125s | ~28s | **78%** ⚡ |

---

## 安装

```bash
npx --yes skills add https://github.com/zcyynl/claw-multi-agent
```

装完即用，**自动使用你 OpenClaw 里已有的模型，零配置**。

---

## 快速上手

安装后直接说：

- "帮我并行调研 LangChain、CrewAI、AutoGen 三个框架"
- "让多个 Agent 同时搜索这几个主题，然后整合报告"
- "用 multi-agent 模式对比几个方案的优缺点"

---

## 两种工作模式

### 🎯 指挥官模式（能联网 + 有工具）

主 Agent 通过 `sessions_spawn` 派发子 Agent，子 Agent 拥有联网搜索、读写文件、执行代码等完整工具。

适合：**需要真实搜索、文件操作的任务**

### 🔄 流水线模式（纯文本，极速并行）

```bash
cd ~/.openclaw/skills/claw-multi-agent

# 并行：多个 Agent 同时回答
python run.py --mode parallel \
  --agents "fast:研究员:调研LangChain的核心特性" \
           "fast:研究员:调研CrewAI的核心特性" \
           "smart:写作者:整合报告"

# 自动路由：让路由器自动拆任务、分配模型
python run.py --auto-route --task "调研三个AI框架并写对比报告"

# 预览模式：只看会执行什么，不实际运行
python run.py --dry-run --agents "fast:研究员:调研X" "smart:写作者:写报告"
```

适合：**纯文本生成、多模型对比、写作分析**

---

## 内置智能路由

自动分析任务类型，派给最合适的 Agent：

```bash
python scripts/router.py classify "写一个 Python 爬虫"
# → Tier: CODE ✅

python scripts/router.py classify "调研 LangChain 框架"  
# → Tier: RESEARCH ✅
```

| 类型 | 触发场景 |
|------|---------|
| `FAST` | 简单查询、翻译、状态检查 |
| `CODE` | 编程、调试、脚本实现 |
| `RESEARCH` | 调研、搜索、对比分析 |
| `CREATIVE` | 写作、文案、文档撰写 |
| `REASONING` | 架构设计、复杂推理 |

---

## 详细文档

见 [SKILL.md](./SKILL.md)，包含：
- contextSharing：给子 Agent 注入背景上下文
- 避坑指南（实战踩坑总结）
- 完整示例和参数速查

---

> 💡 **设计理念**：让每个 Agent 只做一件事，做好一件事。主 Agent 负责整合，子 Agent 负责执行。分工协作，比单打独斗快得多。
