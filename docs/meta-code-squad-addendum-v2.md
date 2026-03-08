# Meta Code Squad — Spec Addendum v2
## Corrections, Missing Systems & Design Decisions
**Date:** 2026-03-06 | **Owner:** Antonio Reid
**Status:** Authoritative — supersedes conflicting sections in master spec and tool inventory

---

## 0. What This Document Covers

Seven categories of gaps and corrections identified in review:

1. Complete verified API key arsenal (no OpenAI, no Anthropic keys)
2. Corrected context window and routing logic (Gemini > Kimi for large context)
3. iFlow CLI — fully documented, now a first-class squad member
4. claude-flow AgentDB / Intelligence Controller — explicitly required
5. Skills system — what is real, what is marketing, how to solve skill bloat
6. Spec-docs repo & constitution — design + structure decision
7. Cross-harness orchestration — BMAD→PRP, PDM/UV, unified context, protocol translation

---

## 1. Verified API Key Arsenal

> No OpenAI API key. No Anthropic API key. Do not route to either in SimpleLLMRouter.

### 1.1 Confirmed Keys (from `notes/blackout/COMPLETE_LLM_API_GUIDE.md`)

| Provider | Key | Free Quota | Rate Limits | Primary Use |
|----------|-----|-----------|-------------|-------------|
| **Mistral** | Yes | 1B req/month | 500K RPM | Bulk orchestration, parallel agents |
| **Groq** | Yes | 14,400 req/day | 30 RPM / 900 RPH | Ultra-fast inference, speed-critical |
| **Cerebras** | Yes | 14,400 req/day (x4 models) | 30 RPM / 900 RPH | High-volume, near-unlimited tokens |
| **VoidAI** | Yes | 125K credits/day (77 models) | Varies | Model variety, daily-rotating discounts |
| **OpenRouter** | Yes | 1,000 req/day | Varies | Diversity, auto-free model rotation |
| **Gemini API** | Yes | 1,500 RPD / 15 RPM | — | Large context (1M-2M), multimodal |
| **GitHub Models** | Yes | Free tier | Rate limited | GPT-4o, Llama, Phi via GitHub auth |
| **Hugging Face** | Yes | Free tier | Rate limited | Research, specialist models |
| **Cloudflare Workers** | Yes | 10K neurons/day | Workers limits | Edge deployment |
| **Z.ai** | Yes | Coding plan quota | Unknown | Code generation CLI + API |
| **Kimi** | Yes | Coding plan (~3x per 5hr) | Per 5hr window | Code generation CLI + API |
| **NVIDIA NIM** | Yes | Unknown — assume generous | Unknown | NIM models (Llama, Mistral variants) |
| **Bonsai** | Yes | Unknown, free | Unknown | Hidden frontier model (GPT-5 or Anthropic) |
| **Giga AI** | Yes | Unknown, free | Unknown | Overflow buffer |

### 1.2 Cerebras is a Standout — Update the Spec

Cerebras was listed as "unknown limits" in the master spec. **Actual limits:**
- 4 production models x 14,400 req/day = **57,600 req/day combined**
- **1M tokens/hour per model** — effectively unlimited token throughput
- `gpt-oss-120b` (65K context), `llama-3.3-70b`, `llama3.1-8b`, `qwen-3-32b`
- This is the best raw throughput provider in the stack. Treat as primary API overflow.

### 1.3 VoidAI Credit Math

125K credits/day. At typical coding model pricing (0.02x input, 0.1x output):
- ~2,400 requests/day at 500 output tokens
- Scales up for smaller outputs, down for large generations
- Daily model list and discounts rotate — SimpleLLMRouter must call `/api/v1/models` at startup

### 1.4 SimpleLLMRouter — What's Already There vs. What's Missing

Repo at `github.com/AReid987/simplellmrouter` already has: Mistral, Groq, Gemini, Cerebras, OpenRouter, VoidAI, Z.ai, Kimi.

