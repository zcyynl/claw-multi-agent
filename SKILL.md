---
name: claw-multi-agent
description: Multi-agent parallel orchestration for OpenClaw. Spawn AI agents as a team — parallel research, multi-model comparison, code pipelines. Proven 50-65% time savings. Trigger words: multi-agent, parallel agents, swarm, spawn multiple agents, parallel research, compare models.
---

# claw-multi-agent 🐝

> **Replace one AI with a team of AIs. Turn serial into parallel. Turn hours into minutes.**

---

## What can it do?

| Scenario | Example | Speedup |
|----------|---------|---------|
| **Parallel research** | Search 5 frameworks simultaneously, each writes a report | ~65% ⚡ |
| **Multi-model compare** | Ask Claude, Gemini, Kimi the same question at the same time | ~50% ⚡ |
| **Code pipeline** | Plan → Code → Review, auto hand-off in sequence | Quality ↑ |
| **Batch processing** | Translate / analyze / summarize multiple docs in parallel | Scales linearly |

---

## ⚡ Get started in 30 seconds

Just say something like:

- "Research LangChain, CrewAI, and AutoGen in parallel"
- "Have multiple agents search these topics and write a combined report"
- "Compare how Claude and Gemini answer this question"
- "Use multi-agent mode to do this research"

---

## Step 0: Always plan first (dynamic agent count)

**Never hardcode how many agents to spawn.** The right number depends on the task complexity. Always start with a planning step:

```
1. Analyze the task → identify subtopics / dimensions
2. Decide: how many agents? which roles? which mode?
3. Spawn accordingly (could be 2, could be 10)
4. Consolidate results
```

**Example planning output:**
```
Task: "Research the top AI agent frameworks"
→ Plan: 5 researchers (one per framework) + 1 analyst for comparison
→ Mode: Orchestrator (needs web search)
→ Spawn: 5 parallel sub-agents
```

**The number of agents should match the task, not a template.**

---

## Two modes — choose based on the task

```
Does the task need web search / file I/O / code execution?
  YES → 🎯 Orchestrator Mode  (sessions_spawn, full tools)
  NO  → 🔄 Pipeline Mode      (run.py, pure text, faster)
```

Both modes support any number of agents. Both can run in parallel or sequential.

---

## 🎯 Orchestrator Mode (with tools, truly parallel)

Sub-agents launched via `sessions_spawn`. Each has full OpenClaw tools: web search, file read/write, code execution.

**⚡ How parallelism works:**
Call multiple `sessions_spawn` in the **same tool-call round** — OpenClaw executes them simultaneously. All sub-agents run at once; the main agent collects all results when they finish.

```
Same round → parallel execution:

sessions_spawn(task="Search LangChain...") ──┐
sessions_spawn(task="Search CrewAI...")    ──┤→ all run simultaneously
sessions_spawn(task="Search AutoGen...")   ──┘
sessions_spawn(task="Search LangGraph...") ─┘

↓  (all finish, main agent receives all 4 results)

Main agent consolidates → writes full report
```

**Sequential = spawn one, wait for result, then spawn next.** Use this only when a later task depends on an earlier result (e.g. write report AFTER research is done).

**How to spawn — always include role, model hint, and what to return:**

```python
# Parallel research: spawn all 4 in the same round → they run simultaneously
sessions_spawn({
    "task": "[CONTEXT] Comparing AI agent frameworks for a tech team report.\n\n[YOUR TASK] Search LangChain: architecture, pros/cons, GitHub stars, latest version. Return 5 bullet points ≤100 words each. Do NOT write a full report.",
    "label": "🔍 researcher-langchain [model: default]"
})
sessions_spawn({
    "task": "[CONTEXT] Same report.\n\n[YOUR TASK] Search CrewAI: architecture, pros/cons, GitHub stars, latest version. Return 5 bullet points ≤100 words each.",
    "label": "🔍 researcher-crewai [model: default]"
})
sessions_spawn({
    "task": "[CONTEXT] Same report.\n\n[YOUR TASK] Search AutoGen: architecture, pros/cons, GitHub stars, latest version. Return 5 bullet points ≤100 words each.",
    "label": "🔍 researcher-autogen [model: default]"
})
sessions_spawn({
    "task": "[CONTEXT] Same report.\n\n[YOUR TASK] Search LangGraph: architecture, pros/cons, GitHub stars, latest version. Return 5 bullet points ≤100 words each.",
    "label": "🔍 researcher-langgraph [model: default]"
})
# All 4 run in parallel → when all return, main agent consolidates and writes report
```

