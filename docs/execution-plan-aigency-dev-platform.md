# Execution Plan — Aigency Developer Platform
## SimpleLLMRouter v2 + @aigency/forge-quality + Meta Code Squad Bootstrap

**Version:** 1.0 | **Date:** 2026-03-05 | **Owner:** Antonio Reid
**Horizon:** 8 weeks to Meta Code Squad operational
**Status:** READY TO START

---

## Your Current Agent Arsenal

Before the plan — a clear-eyed view of what you have and how to use each tool.

### Tier 1 — Primary Workhorses (use first, always)

| Tool | Quota | Best For | Notes |
|------|-------|----------|-------|
| **Claude Code (hive-mind)** | 120 req / 5 hrs (z.ai) | Architecture, complex multi-file, reasoning | Most configured. Use claude-flow for parallel worktrees. Primary executor. |
| **Gemini CLI** | Large pool, abundant skills | Long context, research synthesis, docs, tests | Globally installed skills in abundance. Best for reading the whole codebase and writing docs + tests. |
| **Google Antigravity** | Good quota | High-complexity reasoning, alternative to Claude | Well configured. Use as Claude Code overflow + for architecture reviews. |

### Tier 2 — High Value Alternates (rotate in when Tier 1 quota burns)

| Tool | Quota | Best For | Notes |
|------|-------|----------|-------|
| **Kimi Code CLI** | 3x quota (exact unknown) | Long-context codebase analysis, migrations | 128k+ context. Best for "read this whole app and refactor" tasks. |
| **Qwen Code** | Moderate | Code generation, good quality | Less configured but solid. Rotate in for isolated story implementation. |
| **Roo Code** | Free tier + quota | General coding | Free models available. Good overflow. |
| **Kilo Code** | Free tier + quota | General coding | Same as Roo — free model fallback. |

### Tier 3 — Overflow / Free Runners (burn quota freely, always available)

| Tool | Best For |
|------|----------|
| **iFlow CLI** | Free. Use for any simple scaffolding, boilerplate, or documentation tasks. |
| **Rovo Dev (Acli)** | Atlassian-native. Use for generating Jira/Confluence artifacts if needed. |
| **Augment** | Context-aware background suggestions. Use alongside primary agents. |

### The Core Rule
> **Never use one tool for an 8-hour session. Rotate every 2–3 hours or after each story.**
> Claude Code hive-mind for the hard stuff. Gemini for the wide stuff. Kimi when context is huge.
> Antigravity as your second brain on architecture decisions. Free tools for boilerplate.

---

## Master Timeline

```
Week 1: Foundation + Router Critical Path
Week 2: Router Intelligence + forge-quality Core
Week 3: Router TUI + forge-quality Hooks Complete
Week 4: Integration + End-to-End Tests
Week 5: Meta Code Squad Bootstrap (Sugar + agor.live)
Week 6: First Autonomous Build Run
Week 7: Hardening + Open Source Prep
Week 8: Router v1.0 + forge-quality v1.0 Published
```

---

## PHASE 0 — Setup (Day 1, ~3 hours)
**Do this before writing a single line of application code.**

### Step 0.1 — Monorepo Scaffold (30 min)
**Tool:** Any — this is pure scaffolding. Use iFlow CLI or Claude Code.

```bash
mkdir aigency && cd aigency
pnpm init
pnpm add -D turbo
```

Create these files manually or with `forge init` once it exists:
- `turbo.json` — define tasks: build, test, lint, typecheck, check, dev
- `pnpm-workspace.yaml` — `packages: ['apps/*', 'packages/*']`
- `.gitignore` — node_modules, dist, .turbo, .env*, *.db, vector_store/
- `packages/tsconfig/base.json` — strict TS config shared across all apps

**Verify:** `pnpm install` completes. `turbo.json` is valid.

### Step 0.2 — Move Existing Router Code In (30 min)
**Tool:** Manual file move + Claude Code to fix any import paths.

```bash
mkdir -p apps/simplellmrouter/src
# Move existing simplellmrouter source into apps/simplellmrouter/src/
```

Update `apps/simplellmrouter/package.json` to extend `@aigency/tsconfig/base`.

