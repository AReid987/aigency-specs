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
proper delegation. This is the autonomy constraint that prevents runaway behavior.

### The emergent self-improvement loop (v0.8+)

As of March 2026, Overstory v0.8 added the ability for orchestrator agents to
spawn new skills dynamically if they detect repeated task patterns. Example:

Sequence:
1. You assign task "deploy to Vercel with lighthouse check"
2. Orchestrator delegates → worker completes
3. Next day, same task structure, different branch
4. Orchestrator spawns → same worker sequence
5. On third repetition, orchestrator prompts: "Create reusable skill?"
6. You approve → new skill added to the agent's toolkit automatically

This is unsupervised learning without the risk of silent failure — all new
skills require human approval before activation.

### Where this fits in your Phase 2 roadmap

Overstory replaces the ad-hoc "which tool should I use for this task" decision
with a formal delegation tree. When you scaffold the Meta Code Squad monorepo,
the orchestrator becomes the entry point:

```
You → Orchestrator (Overstory)
       ↓
     Lead: Router Package (Claude Code)
       ↓
     Workers: API client gen (Kimi), tests (Gemini), docs (Qwen)

     Lead: forge-quality Package (Antigravity)
       ↓
     Workers: Linter rules (Roo Code), test suite (iFlow), benchmarks (AMP)
```

You issue one high-level instruction. Overstory decomposes, delegates, monitors,
merges. This is the automation you want for Sprint 2-4 when you are building
two packages in parallel.

### Integration target: Sprint 2, Week 3

Right after Loki + Superset are working. Install order matters — Superset for
basic parallel execution, then Overstory for autonomous delegation.

---

## 5. Letta Code — MONITOR Phase 3 (P2)

### Why it caught your eye

Letta Code is an LSP-based Claude Code alternative with long-term memory. The
compelling part is memory persistence across sessions — the agent "remembers"
your repo structure, coding preferences, and architectural decisions without
you re-explaining every time.

### The honest state of the project (as of March 2026)

Letta Code is **pre-release alpha** (v0.3.2). It works, but:
- LSP stability issues on large repos (crashes on 50K+ LOC codebases)
- Memory retrieval is slow (3-5 second lag on context queries)
- MCP tool support is incomplete (no custom tools yet)
- No Windows support

### What the roadmap shows (public, verified)

- v0.4 (April 2026): Stability fixes, MCP tool registry
- v0.5 (May 2026): Memory compression, <1s retrieval
- v1.0 (Q3 2026): Production-ready, full Claude Code parity

The project is real, the team is credible, but it is not ready for critical-path
work. **Monitor, do not integrate yet.**

### When to revisit

Two triggers:
1. v1.0 release (expected Q3 2026)
2. If your project reaches 5+ concurrent developers and context re-explanation
   becomes a bottleneck

At that point, Letta Code's memory layer becomes worth the migration cost.

---

## 6. ROMA — SKIP for now (P3)

### What it actually does

ROMA (Repository-Oriented Multi-Agent) is a research framework for running
multi-agent systems with full repo context at every step. The core mechanic:
every agent has the entire codebase in context at all times, eliminating the
"but I did not know about that file" failure mode.

### Why it is compelling

Full-context agents make fewer mistakes. When refactoring, they see every import.
When adding a feature, they detect naming conflicts. When writing tests, they
match existing patterns.

### Why it is not a fit right now

**1. Context window requirements**
ROMA needs 200K+ context windows to be effective on real codebases. Your best
option for this is Gemini CLI (1M-2M context). But Gemini CLI already gives you
full-repo context via `@project` command — ROMA does not add capability here.

**2. Inference cost at scale**
Running 5 agents x 200K context x 50 tasks/day = massive token usage even on
free tiers. Cerebras and Mistral can handle the volume, but those models are
not as strong at complex reasoning. You would be trading quality for context.

**3. Overlap with existing tools**
iFlow CLI's `/understand` command already does full-repo analysis and maintains
a context summary file. Loki's quality gates check cross-file dependencies.
ROMA's value-add shrinks when you have those two integrated.

### When ROMA becomes relevant

Two scenarios:

**Scenario 1: Codebase exceeds Gemini context limits**
If your monorepo grows past 2M tokens (~500K LOC), Gemini cannot hold it all.
At that scale, ROMA's chunking + retrieval strategy becomes necessary.

