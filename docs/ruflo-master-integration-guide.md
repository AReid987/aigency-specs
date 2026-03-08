# Ruflo (claude-flow v3.5) — Master Integration Guide
## Complete Setup, Configuration & Integration for the Aigency / agor.live Stack
**Date:** 2026-03-06 | **Owner:** Antonio Reid
**Repo:** https://github.com/ruvnet/ruflo | **Stars:** 19.3k | **Version:** v3.5.7

---

## 0. First: The Big Picture Alignment

Yes — you still have it. The goal is:

> **agor.live** = real-time visual canvas, spatial SDLC regions (kanban-inspired), any CLI/IDE agent instantiable inside it. Agents boot autonomously, tasks call the best agents, work runs in parallel, swarms share context cross-project, self-learning compounds, intervention approaches zero.

Ruflo is the **engine underneath that canvas**. It is not the canvas itself — agor.live is. But Ruflo provides:
- The queen/worker swarm topology that makes autonomous spawning possible
- The persistent cross-session, cross-project memory (AgentDB v3, HNSW, RVF)
- The 3-tier routing that decides which agent handles each task
- The background workers that run continuously (context compaction, drift detection, learning loop)
- The 175+ MCP tools that Claude Code / any IDE agent calls through

Without Ruflo (or claude-flow), agor.live is a canvas with nothing running under it. With Ruflo fully installed and configured, the canvas has a nervous system.

---

## 1. What Ruflo Actually Is (v3.5)

Ruflo is the production rename of claude-flow after 5,800+ commits across 55 alpha iterations. Both names resolve to the same package:

```bash
npx ruflo@latest      # recommended
npx claude-flow@latest   # legacy alias — same thing
npx @claude-flow/cli@latest  # scoped package — same thing
```

**What it ships:**
- 60+ specialized agents (coder, reviewer, tester, security, devops, docs, architect...)
- 175+ MCP tools exposed to Claude Code and any MCP-compatible IDE agent
- 26 CLI commands with 140+ subcommands
- 17 lifecycle hooks
- 12 background workers (daemon mode)
- 42 pre-built skills (workflow patterns)
- 3-tier model routing (WASM Agent Booster → Haiku → Opus)
- AgentDB v3 with 20+ memory controllers
- RVF (RuVector Format) — binary vector storage, no WASM downloads
- HNSW vector search (150x–12,500x faster than SQLite FTS)
- SONA self-learning (<0.05ms adaptation latency)
- Byzantine fault-tolerant consensus (Raft, BFT, Gossip, CRDT)
- Context Autopilot (prevents context window loss)

---

## 2. Prerequisites

```bash
# Required
node --version    # must be 20+
npm --version     # must be 9+

# Required: Claude Code must be installed FIRST
npm install -g @anthropic-ai/claude-code
claude --version

# Optional but recommended: skip permissions prompt during setup
claude --dangerously-skip-permissions
```

---

## 3. Installation — All Options

### 3A. One-Line Full Install (Recommended for your stack)

```bash
# Core install + MCP + diagnostics
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full

# Verify it worked
npx ruflo doctor
```

### 3B. NPX (No global install needed)

```bash
# Interactive wizard — best for first run
npx ruflo@latest init --wizard

# Or headless init
npx ruflo@latest init
```

### 3C. Global NPM Install

```bash
npm install -g ruflo@latest
ruflo init

# With Bun (faster)
bunx ruflo@latest init
```

### 3D. Install Profiles

```bash
# Minimal — core CLI only (~45MB, fastest)
npm install -g ruflo@latest --omit=optional

# Full — includes ML/embeddings/WASM (~340MB, everything)
npm install -g ruflo@latest
```

**For your use case: full install.** You want SONA, HNSW, RuVector, and the embeddings layer running. The 340MB is worth it.

### 3E. OpenAI Codex CLI Support

```bash
# Ruflo works with Codex CLI too — same install, Codex integration is built in
npx ruflo@latest init  # select "Codex" during wizard
```