**Verify:** `pnpm --filter simplellmrouter test` — all existing tests pass.

### Step 0.3 — Scaffold forge-quality Package (30 min)
**Tool:** iFlow CLI or Claude Code.

```bash
mkdir -p packages/forge-quality/src/{hooks,linters,scanners,init,commit,pr}
mkdir -p packages/forge-quality/templates
```

Create `packages/forge-quality/package.json`:
- name: `@aigency/forge-quality`
- bin: `{ "forge": "./dist/cli.js" }`
- exports: `./dist/index.js`

**Verify:** `pnpm install` still works. Package appears in workspace.

### Step 0.4 — Install forge-quality on the Monorepo Itself (15 min)
**Tool:** Manual + forge CLI (dogfooding from day 1).

```bash
cd aigency
pnpm add -D @aigency/forge-quality  # links local package via workspace
npx forge init --skip-github  # will be sparse until forge is built, but sets up git hooks
```

**Goal:** Every commit to the monorepo runs through the hooks we're building.
Start with minimal hooks (commitlint only) and add gitleaks, semgrep later as we build them.

### Step 0.5 — Configure Your Agent Environment (1 hour)
**This is the most important setup step. Don't skip it.**

**Claude Code hive-mind:**
```bash
# In aigency/ root — this gives every claude-flow worktree the full codebase context
echo "You are working inside the aigency/ Turborepo. The two primary packages are:
1. apps/simplellmrouter — LLM routing gateway (TypeScript/Node)
2. packages/forge-quality — commit quality toolchain (@aigency/forge-quality)
3. apps/router-tui — Textual TUI for the router (Python)
Always read the relevant .planning/ docs before starting any task.
Never make architecture decisions without consulting docs/architecture-aigency-dev-platform.md." > CLAUDE.md
```

**Gemini CLI:**
```bash
# Add the full spec as context
gemini context add docs/architecture-aigency-dev-platform.md
gemini context add docs/backlog-aigency-dev-platform.md
```

**Kimi Code:**
```bash
# Kimi's strength is reading the whole codebase — let it index everything
kimi index .
```

**Sugar (install now, configure in Phase 5):**
```bash
pipx install sugarai
sugar init
```

---

## PHASE 1 — Router Critical Path (Days 2–5, Week 1)
**Goal:** Router is production-ready with working quota, circuit breaker, and L1/L2 cache.
**Primary tool:** Claude Code hive-mind | **Overflow:** Antigravity, Kimi

### Step 1.1 — Wire QuotaTracker (STORY 2.1) — 4 hours
**Tool:** Claude Code
**Prompt strategy:** Single focused session. Give it STORY 2.1 acceptance criteria + `quota/tracker.ts`.

What to build:
- `QuotaTracker.record(provider, tokens)` — write to SQLite after every successful response
- `QuotaTracker.isEligible(provider)` — check before routing; returns `{ eligible: bool, score: number }`
- 80% quota → deprioritize (score penalty). 95% → exclude entirely.
- Daily reset at 00:00 UTC (configurable).
- Wire into `router/index.ts` — check before dispatch, record after response.

**Test file:** `tests/quota/tracker.test.ts` — must cover all 6 ACs.

**Commit:** `feat(router): wire QuotaTracker into request handler`

### Step 1.2 — Complete Circuit Breaker (STORY 2.2) — 3 hours
**Tool:** Claude Code (continue session or new session with tracker as context)

What to build:
- State machine: HEALTHY → OPEN → HALF_OPEN → HEALTHY/OPEN
- 3 failures in 60s → OPEN. 120s cooldown → HALF_OPEN. 1 success → HEALTHY.
- Persist circuit state to SQLite.
- Wire into `router/index.ts` — skip provider if circuit is OPEN.

**Test file:** `tests/quota/circuit-breaker.test.ts` — must cover all state transitions.

**Commit:** `feat(router): complete circuit breaker with HALF_OPEN recovery`

### Step 1.3 — Classifier Test Coverage (STORY 2.3) — 6 hours
**Tool:** Gemini CLI (great for generating test suites — long context, reads all existing code)
**Prompt:** "Read classifier.ts. Write classifier.test.ts with 2+ test cases per dimension,
5+ test cases per complexity tier, and edge cases for empty, very long, and non-English prompts.
Target 90% line and function coverage."