**Missing from current `providers.yaml`:** NVIDIA NIM, Bonsai, Giga AI, GitHub Models, Hugging Face, Cloudflare Workers.

These need to be added as provider configs. NVIDIA and Bonsai are highest priority given unknown-but-likely-large quotas.

---

## 2. Context Window Corrections

The original spec over-indexed on Kimi's 128K context window. **Corrected picture:**

| CLI / API | Context Window | Notes |
|-----------|---------------|-------|
| Gemini CLI / Gemini API | **1M – 2M tokens** | Largest in the stack by a massive margin |
| iFlow (Roma model) | 128K | Good for full-repo analysis within iFlow |
| iFlow (Kimi K2) | 128K | Same |
| Kimi Code CLI | 128K (not 256K) | Spec had this wrong |
| Mistral Large | 128K | High-volume API use |
| Cerebras models | 65K | Fast but not long-context |
| Groq | 128K | Fast, not a context play |
| Qwen3 (CLI/API) | **256K** | Larger than Kimi, overlooked in spec |
| Mistral models (many) | **256K** | Multiple Mistral models at 256K |

### Corrected Routing Rule for Context

```
IF context_needed > 200K        → Gemini CLI or Gemini API (1M-2M)
IF context_needed > 64K         → Qwen3 (256K) or Mistral (256K) or Kimi (128K)
IF context_needed <= 64K        → Any provider, optimize for quota/speed
```

**Kimi's actual strengths** (not just context):
- Strong multi-step reasoning and task completion
- Good error recovery in complex workflows
- Often completes multi-step workflows with minimal scaffolding/prompting
- Fast relative to quality
- Skill invocation may differ — does not use `/skill:command` syntax like Claude

---

## 3. iFlow CLI — First-Class Squad Member

### 3.1 What It Is

Free, Alibaba-affiliated, MCP-extensible. Built-in multi-model routing (GLM-5, Roma, Kimi K2, Qwen3, DeepSeek v3). Completely absent from the original spec. **It is not a fallback — it owns specific workflow commands that nothing else replicates as cleanly.**

### 3.2 Install

```bash
npm i -g @iflow-ai/iflow-cli
```

### 3.3 Built-In Commands

| Command | Purpose | Replaces Manual Step |
|---------|---------|---------------------|
| `/init` | Full repo scan + summary context file | Manual AGENT.md bootstrap |
| `/scaffold` | Project scaffolding | Manual turborepo init steps |
| `/qa` | QA workflow — tests, lint, bug scan | Separate forge-quality sweep |
| `/refactor` | Codebase refactor sweep | Manual Kimi/Gemini refactor prompts |
| `/understand` | Full codebase analysis + dependency map | Manual `/init deep` in Claude Code |
| `/mermaid` | Architecture diagram generation | Manual diagram prompting |
| `multi-agent review` | Parallel independent review agents | Sequential review passes |
| `todos-to-issues` | Scan TODOs → create GitHub issues | Manual issue creation |
| `/remember` | Force Letta learning pass | Manual memory consolidation |

### 3.4 Execution Modes

- **Default** — stream + approve at checkpoints
- **Plan** — show full plan before executing (use this for `/scaffold`, `/refactor`)
- **Accepting Edits** — open inline editor during execution
- **YOLO** — auto-execute all steps (use only for `/qa`, `/mermaid`)

### 3.5 Internal Model Routing (iFlow selects automatically)

| Task Type | Model Selected |
|-----------|---------------|
| `/refactor`, `/mermaid` | GLM-5 (fast, general) |
| `/understand`, `/qa` large codebase | Roma (128K, analysis) |
| `/understand --deep`, full-repo `/qa` | Kimi K2 |
| `/scaffold`, config generation | Qwen3 Coder |
| Complex logic in `/refactor` | DeepSeek v3 |

### 3.6 justfile Recipes Enabled by iFlow

