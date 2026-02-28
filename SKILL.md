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

## Two modes

### 🎯 Mode 1: Orchestrator Mode (with tools, can browse the web)

Sub-agents have full OpenClaw tools: **web search, file read/write, code execution**.

**Use when**: tasks require real web search, file operations, or code execution.

```
You say: research three AI frameworks in parallel
   ↓
Main agent spawns 3 sub-agents simultaneously
   ├── 🔍 Agent-1  searches LangChain → returns summary
   ├── 🔍 Agent-2  searches CrewAI   → returns summary
   └── 🔍 Agent-3  searches AutoGen  → returns summary
         ↓ (parallel, ~25 seconds)
Main agent consolidates → writes full comparison report
```

**Key rules** (avoid common pitfalls):
- ✅ Sub-agents return **short summaries only** (≤100 words each)
- ✅ Main agent **writes the full report itself**
- ❌ Don't ask a sub-agent to both search AND write a long report (token limit will cause failure)
- ❌ Always specify `model` when calling `sessions_spawn`

### 🔄 Mode 2: Pipeline Mode (pure text, truly parallel)

Runs agents via Python CLI — **no tool dependency**, great for pure text generation.

**Use when**: writing, translation, analysis, multi-model comparison without web access.

```bash
cd ~/.openclaw/skills/claw-multi-agent

# Parallel: 3 agents answer at the same time
python run.py --mode parallel \
  --agents "fast:analyst:analyze the technical side of this problem" \
           "fast:writer:explain this in plain language" \
           "smart:critic:identify potential risks in this approach" \
  --aggregation compare

# Sequential: plan → implement → review
python run.py --mode sequential \
  --agents "fast:planner:break down the task" \
           "smart:coder:implement the code" \
           "fast:reviewer:review code quality" \
  --aggregation last

# Auto-route: let the router pick the best tier automatically
python run.py --auto-route --task "research LangChain and write a comparison report"

# Dry-run: preview what would run without executing
python run.py --dry-run \
  --agents "fast:researcher:research LangChain" \
           "smart:writer:write the report"
```

---

## Which mode to use?

```
Need web search?           → Orchestrator Mode (sessions_spawn)
Need file read/write?      → Orchestrator Mode
Need code execution?       → Orchestrator Mode
Pure text / writing?       → Pipeline Mode (run.py)
Multi-model comparison?    → Pipeline Mode
```

---

## Smart Router

claw-multi-agent includes a built-in task router that automatically classifies tasks and picks the right tier:

```bash
python scripts/router.py classify "research LangChain framework"
# → Tier: RESEARCH, Confidence: 0.95

python scripts/router.py spawn --json "write a Python web scraper"
# → {"tier": "CODE", "model": null, ...}

python scripts/router.py spawn --json --multi "research LangChain and write a report"
# → [{"tier": "RESEARCH"}, {"tier": "CREATIVE"}]
```

| Tier | Triggers on | Example |
|------|-------------|---------|
| `FAST` | Simple queries, status checks, translation | "what is the weather?" |
| `CODE` | Programming, debugging, scripts | "write a Python scraper" |
| `RESEARCH` | Research, search, compare, survey | "research LangChain" |
| `CREATIVE` | Writing, articles, documentation | "write a blog post" |
| `REASONING` | Architecture, logic, complex analysis | "design a microservice system" |

---

## contextSharing: Give sub-agents context

Sub-agents start as fresh sessions — they don't know your goal or background. Add a `[CONTEXT]` block to align them.

### Three patterns

**Pattern 1: recent (recommended, covers 95% of cases)**
Compress context into 1-2 sentences at the start of the task:

```
[CONTEXT] User is evaluating AI agent frameworks for a team comparison report. Target audience: engineers.

[YOUR TASK] Search LangChain's core pros and cons. Return 5 bullet points, each under 100 words.
```

**Pattern 2: summary (for sequential tasks)**
Attach prior agents' findings to avoid duplicate work:

```
[YOUR TASK] Based on the findings below, supplement with research on AutoGen:

[PRIOR FINDINGS]
- LangChain: richest ecosystem, steep learning curve
- CrewAI: clean role separation, great for multi-agent...

Return 3 unique highlights about AutoGen not covered above.
```

**Pattern 3: full (for complex background)**
Let the sub-agent read a file directly:

```
[CONTEXT FILE] Read /workspace/research/context.md for overall direction.

[YOUR TASK] Search latest Test-Time Compute Scaling advances. Return 3 summaries.
```