**Verify:** `pnpm --filter simplellmrouter test --coverage` — classifier.ts > 90%

**Commit:** `test(router): comprehensive classifier test coverage (28+ cases)`

### Step 1.4 — L1 In-Memory Cache (STORY 3.1) — 4 hours
**Tool:** Claude Code

What to build:
- `cache/l1-memory.ts` — `node-lru-cache`, max 1000 entries, 5min TTL
- Cache key: `SHA256(modelTier + normalizedPrompt)`
- Wire into `cache/manager.ts` — check L1 first on every request

**Test file:** `tests/cache/l1.test.ts` — set, get hit, get miss, TTL, LRU eviction.

**Commit:** `feat(router): L1 in-memory LRU cache`

### Step 1.5 — L2 Disk Cache (STORY 3.2) — 4 hours
**Tool:** Claude Code (or Kimi if session limit approaching — same complexity)

What to build:
- `cache/l2-disk.ts` — `better-sqlite3`, `cache_entries` table
- 24h TTL. Cleanup on startup + every 1h. 10GB max.
- Wire into `cache/manager.ts` — check L2 after L1 miss.

**Test file:** `tests/cache/l2.test.ts`

**Commit:** `feat(router): L2 SQLite disk cache with TTL and eviction`

### Step 1.6 — Cache Manager Orchestrator (STORY 3.4) — 3 hours
**Tool:** Claude Code

What to build:
- `cache/manager.ts` — unified `get(key, prompt)`, `set(key, prompt, response)`, `flush(layer?)`
- L1 → L2 → (L3 later) waterfall
- Async non-blocking writes
- Stats: hit counts per layer, miss count

**Commit:** `feat(router): unified cache manager (L1→L2 waterfall)`

**End of Week 1 checkpoint:**
- [ ] Router starts, accepts requests, routes correctly
- [ ] QuotaTracker wired and persisting
- [ ] Circuit breaker operational
- [ ] L1 + L2 cache live
- [ ] All tests green: `pnpm --filter simplellmrouter test`

---

## PHASE 2 — Router Intelligence + forge-quality Core (Days 6–10, Week 2)
**Primary tool:** Claude Code (complex logic) + Gemini CLI (tests + docs) | **Overflow:** Antigravity, Kimi

### Step 2.1 — L3 Semantic Cache (STORY 3.3) — 8 hours (1 full day)
**Tool:** Kimi Code CLI (long context — needs to understand L1/L2 implementation before writing L3)

This is the hardest cache story. Give Kimi the full `cache/` directory as context.

What to build:
- `cache/l3-semantic.ts` — `hnswlib-node` for vector index, `@xenova/transformers` for MiniLM embeddings
- No external API calls for embeddings — all local
- Cosine similarity threshold: 0.92
- HNSW index persists to disk, loads on startup
- Wire into `cache/manager.ts` — check L3 after L2 miss
- L3 loads async — L1/L2 serve requests during cold start

**This will eat quota. Budget a full day. Use Kimi's 3x quota here.**

**Test file:** `tests/cache/l3.test.ts` — semantic hit (paraphrase), semantic miss, threshold edge cases

**Commit:** `feat(router): L3 semantic cache with local MiniLM embeddings`

### Step 2.2 — Semantic Intent Detector (STORY 4.1) — 1 day
**Tool:** Claude Code (reasoning-heavy — needs to design the embedding space well)

What to build:
- `router/intent-detector.ts` — 9 intent classes with local embeddings
- Confidence score 0.0–1.0
- p95 latency < 100ms on warm model
- Each intent maps to: preferred provider tier + routing strategy + OptiLLM flag

**Intent classes:** CODE_REVIEW, BUG_FIX, ARCHITECTURE, DOCUMENTATION,
SIMPLE_CHAT, REASONING, TOOL_CALL, CREATIVE, UNKNOWN

**Commit:** `feat(router): semantic intent detector with 9 intent classes`

### Step 2.3 — Cascade Fallback Routing (STORY 4.2) — 6 hours
**Tool:** Claude Code