---

## 4. MCP Integration (Most Important Step)

This is what plugs Ruflo into Claude Code and any MCP-compatible IDE agent (Cursor, Windsurf, VS Code with MCP, etc.):

```bash
# Add ruflo as MCP server to Claude Code
claude mcp add ruflo -- npx -y ruflo@latest mcp start

# Verify it registered
claude mcp list

# Start the MCP server manually (if needed)
npx ruflo@latest mcp start
```

Once this is done, Claude Code has access to all 175+ ruflo MCP tools directly:
- `swarm_init` — boot a swarm
- `agent_spawn` — spawn a specialized agent
- `memory_search` — HNSW semantic search across all stored patterns
- `hooks_route` — intelligent task routing
- `task_orchestrate` — decompose a complex task across agents
- `mcp__claude-flow__github_swarm` — GitHub-integrated swarm
- `mcp__claude-flow__neural_train` — train on a successful task
- `mcp__claude-flow__neural_patterns` — inspect learned patterns
- ...and 165+ more

**For agor.live:** The canvas will call these MCP tools to instantiate agents in spatial regions. Each SDLC phase region (Plan → Design → Build → Test → Deploy) maps to agent types. The MCP server is the bridge.

---

## 5. Your First Swarm (Run This Now)

```bash
# 1. Initialize a hierarchical swarm (queen + workers, prevents drift)
npx ruflo swarm init \
  --topology hierarchical \
  --max-agents 8 \
  --strategy specialized

# 2. Spawn the core dev agents
npx ruflo agent spawn -t architect --name arch-01
npx ruflo agent spawn -t coder --name coder-01
npx ruflo agent spawn -t tester --name tester-01
npx ruflo agent spawn -t reviewer --name reviewer-01

# 3. Check swarm status
npx ruflo swarm status

# 4. Route a task through the system
npx ruflo hooks route --task "implement OAuth2 login flow"
```

**Swarm topologies available:**
- `hierarchical` — Queen leads workers (best for SDLC, prevents drift, recommended)
- `mesh` — Peer-to-peer (best for research/planning phase)
- `ring` — Sequential pipeline (best for CI/CD chains)
- `star` — Central coordinator (best for fan-out parallelism)

---

## 6. Background Daemon (Always-On Workers)

This is what makes the system run autonomously without your intervention:

```bash
# Start the daemon (12 background workers)
npx ruflo daemon start

# Check daemon status
npx ruflo daemon status

# Stop daemon
npx ruflo daemon stop
```

**The 12 background workers that run continuously:**

| Worker | What It Does |
|--------|-------------|
| Memory Consolidation | Merges cross-session context |
| Vector Indexing | Maintains HNSW search index |
| Agent Heartbeat | Monitors swarm health |
| Learning Loop | Trains on successful task patterns |
| State Sync | Keeps memory.db in sync |
| Garbage Collector | Cleans stale memory entries |
| Metric Aggregation | Compiles performance data |
| Hook Queue | Processes batched hook events |
| Skill Distillation | Fine-tunes agent behaviors |
| Drift Detection | Alerts on agent goal misalignment |
| Model Router | Optimizes LLM selection (saves 75%) |
| Context Compaction | Maintains context windows, prevents loss |

The Context Compaction worker alone solves one of your biggest pain points — you will never lose context to a full window again. It archives and restores automatically.

---

## 7. Hooks System (17 Lifecycle Hooks)

Hooks are what make the system self-coordinating. They fire automatically at key lifecycle points:

### Core Hooks

