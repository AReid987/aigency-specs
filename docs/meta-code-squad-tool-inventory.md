# Meta Code Squad — Full Tool & Quota Inventory
## Every Free Agent CLI + API Key in the Stack
**Date:** 2026-03-06 | **Owner:** Antonio Reid
**Purpose:** Ground truth for SimpleLLMRouter config and justfile routing logic

---

## 1. CLI Coding Agents (Local Binaries)

| Tool | Model(s) | Daily Quota | Notes |
|------|----------|-------------|-------|
| **Claude Code** (`claude`) | Claude Sonnet/Opus | ~generous but $$ | Complex logic, state machines, ruflo nervous system. Use surgically. |
| **Gemini CLI** (`gemini`) | Gemini 2.5 Pro/Flash | 1,000 req/day | Primary GSD executor. Planning, boilerplate, config, docs. |
| **Kimi Code CLI** (`kimi`) | Kimi K2 (128K) | ~unknown, 3x usage per 5hr window | Full-codebase generation + review sweeps. Strong coder, not just reviewer. |
| **iFlow CLI** (`iflow`) | GLM-5, Roma, Kimi K2, Qwen3, DeepSeek v3 | Free, unknown limit | Multi-model routing built in. Built-in commands: /scaffold, /qa, /refactor, /understand, /mermaid, multi-agent review, todos→GitHub issues. Install: `npm i -g @iflow-ai/iflow-cli` |
| **Qwen Code CLI** | Qwen3 Coder | 2,000 req/day | Highest daily quota of any free CLI. Good for high-volume boilerplate. |
| **Amp CLI** (`amp`) | Gemini family | 1,000 req/day (Google login) | `/mode free` for frontier model access. `/stats model` to check usage. |
| **Rovo Dev CLI** (`rovo`) | Multi-model, #1 SWE-bench (41.98%) | Unknown free tier | Atlassian-native. Jira/Confluence/Bitbucket integration. Adaptive memory. Best for issue-driven workflows. |
| **Kiro CLI** (`kiro`) | Claude Sonnet 4.5 / Auto mix | 50 interactions/month (free preview) | Spec-driven: requirements.md → design.md → tasks.md. Steering files in `.kiro/steering/`. Agent hooks on file events. AWS-native. Low quota — use for spec generation only. |

---

## 2. Free API Keys (Route via SimpleLLMRouter)

| Provider | Key Holder | Models | Daily Quota | Use Case |
|----------|-----------|--------|-------------|----------|
| **NVIDIA API** | Antonio | NVIDIA NIM models (Llama, Mistral, etc.) | Unknown — assume generous | High-throughput overflow. NIM models are fast. |
| **Bonsai API** | Antonio | GPT-5 or Anthropic (hidden for training) | Unknown | Premium quality overflow. Hidden model = treat as frontier-class. |
| **OpenRouter** (`openrouter/free` gateway) | Antonio | Auto model selection from free tier | 1,000 req/day | Auto-selects best available free model. Good general overflow. |
| **Giga AI** | Antonio | Unknown | Unknown | Additional overflow buffer. |

---

## 3. Missing from Previous Spec (Now Added)

### AgentDB Intelligence Controller (Claude-Flow / Ruflo)
The `--full` install of ruflo includes **AgentDB v1.3.9** — this is NOT just storage, it is a full intelligence layer:

- **29 MCP tools** registered with Claude Code automatically
- **SONA** — Self-Optimizing Neural Architecture, <0.05ms routing adaptation
- **EWC++** — Prevents catastrophic forgetting between sessions
- **HNSW vector search** — <100µs pattern retrieval, O(log n)
- **ReasoningBank** — 5-phase trajectory learning: RETRIEVE → JUDGE → DISTILL → CONSOLIDATE → ROUTE
- **9 RL algorithms** — Q-Learning, SARSA, A2C, PPO, DQN, Decision Transformer, MCTS
- **Flash Attention** — 2.49–7.47x speedup
- **29 frontier memory tools** including: causal reasoning graphs, Reflexion memory with self-critique, skill library with semantic search, Merkle proof recall, nightly learner daemon

**Install:** `curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full`
**MCP:** `claude mcp add ruflo -- npx -y ruflo@latest mcp start`
**AgentDB init:** `npx agentdb@latest init ./agents.db`

### iFlow CLI (Completely Missing from Spec)
Free, Alibaba-affiliated, MCP-extensible. Multi-model routing built in (Kimi K2, Qwen3, DeepSeek v3, GLM-5).

**Key built-in commands:**
- `/init` — Full repo scan + summary file (like `/init deep` in Claude Code)
- `/scaffold` — Project scaffolding
- `/qa` — QA workflow
- `/refactor` — Codebase refactoring
- `/understand` — Full codebase analysis
- `/mermaid` — Generate architecture diagrams
- Multi-agent code review
- Convert TODOs → GitHub issues
- SubAgent orchestration
- 4 execution modes: Default, Plan, Accepting Edits, YOLO
- `/remember` — Force Letta learning pass

**This replaces several manual steps** we had planned as separate tools.

---

## 4. Corrected Agent Routing Hierarchy