What to build:
- `router/cascade.ts` — try top candidate → score quality → escalate if < 0.75
- Max 2 escalations per request
- Cascade chain configurable in `providers.json`
- Log all cascade events with reason

**Commit:** `feat(router): cascade fallback routing with quality threshold`

### Step 2.4 — forge-quality: ESLint + Prettier Config (STORY 5.1 equivalent) — 4 hours
**Tool:** iFlow CLI or Qwen Code (straightforward config generation)

What to build in `packages/forge-quality/src/linters/`:
- `eslint-config.ts` — base config + TypeScript rules + React optional
- `prettier-config.ts` — standard Prettier config
- `biome-config.ts` — Biome alternative (faster)
- `install.ts` — installer that detects project type and applies correct config

**Commit:** `feat(forge-quality): ESLint + Prettier + Biome config packages`

### Step 2.5 — forge-quality: commitlint + commitizen (STORY 5.2 equivalent) — 3 hours
**Tool:** Qwen Code or Roo Code

What to build:
- `src/commit/commitlint-config.ts` — conventional commits config
- `src/commit/commitizen-config.ts` — interactive commit prompt config
- `src/commit/index.ts` — install function that wires both into the target project

**Commit:** `feat(forge-quality): commitlint + commitizen conventional commits`

### Step 2.6 — forge-quality: Secret Scanner (STORY 5.3 equivalent) — 4 hours
**Tool:** Claude Code (security logic — use the most careful tool here)

What to build:
- `src/scanners/secrets.ts` — wraps gitleaks + detect-secrets
- Hard block on any detected secret — never auto-fixes, never skips
- Clear error output: file, line, matched pattern type (not the secret itself)
- Exit code 1 on any detection

**This is non-negotiable. Use Claude Code for security-critical code.**

**Commit:** `feat(forge-quality): secret scanner (gitleaks + detect-secrets, hard block)`

---

## PHASE 3 — Router TUI + forge-quality Hooks (Days 11–15, Week 3)
**Primary tool:** Gemini CLI (Python TUI) + Claude Code (hook orchestration) | **Overflow:** Kimi for long context

### Step 3.1 — Router TUI: Core Layout (STORY 6.1) — 1 day
**Tool:** Gemini CLI (Python/Textual is in its wheelhouse + abundant skills)

What to build in `apps/router-tui/`:
- `main.py` — Textual App entry point
- `layout.py` — 4-panel layout: Provider Health (top-left), Request Stream (top-right),
  Performance Metrics (bottom-left), Cache Analytics (bottom-right)
- `theme.py` — color tokens from UX spec (exact hex values)
- `sse_client.py` — SSE connection to router's `/metrics/stream` endpoint

**Refer to:** `docs/ux-specs-aigency-dev-platform.md` Section 4 for exact wireframes and color values.

**Commit:** `feat(router-tui): core 4-panel layout with SSE client`

### Step 3.2 — Router TUI: Provider Health Panel (STORY 6.2) — 4 hours
**Tool:** Gemini CLI

What to build:
- `widgets/provider_health.py` — DataTable showing all providers
- Columns: Provider, Status (colored), Latency p50/p95, TPM, Quota%, Circuit
- Real-time updates from SSE stream
- Color coding: green=healthy, yellow=warning, red=open circuit

**Commit:** `feat(router-tui): provider health panel with real-time updates`

### Step 3.3 — Router TUI: Request Stream Panel (STORY 6.3) — 4 hours
**Tool:** Gemini CLI

What to build:
- `widgets/request_stream.py` — scrolling log of last 200 requests
- Each row: timestamp, tier, provider, latency, tokens, cache hit indicator
- Color coding: green=cache hit, blue=direct, yellow=cascade, red=error
- Filter bar: by tier, provider, cache status, time range

**Commit:** `feat(router-tui): request stream panel with filtering`

### Step 3.4 — forge-quality: lefthook Hook Orchestration (STORY 5.4 equivalent) — 1 day
**Tool:** Claude Code (this is the core of forge-quality — needs careful implementation)