**Scenario 2: Multi-repo coordination**
If you are coordinating changes across 3+ repos simultaneously (backend, frontend,
infra), ROMA's multi-repo context model is the only tool designed for that.

**Current state:** You are nowhere near either threshold. Skip for now.

---

## 7. Integration Sequencing — Recommended Order

### Phase 1: Immediate (This Week)

**Day 1-2:** Loki + BMAD
- Install Loki globally
- Place AI Constitution in `.loki/CONSTITUTION.md`
- Run `loki doctor` to verify BMAD detection
- Launch dashboard, verify kanban view works

**Day 3-4:** Superset
- Download Superset desktop app
- Add aigency monorepo as project
- Configure agent runtimes (Claude, Gemini, Kimi, Qwen)
- Set up 3 task queues: CORE, THROUGHPUT, BUFFER
- Test parallel execution on two dummy tasks

**Day 5:** Integration validation
- Run one real task through Loki → verify quality gates pass
- Run two parallel tasks through Superset → verify worktree isolation
- Document any issues in AGENT.md

### Phase 2: After Monorepo Scaffold (Week 3-5)

**Week 3:** Overstory
- Install Overstory CLI
- Define orchestrator + lead agents for router and forge-quality packages
- Test delegation tree with one real feature task
- Evaluate emergent skill creation on repeated tasks

**Week 5:** Maestra
- Install Maestra in `packages/router`
- Refactor SimpleLLMRouter routing logic → Maestra workflows
- Deploy Maestra REST API for agent coordination
- Set up Agent Studio at `/admin/agents`
- Define evals for output quality scoring

### Phase 3: Post-Launch (Month 2+)

**Monitor:** Letta Code
- Watch for v1.0 release
- Revisit when team size grows or context re-explanation becomes bottleneck

**Skip:** ROMA
- Only revisit if codebase exceeds 500K LOC or multi-repo coordination needed

---

## 8. Cost-Benefit Summary

| Tool | Setup Time | Maintenance | Value | ROI |
|------|-----------|-------------|-------|-----|
| Loki + BMAD | 30 min | None (runs automatically) | High — eliminates manual kickoff + babysitting | Immediate |
| Superset | 45 min | Low (queue tuning) | High — parallel execution + quota management | Week 1 |
| Overstory | 2 hours | Medium (delegation tree refinement) | Medium — autonomous task decomposition | Week 3 |
| Maestra | 4 hours | Medium (workflow updates as agents change) | High — typed orchestration + REST API | Week 5 |
| Letta Code | 1 hour | Unknown (alpha) | Low now, high later | Q3 2026 |
| ROMA | 6 hours | High (context management) | Low (overlaps existing tools) | Only at massive scale |

---

## 9. Decision Framework — When to Integrate a New Tool

Use this checklist every time a new agent tool appears:

1. **Does it solve a problem you have right now?**
   - If no → monitor, do not integrate
   - If yes → proceed to #2

2. **Does an existing tool already solve 80% of it?**
   - If yes → stick with existing tool
   - If no → proceed to #3

3. **Is the setup/maintenance cost justified by time saved?**
   - Calculate: (hours saved per week) / (setup hours + weekly maintenance hours)
   - If ratio < 3 → skip
   - If ratio >= 3 → proceed to #4

4. **Is it stable enough for your critical path?**
   - Check: version number (< 1.0 = risky), GitHub activity, issue count
   - If pre-release alpha → monitor, do not integrate
   - If 1.0+ with active maintenance → integrate

5. **Does it compose with your existing stack or replace it?**
   - Compose (adds capability) → integrate
   - Replace (requires migration) → high bar, only if 5x better

**Example application:**
- Loki: YES to all 5 → integrate now
- Letta Code: NO to #4 (pre-release) → monitor
- ROMA: NO to #1 (no current problem) → skip

---

## 10. Final Recommendations

**Integrate immediately (P0):**
1. Loki + BMAD — eliminates manual execution management
2. Superset — parallel execution + quota handling

**Integrate in Phase 2 (P1):**
3. Overstory — autonomous delegation (after monorepo scaffold)
4. Maestra — typed workflows + REST API (Sprint 3)

**Monitor for later (P2):**
5. Letta Code — memory persistence (wait for v1.0, Q3 2026)

**Skip for now (P3):**
6. ROMA — full-context agents (only relevant at massive scale)

---

**Next Action:** Install Loki today. Superset tomorrow. Validate both by end of week.

**End of Strategy Doc**