# Meta Code Squad — Master System Spec
## Complete Component Map, Relationships & Phased Build Sequence
**Date:** 2026-03-06 | **Owner:** Antonio Reid
**Project Dir:** `meta-code-squad/`
**Status:** Implementation Ready

---

## 0. The Mission in One Paragraph

You are building the Meta Code Squad first — the autonomous dev team that will then build everything else. The squad is a layered system: a quota-busting LLM router at the bottom, a memory/persistence layer in the middle, orchestration and execution agents at the top, and a visual canvas (agor.live) as the command interface. Once the squad is operational, it builds Aigency Core, the sub-domain landing pages, and Project Blackout — largely without you babysitting it.

---

## 1. Full Component Map

### 1.1 The Permanent Stack (runs for everything, forever)

```
┌─────────────────────────────────────────────────────────────────────┐
│                     LAYER 0: QUOTA & ROUTING                        │
│                     SimpleLLMRouter v2                              │
│  ~1B free tokens/month across 8+ providers | sub-100ms routing      │
│  Textual TUI | 3-layer semantic cache | OptiLLM inference boost     │
│  Prevents quota death for ALL layers above                          │
└────────────────────────────┬────────────────────────────────────────┘
                             │ OpenAI-compatible proxy (:8080)
┌────────────────────────────▼────────────────────────────────────────┐
│                     LAYER 1: MEMORY & PERSISTENCE                   │
│            Ruflo (claude-flow v3.5) + Letta Code                    │
│                                                                     │
│  Ruflo:       AgentDB v3 | 12 daemon workers | HNSW vectors         │
│               Context Autopilot | 42 skills | Q-learning router     │
│               SONA self-learning | 175+ MCP tools                   │
│                                                                     │
│  Letta Code:  Stateful agent memory | Sleep-time compute daemon     │
│               Core/Recall/Archival tiers | Skill learning (+36.8%)  │
│               .letta/memory/ synced to git | /init deep indexing    │
└────────────────────────────┬────────────────────────────────────────┘
                             │ MCP servers
┌────────────────────────────▼────────────────────────────────────────┐
│                     LAYER 2: ORCHESTRATION                          │
│         GSD + Loki Mode + Sugar AI + Claude Agent Teams             │
│                                                                     │
│  GSD:         Discuss→Plan→Execute→Verify | wave-based PLAN files   │
│  Loki:        RARV cycle | 9 quality gates | kanban dashboard       │
│  Sugar:       24/7 task queue | Ralph loop | MCP memory             │
│  Agent Teams: CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1                │
│               Parallel agents per package, parallel worktrees       │
└────────────────────────────┬────────────────────────────────────────┘
                             │ task delegation + worktrees
┌────────────────────────────▼────────────────────────────────────────┐
│                     LAYER 3: EXECUTION AGENTS                       │
│        Superset (macOS) → Overstory (Phase 2, any platform)         │
│                                                                     │
│  Primary:     Claude Code (core logic, security, state machines)    │
│  Throughput:  Gemini CLI (tests, config, docs, boilerplate)         │
│  Coding:      Kimi Code CLI (active coding, multi-step tasks)       │
│  Buffer:      Qwen / Roo / Kilo / iFlow (quota overflow)            │
│  Routing:     Superset queues → auto-routes on quota hit            │
└────────────────────────────┬────────────────────────────────────────┘
                             │ visual canvas + zone triggers
┌────────────────────────────▼────────────────────────────────────────┐
│                     LAYER 4: CANVAS (FUTURE)                        │
│                          agor.live                                  │
│  Spatial SDLC zones | session trees | real-time agent view          │
│  Zone drag = agent trigger | fork/spawn hierarchy                   │
│  Phase 3 integration via Motia event bus                            │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 What Gets Built WITH the Squad (the products)

```
Meta Code Squad builds →

  packages/llm-router          SimpleLLMRouter v2 (TypeScript + Python TUI)
  packages/forge-quality       Code quality enforcer (Python hooks + scanner)
  packages/lp-gen              Landing page generator system
       ↓
  Aigency Core Platform        Next.js app, Maestra workflows, Supabase
  Sub-domain landing pages     agor.live, aigency.com, forge, etc.
  Project Blackout             Chrome extension milestone
