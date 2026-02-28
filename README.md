# claw-multi-agent 🐝

> OpenClaw 多 Agent 并行编排 Skill — 让 AI 像团队一样协作

用多个 AI Agent 同时工作，把串行变并行，**实测节省 50-65% 时间**。

自动使用你 OpenClaw 里已有的模型，**零配置，装完即用**。

## 安装

```bash
npx --yes skills add https://github.com/你的账号/claw-multi-agent
```

## 快速上手

安装后直接对 OpenClaw 说：

- "帮我并行调研 LangChain、CrewAI、AutoGen 三个框架"
- "让多个 Agent 同时搜索这几个主题，然后整合报告"
- "用 multi-agent 模式对比几个模型的回答"

## 两种工作模式

### 🎯 指挥官模式（有工具，能联网）

主 Agent 派发子 Agent，子 Agent 拥有**联网搜索、读写文件、执行代码**等完整工具。

```
你说：并行调研三个 AI 框架
  ↓
主 Agent 同时派出 3 个子 Agent
  ├── 🔍 Agent-1 搜索 LangChain → 返回摘要
  ├── 🔍 Agent-2 搜索 CrewAI   → 返回摘要
  └── 🔍 Agent-3 搜索 AutoGen  → 返回摘要
        ↓ 并行执行，约 25 秒
主 Agent 整合 → 写完整对比报告
```

适合：需要联网搜索、文件操作、代码执行的任务。

### 🔄 流水线模式（纯文本，真并行）

```bash
cd ~/.openclaw/skills/claw-multi-agent
python run.py --mode parallel \
  --agents "fast:研究员:调研LangChain的优缺点" \
           "fast:研究员:调研CrewAI的优缺点" \
           "smart:写作者:整合以上调研写对比报告" \
  --aggregation synthesize
```

适合：纯文本生成、多模型对比、写作分析。

## 实测性能

| 场景 | 串行 | 并行 | 节省 |
|------|------|------|------|
| 3 个主题同时调研 | ~75s | ~25s | **67%** ⚡ |
| 3 个模型对比回答 | ~63s | ~22s | **65%** ⚡ |

## 详细文档

完整使用指南见 [SKILL.md](./SKILL.md)，包含：
- contextSharing：给子 Agent 注入背景上下文
- 避坑指南（实战踩坑总结）
- 完整示例