What to build:
- `src/hooks/orchestrator.ts` — reads `lefthook.yml`, installs hooks via lefthook binary
- `src/hooks/pre-commit.ts` — parallel execution groups (see architecture doc)
  - Group 1 (parallel): format, lint
  - Group 2 (sequential): typecheck
  - Group 3 (sequential): secret scan (hard block)
  - Group 4 (parallel): staged-only tests
  - Group 5 (sequential): coverage delta check
- `src/hooks/commit-msg.ts` — commitlint validation
- `src/hooks/pre-push.ts` — full typecheck + full test suite + audit + PR size check
- `src/hooks/post-commit.ts` — non-blocking summary output

**Refer to:** `docs/architecture-aigency-dev-platform.md` Section 5 for exact hook config YAML.

**Commit:** `feat(forge-quality): lefthook hook orchestration (pre-commit, commit-msg, pre-push)`

### Step 3.5 — forge-quality: `forge init` Command (STORY 5.5 equivalent) — 1 day
**Tool:** Claude Code

What to build:
- `src/init/index.ts` — 10-step init sequence (per architecture doc Section 5.2)
- `src/init/detector.ts` — detect: monorepo vs standalone, package manager, existing tools
- `src/init/github.ts` — `gh repo create` wrapper with branch protection rules
- `src/cli.ts` — top-level CLI entry point (`forge init`, `forge commit`, `forge pr`, `forge status`)

**The init output must match the UX spec exactly** — 13 checkmark steps, spinner on each.

**Commit:** `feat(forge-quality): forge init command with 10-step sequence`

---

## PHASE 4 — Integration + End-to-End Tests (Days 16–20, Week 4)
**Primary tool:** Gemini CLI (test writing) + Claude Code (debugging) | Kimi for full-codebase review

### Step 4.1 — Router: 5 Routing Strategies (STORY 4.3) — 1 day
**Tool:** Claude Code

Build all 5 in `router/strategies/`:
- `quality.ts` — maximize response quality score
- `cost.ts` — minimize token cost, stay in free tiers
- `latency.ts` — minimize time-to-first-token
- `balanced.ts` — weighted composite: 40% quality, 30% cost, 30% latency
- `shuffle.ts` — random weighted selection for load distribution

Each strategy implements `RoutingStrategy` interface: `score(providers[], request) → ProviderScore[]`

**Commit:** `feat(router): all 5 routing strategies implemented`

### Step 4.2 — Router: SSE Metrics Stream (STORY 4.4) — 4 hours
**Tool:** Claude Code

What to build:
- `metrics/sse-stream.ts` — SSE endpoint at `/metrics/stream`
- Events: `provider_update`, `request_complete`, `cache_stats`, `quota_update`, `circuit_event`
- Event payload schemas per architecture doc Section 8
- Heartbeat every 30s to keep connections alive

**Commit:** `feat(router): SSE metrics stream with 5 event types`

### Step 4.3 — forge-quality: SAST Scanner (STORY 5.6 equivalent) — 4 hours
**Tool:** Claude Code (security — use careful tool)

What to build:
- `src/scanners/sast.ts` — wraps `semgrep` with `p/typescript` + `p/owasp-top-ten` rulesets
- Configurable severity threshold (default: error-level findings block commit)
- Warning-level findings shown but non-blocking
- Output: file, line, rule ID, severity, message

**Commit:** `feat(forge-quality): SAST scanner via semgrep (OWASP top 10)`

### Step 4.4 — forge-quality: `forge pr` Command — 4 hours
**Tool:** Qwen Code (good at generating structured output from commit history)

What to build:
- `src/pr/index.ts` — auto-generate PR description from git log
- Sections: What changed, Why, Coverage delta, Test count, Checklist
- Block if > 30 files changed (warn at 16)
- Output markdown suitable for GitHub PR body

**Commit:** `feat(forge-quality): forge pr command with auto-generated description`

### Step 4.5 — End-to-End Integration Tests — 1 day
**Tool:** Gemini CLI (write tests) + Claude Code (fix failures)

What to write:
- Router E2E: start router → send 20 requests → verify routing decisions → verify cache hits
- Router E2E: exhaust one provider's quota → verify fallback routing
- Router E2E: force provider failure × 3 → verify circuit breaker opens
- forge-quality E2E: run `forge init` in temp dir → verify all hooks installed → make test commits
- forge-quality E2E: attempt commit with fake secret → verify hard block with correct error output

