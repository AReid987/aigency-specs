# Agentic Tooling Integration Strategy
## Loki + BMAD + Maestra + Superset + Letta + Overstory + ROMA
**Date:** 2026-03-05 | **Owner:** Antonio Reid
**Purpose:** Honest evaluation of 6 tools — what to integrate, when, and how

---

## Executive Summary

You asked about 6 tools. Here is the bottom line before the detail:

| Tool | Verdict | Priority | Effort |
|------|---------|----------|--------|
| **Loki + BMAD** | INTEGRATE NOW | P0 | Low — you already have both |
| **Superset** | INTEGRATE NOW | P0 | Low — install + point at worktrees |
| **Maestra** | INTEGRATE — Phase 2 | P1 | Medium — replaces ad-hoc agent wiring |
| **Overstory** | INTEGRATE — Phase 2 | P1 | Medium — after monorepo is scaffolded |
| **Letta Code** | MONITOR — Phase 3 | P2 | Low now, high value later |
| **ROMA** | SKIP for now | P3 | High effort, overlaps what you have |

---

## 1. Loki + BMAD — INTEGRATE NOW (P0)

### What the research confirmed

The BMAD integration validation doc at `asklokesh/loki-mode` shows Loki was
explicitly designed around BMAD's story/task/wave structure. The integration is
not bolted-on — it is the primary design target. Loki's RARV cycle maps directly
onto BMAD's Execute phase:

```
BMAD Phase          Loki Equivalent
-----------         ---------------
Discuss             (human + GSD — stays as-is)
Plan                GSD plan-phase → PLAN-N.md files
Execute             loki start ./PLAN-N.md (RARV cycle)
Verify              Loki quality gates 1-9 + auto-commit
```

The kanban dashboard (web: `localhost:57374`, terminal: `loki dashboard`) gives
you real-time visibility into wave execution — exactly the task board you want
without building one.

### What this changes in your current workflow

Your current flow:
```
GSD discuss → GSD plan → [MANUAL: kick off claude/gemini/kimi] → [MANUAL: review] → next wave
```

With Loki integrated:
```
GSD discuss → GSD plan → loki start PLAN-N.md → [AUTO: RARV + 9 gates] → loki dashboard → next wave
```

The manual kickoff and mid-wave babysitting goes away. Your one human checkpoint
is reviewing the GSD plan before execution starts. That is the right place for
human judgment.

### Your AI Constitution placement

Your AI Constitution belongs in `.loki/CONSTITUTION.md` at repo root. Loki
reads this file before every task execution and injects it as a constraint layer
on top of the agent's system prompt. This is the cleanest integration point —
it fires on every task without you having to remember to include it.

Secondary placement: also add it as `CLAUDE.md` section header "Constitutional
Constraints" for Claude Code sessions that run outside Loki.

### Integration steps (do these today, ~30 min)

```bash
# 1. Install Loki globally
npm install -g @asklokesh/loki-mode

# 2. Init in your repo root
cd ~/projects/aigency
loki init

# 3. Place your AI Constitution
cp ~/your-constitution.md .loki/CONSTITUTION.md

# 4. Verify BMAD detection
loki doctor
# Should show: BMAD structure detected ✓

# 5. Start dashboard before first run
loki dashboard
# Opens at localhost:57374
```

### Why the kanban dashboard matters specifically for you

You are running 8+ agent tools. Right now you have no single view of what is
running, what finished, and what failed. Loki's dashboard gives you that for
every wave you execute — task states, quality gate pass/fail, which files were
touched, commit hashes. This is the operational visibility you are missing.

---

## 2. Superset — INTEGRATE NOW (P0)

### What it actually is (clarifying the confusion)

Superset (superset.sh) is NOT a data visualization tool (that is Apache Superset,
different product). This is the March 2026 agent orchestration platform from
three ex-YC CTOs. Core mechanic: runs 10+ CLI coding agents in parallel, each
in its own git worktree, with a unified monitoring dashboard.