```bash
# Before a task starts — creates tracking ID
npx ruflo hooks pre-task \
  --description "Implement user authentication" \
  --priority critical \
  --metadata '{"assignee":"coder-swarm"}'
# Returns: task-1753483207250-u7wbmsetj

# After a task completes — stores result in memory
npx ruflo hooks post-task \
  --task-id "task-1753483207250-u7wbmsetj" \
  --status completed \
  --results '{"lines_added":120,"tests_passing":true}' \
  --store-results true

# Before file edit — backup + change tracking
npx ruflo hooks pre-edit \
  --file "src/auth.ts" \
  --operation "update authentication logic" \
  --backup true

# After file edit — validate + sync agents
npx ruflo hooks post-edit \
  --file "src/auth.ts" \
  --memory-key "auth/implementation" \
  --sync-agents true

# Session start — restore context + reload agents
npx ruflo hooks session-start --restore-context --load-agents

# Session end — save state + generate report
npx ruflo hooks session-end --save-state --generate-report --cleanup
```

### Hook Configuration File

Create `.claude/settings.json` in your project root:

```json
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
    "customHooks": {
      "beforeCommit": {
        "command": "npm test && npm run lint",
        "failOnError": true
      },
      "afterDeploy": {
        "command": "npx ruflo notify --channel deployment",
        "async": true
      }
    },
    "memory": {
      "persistence": true,
      "location": ".swarm/memory.db",
      "syncInterval": 30000
    },
    "performance": {
      "tracking": true,
      "alertThresholds": {
        "task": 300000,
        "edit": 5000,
        "agent": 60000
      }
    }
  }
}
```

### All 17 Hook Events

```
PreToolUse Write          — before any file write
PreToolUse Edit           — before any file edit
PreToolUse MultiEdit      — before batch file edits
PreToolUse Task           — before task spawning
PostToolUse Bash          — after shell command execution
PostToolUse Task          — after task completion
PostToolUse Edit          — after edit completion
UserPromptSubmit          — on every user input (routing decision point)
SubagentStop              — when a sub-agent terminates
SubagentEnd               — sub-agent lifecycle cleanup
SessionStart              — on every new Claude Code session
Stop sync                 — on Claude Code stop
PostToolUseFailure        — on any tool failure (error recovery)
AgentSpawn                — agent configuration on creation
AgentComplete             — result collection when agent finishes
PerfStart                 — performance monitoring start
PerfEnd                   — performance monitoring end
```

---

## 8. Memory System — AgentDB v3

This is the brain. 20+ memory controllers, 4 memory tiers, HNSW vector search.

### Memory Commands

```bash
# Store a pattern
npx ruflo memory store \
  --key "auth-pattern" \
  --value "JWT with refresh tokens, bcrypt for passwords" \
  --namespace patterns

# Semantic search (HNSW — 150x faster than text search)
npx ruflo memory search \
  --query "authentication best practices" \
  --limit 5

# List all memories in a namespace
npx ruflo memory list --namespace patterns

# Export memory for backup
npx ruflo memory export --output .swarm/memory-backup.json

# Import memory (cross-project transfer)
npx ruflo memory import --input .swarm/memory-backup.json
```

### 4 Memory Tiers

| Tier | Storage | Lifetime | Size | Content |
|------|---------|----------|------|---------|
| Working | RAM / /tmp | Single task | 100KB–10MB | Active context, scratch space |
| Episodic | Git commits, SQLite | Days–weeks | 100MB–1GB | Task history, recent interactions |
| Short-Term | Git repo (.memory/) | Weeks–months | 10MB–100MB | Project patterns, conventions |
| Long-Term | Global Git + vector DB | Permanent | 100MB–1GB | Universal patterns, meta-knowledge |

### 8 Memory Types (AgentDB)

1. `procedural` — How to do tasks (skills, workflows)
2. `semantic` — Factual knowledge (codebase facts, API shapes)
3. `episodic` — What happened (task history, decisions)
4. `working` — Current active context
5. `associative` — Connected concepts (graph relationships)
6. `temporal` — Time-sensitive data (sprint info, deadlines)
7. `spatial` — File/directory structure knowledge
8. `meta` — Knowledge about knowledge (what the system knows it knows)

### 20+ Memory Controllers (AgentDB v3)

Key controllers you'll use:

```
HierarchicalMemory     — multi-level memory hierarchies
MemoryConsolidation    — cross-session merging
SemanticRouter         — intent-based routing
GNNService             — graph neural network queries
RVFOptimizer           — RuVector field optimization
MutationGuard          — proof-verified writes (no silent corruption)
AttestationLog         — cryptographic audit trail
GuardedVectorBackend   — verified vector operations
LearningBridge         — connects hooks to learning loop
PageRankInsights       — identifies influential patterns
LRUCache               — fast working memory
SQLitePersistence      — durable storage
KnowledgeGraph         — entity relationships
CrossAgentTransfer     — share knowledge between agents
ProjectScope           — isolate per project
LocalScope             — user-level persistence
UserScope              — global across all projects
```

---

## 9. 3-Tier Model Routing (75% Cost Reduction)

Ruflo routes tasks to the cheapest model that can handle them:

| Tier | Handler | Latency | Cost | Use Case |
|------|---------|---------|------|---------|
| 1 | Agent Booster (WASM) | <1ms | $0 | Simple transforms — skip LLM entirely |
| 2 | Haiku / flash model | ~50ms | Low | Moderate reasoning, standard tasks |
| 3 | Opus / full model | ~500ms | High | Complex agentic tasks, architecture |

The Q-Learning router learns over time which tier each task type belongs to. After a few days, 80% of tasks route to Tier 1 or 2 automatically.

### Agent Booster (WASM) — Tier 1

The Agent Booster uses compiled WASM to handle simple code transforms without hitting an LLM at all:

```bash
# Enable agent booster
npx ruflo config set agentBooster.enabled true

# Tasks the booster handles locally:
# - import sorting/organization
# - syntax validation
# - template expansion
# - code smell detection
# - variable renaming
# - simple refactors
```

---

## 10. 42 Skills System

Skills are pre-built workflow patterns. Think of them as reusable playbooks:

```bash
# List all 42 available skills
npx ruflo skills list

# Install from community registry
npx ruflo skills install @community/auth-patterns
npx ruflo skills install @community/react-components
npx ruflo skills install @community/typescript-strict

# Create a skill from a successful task
npx ruflo skills create --from-task "task-1753483207250-u7wbmsetj" --name "aigency-auth-flow"

# Apply a skill
npx ruflo skills apply --skill "aigency-auth-flow" --context "src/auth/"
```

**Skills directly relevant to your stack:**
- `spec-driven-dev` — build full spec, then implement without drift
- `parallel-implementation` — multi-agent simultaneous build
- `testing-coverage` — auto-generate test suites to coverage threshold
- `documentation-generation` — auto-doc from code + comments
- `security-audit` — CVE scan + hardening suggestions

---

## 11. Security System (AIDefence)

```bash
# Run security scan on a file/directory
npx ruflo security scan --target src/

# PII detection
npx ruflo security pii --input logs/

# Prompt injection test
npx ruflo security injection-test --agent coder

# Full security audit
npx ruflo security audit --comprehensive
```

Built-in protections:
- Prompt injection detection and blocking
- Path traversal prevention
- Command injection blocking
- Input validation on all tools
- bcrypt credential handling
- CVE hardening (active)
- PII scanning
- XSS / SQL injection prevention

---

## 12. Environment Variables — Full Config Reference

Create `.env` in your project root (or set in shell profile):

