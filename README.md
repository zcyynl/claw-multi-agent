# claw-multi-agent 🐝

> OpenClaw skill for multi-agent parallel orchestration — run AI agents as a team

Spawn multiple AI agents simultaneously, turn serial into parallel. **Proven 50–65% time savings.**

Uses your existing OpenClaw models — **zero config, works out of the box.**

## Install

```bash
npx --yes skills add https://github.com/zcyynl/claw-multi-agent
```

## Quick start

After installing, just tell OpenClaw:

- "Research LangChain, CrewAI, and AutoGen in parallel"
- "Spawn multiple agents to search these topics and combine the results"
- "Compare how different models answer this question"
- "Use multi-agent mode to do this research task"

## Two modes

### 🎯 Orchestrator Mode (tools + web search)

Main agent spawns sub-agents via `sessions_spawn`. Each sub-agent has full OpenClaw tools: web search, file I/O, code execution.

```
You: research three AI frameworks in parallel
  ↓
Main agent spawns 3 sub-agents simultaneously
  ├── 🔍 Agent-1  LangChain → summary
  ├── 🔍 Agent-2  CrewAI    → summary
  └── 🔍 Agent-3  AutoGen   → summary
        ↓ ~25 seconds (parallel)
Main agent consolidates → full comparison report
```

### 🔄 Pipeline Mode (pure text, truly parallel)

```bash
cd ~/.openclaw/skills/claw-multi-agent

# Parallel comparison
python run.py --mode parallel \
  --agents "fast:researcher:research LangChain pros and cons" \
           "fast:researcher:research CrewAI pros and cons" \
           "smart:writer:write a comparison report" \
  --aggregation synthesize

# Auto-route: router picks tiers automatically
python run.py --auto-route --task "research LangChain and write a report"

# Preview without running
python run.py --dry-run --agents "fast:researcher:research X" "smart:writer:write report"
```

## Smart Router

Built-in task classifier — automatically picks the right agent tier:

```bash
python scripts/router.py classify "write a Python scraper"
# → Tier: CODE

python scripts/router.py classify "research LangChain framework"
# → Tier: RESEARCH
```

| Tier | Used for |
|------|---------|
| `FAST` | Simple queries, status, translation |
| `CODE` | Programming, debugging, implementation |
| `RESEARCH` | Research, search, compare, survey |
| `CREATIVE` | Writing, articles, documentation |
| `REASONING` | Architecture, logic, complex analysis |

## Benchmarks

| Scenario | Serial | Parallel | Saved |
|----------|--------|----------|-------|
| 3 topics researched simultaneously | ~75s | ~25s | **67%** ⚡ |
| 3 models answering same question | ~63s | ~22s | **65%** ⚡ |

## Full docs

See [SKILL.md](./SKILL.md) for:
- contextSharing: inject background into sub-agents
- Gotchas & lessons learned
- Complete examples