```bash
just diagram          # iflow /mermaid --type=architecture
just understand       # iflow /understand --mode plan
just todos-to-issues  # iflow todos-to-issues
just qa               # iflow /qa
just scaffold PKG     # iflow /scaffold packages/PKG
just multi-review     # iflow multi-agent review
```

---

## 4. claude-flow AgentDB & Intelligence Controller

These were **completely absent** from the spec. The `--full` install is not optional — it is the entire intelligence layer.

### 4.1 Install Command (Required — not the basic install)

```bash
# FULL install — required for AgentDB + Intelligence Controller
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full

# Register MCP with Claude Code
claude mcp add ruflo -- npx -y ruflo@latest mcp start

# Initialize AgentDB for project
npx agentdb@latest init $PROJECT_ROOT/agents.db
```

### 4.2 What AgentDB Actually Does

AgentDB v1.3.9 is not just storage. It is a full intelligence layer:

| Component | Description |
|-----------|-------------|
| **29 MCP tools** | Auto-registered with Claude Code on `mcp add` |
| **SONA** | Self-Optimizing Neural Architecture — <0.05ms routing adaptation |
| **EWC++** | Elastic Weight Consolidation — prevents catastrophic forgetting between sessions |
| **HNSW vector search** | <100µs pattern retrieval, O(log n) — semantic memory lookup |
| **ReasoningBank** | 5-phase trajectory learning: RETRIEVE → JUDGE → DISTILL → CONSOLIDATE → ROUTE |
| **9 RL algorithms** | Q-Learning, SARSA, A2C, PPO, DQN, Decision Transformer, MCTS, TD(λ), R-Max |
| **Flash Attention** | 2.49–7.47x speedup on long contexts |
| **Nightly learner daemon** | Runs background memory consolidation every night |
| **Causal reasoning graphs** | Maps cause-effect relationships across sessions |
| **Reflexion memory** | Self-critique loops on past decisions |
| **Skill library** | Semantic search across 42 bundled skills |
| **Merkle proof recall** | Cryptographically verified memory integrity |

### 4.3 The 60+ Bundled Agents

The `--full` install includes 60+ pre-built specialist agents: coder, reviewer, tester, security auditor, architect, documenter, optimizer, debugger, DevOps engineer, and more. These are available as Claude Code slash commands and MCP tools after install.

### 4.4 Required Additions to `just setup`

```bash
# In justfile setup recipe — ADD these:
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full
claude mcp add ruflo -- npx -y ruflo@latest mcp start
npx agentdb@latest init ./agents.db
```

---

## 5. Skills System — Reality Check

### 5.1 Anthropic Skills 2.0 — What Is Real

Announced March 3, 2026. **This is real**, not marketing hype, but the terminology is imprecise:

- **Real:** Skill Creator 2.0 with structured evals (automated tests for skills)
- **Real:** Benchmark mode — tracks pass rates, token usage, elapsed time across runs
- **Real:** A/B comparison — skill vs. no-skill via blind judge agents
- **Real:** Multi-agent eval execution in parallel (no cross-contamination)
- **Real:** Auto-generated descriptions based on system prompt structure
- **Not real (yet):** "Skill Marketplace" — not launched
- **Not real (yet):** Organization-level shared skill repositories
- **Misleading:** "1,000s of skills" — means starter templates, not production-tested code

### 5.2 The Bloat Problem

Skills 2.0 makes it **too easy** to create skills. Every small fix becomes "maybe this should be a skill." This is a recipe for a 300-skill repository where 280 of them are used once.

