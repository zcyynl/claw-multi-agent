---
name: claw-multi-agent
description: OpenClaw 多 Agent 并行编排系统，让 AI 像团队一样协作。支持并行调研、多模型对比、代码流水线等场景，实测节省 50-65% 时间。触发词：multi-agent、多个agent、并行执行、多模型对比、分发子任务、并行调研、指挥官模式、让多个AI同时。
---

# claw-multi-agent 🐝

> **用 AI 团队替代单个 AI，把串行变并行，把小时变分钟。**

---

## 能做什么？

| 场景 | 例子 | 提速 |
|------|------|------|
| **并行调研** | 同时搜索 5 个技术框架，各自出报告 | ~65% ⚡ |
| **多模型对比** | 让 Kimi、Claude、Gemini 同时回答同一问题 | ~50% ⚡ |
| **代码流水线** | 规划 → 编码 → 审查，自动串行交接 | 质量↑ |
| **批量处理** | 同时翻译/分析/总结多份文档 | 按数量倍增 |

---

## ⚡ 30 秒上手

说以下任何一句话就能触发：

- "帮我并行调研 LangChain、CrewAI、AutoGen 三个框架"
- "让 Kimi 和 Claude 同时回答这个问题，对比一下"
- "用 swarm 模式帮我做这个调研"
- "派多个 Agent 同时搜索这几个主题"

---

## 两种工作模式

### 🎯 模式一：指挥官模式（有工具，能联网）

子 Agent 拥有**联网搜索、读写文件、执行代码**等完整工具。

**适用**：需要真实搜索、文件操作、代码执行的任务。

```
用户说：并行调研三个 AI 框架
   ↓
主 Agent（我）同时派出 3 个子 Agent
   ├── 🔍 Agent-1 搜索 LangChain → 返回摘要
   ├── 🔍 Agent-2 搜索 CrewAI   → 返回摘要  
   └── 🔍 Agent-3 搜索 AutoGen  → 返回摘要
         ↓（并行，约 25 秒）
主 Agent 整合 → 写完整对比报告
```

**关键规则**（避坑）：
- ✅ 子 Agent 只返回**搜索摘要**（100字以内每条）
- ✅ 主 Agent 自己**写完整报告**
- ❌ 不要让子 Agent 又搜索又写长文（token 上限会挂）
- ❌ 每次 spawn 必须指定 model，否则走默认不可控

### 🔄 模式二：流水线模式（纯文本，真并行）

通过 Python CLI 调度多个模型，**无工具限制**，适合纯文本生成。

**适用**：写作、翻译、分析、多模型对比等不需要联网的任务。

```bash
cd /workspace/openclaw/skills/claw-swarm

# 并行：3个模型同时回答
python run.py --mode parallel \
  --agents "kimi:分析师:分析这个问题的技术层面" \
           "gemini:写作者:用通俗语言解释这个问题" \
           "claude:评论家:指出这个方案的潜在风险" \
  --aggregation compare

# 串行：规划→执行→审核
python run.py --mode sequential \
  --agents "glm:规划师:拆解任务" \
           "kimi:开发者:实现代码" \
           "claude:审核员:审查质量" \
  --aggregation last
```

---

## 选哪种模式？

```
需要联网搜索？       → 指挥官模式
需要读写文件？       → 指挥官模式
需要执行代码？       → 指挥官模式
纯文本生成/写作？    → 流水线模式
多模型对比同一问题？  → 流水线模式
```

---

## 模型选择指南

| 任务类型 | 推荐模型 | 理由 |
|----------|----------|------|
| 联网搜索、整理摘要 | `glm` | 便宜够用，省成本 |
| 代码编写/调试 | `kimi` | 长上下文，中文代码好 |
| 写作、文档、创意 | `gemini` | 表达流畅，结构清晰 |
| 深度分析、最终交付 | `claude` | 质量有保证 |
| 极难推理、关键决策 | `opus` | 最强，但贵 |

**成本优化原则**：搜索/初稿用 `glm`，最终输出用 `claude`，中间步骤按需选。

---

## 🔗 contextSharing：让子 Agent 知道背景

子 Agent 默认是全新会话，**不知道你的目标和背景**。通过在 task 中加 `【背景】` 段，让它们对齐。

### 三种方式

**方式一：recent（推荐，95% 场景）**
把背景压缩成 1-2 句话放在 task 开头：

```
【背景】用户在研究 AI Agent 框架选型，目标是写一篇对比报告给技术团队看。

【你的任务】搜索 LangChain 的核心优缺点，返回 5 条摘要，每条 100 字以内。
```

**方式二：summary（串行场景）**
把前序 Agent 的结论附在任务末尾，避免重复搜索：

```
【你的任务】在以下调研结论基础上，补充搜索 AutoGen 的资料：

【已有结论】
- LangChain：生态最丰富，学习曲线陡
- CrewAI：角色分工清晰，适合多 Agent...

请返回 AutoGen 的 3 条独特亮点，与上面的结论不重复。
```

**方式三：full（上下文复杂时）**
让子 Agent 自己读文件：