**Commit:** `test: comprehensive E2E test suite for router and forge-quality`

### Step 4.6 — Full Codebase Review (Kimi) — 2 hours
**Tool:** Kimi Code CLI (128k context — can read everything at once)

**Prompt:** "Read the entire aigency/ monorepo. Identify:
1. Any incomplete implementations (TODO, stub, missing error handling)
2. Any security issues not caught by automated scanners
3. Performance bottlenecks in the critical request path
4. Test coverage gaps not already covered by STORY 2.3
5. Inconsistencies between implementation and the architecture doc"

Generate a findings report. Fix P0/P1 findings before Phase 5.

**Commit (for any fixes):** `fix(router|forge-quality): [findings from Kimi codebase review]`

---

## PHASE 5 — Meta Code Squad Bootstrap (Days 21–25, Week 5)
**Primary tool:** Claude Code hive-mind (multi-agent coordination) + Sugar
**This is where the Meta Code Squad starts building Aigency.**

### Step 5.1 — Install and Configure Sugar (2 hours)
**Tool:** Manual + iFlow CLI for config generation

```bash
pipx install sugarai
cd aigency
sugar init
```

Create `.sugar/config.yaml`:
```yaml
sugar:
  dry_run: false
  loop_interval: 300
  max_concurrent_work: 3

claude:
  enable_agents: true

discovery:
  github:
    enabled: true
    repo: "aigency/aigency"
  error_logs:
    enabled: true
    paths: ["logs/errors/"]
  code_quality:
    enabled: true
    source_dirs: ["apps", "packages"]
    excluded_dirs: [".next", "node_modules", "dist", ".turbo"]
```

Wire Sugar to MCP in Claude Code:
```bash
claude mcp add sugar -- sugar mcp memory
```

### Step 5.2 — Load Backlog into Sugar (1 hour)
**Tool:** Manual + Claude Code to generate the Sugar task commands

Convert the backlog doc into Sugar tasks. For each P0/P1 story not yet complete:
```bash
sugar add "STORY X.Y — [title]" \
  --context "docs/backlog-aigency-dev-platform.md" \
  --ralph \
  --completion-criteria "all acceptance criteria in STORY X.Y checked"
```

Use Claude Code to generate all the `sugar add` commands from the backlog in one shot:
**Prompt:** "Read docs/backlog-aigency-dev-platform.md. Generate sugar add commands for every
P0 and P1 story that isn't already completed. Include --ralph for any story estimated > 4h."

### Step 5.3 — Configure agor.live (2 hours)
**Tool:** Manual setup via agor.live web UI

1. Create a new board: "Aigency Dev Platform"
2. Create zones: Research → Planning → Development → Review → Testing → Deploy → Done
3. Create a worktree per active story (start with the 3 highest priority incomplete stories)
4. Wire zone automations:
   - Development Zone → start Sugar task → route to Claude Code
   - Review Zone → Guardian agent review → approve/reject
   - Testing Zone → run test suite → report results

### Step 5.4 — First Autonomous Run (4 hours)
**Tool:** Sugar + Claude Code hive-mind

Start Sugar with 3 tasks in queue:
```bash
sugar run --dry-run  # verify task queue looks correct
sugar run           # start autonomous execution
```

Watch the first 2–3 tasks complete. Verify:
- [ ] Sugar picks task, routes to Claude Code, implementation starts
- [ ] Tests run automatically after implementation
- [ ] Ralph loop activates on test failure (iterate until green)
- [ ] Completed task committed with correct conventional commit message
- [ ] forge-quality hooks run on every commit, blocking bad ones

This is the proof of concept. Once you see Sugar + Claude Code completing a story end-to-end
without you touching the keyboard, the Meta Code Squad is operational.

### Step 5.5 — Configure Squad Agent Routing (1 day)
**Tool:** Claude Code

Create `.sugar/agent-routing.yaml` that maps task types to execution agents:
```yaml
routing:
  feature: claude-code
  bug_fix: claude-code --ralph
  test: gemini-cli
  refactor: kimi-code
  documentation: gemini-cli
  research: gemini-cli
  infrastructure: claude-code
  security_audit: claude-code
  boilerplate: iflow-cli
```