**Mixed: parallel then sequential** (most common pattern):
```python
# Phase 1: parallel research (spawn all at once)
sessions_spawn({"task": "[CONTEXT] ...\n\n[TASK] Search LangChain. 5 bullets ≤100 words.", "label": "🔍 researcher-langchain"})
sessions_spawn({"task": "[CONTEXT] ...\n\n[TASK] Search CrewAI. 5 bullets ≤100 words.", "label": "🔍 researcher-crewai"})
sessions_spawn({"task": "[CONTEXT] ...\n\n[TASK] Search AutoGen. 5 bullets ≤100 words.", "label": "🔍 researcher-autogen"})

# Phase 2: after all 3 return → main agent writes report (sequential, depends on research)
# (main agent does this directly, no need to spawn a writer)
```

**Key rules:**
- ✅ **Same round = parallel**: spawn multiple agents at once for independent tasks
- ✅ **Sequential**: spawn one, wait for result, then spawn next — only when tasks depend on each other
- ✅ Sub-agents return **summaries only** (≤100 words per point)
- ✅ Main agent **writes the full report** (avoids token limit failures)
- ✅ Label each agent clearly: role + what model it's using
- ❌ Don't ask a sub-agent to both search AND write a long report

---

## 🔄 Pipeline Mode (pure text, any task)

Runs agents via Python CLI. **No web search, but works for any pure-text task**: writing, analysis, translation, multi-model comparison, brainstorming, code generation.

```bash
cd ~/.openclaw/skills/claw-multi-agent

# Parallel: multiple agents tackle different angles simultaneously
python run.py --mode parallel \
  --agents "fast:🔍 researcher:summarize the pros of microservice architecture" \
           "fast:🔍 researcher:summarize the cons of microservice architecture" \
           "fast:🔍 researcher:list real-world companies using microservices and outcomes" \
           "smart:📊 analyst:compare microservices vs monolith for a 10-person startup" \
  --aggregation synthesize

# Sequential: chain agents, each builds on the previous output
python run.py --mode sequential \
  --agents "fast:📋 planner:break down how to build a REST API in Python" \
           "smart:👨‍💻 coder:implement the API based on the plan above" \
           "fast:🔎 reviewer:review the code for bugs and security issues" \
  --aggregation last

# Auto-route: router classifies task and picks tiers automatically
python run.py --auto-route --task "write a technical blog post about GRPO vs PPO"

# Dry-run: preview the plan without executing
python run.py --dry-run \
  --agents "fast:researcher:research X" "smart:writer:write report"
```

**Pipeline mode works great for:**
- Multi-angle analysis (spawn one agent per dimension)
- Multi-model comparison (same task, different models)
- Code pipeline (plan → code → review)
- Batch writing (translate/summarize N documents in parallel)

---

## Smart Router

Built-in task classifier. Auto-picks the right tier based on keywords:

```bash
python scripts/router.py classify "write a Python web scraper"
# → Tier: CODE  (routes to smart model)

python scripts/router.py classify "research the latest LLM papers"
# → Tier: RESEARCH  (routes to fast model)

python scripts/router.py spawn --json --multi "research X and write a report"
# → splits into 2 tasks: RESEARCH + CREATIVE
```

| Tier | Model | Used for |
|------|-------|---------|
| `FAST` | default (light) | Simple queries, status, translation, search |
| `CODE` | default (smart) | Programming, debugging, implementation |
| `RESEARCH` | default (light) | Research, search, compare, survey |
| `CREATIVE` | default (smart) | Writing, articles, documentation |
| `REASONING` | default (best) | Architecture, logic, complex analysis |

---

## contextSharing: Give sub-agents background

Sub-agents start as fresh sessions — they don't know your goal. Add a `[CONTEXT]` block.

**Pattern 1: recent** (recommended — works for 95% of cases)
```
[CONTEXT] User is comparing AI agent frameworks for a team report. Audience: engineers.

[YOUR TASK] Search LangChain pros and cons. Return 5 bullet points ≤100 words each.
```