### Why this is a direct fit for your setup

You have the exact agent inventory Superset was designed for:
- Claude Code (primary)
- Gemini CLI
- Kimi Code CLI
- Qwen Code, Roo Code, Kilo Code, Rovo Dev, AMP CLI, iFlow CLI

Right now when you hit the z.ai 120-request quota limit, you manually switch
tools and manage parallel work in your head. Superset eliminates that entirely:

```
Without Superset:              With Superset:
- Agent A hits quota           - Agent A hits quota
- You notice, manually switch  - Superset detects idle state
- Open new terminal            - Routes next task to Agent B automatically
- Set up context again         - Context already loaded in worktree
- Resume work                  - Zero interruption
```

### The worktree isolation model solves your monorepo problem

The Meta Code Squad monorepo has two packages being built simultaneously:
`packages/router` and `packages/forge-quality`. Without isolation, parallel
agents step on each other. Superset's git worktree model gives each agent
its own branch + directory. Merging is handled with tiered conflict resolution.

This is the infrastructure that makes Claude Agent Teams actually safe at scale.

### Quota strategy with Superset

Configure task queues by agent:

```
Queue: CORE (Claude Code + Antigravity)
  → Routing logic, security code, state machines
  → Max concurrent: 3

Queue: THROUGHPUT (Gemini CLI + Kimi)
  → Tests, config, documentation, boilerplate
  → Max concurrent: 5

Queue: BUFFER (Qwen + Roo + Kilo + iFlow)
  → Overflow when CORE hits quota limits
  → Max concurrent: unlimited (free tier)
```

When CORE queue drains (quota hit), Superset auto-promotes BUFFER tasks. You
stop babysitting quota counters entirely.

### Integration steps (~45 min)

```bash
# 1. Download Superset (macOS)
# From superset.sh — desktop app, Electron-based

# 2. Add your monorepo as a project
# File → Add Project → ~/projects/aigency

# 3. Configure agent runtimes
# Settings → Agents → Add: claude-code, gemini-cli, kimi, qwen-code, etc.

# 4. Set up task queues per priority tier
# Projects → aigency → Queues → New Queue

# 5. Launch first parallel run
# Queue task for router package + queue task for forge-quality
# Both run simultaneously in isolated worktrees
```

### One honest limitation

Superset is macOS-only right now. If you ever need to run this on Linux/cloud,
you will need Overstory instead (which runs anywhere via tmux). Keep that in
mind for Phase 2 when you think about CI/CD agent runs.

---

## 3. Maestra — INTEGRATE Phase 2 (P1)

### What grabbed your attention was probably this

The thing you saw in the YouTube video that you cannot remember — it was almost
certainly the **Agent Studio with live trace visualization**. Maestra shows you
every model call, tool execution, and workflow step as it happens in a tree view.
For someone building an agent network like yours, seeing the execution graph live
is genuinely compelling.

### The real value for your stack

Maestra's highest-value features for your specific situation, ranked:

**1. Workflows with deterministic orchestration**
Maestra workflows use `.then()`, `.parallel()`, `.branch()`, `.dowhile()` —
typed, validated, reproducible. This is what your LLM Router's execution engine
should be built on top of, not custom-rolled. It handles the suspend/resume,
parallel branching, and loop logic that SimpleLLMRouter v2 needs natively.

**2. REST API server mode**
`mastra build` produces a standalone Hono server with full OpenAPI docs. Your
LLM Router can be deployed as a Maestra-powered REST API that any agent or
service in your stack can call. This is the integration bridge between your
CLI tools and your Next.js apps.

**3. Deployed in Next.js**
The Aigency platform IS a Next.js app. Maestra's embedded Next.js integration
means your agent workflows run inside the same app — no separate backend to
maintain. Agent Studio becomes your admin panel at `/admin/agents`.

**4. Evals + Scorers**
You have forge-quality as a code quality enforcer. Maestra's evals framework
gives you the same capability for agent output quality — score responses,
track regression, identify which models degrade on which task types.