**Design Decision:** Skills must meet one of these criteria to justify existence:
- Used across 3+ distinct projects
- Encapsulates proprietary logic or domain knowledge
- Requires structured state management (DB, config, multi-step)
- Has non-trivial evals (can't be tested with a simple assert)

**Anti-pattern:** "search-codebase" skill — this is one find/grep command. Do not make a skill.

### 5.3 Skill Organization Strategy

```
skills/
├── core/              # < 10 skills — universal primitives
│   ├── repo-init
│   ├── test-runner
│   ├── deploy-preview
│   └── security-audit
├── domain/            # Project-specific logic
│   ├── aigency-deploy
│   └── nexus-signal-parse
└── experimental/      # Unproven — delete after 30 days if unused
    └── auto-docstring
```

### 5.4 Real Skills in This Project (Justified)

1. **repo-init** — Full turborepo + PDM + justfile scaffold (multi-step, reusable)
2. **test-runner** — Vitest + Playwright orchestration with coverage checks
3. **deploy-preview** — Vercel preview deploy + status check loop
4. **security-audit** — Trufflehog + npm audit + SAST scan aggregation
5. **refactor-sweep** — Multi-file refactor with rollback on test failure
6. **memory-consolidate** — AgentDB learning pass + Letta reflection

Anything beyond these six must justify its existence with usage data.

---

## 6. Spec-Docs Repo & Constitution

### 6.1 Decision: Separate Repo

All specs, ADRs, RFCs, and constitutions live in `aigency-specs` (separate from code repos). This repo is the **single source of truth** for project design.

### 6.2 Repository Structure

```
aigency-specs/
├── architecture/
│   ├── prd.md
│   ├── architecture.md
│   ├── backlog.md
│   └── execution-plan.md
├── constitutions/
│   ├── meta-code-squad-master-spec.md
│   ├── meta-code-squad-addendum-v2.md
│   ├── meta-code-squad-tool-inventory.md
│   └── meta-code-squad-launch-guide.md
├── org/
│   ├── team-charter.md
│   ├── coding-standards.md
│   └── incident-response.md
└── templates/
    ├── adr-template.md
    ├── rfc-template.md
    └── postmortem-template.md
```

### 6.3 Constitution Contents

The "constitution" is four docs:
1. **Master Spec** — Full system design, tool selection, workflows
2. **Addendum v2** (this doc) — Corrections and missing systems
3. **Tool Inventory** — Complete CLI/API/MCP catalog
4. **Launch Guide** — Step-by-step execution playbook

These four docs together form the **immutable contract** for Phase 3.

### 6.4 Update Protocol

Constitution updates require:
- New addendum document (never edit master spec directly)
- Increment addendum version (v2 → v3)
- Git tag + release notes
- Notification to all agents (update AGENT.md references)

---

## 7. Cross-Harness Orchestration

### 7.1 The Orchestration Problem

We have **four distinct agent execution harnesses**, each with different:
- Command syntax (`/skill:name` vs. `/refactor` vs. `just recipe` vs. `npx agentdb`)
- Context protocols (MCP vs. BMAD vs. Letta memory)
- Skill invocation (Claude-native vs. ruflo agents vs. iFlow commands)
- State management (AgentDB vs. Letta episodic memory vs. justfile ENV)

**Critical:** These are not competitors. They are **composable layers** with different strengths.

### 7.2 The Four Harnesses

| Harness | Strength | Command Syntax | Context Protocol | State Storage |
|---------|----------|---------------|-----------------|---------------|
| **Claude Code** | Interactive dev, skills | `/skill:name` | MCP tools + Knowledge | Conversation state |
| **claude-flow** | Agents, orchestration | `ruflo run AGENT` | BMAD → ruflo translator | AgentDB SQLite |
| **iFlow CLI** | Repo automation | `/command --opts` | iFlow memory files | `.iflow/` folder |
| **justfile** | Dev workflows | `just recipe` | ENV vars + args | Shell context only |

### 7.3 Unified Orchestration — The justfile Layer

The **justfile is the unifying harness**. All other tools are invoked through `just` recipes:

```bash
just skill NAME       # → claude skill:NAME
just agent AGENT      # → ruflo run AGENT
just iflow CMD        # → iflow /CMD
just test             # → pnpm test (direct)
```

This provides:
- Single command interface for all agent systems
- ENV injection at execution (API keys, PROJECT_ROOT)
- Consistent error handling and logging
- Dependency checks before invocation

### 7.4 Context Unification — BMAD as Translation Layer

BMAD (Builder/Manager/Advisor/Director) is the **protocol translator** between Claude's MCP tools and ruflo's agent system.

**How it works:**
1. Claude Code invokes skill via `/skill:deploy-preview`
2. Skill internally calls `ruflo run deploy-agent`
3. BMAD translates Claude's MCP context into ruflo's expected format
4. ruflo agent executes with full AgentDB memory access
5. Result is returned to Claude via BMAD response formatter

This is **already built** in claude-flow 1.3.9. No custom integration needed.

### 7.5 Memory Consolidation — AgentDB ← Letta ← Claude

Memory flows one direction: **Claude → Letta → AgentDB**

```
Claude conversation → Letta episodic memory (via MCP Letta tool)
                   → AgentDB consolidation (nightly learner)
                   → Semantic vector search available to all agents
```

**Implementation:**
- Claude Code has Letta MCP tool registered
- Letta auto-logs important decisions/context as episodic memory
- AgentDB nightly learner runs EWC++ consolidation on Letta's episodic memory
- Future sessions query AgentDB via semantic search

No custom glue code required — this is the intended flow per ruflo docs.

### 7.6 PDM + UV — Python Dependency Unification

Both PDM and UV are installed. **PDM is primary**, UV is fallback for speed-critical installs.

**Decision tree:**
```
IF adding dependency          → pdm add PACKAGE
IF lockfile regen needed      → pdm lock
IF CI/production install      → pdm sync --prod
IF dev environment from zero  → uv venv && uv pip install -r requirements.txt (faster cold start)
IF tool install (ruff, mypy)  → uvx TOOL (UV's pipx equivalent)
```

PDM for project state, UV for throwaway speed.

### 7.7 Execution Flow — Real Example

**Task:** Deploy a Vercel preview and run QA checks

```bash
just deploy-preview BRANCH
```

**What happens:**
1. justfile checks ENV for VERCEL_TOKEN
2. Invokes `/skill:deploy-preview` in Claude Code
3. Skill calls `ruflo run deploy-agent --branch BRANCH`
4. BMAD translates request → ruflo agent context
5. Deploy agent (from AgentDB) executes Vercel CLI deploy
6. Agent writes deploy URL to AgentDB memory
7. justfile invokes `iflow /qa --target DEPLOY_URL`
8. iFlow runs Playwright + Lighthouse + axe checks
9. Results streamed back to justfile → formatted output
10. Exit code determines CI pass/fail
```

This is **three harnesses in one command** — orchestrated via justfile, unified context via BMAD/AgentDB.

---

## 8. Implementation Checklist

These are **required changes** to align with this addendum:

### Phase 3A — Pre-Launch Fixes
- [ ] Update SimpleLLMRouter `providers.yaml` — add NVIDIA NIM, Bonsai, Giga AI
- [ ] Add iFlow CLI install to `just setup`
- [ ] Add claude-flow `--full` install to `just setup`
- [ ] Create `aigency-specs` repo with constitution structure
- [ ] Add AgentDB init to `just setup`
- [ ] Update justfile with iFlow recipe wrappers
- [ ] Document skill bloat prevention rules in launch guide
- [ ] Add Cerebras as primary API overflow in routing logic

### Phase 3B — Post-Launch Validation
- [ ] Run full context window test suite (64K, 128K, 256K, 1M inputs)
- [ ] Verify BMAD → PRP translation with sample ruflo agent
- [ ] Test Letta → AgentDB nightly consolidation
- [ ] Validate iFlow `/scaffold` with real turborepo init
- [ ] Run skill A/B evals on all six justified skills
- [ ] Audit for skills that don't meet criteria — delete or justify

---

## 9. Final Authority

This document **supersedes** conflicting statements in:
- Meta Code Squad Master System Spec
- Meta Code Squad Tool Inventory
- Any prior verbal or written guidance

**When in conflict:** Addendum v2 is authoritative.

**Effective immediately.**

---

**End of Addendum v2**