```bash
# === LLM Providers ===
ANTHROPIC_API_KEY=your_key            # Claude (primary)
OPENAI_API_KEY=your_key               # GPT-4 (failover)
GOOGLE_API_KEY=your_key               # Gemini (cost routing)
COHERE_API_KEY=your_key               # Cohere (optional)
GROQ_API_KEY=your_key                 # Groq (fast inference)
OPENROUTER_API_KEY=your_key           # OpenRouter (multi-model)

# === Model Routing ===
RUFLO_DEFAULT_MODEL=claude-sonnet     # fallback model
RUFLO_COST_OPTIMIZATION=true          # enable 3-tier routing
RUFLO_BOOSTER_ENABLED=true            # WASM tier 1 routing

# === Memory & Storage ===
RUFLO_MEMORY_PATH=.swarm/memory.db    # AgentDB location
RUFLO_VECTOR_BACKEND=rvf              # rvf | postgresql
RUFLO_HNSW_M=16                       # HNSW graph degree
RUFLO_HNSW_EF=200                     # HNSW search depth
RUFLO_CACHE_SIZE=1000                 # LRU cache entries

# === Swarm ===
RUFLO_MAX_AGENTS=20                   # max concurrent agents
RUFLO_TOPOLOGY=hierarchical           # hierarchical | mesh | ring | star
RUFLO_CONSENSUS=byzantine             # byzantine | raft | gossip | majority
RUFLO_AUTO_SCALE=true                 # scale up/down on load
RUFLO_DRIFT_DETECTION=true            # alert on goal drift

# === Learning ===
RUFLO_LEARNING_ENABLED=true           # SONA learning loop
RUFLO_LEARNING_RATE=0.15              # adaptation speed
RUFLO_SKILL_DISTILLATION=true         # auto-build skills from tasks

# === Guidance (Long-Horizon Control) ===
GUIDANCE_MAX_TOKENS=200000            # context window target
GUIDANCE_AUTOPILOT=true               # context compaction on
GUIDANCE_CHECKPOINT_INTERVAL=50       # save state every N turns
GUIDANCE_RECOVERY_ENABLED=true        # auto-restore on crash

# === Security ===
RUFLO_SECURITY_LEVEL=high             # low | medium | high | paranoid
RUFLO_PII_SCANNING=true               # scan outputs for PII
RUFLO_INJECTION_DETECTION=true        # block prompt injection

# === MCP Server ===
RUFLO_MCP_PORT=7070                   # MCP server port
RUFLO_MCP_HOST=localhost              # MCP bind address

# === Diagnostics ===
RUFLO_LOG_LEVEL=info                  # debug | info | warn | error
RUFLO_METRICS=true                    # enable metrics collection
RUFLO_TELEMETRY=false                 # opt out of usage telemetry
```

---

## 13. GUIDANCE Plugin (Long-Horizon Control Plane)

The `@claude-flow/guidance` package is the long-horizon governance layer. It prevents agents from losing track of goals over extended sessions:

```bash
# Install guidance plugin
npm install -g @claude-flow/guidance

# Initialize in project
npx guidance init

# Start with autopilot (context never lost)
npx guidance start --autopilot

# Set a goal boundary (agents cannot deviate)
npx guidance goal set "Build the auth module for aigency — TypeScript, JWT, bcrypt, full test coverage"

# Check goal adherence score
npx guidance goal status

# Checkpoint current state
npx guidance checkpoint save --label "auth-complete"

# Restore from checkpoint
npx guidance checkpoint restore --label "auth-complete"
```

**GUIDANCE_* environment variables** (set these in your .env):

```bash
GUIDANCE_GOAL_ENFORCEMENT=strict      # strict | advisory
GUIDANCE_DRIFT_THRESHOLD=0.15         # halt if drift > 15%
GUIDANCE_CONTEXT_WINDOW_TARGET=180000 # token target
GUIDANCE_COMPACTION_STRATEGY=smart    # smart | aggressive | conservative
GUIDANCE_CHECKPOINT_INTERVAL=50       # every 50 turns
GUIDANCE_RECOVERY_MODE=auto           # auto | manual | disabled
```

---

## 14. RVF Storage — RuVector Format

RVF is Ruflo's binary vector storage. It replaces the 18MB sql.js WASM dependency with pure TypeScript:

```bash
# RVF is the default — no configuration needed
# But you can tune it:

# Inspect RVF store
npx ruflo rvf inspect --path .swarm/vectors.rvf

# Compact (defrag) the store
npx ruflo rvf compact --path .swarm/vectors.rvf

# Export to JSON (for inspection/migration)
npx ruflo rvf export --path .swarm/vectors.rvf --output vectors.json

# Migrate from sql.js to RVF (if upgrading from v2)
npx ruflo rvf migrate --from .swarm/memory.db --to .swarm/vectors.rvf
```