### Where it fits in your architecture

```
Current:                          With Maestra:
CLI tools → ad-hoc execution      CLI tools → Maestra workflows → typed outputs
No agent API                      Maestra REST API at /api/agents
No eval framework                 Maestra evals scoring every agent response
Separate observability needed     Built-in Langfuse integration
```

### When to integrate (not now — here is why)

Maestra is a framework, not a tool. Integrating it means refactoring how your
agents are defined and invoked. Do this AFTER the Meta Code Squad monorepo is
scaffolded and the basic router is working. The right moment is Sprint 3 when
you are wiring up the agent routing table — that is when Maestra workflows
replace your custom routing logic and the ROI is immediate.

### Integration target: Sprint 3, Week 5

```
packages/
  router/           # SimpleLLMRouter v2 — Maestra workflow engine
  forge-quality/    # Existing
  agents/           # NEW: Maestra agent definitions
    studio/         # Agent Studio admin UI
    workflows/      # Deterministic orchestration graphs
    evals/          # Scorer definitions
```

---

## 4. Overstory — INTEGRATE Phase 2 (P1)

### What makes it different from Superset

Both use git worktrees. The key difference:

| | Superset | Overstory |
|--|---------|----------|
| Interface | Desktop GUI (Electron) | CLI + tmux |
| Agent model | You define tasks manually | Orchestrator spawns workers |
| Hierarchy | Flat (you → agents) | Tree (orchestrator → leads → workers) |
| Platform | macOS only | Any (tmux-based) |
| Self-improvement | No | Yes (emergent, v0.8+) |
| Best for | Parallel task management | Autonomous multi-level delegation |

### The forced continuation mechanic you noticed

When a lead agent tries to do something outside its scope (edit files directly
instead of delegating), Overstory denies it at the system level and forces
proper delegation. This is the autonomy constraint you need when running 10+
agents — without it, the hierarchy collapses into chaos. This is the feature
that makes Overstory production-safe rather than just a demo toy.

### The 55-minute autonomous run

The self-improving swarm demo (25 agents, 40 commits, 21 branch merges, invented
its own review loop mid-run) is real. That is what your Meta Code Squad is
working toward — a single `ov start` that builds a feature while you sleep.
Overstory v0.8.5 supports all your primary agents as runtime adapters: Claude
Code, Gemini CLI, and it has per-capability routing (different agents for
different roles).

### Why Phase 2, not now

Overstory requires a working monorepo with clear package boundaries to be
effective. Running it on an unscaffolded repo creates the exact architectural
drift and compounding error rates their own STEELMAN.md warns about. The
right sequence:

```
Phase 1: Scaffold monorepo + basic router (manual execution, Superset for parallel mgmt)
Phase 2: Overstory takes over as the autonomous execution layer
Phase 3: Overstory + Maestra = full autonomous build pipeline
```

### Integration target: Sprint 4-5, after monorepo structure is stable

```bash
npm install -g overstory

# .overstory/config.yaml
orchestrator: claude-code
team_leads:
  - runtime: gemini-cli
    role: builder
    capabilities: [test, config, docs]
  - runtime: kimi-code
    role: reviewer
    capabilities: [large-context-review, codebase-scan]
workers:
  - runtime: qwen-code
    role: specialist
  - runtime: iflow
    role: specialist
depth_limit: 2
worktree_base: ./.worktrees
```

---

## 5. Letta Code — MONITOR Phase 3 (P2)

### The genuine innovation

Letta Code is #1 in the model-agnostic open-source coding agent category on
Terminal-Bench. The memory architecture is legitimately different:

- `/init` deep-scans your codebase on first run, writes memories
- `/remember` corrects mistakes permanently — the agent learns from your feedback
- `.skills/` directory stores reusable task patterns as git-tracked markdown files
- Context Repositories: git-backed memory that survives across all sessions

### Why this matters for your workflow specifically

