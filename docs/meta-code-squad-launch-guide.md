# Meta Code Squad — Launch Guide
## Optimized Workflow: GSD + Loki + Claude Agent Teams + Sugar
**Date:** 2026-03-05 | **Owner:** Antonio Reid
**Purpose:** Exact steps to kick off the Meta Code Squad build today

---

## Honest Assessment of Your Current Workflow

Your existing flow is already strong. Here is what it actually does well and where the gaps are:

### What Works (Keep It)

**GSD initialization** is legitimately excellent. The Discuss → Plan → Execute → Verify cycle
with fresh subagent context per task solves context rot — the #1 reason agentic builds degrade
mid-project. The wave-based planning output is exactly what Loki and Claude Agent Teams need
to consume.

**Hive Mind / Claude Agent Teams** (enabled via
`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) is now a first-class feature. Anthropic's own
internal team built a C compiler — 100K lines of Rust, 2,000 sessions — using 16 agent teams.
For the Meta Code Squad monorepo, parallel agents per package (router + forge-quality) is
the right call.

**Gemini CLI + Conductor** is your best throughput tool. Free tier is 1,000 req/day, rate
at 60 req/min. Run it for the high-volume, lower-stakes work: test generation, config files,
boilerplate, documentation passes.

**Kimi Code CLI** for full-codebase review passes. 128K context means it reads the entire
monorepo at once without chunking. Use it at phase boundaries, not during active development.

### What to Add (Gap Fillers)

**Loki Mode** fills the gap between your GSD plan output and actual autonomous execution.
It is the execution engine your workflow is missing. GSD gives you the PLAN files. Loki
consumes those PLAN files and drives the build through its RARV cycle
(Reason → Pre-Act Attention → Act → Reflect → Verify) with automated quality gates.
Without Loki or something like it, you are manually kicking off each wave.

**Sugar AI** is a task queue and memory layer, not an orchestrator. Think of it as the
persistent brain between sessions. It solves context amnesia — when you close a session and
come back, Sugar knows what was decided, what failed, and what is next. It integrates
natively with Claude Code via MCP. It is worth adding but is NOT a replacement for GSD or
Loki — it layers on top.

**Loki vs Sugar — which to use when:**

| Need | Use |
|------|-----|
| Convert GSD plan into autonomous build waves | Loki Mode |
| Queue tasks for overnight / next-session runs | Sugar |
| Persistent memory of decisions across sessions | Sugar |
| Parallel multi-agent execution with quality gates | Loki + Claude Agent Teams |
| Simple task delegation mid-conversation | Sugar `/sugar-add` |

---

## Recommended Workflow (Optimized)

```
CONTEXT LOAD
    │
    ▼
GSD INITIALIZATION                      ← You already do this well
    │  /gsd:new-project                 ← Generates PROJECT.md, REQUIREMENTS.md,
    │  /gsd:discuss-phase N             ←   ROADMAP.md, STATE.md, CONTEXT.md
    │  /gsd:plan-phase N                ← Generates PLAN files (wave-based)
    ▼
LOKI EXECUTION                          ← NEW: replaces manual wave kickoff
    │  loki start ./PLAN-1.md           ← Loki consumes GSD output directly
    │  RARV cycle runs autonomously     ← Reason, Act, Reflect, Verify per task
    │  Quality gates enforce standards  ← Tests, security, spec compliance built in
    ▼
CLAUDE AGENT TEAMS (parallel)           ← For tasks Loki splits across packages
    │  router package agents            ← TypeScript, routing core, TUI
    │  forge-quality package agents     ← Python hooks, scanner, CLI
    ▼
SUGAR MEMORY LAYER                      ← NEW: runs alongside everything
    │  sugar recall "auth decisions"    ← Surfaces relevant past decisions
    │  /sugar-add "Fix X" mid-session   ← Queues for next autonomous run
    │  MCP server keeps context live    ← Persistent across session resets
    ▼
KIMI / GEMINI REVIEW PASSES             ← Phase boundary reviews, doc gen
    │  kimi: full-codebase sweep        ← 128K context, sees everything at once
    │  gemini: test gen, boilerplate    ← High volume, low stakes work
    ▼
NEXT PHASE → GSD again
```

---

## Exact Steps to Start RIGHT NOW

### Step 0 — One-Time Tool Setup (15 min, do once)

**Enable Claude Agent Teams:**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
# Add to ~/.zshrc or ~/.bashrc to persist
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
```

**Install Loki Mode:**
```bash
npm install -g loki-mode
loki doctor                  # verify environment — should show Claude Code detected
```

**Install Sugar:**
```bash
pipx install sugarai
sugar init                   # creates .sugar/ in your project
```

**Verify Gemini CLI has Loki extension:**
```bash
gemini /extensions           # confirm loki shows up
```

---

### Step 1 — Create Project Directory (5 min)

```bash
mkdir aigency && cd aigency
git init
mkdir -p .context/docs
```

**Copy all 5 docs into .context/docs/:**
```
.context/docs/
├── project-brief-aigency-dev-platform.md
├── prd-aigency-dev-platform.md
├── architecture-aigency-dev-platform.md
├── ux-specs-aigency-dev-platform.md
└── backlog-aigency-dev-platform.md
```

Why `.context/docs/` not root? Loki, GSD, and Sugar all scan for context files.
Keeping them in a dedicated folder prevents them from being treated as source code.

---

### Step 2 — Initialize Claude Code Hive Mind (10 min)

Create `CLAUDE.md` at project root — this is what Hive Mind reads on every agent spawn:

```markdown
# CLAUDE.md — aigency/ Monorepo

## Project
Building SimpleLLMRouter v2 (TypeScript) + @aigency/forge-quality (Python/TypeScript).
Target: production-ready, open-source, NPM-publishable packages.

## Architecture
- Turborepo monorepo
- packages/router — LLM routing gateway + Textual TUI
- packages/forge-quality — git hook quality enforcement
- shared/types — TypeScript interfaces shared across packages

## Context Docs
All spec documents are in .context/docs/. Read them before starting any task.
Priority order: architecture > PRD > UX spec > backlog > project brief

## Code Standards
- TypeScript strict mode, no `any`
- Python 3.11+, type hints required
- All commits must pass forge-quality hooks once installed
- Tests required for every new module (vitest for TS, pytest for Python)

## Agent Team Rules
- router agents: work only in packages/router/
- forge-quality agents: work only in packages/forge-quality/
- shared agents: work only in shared/
- Never cross package boundaries without orchestrator approval

## Current Phase
[UPDATE THIS AS YOU PROGRESS]
Phase 0: Turborepo scaffold

## Blocked Tasks
[LIST ANY BLOCKED ITEMS HERE]
None currently.
```

Start Claude Code with hive mind:
```bash
claude --dangerously-skip-permissions
# In session: "Initialize hive mind for aigency/ project. Read CLAUDE.md and all docs
# in .context/docs/. Confirm you have full context before we begin Phase 0."
```

---

### Step 3 — GSD Initialization in Gemini CLI (20 min)

Open a second terminal. Run Gemini CLI:

```bash
gemini
```

Then run GSD initialization sequence:

```
/gsd:new-project
```

When GSD asks questions, your answers:
- **Project name:** aigency
- **What are you building?** Two NPM packages in a Turborepo monorepo. Package 1:
  SimpleLLMRouter v2 — an LLM routing gateway with intelligent cost/latency optimization
  and a Textual TUI. Package 2: @aigency/forge-quality — a git hook enforcement system
  with AI-powered code quality and security scanning. Detailed specs are in .context/docs/.
- **Tech stack:** TypeScript (router), Python + TypeScript (forge-quality), Turborepo,
  Vitest, Pytest, SQLite, Redis, Docker
- **Constraints:** Must be open-source publishable. Router must achieve <50ms routing
  latency. forge-quality hooks must complete in <30s total.
- **v1 scope:** Core routing logic, basic TUI, git hooks, basic scanner, CLI init command
- **Out of scope for v1:** Advanced ML routing, SaaS dashboard, paid tier features

After GSD generates PROJECT.md, REQUIREMENTS.md, ROADMAP.md:

```
/gsd:discuss-phase 1
```

Answer the discuss questions for Phase 1 (Turborepo scaffold + router core). Then:

```
/gsd:plan-phase 1
```

This generates your PLAN files. These are what Loki will consume. Review them before
proceeding — this is your only human checkpoint before autonomous execution starts.

---

### Step 4 — Sugar Memory Layer Setup (5 min)

In the project directory:

```bash
sugar init
sugar add "Read all docs in .context/docs/ before starting any task" --type preference
sugar add "Turborepo scaffold - packages/router and packages/forge-quality" --type feature --urgent
sugar add "CLAUDE.md must be updated at each phase boundary" --type preference
```

Set up the MCP server for Claude Code integration:
```bash
sugar mcp setup
# This adds Sugar's MCP server to Claude Code's tool registry
# Claude Code can now use /sugar-add and /sugar-recall mid-session
```

---

### Step 5 — Loki Execution (ongoing, autonomous)

Once GSD plan files are reviewed and approved:

```bash
loki start ./1-1-PLAN.md    # GSD plan file naming convention
```

Loki will:
1. Read the plan, decompose into atomic tasks
2. Spawn Claude Code (or Gemini in degraded mode) per task
3. Run RARV cycle for each task
4. Execute 9 quality gates before marking complete
5. Auto-commit passing work

**Monitor via Loki dashboard:** `http://localhost:57374`

**If Loki hits a quality gate failure 5 times:** It marks the task as blocked and moves on.
You review the `.loki/CONTINUITY.md` to see what failed and why, then either fix context
or decompose the task further.

---

### Step 6 — Quota Management (ongoing)

Your quota rotation order when primary tools are exhausted:

```
Primary (use first):
  1. Claude Code + Agent Teams   — z.ai 120 req/5hr — best for complex code
  2. Gemini CLI                  — 1,000 req/day    — best for volume/TUI/tests
  3. Kimi Code CLI               — 3x quota         — best for large context review

Secondary (when primaries are depleted):
  4. Antigravity                 — unknown quota     — solid fallback
  5. Qwen Code                   — good quality      — underutilized, ramp this up
  6. Roo Code / Kilo Code        — quota buffer      — good for config/boilerplate
  7. iFlow CLI                   — free              — use for any task that fits
  8. AMP CLI / Rovo Dev          — use as buffer     — configure these next session

Rule: Never burn Tier 1 quota on boilerplate. Route that to Tier 2/3.
Rule: Save Kimi for phase-boundary full-codebase reviews only.
Rule: Gemini CLI is your highest throughput. Use it for test gen, docs, config.
```

---

## What NOT to Do

**Do not skip the GSD discuss phase.** The wave plans it generates without discuss are
generic. The discuss phase is 20 minutes that saves 4 hours of wrong-direction code.

**Do not run Claude Agent Teams without CLAUDE.md package boundaries defined.**
Without the boundary rules, agents will step on each other's work across packages.

**Do not use Sugar as your primary orchestrator.** It is a memory and queue layer.
Loki or Claude Agent Teams do the actual execution. Sugar remembers what they decided.

**Do not let Loki run without reviewing the GSD plan first.** Loki executes autonomously.
If the plan is wrong, it executes the wrong thing autonomously and commits it. Review
the PLAN files. That is your one checkpoint.

**Do not start with all tools at once.** This session: GSD + Claude Agent Teams + Sugar.
Add Loki on Phase 1 once you have confirmed GSD output looks right.

---

## Today's Session Target

By end of today's session you should have:

- [ ] aigency/ directory initialized with git
- [ ] All 5 docs in .context/docs/
- [ ] CLAUDE.md written and Claude Code confirmed it has full context
- [ ] GSD ran: PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md generated
- [ ] GSD Phase 1 PLAN files generated and reviewed by you
- [ ] Sugar initialized with base preferences loaded
- [ ] Turborepo scaffold complete (packages/router, packages/forge-quality, shared/types)
- [ ] package.json files in place for all packages
- [ ] forge-quality dogfooded on the monorepo itself (hooks installed on aigency/)

That is Story 1.1 + 1.2 from the backlog. One day. Realistic.

---

## Integration Map (How the Tools Relate)

```
                    ┌─────────────────────┐
                    │   YOU (Architect)   │
                    │  Review GSD plans   │
                    │  Set phase targets  │
                    └──────────┬──────────┘
                               │ approve plan
                    ┌──────────▼──────────┐
                    │    GSD SYSTEM       │
                    │  Discuss → Plan     │
                    │  Generates PLAN.md  │
                    └──────────┬──────────┘
                               │ PLAN files
              ┌────────────────▼────────────────┐
              │          LOKI MODE               │
              │  RARV cycle per task             │
              │  9 quality gates                 │
              │  Auto-commit passing work        │
              └──────┬──────────────┬────────────┘
                     │              │ parallel
          ┌──────────▼──┐    ┌──────▼──────────┐
          │ Claude Agent │    │   Gemini CLI    │
          │ Teams        │    │   Conductor     │
          │ (complex)    │    │   (volume)      │
          └──────────────┘    └─────────────────┘
                     │              │
              ┌──────▼──────────────▼──────┐
              │       SUGAR AI              │
              │  Memory across sessions     │
              │  Task queue for next run    │
              │  Recall past decisions      │
              └────────────────────────────┘
                               │ phase complete
                    ┌──────────▼──────────┐
                    │    KIMI CODE CLI     │
                    │  Full-codebase scan  │
                    │  128K context sweep  │
                    └─────────────────────┘
```

---

## One-Line Summary Per Tool

| Tool | One Line |
|------|----------|
| GSD | Plans what to build, wave by wave |
| Loki Mode | Executes the plan autonomously with quality gates |
| Claude Agent Teams | Parallel agents per package, peer-to-peer capable |
| Sugar | Remembers everything across sessions, queues overnight work |
| Gemini CLI + Conductor | High-throughput execution for volume tasks |
| Kimi Code CLI | Phase-boundary full-codebase review (128K) |
| Antigravity / Qwen | Quota buffer — configured and ready, use when Tier 1 depleted |

---

*Start with Step 1. You have everything you need.*