**PostgreSQL option** (for multi-node / team scenarios):

```bash
# Install RuVector PostgreSQL extension
npx ruflo rvf postgres install --db aigency_dev

# Configure
RUFLO_VECTOR_BACKEND=postgresql
DATABASE_URL=postgresql://localhost/aigency_dev
```

RuVector PostgreSQL gives you:
- 77+ SQL functions for vector operations
- ~61µs search latency
- 16,400 QPS throughput
- Native pgvector compatibility
- GNN (Graph Neural Network) search

---

## 15. Claims & Work Coordination (Human-Agent Task Management)

Claims is how Ruflo coordinates work assignment between humans and agents — relevant for the agor.live canvas:

```bash
# Claim a task (agent reserves it, others back off)
npx ruflo claims create \
  --task "implement payment module" \
  --assignee coder-01 \
  --priority high

# List all active claims
npx ruflo claims list

# Release a claim (task goes back to pool)
npx ruflo claims release --task-id "claim-abc123"

# Human claim override (you take a task back)
npx ruflo claims override --task-id "claim-abc123" --assignee human
```

This is the primitive that prevents two agents from working on the same thing simultaneously — critical when you have parallel swarms running.

---

## 16. Context Autopilot — Never Lose Context Again

```bash
# Enable (should be on by default after init)
npx ruflo config set contextAutopilot.enabled true

# Set compaction threshold
npx ruflo config set contextAutopilot.threshold 0.85   # compact at 85% full

# View current context utilization
npx ruflo context status

# Force compaction (manual)
npx ruflo context compact --smart

# Restore from last archive
npx ruflo context restore --latest
```

How it works:
1. Monitors token usage in real time
2. At 85% capacity, archives lower-priority context to RVF storage
3. Generates a compressed summary that replaces raw history
4. On session resume, restores the archived context seamlessly
5. You never hit a context wall and lose your working state

---

## 17. System Diagnostics with Auto-Fix

```bash
# Full diagnostic
npx ruflo doctor

# Auto-fix detected issues
npx ruflo doctor --fix

# Deep diagnostic (includes memory, hooks, agents)
npx ruflo doctor --deep

# Expected healthy output:
#   ✓ node: v22.14.0
#   ✓ npm: 10.9.2
#   ✓ git: 2.47.1
#   ✓ config: ruflo.config.json found
#   ✓ daemon: Running (PID 12345)
#   ✓ memory: AgentDB initialized (RVF + HNSW)
#   ✓ mcp: Server running on :7070
#   ✓ agentic-flow: v3.0.0-alpha.1
#   ✓ hooks: 17/17 registered
#   ✓ workers: 12/12 active
```

---

## 18. Spec-Driven Development (Critical for Your Use Case)

This is one of Ruflo's most powerful features for autonomous coding. You write a spec, agents implement it without drifting:

```bash
# Initialize spec-driven mode
npx ruflo spec init --name "aigency-auth-module"

# Write or import your spec
npx ruflo spec set --file docs/auth-spec.md

# Validate spec completeness before implementation
npx ruflo spec validate

# Start implementation (agents read spec, implement, test, doc)
npx ruflo spec implement --parallel --agents 4 --coverage 90

# Track implementation progress against spec
npx ruflo spec status

# Generate coverage report vs spec requirements
npx ruflo spec coverage-report
```

**Connect your existing PRD/specs:**
All of your docs (prd-aigency-dev-platform.md, ux-specs-aigency-dev-platform.md, architecture-aigency-dev-platform.md, backlog-aigency-dev-platform.md) can feed directly into Ruflo's spec system. Each sprint, you point agents at the relevant spec sections and they implement autonomously.

---

## 19. Agent Browser

Ruflo includes a built-in browser automation agent:

```bash
# Launch agent browser
npx ruflo browser start

# Or via MCP (Claude Code calls this directly)
# mcp__claude-flow__browser_navigate
# mcp__claude-flow__browser_click
# mcp__claude-flow__browser_screenshot
# mcp__claude-flow__browser_extract
```

