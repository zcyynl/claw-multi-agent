# claw-multi-agent 🐝

**OpenClaw 多智能体并行编排 Skill**

---

## OpenClaw 原生能力 vs claw-multi-agent

> 先说清楚原生有什么、我们加了什么。

**OpenClaw 原生**只有一种多 Agent 能力：`sessions_spawn`

```
sessions_spawn → 派一个子 Agent → 等它完成 → 再派下一个
```

- ✅ 子 Agent 有完整工具（联网搜索、读写文件、执行代码）
- ❌ **只能串行**：等一个完成才能派下一个
- ❌ 没有并行机制，没有批量调度，没有结果自动聚合

**claw-multi-agent 加了什么：**

| 能力 | 原生 OpenClaw | claw-multi-agent |
|------|--------------|-----------------|
| 派子 Agent | ✅ sessions_spawn（串行） | ✅ 指挥官模式（继承并优化） |
| **真并行多 Agent** | ❌ | ✅ 流水线模式（CLI 并行） |
| 结果自动聚合 | ❌ | ✅ |
| 任务类型自动路由 | ❌ | ✅ router.py |
| **无工具轻量 Agent**（省 token） | ❌ | ✅ 流水线模式 |

---

## 三种模式的区别

### 🎯 指挥官模式 — 子 Agent 有工具，能联网，真并行

子 Agent 通过 `sessions_spawn` 派发，**每个子 Agent 都有完整工具**：
- 能 `web_search` 联网搜索
- 能 `read` / `write` 操作文件
- 能 `exec` 执行代码

**并行原理**：在同一轮工具调用里批量发出多个 `sessions_spawn`，OpenClaw 会同时执行它们，全部完成后主 Agent 统一收结果。

```
同一轮发出：
  sessions_spawn(搜索LangChain) ──┐
  sessions_spawn(搜索CrewAI)   ──┤→ 同时跑
  sessions_spawn(搜索AutoGen)  ──┘
↓ 全部返回后，主 Agent 整合写报告
```

适合：**需要真实联网搜索、文件读写的任务**

---

### 🔀 混合模式 — 先联网搜索，再并行多版草稿

两种模式的组合：**有工具的调研 + 极速的并行写作**。

```
Phase 1（指挥官，联网，并行）：
  sessions_spawn × N → 搜索，返回摘要

Phase 2（流水线，无工具，并行）：
  多个 Agent 各自写一版草稿 → 你来挑最好的
```

适合：**"帮我搜索竞品资料，然后写几个版本的分析报告"**

---

### 🔄 流水线模式 — 真并行，无工具，纯文本

通过 OpenClaw CLI 调度多个独立 Agent 会话，**真正同时并行运行**：

```
3 个 Agent 同时启动 → 同时跑 → 同时返回结果
串行 75s → 并行 25s，节省 67%
```

**限制**：子 Agent **没有工具**，只能基于自身知识回答，不能联网

适合：
- 多模型对比（同一问题让 3 个 AI 各自回答，看谁更好）
- 基于已有知识的写作、分析、翻译
- 快速生成多个角度的草稿
- 不需要实时信息的任务

```bash
cd ~/.openclaw/skills/claw-multi-agent

# 3 个 Agent 并行分析同一问题的不同角度
python run.py --mode parallel \
  --agents "default:技术专家:从技术角度分析 LangChain 的优缺点" \
           "default:产品经理:从产品角度分析 LangChain 的优缺点" \
           "default:初学者:从易用性角度分析 LangChain 的优缺点"

# 自动路由：路由器自动拆任务
python run.py --auto-route --task "对比分析三个框架的设计思路"

# 预览模式：只看会执行什么，不实际运行
python run.py --dry-run --agents "default:研究员:分析X" "default:写作者:写报告"
```

---

## 实测数据

| 场景 | 模式 | 串行 | 并行 | 节省 |
|------|------|------|------|------|
| 2 个主题同时分析 | 流水线 | ~13s | ~7s | **46%** ⚡ |
| 3 个主题同时调研 | 指挥官 | ~75s | ~25s | **67%** ⚡ |
| 4 Agent 深度调研 | 指挥官 | ~322s | ~96s | **70%** ⚡ |
| 5 个 Agent 对比分析 | 流水线 | ~125s | ~28s | **78%** ⚡ |

---

## 安装

```bash
npx --yes skills add https://github.com/zcyynl/claw-multi-agent
```

装完即用，**自动使用你 OpenClaw 里已有的模型，零配置**。

---

## 快速上手

安装后直接对话：

- "帮我并行分析 LangChain 和 CrewAI 各自的优缺点"
- "用 multi-agent 模式让多个角色同时分析这个方案"
- "让 3 个 AI 同时写这篇文案，我挑最好的"

**不需要说模式**，直接描述需求，skill 自动判断：
- 说"搜索/查找/调研/最新" → 自动用指挥官模式联网
- 说"几个版本/多个角度/让我挑" → 自动生成多版草稿
- 两个都说 → 自动进入混合模式（先搜索，再多版）

---

## 内置智能路由

自动分析任务类型，选择合适的执行方式：

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