```

---

## 2. Component Relationships — The Honest Map

### 2.1 What Replaces What

| Old/Manual Approach | Replaced By | Notes |
|---|---|---|
| Hitting quota, manually switching | SimpleLLMRouter v2 | All agents proxy through :8080 |
| Context loss between sessions | Ruflo Context Autopilot + Letta Code | Both run; ruflo for swarm, letta for codebase memory |
| Manual wave kickoff | Loki Mode (RARV + gates) | Consumes GSD PLAN files |
| Remembering past decisions | Sugar AI memory + Letta archival | Sugar for task decisions, Letta for code patterns |
| Manually managing parallel agents | Superset (now) → Overstory (Phase 2) | Superset is macOS GUI; Overstory is CLI/anywhere |
| ROMA | Skip entirely | TypeScript/Python mismatch; ruflo covers it |

### 2.2 What Does NOT Overlap (keep all of these)

| Tool | Unique Job | Layer |
|---|---|---|
| SimpleLLMRouter v2 | Quota routing + provider distribution | 0 |
| Ruflo | Swarm memory, agent coordination, lifecycle hooks | 1 |
| Letta Code | Codebase-specific stateful memory, sleep-time learning | 1 |
| GSD | Structured planning output (PLAN files) | 2 |
| Loki Mode | Autonomous PLAN execution + quality gates | 2 |
| Sugar AI | Persistent task queue + cross-session memory | 2 |
| Superset | macOS parallel worktree management + quota routing | 3 |
| Overstory | Hierarchical autonomous swarm (Phase 2) | 3 |
| Maestra | Typed workflow engine + REST API + evals (Phase 3) | 3+ |
| agor.live | Visual canvas + zone-based automation (Phase 3) | 4 |

### 2.3 How Letta Code Actually Learns (answering your question directly)

**You do NOT need to actively execute something for Letta to learn.** Here is the full picture:

**Active initialization (do once):**
```bash
# Run /init deep — fires ~100 tool calls, reads everything
# Reads: README, package.json, git history, architecture docs,
#        contributors, conventions, pain points
# Writes to: .letta/memory/ blocks (synced to git)
# Time: ~5-10 minutes for a medium repo
```

**Passive learning (automatic after init):**
- The **sleep-time compute daemon** runs in the background while you work
- A secondary sleep-time agent continuously reorganizes and improves memory quality
- Memory blocks in `.letta/memory/` update as the codebase evolves
- You can also run `/remember` explicitly to force a learning pass

**What it learns automatically:**
- Code conventions and patterns it observes during your sessions
- Architecture decisions made in conversations
- Bug patterns and their fixes
- Which approaches failed and why

**Bottom line:** After `/init deep` once, Letta Code is the passive librarian. It learns from every session automatically via the sleep-time daemon. You do not babysit it. The more sessions you run, the smarter it gets. The `.letta/memory/` folder is version-controlled so knowledge compounds and never gets lost.

---

## 3. The Justfile — Single Command Installation

**Yes — use `just`. No, you do not run every command manually every time.**

One `justfile` at repo root handles everything. You run `just setup` once. After that, individual `just <recipe>` commands replace all the multi-step manual sequences.

**Install just first (one time, globally):**
```bash
brew install just          # macOS
# or
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin
```

**The complete `justfile` for meta-code-squad:**
```just
# meta-code-squad/justfile
# Run `just` to see all available recipes

set dotenv-load := true

# ── DEFAULTS ──────────────────────────────────────────────────────────
default:
    @just --list

# ── ONE-TIME FULL SETUP ───────────────────────────────────────────────
# Run this ONCE after cloning. Installs everything in correct order.
setup:
    just _check-prereqs
    just install-tools
    just init-ruflo
    just init-letta
    just init-sugar
    just init-loki
    just init-monorepo
    just doctor
    @echo "✓ Meta Code Squad ready. Run: just dev"

# ── PREREQUISITE CHECK ────────────────────────────────────────────────
_check-prereqs:
    @command -v node >/dev/null || (echo "ERROR: node not found" && exit 1)
    @command -v python3 >/dev/null || (echo "ERROR: python3 not found" && exit 1)
    @command -v pipx >/dev/null || (echo "ERROR: pipx not found — run: brew install pipx" && exit 1)
    @command -v git >/dev/null || (echo "ERROR: git not found" && exit 1)
    @echo "✓ Prerequisites OK"