### Tier 1 — Primary Executors (highest quality, use first)
```
Gemini CLI    → GSD planning, boilerplate, config, tests, docs
Kimi Code CLI → Full-codebase tasks requiring 128K context, generation + review
Claude Code   → Complex logic, security, state machines, ruflo nervous system itself
```

### Tier 2 — Specialized Roles
```
iFlow CLI     → /scaffold, /qa, /refactor, /understand, /mermaid, todos→issues
               Also acts as secondary multi-model router (Kimi K2/Qwen3/DeepSeek/GLM-5)
Rovo Dev CLI  → Issue-driven workflows, Jira/Confluence sync, SWE-bench-quality fixes
Amp CLI       → Gemini family overflow (1k/day separate from Gemini CLI quota)
Qwen Code CLI → High-volume boilerplate (2k/day, highest CLI quota)
Kiro CLI      → Spec generation only (requirements.md, design.md, tasks.md) — 50/month, use sparingly
```

### Tier 3 — API Overflow via SimpleLLMRouter (auto-selected)
```
OpenRouter/free  → Auto model selection, 1k/day
NVIDIA NIM       → High-throughput overflow, assume large quota
Bonsai           → Frontier-class overflow (GPT-5 or Anthropic hidden)
Giga AI          → Additional overflow buffer
```

### Routing Logic (SimpleLLMRouter v2 Config)
```
IF task.context_window > 64K     → Kimi Code CLI
IF task.type == "spec"           → Kiro CLI (if monthly quota available)
IF task.type == "scaffold"       → iFlow /scaffold OR Gemini CLI
IF task.type == "qa"             → iFlow /qa OR Kimi Code CLI
IF task.type == "refactor"       → iFlow /refactor (uses own multi-model routing)
IF task.type == "diagram"        → iFlow /mermaid
IF task.type == "issue-driven"   → Rovo Dev CLI
IF gemini_quota_remaining < 100  → Amp CLI (separate Gemini quota)
IF all_cli_quotas_exhausted      → OpenRouter → NVIDIA → Bonsai → Giga
IF task.complexity == "critical" → Claude Code (always available, use surgically)
```

---

## 5. What `just setup` Must Now Install

```bash
# CLIs
npm i -g @iflow-ai/iflow-cli          # iFlow (missing from original spec)
curl -fsSL https://cli.kiro.dev/install | bash  # Kiro CLI
# amp, rovo, qwen — verify install commands before adding

# Ruflo FULL install (not basic)
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full

# AgentDB MCP registration
claude mcp add ruflo -- npx -y ruflo@latest mcp start
npx agentdb@latest init $PROJECT_ROOT/agents.db

# SimpleLLMRouter — add all API keys to .env
NVIDIA_API_KEY=...
BONSAI_API_KEY=...
OPENROUTER_API_KEY=...
GIGA_AI_KEY=...
```

---

## 6. Claude-Flow Features Now Explicitly Included

Previously missing from the spec, now required:

| Feature | Status | Notes |
|---------|--------|-------|
| AgentDB v1.3.9 | Required — install with `--full` | 29 MCP tools, vector DB, RL |
| SONA intelligence routing | Auto-enabled via AgentDB | |
| ReasoningBank | Auto-enabled | Pattern learning across sessions |
| 60+ specialized agents | Bundled with ruflo `--full` | coder, reviewer, tester, security, architect... |
| Byzantine fault tolerance | Available | Raft, BFT, Gossip, CRDT |
| 42 pre-built skills | Auto-loaded | SPARC, TDD, swarm orchestration, GitHub review |
| Nightly learner daemon | Background — auto-runs | Reorganizes memory each night |
| Claude MCP integration | Requires `claude mcp add ruflo` | Gives Claude Code access to all 29 AgentDB tools |

---

## 7. iFlow Integration Points in the Squad

iFlow is not just a fallback — it becomes a **primary workflow tool** for specific commands:

```
just diagram          → iflow /mermaid (generates architecture diagrams)
just understand       → iflow /understand (full codebase analysis)
just todos-to-issues  → iflow (convert TODO comments → GitHub issues)
just qa               → iflow /qa OR kimi (QA sweep)
just scaffold PKG     → iflow /scaffold (package scaffolding)
just multi-review     → iflow multi-agent review (parallel review agents)
```

---

## 8. Updated Bootstrap Sequence

```bash
just scaffold          # turborepo + monorepo structure
just constitution      # copy docs/ + verify
just write-agent-context  # AGENT.md + CLAUDE.md + GEMINI.md
just install-stack     # all CLIs + ruflo --full + agentdb + venv
just configure-hooks   # ruflo hooks + git hooks + mcp registration
just dev               # start router + letta + sugar + loki
# In Claude Code:
/init deep             # one-time Letta bootstrap (~10 min)
# In iFlow:
/init                  # iFlow repo scan + summary
# Then:
just gsd               # Gemini CLI GSD planning → first PLAN-1.md
just run PLAN-1.md     # Loki RARV cycle begins
```

---

*This document supersedes the tool inventory section of meta-code-squad-master-system-spec.md*
*Next: Rewrite justfile to include all CLIs + AgentDB + iFlow commands*