### Parallel research template

```python
# Write context once, reuse across all sub-agents
BG = "User researches RL post-training. Goal: comparison report for ML engineers. Topics: GRPO/DAPO/PPO, veRL."

sessions_spawn({"task": f"[CONTEXT] {BG}\n\n[TASK] Search GRPO vs PPO latest benchmarks. Return 5 bullet points, each under 100 words.", "model": "glm", "label": "s1"})
sessions_spawn({"task": f"[CONTEXT] {BG}\n\n[TASK] Search DAPO algorithm design and use cases. Return 5 bullet points.", "model": "glm", "label": "s2"})
sessions_spawn({"task": f"[CONTEXT] {BG}\n\n[TASK] Search veRL framework architecture and performance. Return 5 bullet points.", "model": "glm", "label": "s3"})
# After all results return, main agent consolidates and writes the full report
```

---

## Preset roles

| Role | Emoji | Best for |
|------|-------|---------|
| `researcher` | 🔍 | Web search, info gathering |
| `writer` | ✍️ | Reports, documentation |
| `coder` | 👨‍💻 | Code writing, debugging |
| `analyst` | 📊 | Data analysis, statistics |
| `reviewer` | 🔎 | Code / content review |
| `planner` | 📋 | Task planning, decomposition |

---

## Full example: Technical research report

### Step 1: Parallel search (Orchestrator Mode, ~25s)

```python
sessions_spawn({"task": "Search FastAPI framework features and benchmarks. Return 5 bullet points under 80 words each.", "label": "r1"})
sessions_spawn({"task": "Search Django framework features and use cases. Return 5 bullet points under 80 words each.", "label": "r2"})
sessions_spawn({"task": "Search Flask framework features and real-world examples. Return 5 bullet points under 80 words each.", "label": "r3"})
```

### Step 2: Main agent consolidates and writes

Once all three agents report back, the main agent uses the `write` tool to produce the full report — no sub-agent involved.

**Benchmark**: 3 topics serially takes ~75s, in parallel ~25s — **67% faster** ⚡

---

## Execution summary output

After every multi-agent run, print a standard summary:

```
## 📊 Execution Summary

| Agent | Task | Model | Time | Status |
|-------|------|-------|------|--------|
| 🔍 r1 | FastAPI search | default | 22s | ✅ |
| 🔍 r2 | Django search  | default | 24s | ✅ |
| 🔍 r3 | Flask search   | default | 21s | ✅ |
| ✍️ main | Write report | default | 35s | ✅ |

Parallel saving: ~50s | Total: ~60s (vs serial ~110s)
```

---

## ⚠️ Gotchas (learned the hard way)

### Gotcha 1: Sub-agent output token limit
Sub-agents have a ~4096 token output cap. If exceeded, tool call arguments get truncated to `{}` and file writes silently fail.

- ❌ Task: "search AND write a 2000-word report"
- ✅ Sub-agent returns summaries only; **main agent writes the report**

### Gotcha 2: Missing model parameter
Without `model`, `sessions_spawn` uses the session default — unpredictable and potentially expensive.

- ❌ `sessions_spawn({"task": "...", "label": "x"})`
- ✅ `sessions_spawn({"task": "...", "model": "glm", "label": "x"})`

### Gotcha 3: Using web tools in Pipeline Mode
`python run.py` sub-processes have **no** OpenClaw tools (no `web_search`, `exec`, etc.).

- ❌ Pipeline mode: "search the latest news on X"
- ✅ Anything needing tools → use Orchestrator Mode

### Gotcha 4: Parallel tasks depending on each other
Agents spawned in the same round run simultaneously — they can't wait for each other.

- ❌ Agent-2 task says "based on Agent-1's results..."
- ✅ Parallel agents work independently; main agent consolidates after all return

---

## Pipeline mode quick reference

```bash
python run.py
  --mode parallel|sequential       # run agents in parallel or serial
  --agents "tier:role:task"        # repeatable; tier = fast|smart|best or custom model
  --aggregation synthesize|compare|concatenate|last
  --timeout 300                    # seconds before giving up
  --dry-run                        # preview without executing
  --auto-route                     # let router classify and split task automatically
  --list-models                    # show current model config
```

| Aggregation | Effect |
|-------------|--------|
| `synthesize` | Main agent summarizes all outputs (default) |
| `compare` | Side-by-side comparison of each agent's output |
| `concatenate` | Outputs joined in order |
| `last` | Only the final agent's output (use with sequential) |