Install routing config:
```bash
sugar config apply .sugar/agent-routing.yaml
```

**Result:** Sugar now routes different task types to different agents automatically.
You don't pick the agent — Sugar does.

---

## PHASE 6 — First Meta Code Squad Build Run (Days 26–30, Week 6)
**The Meta Code Squad builds the next feature of the platform, autonomously.**

### Step 6.1 — Choose the First Autonomous Build Target
Pick one of these (in order of risk, low to high):
1. **Router: History-Augmented Routing (STORY 4.4)** — isolated feature, good test coverage
2. **forge-quality: Dependency Audit Step** — isolated package, clear success criteria
3. **Router TUI: Performance Metrics Panel** — isolated widget, visual verification easy

Recommended: start with Option 1 (Router history routing). It's the most self-contained.

### Step 6.2 — Write the PRP for the Target Feature (1 hour)
**Tool:** Nexus (your aigency-agile-squad-newton-nexus-chen agent) + Meridian

Create `.planning/execution/prp-history-routing.md`:
- Background: what exists, what's missing
- Goal: exactly what "done" looks like
- Tests: what test cases prove it works
- Constraints: don't break existing routing strategies

Feed this to Sugar:
```bash
sugar add "Router: History-Augmented Routing" \
  --context ".planning/execution/prp-history-routing.md" \
  --ralph \
  --completion-criteria "all tests in STORY 4.4 pass with 90%+ coverage"
```

### Step 6.3 — Let It Run (hands off)
```bash
sugar run
```

Walk away. Come back when Sugar notifies you (Telegram via NEXUS or terminal notification).

**If it completes:** Review the PR. If it passes Guardian review, merge it.
**If it stalls:** Check the Sugar log. Unblock the one thing blocking it. Let it continue.

This is the inflection point. After this, the Meta Code Squad can build the platform itself.

---

## PHASE 7 — Hardening + Open Source Prep (Days 31–35, Week 7)

### Step 7.1 — Security Hardening
- Run full semgrep + gitleaks audit on both packages
- Fix all findings (Claude Code)
- Add rate limiting to the router HTTP server
- Add input sanitization for all prompt text
- Audit SQLite schema for injection vectors

### Step 7.2 — Performance Benchmarking
- Benchmark: router request latency (target: p50 < 100ms, p95 < 500ms)
- Benchmark: L1 cache hit (target: < 1ms)
- Benchmark: L3 semantic cache (target: < 50ms)
- Benchmark: `forge init` total time (target: < 30 seconds)
- Fix any benchmarks that miss targets

### Step 7.3 — Documentation Pass
**Tool:** Gemini CLI (long context, excellent at documentation)

Generate for each package:
- `README.md` — install, configure, use (with exact command examples)
- `CONTRIBUTING.md` — how to contribute (uses forge-quality for all contributions)
- `CHANGELOG.md` — from commit history
- API reference from JSDoc/TSDoc comments

### Step 7.4 — Open Source Checklist
- [ ] MIT LICENSE added
- [ ] `providers.example.json` committed (no real API keys)
- [ ] `.env.example` committed
- [ ] All secrets removed from history (gitleaks confirms clean)
- [ ] GitHub Actions CI workflow added: test + lint + typecheck on every PR
- [ ] Both packages publishable to npm: `pnpm publish --dry-run` succeeds

---

## PHASE 8 — Publish (Days 36–40, Week 8)

### Step 8.1 — Cut Release Candidates
```bash
pnpm changeset       # describe changes
pnpm changeset version  # bump versions
pnpm --filter simplellmrouter build
pnpm --filter @aigency/forge-quality build
```

Versions:
- `simplellmrouter`: `2.0.0-rc.1`
- `@aigency/forge-quality`: `1.0.0-rc.1`

### Step 8.2 — Publish
```bash
pnpm --filter simplellmrouter publish --access public
pnpm --filter @aigency/forge-quality publish --access public
```

### Step 8.3 — Create GitHub Releases
- Tag: `simplellmrouter@2.0.0`
- Tag: `@aigency/forge-quality@1.0.0`
- Release notes: generated from CHANGELOG by Gemini CLI