**Pattern 2: summary** (sequential tasks — pass prior results forward)
```
[PRIOR FINDINGS]
- LangChain: richest ecosystem, steep curve
- CrewAI: clean role separation...

[YOUR TASK] Based on above, search AutoGen. Return 3 unique points not covered above.
```

**Pattern 3: full** (complex background — let agent read a file)
```
[CONTEXT FILE] Read /workspace/research/context.md for full background.

[YOUR TASK] Search latest Test-Time Compute Scaling advances. Return 3 summaries.
```

**Reuse context across parallel agents:**
```python
BG = "Researching RL post-training for ML engineers. Topics: GRPO/DAPO/PPO, veRL."

sessions_spawn({"task": f"[CONTEXT] {BG}\n\n[TASK] Search GRPO vs PPO benchmarks. 5 bullets ≤100 words.", "label": "🔍 researcher-grpo [model: default]"})
sessions_spawn({"task": f"[CONTEXT] {BG}\n\n[TASK] Search DAPO design. 5 bullets ≤100 words.", "label": "🔍 researcher-dapo [model: default]"})
sessions_spawn({"task": f"[CONTEXT] {BG}\n\n[TASK] Search veRL architecture. 5 bullets ≤100 words.", "label": "🔍 researcher-verl [model: default]"})
```

---

## Execution summary — always output this

After every multi-agent run, print a standard card:

```
## 📊 Execution Summary

Mode: 🎯 Orchestrator Mode (sessions_spawn, with tools)

| Agent | Role | Model | Time | Status |
|-------|------|-------|------|--------|
| 🔍 researcher-langchain | Researcher | default | 22s | ✅ |
| 🔍 researcher-crewai    | Researcher | default | 19s | ✅ |
| 🔍 researcher-autogen   | Researcher | default | 24s | ✅ |
| 🔍 researcher-langgraph | Researcher | default | 21s | ✅ |
| ✍️ main (consolidate)   | Writer     | default | 38s | ✅ |

Agents spawned: 4  |  Parallel time: ~24s  |  Serial equivalent: ~86s  |  Saved: ~62s (72%)
```

**Always include:**
- Mode (Orchestrator / Pipeline + Sequential/Parallel)
- Each agent's role emoji + name + model used
- Actual elapsed time per agent
- Total parallel time vs serial equivalent

---

## Preset roles

| Role | Emoji | Best for |
|------|-------|---------|
| `researcher` | 🔍 | Web search, info gathering |
| `writer` | ✍️ | Reports, documentation, articles |
| `coder` | 👨‍💻 | Code writing, debugging, implementation |
| `analyst` | 📊 | Data analysis, comparison, statistics |
| `reviewer` | 🔎 | Code / content review, QA |
| `planner` | 📋 | Task planning, decomposition |
| `critic` | 🧐 | Risk analysis, devil's advocate |

---

## ⚠️ Gotchas

### Gotcha 1: Sub-agent output token limit
Sub-agents have a ~4096 token output cap. Exceeded → tool args truncated → file writes silently fail.

- ❌ "search AND write a 2000-word report"
- ✅ Sub-agent returns summaries; **main agent writes the report**

### Gotcha 2: Orchestrator Mode has no tools in Pipeline Mode
`python run.py` processes have no `web_search`, `exec`, etc.

- ❌ Pipeline mode: "search the latest news on X"
- ✅ Anything needing real web access → Orchestrator Mode

### Gotcha 3: Parallel agents can't depend on each other
Agents spawned in the same round run simultaneously.

- ❌ Agent-2: "based on Agent-1's results..."
- ✅ Parallel = independent; sequential = chained

### Gotcha 4: Don't hardcode agent count
Match agents to the task, not to a template.

- ❌ Always spawn exactly 3 agents
- ✅ Plan first, then decide: simple task → 2 agents, complex → 8+ agents

---

## Pipeline mode quick reference

```bash
python run.py
  --mode parallel|sequential
  --agents "tier_or_model:🎭role:task description"   # repeatable, any number
  --aggregation synthesize|compare|concatenate|last
  --timeout 300
  --dry-run          # preview without executing
  --auto-route       # router picks tiers automatically
  --list-models      # show current model config
```

| Aggregation | Effect |
|-------------|--------|
| `synthesize` | Main agent summarizes all outputs (default) |
| `compare` | Side-by-side of each agent's output |
| `concatenate` | Outputs joined in order |
| `last` | Final agent's output only (sequential) |