You have described a common pattern: you configure a tool extensively (Claude
Code hive-mind, Gemini with conductor, Kimi), get good results, but each new
session starts fresh and you redo context setup. Letta Code eliminates that
entirely. The agent that worked on your router last Tuesday still knows your
router architecture today.

### Why not now

You already have deeply configured Claude Code, Gemini CLI, and Kimi Code.
Switching primary agents mid-build creates context fragmentation — the opposite
of what Letta Code promises. The right time to introduce it is at the start of
a new package or project, not mid-sprint.

**Ideal introduction point:** When you start `packages/agents` (the Maestra
agent definitions package in Sprint 3). Let Letta Code own that package from
day one. It will accumulate memory about your agent patterns and become
genuinely useful by Sprint 5-6.

### One thing to do now (5 min)

```bash
npm install -g @letta-ai/letta-code
cd ~/projects/aigency
letta-code init   # Runs /init scan, writes initial memory
# Then leave it — just check in once a month
```

The memory compounds over time. Starting the clock now costs nothing.

---

## 6. ROMA — SKIP for now (P3)

### What it does well

ROMA's recursive decomposition (Atomizer → Planner → Executor → Aggregator →
Verifier) is genuinely well-designed. The React Flow execution visualization is
exactly as compelling as it looks in the demos — a live DAG showing which
subtasks are running, their states, latencies, and I/O. The SEALQA benchmark
result (45.6% vs Kimi's 36%) is impressive for complex multi-step reasoning.

### Why skip it

**Overlap:** ROMA is a task decomposition and execution framework. Maestra
already covers this for your agent workflows. Overstory already covers this for
your coding agent swarms. Adding ROMA creates a third overlapping orchestration
layer with no clear ownership boundary.

**Stack:** ROMA is Python/DSPy. Your entire stack is TypeScript/Node. Introducing
a Python orchestration layer creates a cross-language maintenance burden that
is not justified by the marginal capability gain.

**Maturity:** v0.2.0-beta, October 2024. Real, but the ecosystem is thin and
debugging a recursive agent failure across PostgreSQL + MLflow + MinIO is
non-trivial.

### When to reconsider

If you ever build a **research or reasoning-heavy agent** (not coding, but
something like a market research pipeline or competitive analysis system),
ROMA's recursive decomposition + DSPy optimization would be the right fit.
File it under "use for the right problem, not this problem."

---

## Integration Roadmap

### Today (Phase 0 — ~2 hours total)

```
[ ] Install Loki globally, run loki init in repo
[ ] Place AI Constitution at .loki/CONSTITUTION.md
[ ] Add AI Constitution as section in CLAUDE.md
[ ] Install Superset desktop app
[ ] Add aigency monorepo as Superset project
[ ] Configure agent runtime tiers in Superset
[ ] Install Letta Code globally, run letta-code init (5 min, then forget)
```

### Sprint 1-2 (Phase 1 — building)

```
[ ] GSD → Loki workflow for every wave
[ ] Superset managing parallel router + forge-quality work
[ ] Loki dashboard as primary monitoring view
```

### Sprint 3 (Phase 2a — Maestra)

```
[ ] Add packages/agents to monorepo
[ ] Implement SimpleLLMRouter v2 as Maestra workflow
[ ] Deploy Maestra REST API server
[ ] Wire Agent Studio into Aigency Next.js admin
[ ] Set up Maestra evals for forge-quality scoring
```

### Sprint 4-5 (Phase 2b — Overstory)

```
[ ] Install Overstory, write .overstory/config.yaml
[ ] Run first supervised autonomous build (one small story)
[ ] Tune hierarchy depth and runtime routing
[ ] Graduate to unsupervised overnight builds
```

### Sprint 6+ (Phase 3 — Letta)

```
[ ] Letta Code fully onboarded on packages/agents
[ ] Memory compounding across all sessions
[ ] .skills/ library growing with each completed task
[ ] Evaluate ROMA for any non-coding reasoning pipelines
```

---

## The Integrated Stack Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    AIGENCY PLATFORM (Next.js)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐   │
│  │ Agent Studio  │  │  Workflows   │  │  Evals / Scorers    │   │
│  │  (Maestra)    │  │  (Maestra)   │  │  (Maestra + forge)  │   │
│  └──────────────┘  └──────────────┘  └─────────────────────┘   │
└─────────────────────────────┬───────────────────────────────────┘
                               │ REST API
┌──────────────────────────────▼──────────────────────────────────┐
│                  ORCHESTRATION LAYER                             │
│  ┌──────────────────────┐  ┌──────────────────────────────────┐ │
│  │  Overstory           │  │  Superset                        │ │
│  │  (autonomous swarms) │  │  (parallel task management)      │ │
│  │  orchestrator →      │  │  worktree isolation              │ │
│  │  leads → workers     │  │  quota routing                   │ │
│  └──────────────────────┘  └──────────────────────────────────┘ │
└─────────────────────────────┬───────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                  EXECUTION LAYER                                 │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌─────────────┐  │
│  │ Claude Code│ │ Gemini CLI │ │  Kimi Code │ │ Letta Code  │  │
│  │ (Loki/BMAD)│ │ (conductor)│ │  (128k ctx)│ │ (memory)    │  │
│  └────────────┘ └────────────┘ └────────────┘ └─────────────┘  │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌─────────────┐  │
│  │ Qwen Code  │ │  Roo Code  │ │  Kilo Code │ │  iFlow CLI  │  │
│  │  (buffer)  │ │  (buffer)  │ │  (buffer)  │ │  (buffer)   │  │
│  └────────────┘ └────────────┘ └────────────┘ └─────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                  PLANNING + MEMORY LAYER                         │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────┐ │
│  │     GSD      │  │    Loki      │  │     Letta Memory      │ │
│  │  (planning)  │  │  (quality    │  │  (.skills/ git-backed) │ │
│  │  wave files  │  │   gates +    │  │  context repositories │ │
│  │              │  │  dashboard)  │  │                       │ │
│  └──────────────┘  └──────────────┘  └───────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Your AI Constitution — Placement Strategy

You mentioned you made your own version of Maestra's AI Constitution and are
thinking about where to place it. Here is the answer:

### Place it in 3 locations, each serving a different purpose

**1. `.loki/CONSTITUTION.md` (enforcement layer)**
Loki reads this before every task. It becomes a hard constraint on every
autonomous execution. This is the primary enforcement point.

**2. `CLAUDE.md` → section: "Constitutional Constraints" (Claude sessions)**
All Claude Code sessions (hive-mind, solo, agent teams) read CLAUDE.md at
startup. This covers executions that happen outside Loki.

**3. `packages/agents/src/constitution.ts` (programmatic layer)**
When you build the Maestra agent definitions in Sprint 3, export your
constitution as a typed TypeScript object that gets injected into every
agent's system prompt at instantiation. This makes it enforceable in code,
not just in prose.

```typescript
// packages/agents/src/constitution.ts
export const AI_CONSTITUTION = {
  principles: [...],
  constraints: [...],
  escalation_rules: [...],
} as const satisfies Constitution;
```

This triples enforcement surface area without tripling maintenance — you edit
one source file, it propagates to all three locations via the build.

---

## One More Thing — The Integration You Already Have

Before chasing new tools, confirm what you already have configured is firing
correctly. The research surfaced something important: your most-used tools
(Claude Code hive-mind, Gemini with conductor, Kimi) are all capable of
Overstory-level autonomous behavior when given the right PLAN files from GSD.

The constraint was never capability — it was orchestration and visibility.
Loki + Superset solve that today without adding new agent tools.

Add Maestra in Sprint 3 to make your architecture programmable.
Add Overstory in Sprint 4 to make it autonomous.
Let Letta Code accumulate memory in the background starting now.
Skip ROMA until the problem demands it.

That is the sequence.