---

## Daily Working Pattern

### How to start each working session:
1. **Check Sugar queue:** `sugar status` — what's in progress, what's next
2. **Check circuit breakers:** `pnpm --filter simplellmrouter dev` → TUI shows provider health
3. **Pick the next story** from the backlog priority order (P0 → P1 → P2)
4. **Choose your tool** based on the story type (see arsenal table above)
5. **Work the story** — write code, run tests, commit
6. **Rotate tools** every 2–3 hours if quota is burning

### How to avoid wasting quota:
- **Gemini CLI for tests.** Don't use Claude Code to write boilerplate tests.
- **iFlow CLI for scaffolding.** Don't use Claude Code to create empty files and directories.
- **Kimi for large refactors.** When you need to read the whole codebase first, use Kimi.
- **Claude Code for hard logic only.** Routing algorithms, security code, complex state machines.
- **Pre-load context.** Feed the architecture doc and relevant source files at session start.
  A 200-token context investment saves 2000 tokens of back-and-forth.

### When Claude Code (z.ai) hits 120 req limit:
1. Switch to Antigravity immediately — same capability, different quota pool
2. If Antigravity is also low → Kimi Code (3x quota)
3. If Kimi is low → Qwen Code or Roo Code
4. If everything is low → use iFlow/free tools for scaffolding, take a 30-min break

**The 5-hour window resets. Don't stop building.**

---

## Quick Reference: Story → Tool Assignment

| Phase | Story | Recommended Tool | Why |
|-------|-------|-----------------|-----|
| 1 | QuotaTracker | Claude Code | State machine logic |
| 1 | Circuit Breaker | Claude Code | State machine logic |
| 1 | Classifier Tests | Gemini CLI | Test generation at scale |
| 1 | L1 Cache | Claude Code | Performance-critical |
| 1 | L2 Cache | Claude Code / Kimi | SQLite integration |
| 2 | L3 Semantic Cache | Kimi Code CLI | Huge context needed |
| 2 | Intent Detector | Claude Code | Embedding + reasoning |
| 2 | Cascade Routing | Claude Code | Complex fallback logic |
| 2 | ESLint Config | iFlow / Qwen | Config generation |
| 2 | commitlint | Qwen / Roo | Straightforward config |
| 2 | Secret Scanner | Claude Code | Security — use best tool |
| 3 | TUI Layout | Gemini CLI | Python + Textual |
| 3 | TUI Panels | Gemini CLI | Python + Textual |
| 3 | Hook Orchestration | Claude Code | Core forge-quality logic |
| 3 | forge init | Claude Code | UX precision required |
| 4 | 5 Strategies | Claude Code | Routing algorithm logic |
| 4 | SSE Stream | Claude Code | Real-time event logic |
| 4 | SAST Scanner | Claude Code | Security |
| 4 | forge pr | Qwen Code | Structured output gen |
| 4 | E2E Tests | Gemini CLI | Test generation at scale |
| 4 | Codebase Review | Kimi Code CLI | Full context needed |
| 5+ | All autonomous | Sugar routes | Squad picks the tool |

---

## Success Criteria

**Week 4 (Router + forge-quality complete):**
- [ ] `curl localhost:8080/v1/chat/completions` routes correctly to the right provider
- [ ] L1/L2/L3 cache all returning hits for repeated + similar prompts
- [ ] `forge init` in a fresh directory installs all hooks in < 30 seconds
- [ ] A commit with a fake AWS key is hard-blocked with a clear error message
- [ ] `pnpm turbo test` — all tests green, > 85% coverage across both packages

**Week 6 (Meta Code Squad operational):**
- [ ] Sugar completes at least one full story end-to-end without human intervention
- [ ] `sugar run` processes 3 concurrent tasks
- [ ] Guardian agent reviews and approves/rejects PRs automatically
- [ ] Every autonomous commit passes forge-quality hooks

**Week 8 (Published):**
- [ ] `npm install @aigency/forge-quality` works
- [ ] SimpleLLMRouter v2 has > 50 GitHub stars within 30 days
- [ ] Zero known security vulnerabilities at publish time
- [ ] Meta Code Squad is building Aigency platform features autonomously