```
【背景文件】请先读 /workspace/research/context.md 了解整体方向。

【你的任务】基于背景，搜索最新的 Test-Time Compute Scaling 进展，返回 3 条摘要。
```

### 并行调研标准模板

```python
# 一次写好背景，复用给所有子 Agent
BG = "用户研究 RL 训练技术，目标写对比报告给算法工程师。关注：GRPO/DAPO/PPO 对比、veRL 框架。"

sessions_spawn({"task": f"【背景】{BG}\n\n【任务】搜索 GRPO vs PPO 最新对比数据，返回 5 条摘要，每条 100 字以内。", "model": "glm", "label": "s1"})
sessions_spawn({"task": f"【背景】{BG}\n\n【任务】搜索 DAPO 算法原理和适用场景，返回 5 条摘要，每条 100 字以内。", "model": "glm", "label": "s2"})
sessions_spawn({"task": f"【背景】{BG}\n\n【任务】搜索 veRL 框架架构和性能数据，返回 5 条摘要，每条 100 字以内。", "model": "glm", "label": "s3"})
# 收到全部结果后，主 Agent 自己整合写完整报告
```

---

## 预设角色

| 角色 | Emoji | 默认模型 | 擅长 |
|------|-------|----------|------|
| `researcher` | 🔍 | glm | 联网搜索、信息整理 |
| `writer` | ✍️ | gemini | 报告写作、文档整理 |
| `coder` | 👨‍💻 | kimi | 代码编写、调试 |
| `analyst` | 📊 | kimi | 数据分析、统计 |
| `reviewer` | 🔎 | glm | 代码/内容审查 |
| `planner` | 📋 | glm | 任务规划、需求拆解 |

---

## 完整案例：技术调研报告

### 第一步：并行搜索（指挥官模式，~25秒）

```python
sessions_spawn({"task": "搜索 FastAPI 框架特点、性能数据，返回 5 条摘要，每条 80 字内。", "model": "glm", "label": "r1"})
sessions_spawn({"task": "搜索 Django 框架特点、适用场景，返回 5 条摘要，每条 80 字内。", "model": "glm", "label": "r2"})
sessions_spawn({"task": "搜索 Flask 框架特点、典型案例，返回 5 条摘要，每条 80 字内。", "model": "glm", "label": "r3"})
```

### 第二步：主 Agent 整合写报告

收到三个 Agent 的摘要后，主 Agent 直接调用 write 工具写完整报告（不再派发子 Agent）。

### 第三步（可选）：推送飞书文档

配合 `feishu-all-operations` skill，一行代码推送到飞书。

**实测数据**：3 个主题串行需要 ~75 秒，并行只需 ~25 秒，**节省 67%** ⚡

---

## 执行完成后的统计输出

每次任务完成，输出标准统计卡：

```
## 📊 执行统计

| Agent | 任务 | 模型 | 耗时 | 状态 |
|-------|------|------|------|------|
| 🔍 r1 | FastAPI 搜索 | glm | 22s | ✅ |
| 🔍 r2 | Django 搜索  | glm | 24s | ✅ |
| 🔍 r3 | Flask 搜索   | glm | 21s | ✅ |
| ✍️ 主 | 整合写报告   | claude | 35s | ✅ |

并行节省：~50s | 总耗时：~60s（vs 串行 ~110s）
```

---

## ⚠️ 避坑指南（实战血泪总结）

### 坑①：子 Agent 写大文件必挂
子 Agent 单次输出约 **4096 token 上限**，超出后工具调用截断，写文件死循环。

- ❌ task 里让子 Agent「搜索 + 写 2000 字报告」
- ✅ 子 Agent 只返回摘要，**主 Agent 自己写报告**

### 坑②：忘传 model 参数
不传 model = 走 session 默认模型 = 不可控、成本不符预期。

- ❌ `sessions_spawn({"task": "...", "label": "x"})`
- ✅ `sessions_spawn({"task": "...", "model": "glm", "label": "x"})`

### 坑③：流水线模式里用联网工具
`python run.py` 的子进程**没有** web_search、exec 等工具，纯文本只。

- ❌ 流水线模式里要求搜索最新信息
- ✅ 联网任务 → 指挥官模式（sessions_spawn）

### 坑④：并行任务互相依赖
同一批 spawn 的任务是并行的，**不能让 A 等待 B 的输出**。

- ❌ Agent-2 的 task 里说「基于 Agent-1 的结果继续」
- ✅ 并行任务各自独立，主 Agent 收齐后再整合

---

## 流水线模式参数速查

```bash
python run.py
  --mode parallel|sequential   # 并行或串行
  --agents "model:role:task"   # 可重复多次
  --aggregation synthesize|concatenate|compare|last
  --timeout 300                # 超时秒数
```

| aggregation | 效果 |
|-------------|------|
| `synthesize` | 主 Agent 汇总整理（默认） |
| `compare` | 并排对比各模型输出 |
| `concatenate` | 顺序拼接 |
| `last` | 只取最后一个结果（串行用） |