# ── TOOL INSTALLATION ─────────────────────────────────────────────────
install-tools:
    just _install-ruflo
    just _install-loki
    just _install-sugar
    just _install-letta
    just _install-gemini
    just _set-env

_install-ruflo:
    @echo "→ Installing Ruflo (claude-flow v3.5)..."
    curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full
    @echo "✓ Ruflo installed"

_install-loki:
    @echo "→ Installing Loki Mode..."
    npm install -g loki-mode
    @echo "✓ Loki installed"

_install-sugar:
    @echo "→ Installing Sugar AI..."
    pipx install sugarai
    @echo "✓ Sugar installed"

_install-letta:
    @echo "→ Installing Letta Code..."
    pipx install letta
    letta server start &
    sleep 3
    @echo "✓ Letta installed and server started"

_install-gemini:
    @echo "→ Installing Gemini CLI..."
    npm install -g @google/gemini-cli
    @echo "✓ Gemini CLI installed"

_set-env:
    @echo "→ Setting persistent environment variables..."
    echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
    echo 'export SIMPLELLMROUTER_URL=http://localhost:8080' >> ~/.zshrc
    @echo "✓ Environment variables set (restart terminal or: source ~/.zshrc)"

# ── INITIALIZATION ────────────────────────────────────────────────────
init-ruflo:
    @echo "→ Initializing Ruflo in project..."
    npx ruflo@latest init --wizard
    claude mcp add ruflo -- npx -y ruflo@latest mcp start
    @echo "✓ Ruflo initialized + MCP registered"

init-letta:
    @echo "→ Initializing Letta Code in project..."
    letta code init
    @echo "✓ Letta Code initialized — run /init deep in your first session"

init-sugar:
    @echo "→ Initializing Sugar AI..."
    sugar init
    @echo "✓ Sugar initialized"

init-loki:
    @echo "→ Initializing Loki Mode..."
    loki init
    loki doctor
    @echo "✓ Loki initialized — place CONSTITUTION.md at .loki/CONSTITUTION.md"

init-monorepo:
    @echo "→ Scaffolding Turborepo monorepo..."
    npx create-turbo@latest . --skip-install
    npm install
    just _scaffold-packages
    @echo "✓ Monorepo scaffolded"