59 browser MCP tools available. Uses accessibility-tree snapshots (93% context reduction vs CSS selectors). Includes trajectory learning — successful browser patterns get stored and reused.

---

## 20. Upgrading

```bash
# Update helpers/statusline (preserves your data and config)
npx ruflo@v3alpha init upgrade

# Update AND install any new skills/agents/commands added in newer versions
npx ruflo@v3alpha init upgrade --add-missing
```

The `--add-missing` flag is important — it adds newly shipped skills and agents without overwriting your customizations.

---

## 21. Does Ruflo Replace Anything from Previous Iterations?

Here is the honest replacement map vs. what you were building before:

| What You Had / Were Building | Ruflo Equivalent | Replace? |
|------------------------------|------------------|---------|
| Manual claude-flow setup (partial) | Full ruflo v3.5 init | YES — ruflo IS claude-flow v3.5 |
| Loki orchestration layer | Ruflo swarm + hooks | PARTIAL — Loki stays for terminal UI, ruflo handles orchestration |
| Maestra workflow engine | Ruflo swarm topologies + skill system | PARTIAL — evaluate overlap in Sprint 3 |
| Superset monitoring | Ruflo metrics + AgentDB dashboards | PARTIAL — Superset better for SQL/BI, ruflo better for agent telemetry |
| Manual agent instantiation in agor.live | Ruflo swarm init via MCP | YES — ruflo handles this |
| Context loss on long sessions | Context Autopilot + GUIDANCE | YES — fully replaced |
| Ad-hoc memory between sessions | AgentDB v3 + RVF | YES — fully replaced |
| Manual task routing | 3-tier router + Q-Learning | YES — fully replaced |
| Per-agent scope isolation | AgentDB project/local/user scopes | YES — fully replaced |
| Custom background workers | 12 daemon workers | MOSTLY — extend with custom hooks if needed |
| Overstory (planned Sprint 4-5) | Ruflo queen/worker hierarchy | VERY SIMILAR — evaluate if Overstory adds value on top |
| ROMA (skip recommendation) | Ruflo full feature set | YES — Ruflo does everything ROMA does in TypeScript |

**The short version:** Ruflo does NOT replace Loki (terminal UI + Loki Mode), agor.live (visual canvas), Superset (BI dashboards), or Letta Code (persistent codebase memory for IDE sessions). Everything else in your previous tool inventory — Ruflo covers it or does it better.

---

## 22. Full Startup Sequence (Run Once to Bootstrap)

This is the exact sequence to get everything running:

```bash
# === Step 1: Prerequisites ===
npm install -g @anthropic-ai/claude-code
node --version  # must be 20+

# === Step 2: Install Ruflo ===
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/claude-flow@main/scripts/install.sh | bash -s -- --full

# === Step 3: Initialize Project ===
cd /your/monorepo/root
npx ruflo@latest init --wizard
# Follow prompts: select hierarchical topology, enable SONA, enable Context Autopilot

# === Step 4: Connect to Claude Code ===
claude mcp add ruflo -- npx -y ruflo@latest mcp start

# === Step 5: Verify ===
npx ruflo doctor
claude mcp list

# === Step 6: Start Daemon (background workers) ===
npx ruflo daemon start
npx ruflo daemon status   # confirm 12 workers active

# === Step 7: Initialize Your First Swarm ===
npx ruflo swarm init --topology hierarchical --max-agents 8 --strategy specialized

# === Step 8: Start Memory System ===
npx ruflo memory store --key "project/aigency" --value "Aigency dev platform monorepo" --namespace meta

# === Step 9: Enable Context Autopilot ===
npx ruflo config set contextAutopilot.enabled true
npx ruflo config set contextAutopilot.threshold 0.85

# === Step 10: Run Health Check ===
npx ruflo doctor --deep
```