_scaffold-packages:
    mkdir -p packages/llm-router/src
    mkdir -p packages/forge-quality/src
    mkdir -p packages/lp-gen/src
    mkdir -p apps/aigency-core
    mkdir -p .context/docs
    cp docs/*.md .context/docs/ 2>/dev/null || true
    @echo "✓ Package directories created"

# ── HOOKS CONFIGURATION ───────────────────────────────────────────────
# Hooks are configured via .claude/settings.json — NOT run manually each session.
# This recipe writes the config once; hooks fire automatically after that.
configure-hooks:
    mkdir -p .claude
    cat > .claude/settings.json << 'EOF'
    {
      "hooks": {
        "enabled": true,
        "autoExecute": {
          "preTask": true,
          "postTask": true,
          "preEdit": true,
          "postEdit": true,
          "sessionStart": true,
          "sessionEnd": true
        },
        "memory": {
          "persistence": true,
          "location": ".swarm/memory.db",
          "syncInterval": 30000
        },
        "performance": {
          "tracking": true
        }
      }
    }
    EOF
    npx ruflo daemon start
    @echo "✓ Hooks configured — they fire automatically, no manual commands needed"

# ── DAILY DEVELOPMENT ─────────────────────────────────────────────────
# These are what you run day-to-day (not setup commands)

# Start everything needed for a dev session
dev:
    just start-router
    just start-ruflo-daemon
    @echo "✓ Stack running. Start Claude Code — hooks fire automatically."
    @echo "  Router TUI: just tui"
    @echo "  Loki dash:  just dashboard"

start-router:
    @echo "→ Starting SimpleLLMRouter v2..."
    cd packages/llm-router && npm run dev &
    @echo "✓ Router running at http://localhost:8080"

start-ruflo-daemon:
    npx ruflo daemon start
    @echo "✓ Ruflo daemon running (12 background workers active)"

tui:
    cd packages/llm-router && python -m router_tui

dashboard:
    loki dashboard

# ── EXECUTION WORKFLOW ────────────────────────────────────────────────
# Run a GSD plan through Loki
run plan:
    loki start {{plan}}

# Add a task to Sugar queue
task description:
    sugar add "{{description}}" --triage

# Active coding task or codebase review with Kimi
review:
    kimi "Review the full codebase for issues, architecture violations, and improvement opportunities"

# ── HEALTH & DIAGNOSTICS ─────────────────────────────────────────────
doctor:
    npx ruflo doctor --deep
    loki doctor
    sugar status
    @echo "✓ Health check complete"

# ── PHASE SHORTCUTS ───────────────────────────────────────────────────
# Phase 1 packages
build-router:
    cd packages/llm-router && npm run build

build-forge:
    cd packages/forge-quality && python -m build

build-lp-gen:
    cd packages/lp-gen && npm run build

# Test all packages
test:
    turbo run test

lint:
    turbo run lint

# ── LETTA SHORTCUTS ───────────────────────────────────────────────────
letta-init-deep:
    @echo "Run this inside a Letta Code session: /init deep"
    @echo "It fires ~100 tool calls and indexes the full codebase."
    @echo "Do this once per project, then memory compounds automatically."

letta-status:
    letta agents list
```

**Key answer: hooks are NOT run manually every session.** The `configure-hooks` recipe writes `.claude/settings.json` once. After that, `sessionStart`, `preTask`, `postTask`, etc. fire automatically every time Claude Code starts. You never type a hook command again.

---

## 4. Ruflo Hooks — Automatic, Not Manual

This is explicitly answered: **set `autoExecute: true` in `.claude/settings.json` once, and all 17 hooks register permanently.**

The hooks that matter most:
- `sessionStart` → restores context, reloads swarm agents automatically
- `postTask` → feeds the SONA learning loop, updates memory
- `preEdit` / `postEdit` → change tracking, agent sync

After `just configure-hooks`, you never touch hooks again. The daemon runs in background. `just dev` is all you need at the start of each session.

---

## 5. Phased Build Sequence

### PHASE 0 — Environment (Day 1, ~2 hours)
**Goal:** Everything installed, configured, no manual commands needed ever again.

```
1. mkdir meta-code-squad && cd meta-code-squad
2. git init
3. Copy docs into docs/ (see Section 7 for exact list)
4. brew install just
5. Create justfile (copy from Section 3)
6. just setup          ← installs ALL tools in correct order
7. just configure-hooks ← hooks are now permanent
8. just doctor         ← verify everything green
```

After Phase 0: One command (`just dev`) starts the full stack. Hooks fire automatically. You never run individual tool commands again.

---

### PHASE 1 — Build the Meta Code Squad Itself (Weeks 1–3)
**Goal:** `packages/llm-router` + `packages/forge-quality` working and tested.

**Sprint 1 — LLM Router Core (Week 1)**
```
packages/llm-router/
  src/
    router.ts           ← 14-dimension classifier + provider config
    quota-tracker.ts    ← wired up (this is the main gap to fix)
    cache/
      l1-memory.ts      ← node-lru-cache, 5min TTL
      l2-disk.ts        ← better-sqlite3, 24h TTL
      l3-semantic.ts    ← hnswlib-node, 72h TTL
    providers/          ← 8 free-tier providers configured
    templates/          ← Handlebars-style prompt templates
  router-tui/
    app.py              ← Textual TUI, 9 screens
    screens/            ← boot, dashboard, quota, metrics, etc.
```

Execution: `just run PLAN-1.md` (after GSD generates PLAN-1.md)
Agents: Claude Code (core) + Gemini CLI (tests + boilerplate)
Quota strategy: Router routes itself during build (meta)

**Sprint 2 — Forge Quality (Week 2)**
```
packages/forge-quality/
  src/
    scanner.py          ← AST analysis, complexity metrics
    hooks/              ← pre-commit, pre-push quality gates
    cli.py              ← forge check, forge fix, forge report
    reporters/          ← terminal + JSON + HTML output
```

**Sprint 3 — Integration + LP-Gen Package (Week 3)**
```
packages/lp-gen/        ← Landing page generator
  src/
    agents/             ← Discovery, Strategy, Copy, Design, Architect
    pipeline.ts         ← orchestrated generation pipeline
    templates/          ← section templates per page type
```

Loki + Maestra integration happens here (Sprint 3 is when Maestra enters).

---

### PHASE 2 — Aigency Core Platform (Weeks 4–8)
**Goal:** The main Aigency web app, built by the Meta Code Squad.

```
apps/aigency-core/      ← Next.js 15, App Router
  src/
    app/                ← routes
    components/         ← UI components
    lib/
      maestra/          ← Agent workflows (Maestra enters here)
      supabase/         ← DB client
      router-client.ts  ← talks to SimpleLLMRouter v2
    api/
      agents/           ← Maestra REST endpoints
```

Overstory replaces Superset for autonomous overnight runs in Phase 2.
Maestra workflows replace ad-hoc agent wiring.

---

### PHASE 3 — Sub-Domain Landing Pages (Weeks 9–11)
**Goal:** agor.live, aigency.com/forge, aigency.com/squad, etc. — all built by lp-gen.

The `lp-gen` package built in Phase 1 Sprint 3 now builds all landing pages. This is the payoff: the squad builds the builder, the builder builds the pages. DSPy optimization runs here against real usage data from Phase 2.

agor.live canvas integration happens in Phase 3 (Motia event bus bridges Router + agor.live).

---

### PHASE 4 — Project Blackout Chrome Extension (Weeks 12–14)
**Goal:** Chrome extension milestone.

Built entirely by the Meta Code Squad. Loki quality gates enforce extension security standards. Kimi handles active coding tasks and multi-step implementation workflows before each submission build.

---

## 6. Quota Strategy (How You Never Hit Limits)

```
All agents → SIMPLELLMROUTER_URL=http://localhost:8080 (set in .zshrc by just setup)

Router distributes across:
  Gemini Flash       600 req/min, 1500/day free
  Gemini Pro         60 req/min, 50/day free
  Claude Haiku       via Anthropic free tier
  GPT-4o-mini        via OpenAI free tier
  Groq Llama         6000 tokens/min free
  Mistral            via free tier
  Cohere             via free tier
  DeepSeek           via free tier
  Total: ~1B tokens/month free

3-layer semantic cache (30-60% additional token savings for agentic workloads)
OptiLLM inference boost for COMPLEX tasks only (routes through after provider selection)
Circuit breaker per provider (fails 3 reqs in 60s → excluded until recovery)
Canary testing (X% of traffic to new providers for A/B quality testing)

Ruflo WASM tier (Agent Booster): simple transforms skip LLM entirely → <1ms, 0 tokens
```

---

## 7. Docs to Copy into `meta-code-squad/docs/` RIGHT NOW

Copy these before running any commands. They become the context foundation that GSD, Loki, Sugar, and Letta all read on initialization.

### Required (copy all of these):

| File | Why It's Needed |
|---|---|
| `project-brief-aigency-dev-platform.md` | Top-level project intent — GSD + Sugar read this to understand the mission |
| `prd-aigency-dev-platform.md` | Feature requirements — Loki quality gates validate against this |
| `architecture-aigency-dev-platform.md` | System architecture — Letta `/init deep` uses this as primary architecture doc |
| `backlog-aigency-dev-platform.md` | Sprint backlog — Sugar queue seeds from this |
| `meta-code-squad-spec.md` | The squad's own spec — every agent needs to know how the squad works |
| `meta-code-squad-launch-guide.md` | Workflow guide — GSD reads this for process context |
| `simplellmrouter-v2-spec.md` | Router spec — Claude Code implements from this |
| `agentic-tooling-integration-strategy.md` | Tool decisions — prevents agents from re-debating already-decided choices |
| `aigency-squad-install-spec.md` | Squad install spec — Loki gates reference this |
| `ruflo-master-integration-guide.md` | Ruflo integration decisions — prevents re-research |
| `meta-code-squad-master-system-spec.md` | THIS FILE — the single source of truth |

### Optional but valuable:
| File | Why |
|---|---|
| `ux-specs-aigency-dev-platform.md` | Needed in Phase 2 when building Aigency Core UI |
| `execution-plan-aigency-dev-platform.md` | Sprint timeline reference |
| `aigency-agent-personas.md` | Agent identity specs — needed when squad agents spawn persona-aware tasks |
| `design-system.md` | Needed in Phase 2/3 for landing pages |
| `tech-stack.md` | Tech decisions — prevents agents from choosing wrong stack |

### Folder structure after copying:
```
meta-code-squad/
  docs/
    project-brief-aigency-dev-platform.md
    prd-aigency-dev-platform.md
    architecture-aigency-dev-platform.md
    backlog-aigency-dev-platform.md
    meta-code-squad-spec.md
    meta-code-squad-launch-guide.md
    simplellmrouter-v2-spec.md
    agentic-tooling-integration-strategy.md
    aigency-squad-install-spec.md
    ruflo-master-integration-guide.md
    meta-code-squad-master-system-spec.md  ← this file
  justfile                                  ← created in Phase 0
  CLAUDE.md                                 ← created by just init-ruflo
  .loki/
    CONSTITUTION.md                         ← your AI constitution goes here
```

**Why docs first?** `just setup` calls `just init-monorepo` which runs `cp docs/*.md .context/docs/`. The context folder is what Loki, GSD, Sugar, and Letta scan during their own init sequences. If you run setup before copying docs, you get empty context and have to re-init every tool manually.

---

## 8. The Exact Sequence — Start to First Autonomous Build

```bash
# ── BEFORE ANY COMMANDS ──────────────────────────────────────────────
# Copy all docs from Section 7 into meta-code-squad/docs/
# Copy the justfile from Section 3 into meta-code-squad/justfile

# ── PHASE 0: ONE-TIME SETUP (~2 hours) ──────────────────────────────
brew install just
cd meta-code-squad
just setup                     # installs ALL tools in correct order
just configure-hooks           # hooks permanent — never run manually again
just doctor                    # verify green

# ── FIRST SESSION ────────────────────────────────────────────────────
just dev                       # starts router + ruflo daemon
# Open Claude Code
# Run: /init deep              # Letta indexes the codebase (~10 min, once)
# Run: /gsd:new-project        # GSD reads docs/ and generates PROJECT.md etc.
# Run: /gsd:discuss-phase 1    # Discuss the router build
# Run: /gsd:plan-phase 1       # Generates PLAN-1.md

# ── FIRST AUTONOMOUS BUILD ───────────────────────────────────────────
just run PLAN-1.md             # Loki consumes PLAN-1.md, RARV cycle starts
just dashboard                 # Watch it build in the kanban dashboard
# Walk away. Come back when it's done.
```

---

## 9. Tool-by-Tool "Do I Need to Do Anything?" Reference

| Tool | Ongoing manual action? | What's automatic |
|---|---|---|
| SimpleLLMRouter | `just start-router` once per dev session | All routing, caching, failover |
| Ruflo daemon | `just dev` starts it | All 12 workers, hooks, memory, SONA learning |
| Ruflo hooks | Never — configured once | All 17 hooks fire on every event |
| Letta Code | `just letta-init-deep` once per project | Sleep-time daemon learns continuously |
| GSD | Run slash commands to start phases | Generates all PLAN files |
| Loki | `just run PLAN-N.md` to start | RARV cycle, quality gates, commits |
| Sugar | `just task "description"` to queue | Picks up tasks, Ralph loop, memory |
| Superset | Launch app, queue tasks | Quota routing, worktree isolation |
| Claude Agent Teams | Enabled via env var (set by just setup) | Parallel agents per package |

---

## 10. Phase 3+ Additions (Don't Touch These Yet)

| Tool | When | What It Does |
|---|---|---|
| Maestra | Sprint 3 (Week 3) | Typed workflow engine, REST API, evals framework |
| Overstory | Phase 2 (Week 4+) | Replaces Superset with hierarchical autonomous swarm |
| agor.live | Phase 3 | Visual canvas, zone-based automation |
| OptiLLM | Phase 2 | Inference boost layer between router and providers |
| DSPy | Phase 3 | Prompt optimization after real usage data exists |
| Letta Skills | Phase 2 | Explicit skill files for domain knowledge |

---

*This is the single source of truth. Every other doc feeds into this one.*
*When in doubt: check this file. When something changes: update this file first.*