After this sequence, your system is running. Every Claude Code session will have access to 175+ ruflo tools, agents spawn on demand, memory persists cross-session, and the daemon keeps 12 workers running in the background.

---

## 23. agor.live Integration Points

Once Ruflo is running, here is how agor.live connects to it:

| agor.live Canvas Region | Ruflo Integration |
|------------------------|------------------|
| Plan region (research/brief) | `mesh` swarm topology — peer agents collaborate on research |
| Design region (architecture/UX) | `star` topology — central architect agent coordinates |
| Build region (implementation) | `hierarchical` topology — queen coder + worker coders in parallel |
| Test region (QA/coverage) | `ring` topology — tester → reviewer → fixer pipeline |
| Deploy region (CI/CD/release) | Ruflo + GitHub swarm tools, deployment hooks |
| Memory panel | AgentDB v3 search UI — `npx ruflo memory search` as API |
| Agent inspector | `npx ruflo agent list` + `swarm status` as API |
| Task claims board | `npx ruflo claims list` as API |
| Routing visualization | Q-Learning router state as API |

The canvas calls these via the MCP server or directly via the ruflo CLI API. Each spatial region in agor.live maps to a swarm topology + agent type combination. Clicking "instantiate agent" in the Build region calls `swarm_init hierarchical` + `agent_spawn coder` through MCP.

---

## 24. The Self-Improving Loop (SONA)

This is the feature you have never fully used in claude-flow. SONA is the self-optimization engine:

```
Successful task completes
  → PostTask hook fires
  → SONA captures: what worked, which agents, how long, which model tier
  → Pattern stored in AgentDB (HNSW-indexed)
  → Learning Loop worker consolidates pattern overnight
  → Next similar task: router recognizes pattern (89% accuracy)
  → Routes to same winning configuration
  → No LLM call needed for routing decision (WASM tier)
  → Latency < 1ms for routing
  → Pattern gets reinforced
  → System gets smarter with every completed task
```

To actively feed SONA:

```bash
# After any successful task — always do this
npx ruflo hooks post-task \
  --task-id "task-xyz" \
  --status completed \
  --success true \
  --store-results true

# Train neural model on accumulated patterns
npx ruflo neural train --episodes 100

# Check what SONA has learned
npx ruflo neural patterns --top 20

# Check routing accuracy
npx ruflo neural status
```

The more you use it, the smarter it gets. This is the compounding return on the whole system.

---

## 25. Quick Reference — Most Used Commands

```bash
# Install & init
npx ruflo@latest init --wizard
claude mcp add ruflo -- npx -y ruflo@latest mcp start
npx ruflo doctor --fix

# Daemon
npx ruflo daemon start
npx ruflo daemon status

# Swarms
npx ruflo swarm init --topology hierarchical --max-agents 8
npx ruflo agent spawn -t coder --name coder-01
npx ruflo swarm status

# Task routing
npx ruflo hooks route --task "your task description here"
npx ruflo hooks pre-task --description "..." --priority high
npx ruflo hooks post-task --task-id "..." --success true --store-results true

# Memory
npx ruflo memory store --key "key" --value "value" --namespace ns
npx ruflo memory search --query "what to find" --limit 5

# Skills
npx ruflo skills list
npx ruflo skills create --from-task "task-id" --name "my-skill"

# Context
npx ruflo context status
npx ruflo context compact --smart

# Learning
npx ruflo neural train
npx ruflo neural patterns --top 10
npx ruflo neural status

# Diagnostics
npx ruflo doctor --deep --fix
npx ruflo swarm status
npx ruflo daemon status
```

---

## Sources

- https://github.com/ruvnet/ruflo (main repo, v3.5.7)
- https://github.com/ruvnet/ruflo/wiki/hooks (hooks reference)
- https://github.com/ruvnet/ruflo/issues/1240 (v3.5 release spec)
- https://github.com/ruvnet/ruflo/issues/1291 (environment variables)
- Local file: code/ruflow-repo.md (full README scrape)
- CLAUDE.md, AGENTS.md from repo
