# Product Backlog
## SimpleLLMRouter v2 + @aigency/forge-quality

**Version:** 1.0 | **Date:** 2026-03-05 | **Owner:** Antonio Reid
**Format:** Epic → Story → Acceptance Criteria | **Priority:** P0 (blocker) → P3 (nice-to-have)

---

## Priority Legend

| Priority | Meaning | Must ship in |
|----------|---------|-------------|
| P0 | Blocker — nothing works without this | Phase 1 |
| P1 | Core feature — product is incomplete without this | Phase 1-2 |
| P2 | High value — significantly improves the product | Phase 2-3 |
| P3 | Enhancement — polish and open source readiness | Phase 3-4 |

---

## EPIC 1 — Monorepo Foundation
**Goal:** The aigency/ Turborepo exists, both packages scaffold, forge-quality dogfoods itself.
**Phase:** 1 | **Estimate:** 3 days

---

### STORY 1.1 — Initialize Turborepo (P0)
**As a** developer,
**I want** the aigency/ monorepo scaffolded with Turborepo, pnpm workspaces, and shared configs,
**so that** I can develop both packages in a single repo with shared tooling.

**Acceptance Criteria:**
- [ ] `aigency/` directory exists with `turbo.json`, `pnpm-workspace.yaml`, root `package.json`
- [ ] `apps/simplellmrouter/` exists as a Turborepo app with its own `package.json`
- [ ] `apps/router-tui/` exists with `pyproject.toml`
- [ ] `packages/forge-quality/` exists with `package.json` named `@aigency/forge-quality`
- [ ] `packages/tsconfig/` exists with `base.json`
- [ ] `pnpm install` completes without errors
- [ ] `pnpm turbo build` runs successfully across all packages
- [ ] `turbo.json` defines tasks: `build`, `test`, `lint`, `typecheck`, `check`, `dev`

**Estimate:** 4h

---

### STORY 1.2 — Migrate SimpleLLMRouter into Monorepo (P0)
**As a** developer,
**I want** the existing simplellmrouter codebase moved into `apps/simplellmrouter/`,
**so that** it builds and tests correctly inside the Turborepo.

**Acceptance Criteria:**
- [ ] All existing source files moved to `apps/simplellmrouter/src/`
- [ ] All existing tests pass via `pnpm turbo test --filter=simplellmrouter`
- [ ] `tsconfig.json` extends `@aigency/tsconfig/base`
- [ ] No hardcoded paths that break in monorepo context
- [ ] `pnpm --filter simplellmrouter dev` starts the router on port 8080

**Estimate:** 4h

---

### STORY 1.3 — forge-quality Dogfoods Itself (P1)
**As a** developer,
**I want** `@aigency/forge-quality` to use its own hooks and configs,
**so that** every commit to the package is validated by the very tool being built.

**Acceptance Criteria:**
- [ ] `forge-quality` has `.eslintrc.json` extending its own config
- [ ] `forge-quality` has `lefthook.yml` installed and active
- [ ] A commit with a non-conventional message to `forge-quality` is blocked
- [ ] A commit with a staged secret to `forge-quality` is blocked
- [ ] `pnpm turbo check --filter=@aigency/forge-quality` passes

**Estimate:** 3h

---

## EPIC 2 — Router: Critical Bug Fixes
**Goal:** The router is production-ready: QuotaTracker wired, circuit breaker complete, tests green.
**Phase:** 1 | **Estimate:** 2 days

---

### STORY 2.1 — Wire QuotaTracker into Request Handler (P0)
**As a** system operator,
**I want** the QuotaTracker to record every token usage and gate routing when limits are reached,
**so that** the router never exceeds provider quotas.

**Acceptance Criteria:**
- [ ] `QuotaTracker.record(provider, tokens)` is called after every successful provider response
- [ ] `QuotaTracker.isEligible(provider)` is checked before dispatching to any provider
- [ ] Quota state persists to SQLite across restarts
- [ ] Provider at 80% quota is deprioritized (lower score in strategy ranking)
- [ ] Provider at 95% quota is excluded from routing entirely
- [ ] Quota resets daily at 00:00 UTC (configurable)
- [ ] Unit test: `tracker.test.ts` covers: record, isEligible, warning threshold, hard stop, reset
- [ ] Integration test: router routes away from a provider when its quota is exhausted

**Estimate:** 4h

---

### STORY 2.2 — Complete Circuit Breaker Implementation (P0)
**As a** system operator,
**I want** each provider to have a functional circuit breaker,
**so that** a failing provider is automatically excluded until it recovers.

**Acceptance Criteria:**
- [ ] Circuit opens after 3 consecutive failures within 60 seconds (configurable)
- [ ] Open circuit excludes provider from routing for 120 seconds (configurable)
- [ ] After cooldown, circuit enters HALF_OPEN state — one test request allowed
- [ ] Successful test request → circuit closes (HEALTHY)
- [ ] Failed test request → circuit re-opens, cooldown resets
- [ ] Circuit state persists to SQLite
- [ ] Unit test: `circuit-breaker.test.ts` covers all state transitions
- [ ] Integration test: router continues serving requests when one provider's circuit is open

**Estimate:** 3h

---

### STORY 2.3 — Classifier Test Coverage (P0)
**As a** developer,
**I want** comprehensive tests for the 14-dimension classifier,
**so that** I can refactor and extend it with confidence.

**Acceptance Criteria:**
- [ ] `classifier.test.ts` has at least 2 test cases per dimension (28 minimum)
- [ ] Edge cases tested: empty prompt, very long prompt (>8k tokens), non-English input
- [ ] Complexity tier assignment tested: at least 5 prompts per tier (SIMPLE/MEDIUM/COMPLEX/REASONING)
- [ ] Test coverage on classifier.ts: > 90% lines, > 90% functions
- [ ] All tests pass in < 5 seconds

**Estimate:** 6h

---

## EPIC 3 — Router: Three-Layer Cache
**Goal:** L1/L2/L3 caching operational, delivering 30%+ token savings.
**Phase:** 1 | **Estimate:** 3 days

---

### STORY 3.1 — L1 In-Memory Cache (P0)
**As a** developer,
**I want** an in-memory LRU cache for exact-match prompt lookups,
**so that** repeated identical requests within a session are served instantly.

**Acceptance Criteria:**
- [ ] `l1-memory.ts` uses `node-lru-cache` with configurable max entries (default 1000)
- [ ] Cache key: `SHA256(model_tier + normalizedPrompt)`
- [ ] TTL: 300 seconds (configurable via env)
- [ ] Cache hit returns response in < 1ms
- [ ] LRU eviction works correctly when max entries reached
- [ ] `l1.test.ts`: set, get hit, get miss, TTL expiry, LRU eviction, key normalization
- [ ] Cache manager checks L1 first before L2/L3

**Estimate:** 4h

---

### STORY 3.2 — L2 Disk Cache (P1)
**As a** developer,
**I want** a SQLite-backed disk cache for exact-match lookups that survive restarts,
**so that** previously seen prompts are served from cache across sessions.

**Acceptance Criteria:**
- [ ] `l2-disk.ts` uses `better-sqlite3` with cache_entries table (schema per architecture doc)
- [ ] TTL: 86400 seconds (24h), configurable
- [ ] Cache hit returns response in < 10ms
- [ ] Expired entries are cleaned up on startup and periodically (every 1h)
- [ ] Max disk size: 10GB (configurable), evict oldest on overflow
- [ ] `l2.test.ts`: set, get hit, get miss, TTL expiry, persistence across restart simulation
- [ ] Cache manager checks L2 after L1 miss

**Estimate:** 4h

---

### STORY 3.3 — L3 Semantic Cache (P1)
**As a** developer,
**I want** an embedding-based semantic cache that matches similar (not just identical) prompts,
**so that** paraphrased versions of the same question are served from cache.

**Acceptance Criteria:**
- [ ] `l3-semantic.ts` uses `hnswlib-node` with cosine similarity
- [ ] Embeddings computed locally using `@xenova/transformers` (all-MiniLM-L6-v2, no API)
- [ ] Similarity threshold: 0.92 (configurable via env)
- [ ] TTL: 259200 seconds (72h), configurable
- [ ] Cache hit returns response in < 50ms
- [ ] HNSW index persists to disk and loads on startup
- [ ] `l3.test.ts`: semantic hit (paraphrase), semantic miss (unrelated), threshold edge cases
- [ ] Cold start: L1/L2 serve requests while L3 index loads asynchronously

**Estimate:** 8h

---

### STORY 3.4 — Cache Manager Orchestrator (P0)
**As a** developer,
**I want** a unified cache manager that coordinates L1→L2→L3 lookups and writes,
**so that** the router has a single cache interface.

**Acceptance Criteria:**
- [ ] `cache/manager.ts` exposes: `get(key, prompt)`, `set(key, prompt, response)`, `flush(layer?)`
- [ ] Lookup order: L1 → L2 → L3, returns first hit
- [ ] Write order: async non-blocking write to all three layers on miss+response
- [ ] Cache stats available: hit counts per layer, miss count, total entries
- [ ] `flush('l1' | 'l2' | 'l3' | 'all')` clears the specified layer(s)
- [ ] Integration test: full request through router results in correct cache population

**Estimate:** 3h

---

## EPIC 4 — Router: Intelligence Layer
**Goal:** Semantic intent detection, cascade routing, 5 strategies, memory routing all operational.
**Phase:** 2 | **Estimate:** 4 days

---

### STORY 4.1 — Semantic Intent Detector (P1)
**As a** router,
**I want** to classify every request's intent using local embeddings,
**so that** routing decisions use semantic understanding, not just keyword matching.

**Acceptance Criteria:**
- [ ] `intent-detector.ts` classifies into: CODE_REVIEW, BUG_FIX, ARCHITECTURE, DOCUMENTATION,
  SIMPLE_CHAT, REASONING, TOOL_CALL, CREATIVE, UNKNOWN
- [ ] Classification uses local embeddings (no external API call)
- [ ] Returns confidence score 0.0–1.0
- [ ] p95 latency < 100ms on warm embedding model
- [ ] Each intent maps to: preferred provider tier, routing strategy, OptiLLM flag
- [ ] `intent-detector.test.ts`: 3+ canonical examples per intent class (27+ test cases)
- [ ] Unknown intent falls through to 14-dimension classifier without error

**Estimate:** 1 day

---

### STORY 4.2 — Cascade Fallback Routing (P1)
**As a** router,
**I want** to attempt cheap models first and escalate to stronger models only when needed,
**so that** I maximize free-tier usage while maintaining quality.

**Acceptance Criteria:**
- [ ] `cascade.ts` implements: try top candidate → score response quality → escalate if < threshold
- [ ] Quality threshold configurable (default 0.75), set per routing strategy
- [ ] Cascade chain configurable in `providers.json`
- [ ] Max 2 escalations per request (prevents infinite loops)
- [ ] Cascade events logged with: attempted model, quality score, reason for escalation
- [ ] Escalation recorded in `RoutingDecision.escalated` and emitted on SSE stream
- [ ] `cascade.test.ts`: successful first attempt, single escalation, double escalation, max hit

**Estimate:** 1 day

---

### STORY 4.3 — Five Routing Strategies (P1)
**As a** user,
**I want** to select from five routing strategies,
**so that** I can optimize for quality, cost, or latency depending on my current task.

**Acceptance Criteria:**
- [ ] `strategies/quality.ts` — ranks providers by model capability score
- [ ] `strategies/cost.ts` — ranks providers by token cost (free tiers first, smallest models first)
- [ ] `strategies/latency.ts` — ranks providers by historical p50 latency
- [ ] `strategies/balanced.ts` — weighted composite: 40% quality + 30% cost + 30% latency
- [ ] `strategies/shuffle.ts` — round-robin across eligible providers
- [ ] Active strategy switchable via `PATCH /config` without restart
- [ ] Each strategy tested with a mock provider set in `strategies/*.test.ts`
- [ ] Strategy selection visible and editable in TUI Router Config screen

**Estimate:** 1 day

---

### STORY 4.4 — Memory-Augmented Routing (P2)
**As a** router,
**I want** to learn from past routing decisions,
**so that** I route future similar requests to historically successful providers.

**Acceptance Criteria:**
- [ ] `memory.ts` stores routing history in SQLite (schema per architecture doc)
- [ ] On new request: query history for similar prompts (similarity > 0.88)
- [ ] If match found with high success rate: boost matching provider's score by configurable weight
- [ ] History capped at 10,000 entries (LRU eviction by timestamp)
- [ ] History persists across restarts
- [ ] `memory.test.ts`: boost applied on similar history, no boost on no history, eviction
- [ ] Memory routing can be disabled via config without code change

**Estimate:** 1 day

---

### STORY 4.5 — SSE Metrics Stream Endpoint (P0)
**As a** TUI developer,
**I want** a Server-Sent Events endpoint that streams all routing events in real time,
**so that** the TUI can display live data without polling.

**Acceptance Criteria:**
- [ ] `GET /metrics/stream` returns `Content-Type: text/event-stream`
- [ ] Emits event types: `routing_decision`, `cache_hit`, `quota_update`, `quota_warning`,
  `circuit_change`, `provider_error`, `system_status` (every 5s)
- [ ] Each event is JSON with fields per architecture doc spec
- [ ] Supports at least 10 concurrent SSE connections
- [ ] API keys masked as `sk-...xxxx` in all event payloads
- [ ] Connection drop handled gracefully (client can reconnect, no server crash)
- [ ] Integration test: fire 5 requests, verify all 5 `routing_decision` events emitted

**Estimate:** 4h

---

### STORY 4.6 — OptiLLM Integration (P2)
**As a** router,
**I want** to route COMPLEX and REASONING requests through OptiLLM for enhanced inference,
**so that** high-stakes requests get 2-10x quality improvement at no extra cost.

**Acceptance Criteria:**
- [ ] `optillm/client.ts` proxies requests to OptiLLM at configurable URL
- [ ] Integration is gated by: `OPTILLM_ENABLED=true` AND complexity >= threshold
- [ ] Technique selected automatically by intent: MCTS (reasoning), self-consistency (code),
  chain-of-thought (architecture)
- [ ] If OptiLLM unreachable: falls through to direct provider call, logs warning
- [ ] OptiLLM usage logged in `RoutingDecision.optillm_used` + `optillm_technique`
- [ ] Integration test: mock OptiLLM server receives request when threshold met

**Estimate:** 4h

---

## EPIC 5 — Router: Prompt Templates
**Goal:** Template system operational, 5 built-in templates, partials working.
**Phase:** 2 | **Estimate:** 1 day

---

### STORY 5.1 — Template Registry and Renderer (P2)
**As a** Meta Code Squad operator,
**I want** prompt templates with variable substitution and partials,
**so that** agents use consistent, optimized prompts without hardcoding them.

**Acceptance Criteria:**
- [ ] `templates/registry.ts` loads all `.md` files from the `templates/` directory on startup
- [ ] `{{variable}}` syntax replaced with request context values
- [ ] `{{> partial-name}}` syntax inlines content from `templates/partials/`
- [ ] Template resolution happens before the request is forwarded to the provider
- [ ] Built-in templates: `code-review.md`, `bug-fix.md`, `architecture.md`, `documentation.md`,
  `chat.md` — all using relevant partials
- [ ] Partials: `system-base.md` (shared system prompt), `code-context.md` (language/framework context)
- [ ] Template missing: falls through to raw prompt without error
- [ ] Unit test: render each template with mock context, verify variable substitution

**Estimate:** 1 day

---

## EPIC 6 — Textual TUI — All Screens
**Goal:** All 9 TUI screens complete, connected to SSE stream, boot animation polished.
**Phase:** 3 | **Estimate:** 5 days

---

### STORY 6.1 — TUI App Skeleton + Navigation (P1)
**As a** user,
**I want** the TUI app to launch, display the global chrome, and navigate between screens,
**so that** I have a working shell before individual screens are built.

**Acceptance Criteria:**
- [ ] `app.py` defines a Textual `App` class with all 9 screens registered
- [ ] Global header bar renders: logo, live indicator, provider count, request count, uptime
- [ ] Global nav bar renders with number key shortcuts for all screens
- [ ] `1`–`9` keys navigate to correct screens
- [ ] `Tab`/`Shift+Tab` cycle through screens
- [ ] `?` opens a keyboard help overlay listing all shortcuts
- [ ] `q` shows confirmation prompt and exits cleanly
- [ ] `router_client.py` establishes SSE connection to `ROUTER_URL/metrics/stream`
- [ ] If router unreachable on launch: shows clear error with retry option (not crash)

**Estimate:** 4h

---

### STORY 6.2 — Boot Animation Screen (P2)
**As a** user,
**I want** a branded boot animation that shows real initialization progress,
**so that** startup is informative and establishes the Aigency brand.

**Acceptance Criteria:**
- [ ] ASCII logo reveals left-to-right over 800ms using `AnimatedLabel` widget
- [ ] Version and tagline fade in after logo
- [ ] Progress bar fills in real time as each subsystem initializes
- [ ] Each subsystem line shows spinner → checkmark transition as it completes
- [ ] Provider key-missing warnings shown in yellow with provider name and env var
- [ ] Fatal error (no providers) shown in red with clear instruction; boot halts
- [ ] Boot completes and auto-transitions to Dashboard in < 3 seconds on a modern machine
- [ ] `--no-animation` flag skips directly to Dashboard
- [ ] Logo degrades gracefully at 80-column width

**Estimate:** 4h

---

### STORY 6.3 — Main Dashboard Screen (P1)
**As a** user,
**I want** to see all critical metrics on one screen,
**so that** I can assess system health at a glance without navigating elsewhere.

**Acceptance Criteria:**
- [ ] System Status panel: router state, cache state, OptiLLM state, queue depth, error rate, p50/p95
- [ ] Quota Overview panel: bar chart per provider, color-coded green/yellow/red, warning badges
- [ ] Routing panel: active strategy, cascade state, cache layers, OptiLLM threshold — all clickable
- [ ] Activity sparkline: 60-second rolling req/s bar chart
- [ ] Summary stats row: cache hit rate, tokens saved today, error rate
- [ ] Recent Requests feed: last 10 requests, scrollable, shows icons/badges per spec
- [ ] Clicking a request row opens a detail overlay with full `RoutingDecision` data
- [ ] All panels update within 2 seconds of SSE event receipt

**Estimate:** 1 day

---

### STORY 6.4 — Live Inference Stream Screen (P1)
**As a** user,
**I want** to watch every routing decision as a real-time collapsible tree,
**so that** I can understand exactly how the router is thinking.

**Acceptance Criteria:**
- [ ] Every `routing_decision` SSE event renders as a new tree node at the top
- [ ] Each tree shows all decision steps: intent, complexity, dimensions, cache, strategy, quota,
  selected, result — per UX spec layout
- [ ] Most recent request auto-expanded; all others collapsed
- [ ] `Enter` or click toggles expand/collapse on any request
- [ ] `Space` pauses/resumes the stream; shows buffered count while paused
- [ ] `/` opens filter dropdown (All/Errors/Cache Hits/Cascades/By Intent/By Provider)
- [ ] Last 500 requests kept in scrollable history
- [ ] Cache hit events render as distinct `◈ CACHE HIT Lx` rows (collapsed by default)
- [ ] Cascade escalation events render with `⚠` icon and escalation detail

**Estimate:** 1 day

---

### STORY 6.5 — Quota Manager Screen (P1)
**As a** operator,
**I want** full visibility into every provider's quota state and control over providers,
**so that** I can manage capacity and avoid quota exhaustion.

**Acceptance Criteria:**
- [ ] Table shows all providers: model tier, usage bar, tokens used/limit, % used, state badge
- [ ] Usage bars color-coded: green < 80%, yellow 80-95%, red > 95%
- [ ] Circuit breaker state shown per provider (CLOSED/OPEN/HALF_OPEN)
- [ ] Expanding a provider row shows: models list, reset time, failure count, circuit detail
- [ ] Monthly aggregate panel: total capacity, used, saved via cache, projected EOM
- [ ] `A` opens Add Provider modal (all fields per spec)
- [ ] `E` opens Edit modal for selected provider
- [ ] `D` disables/enables selected provider with confirmation
- [ ] `T` fires a test request and shows result inline
- [ ] `R` resets quota for selected provider with confirmation

**Estimate:** 1 day

---

### STORY 6.6 — Metrics, Config, Cache, and Logs Screens (P2)
**As a** user,
**I want** the remaining four screens (Metrics, Router Config, Cache Inspector, Logs) to be functional,
**so that** the TUI is complete and provides full operational control.

**Acceptance Criteria — Metrics (Screen 4):**
- [ ] Four tabs: Overview, Routing Dimensions, Provider Performance, Cache ROI
- [ ] Overview: request volume hourly bar chart, success/error/fallback rates, cache hit breakdown
- [ ] Routing Dimensions: all 14 dimensions with triggered %, avg score, influence bar
- [ ] Provider Performance: per-provider latency/error/throughput/quality table
- [ ] Cache ROI: tokens saved per layer, cost savings estimate, 7-day hit rate trend

**Acceptance Criteria — Router Config (Screen 6):**
- [ ] All config fields editable per UX spec (strategy, cascade, cache, OptiLLM, quota guards, memory)
- [ ] Changes dispatch `PATCH /config` within 500ms of field change
- [ ] `[✓ Saved]` flash on successful save
- [ ] Revert to defaults button with confirmation

**Acceptance Criteria — Cache Inspector (Screen 7):**
- [ ] Layer stats panel: entries, size, oldest entry age, hit rate per layer
- [ ] Browse panel: scrollable entry list with key preview, TTL, hit count
- [ ] Entry detail panel: full metadata on selected entry (intent, model tier, prompt preview, timestamps)
- [ ] Flush individual layer or all with confirmation
- [ ] Delete individual entry

**Acceptance Criteria — Logs (Screen 9):**
- [ ] Full log stream with auto-scroll
- [ ] Filter by level (DEBUG/INFO/WARN/ERROR — cumulative)
- [ ] Filter by component (router/cache/classifier/provider/quota/optillm/sse)
- [ ] `/` opens search box — filters to matching lines
- [ ] `E` exports visible logs to a timestamped file
- [ ] Auto-scroll toggle (`A`)

**Estimate:** 2 days

---

## EPIC 7 — @aigency/forge-quality: Core Package
**Goal:** forge init, all hooks, and shareable configs fully operational.
**Phase:** 1-2 | **Estimate:** 5 days

---

### STORY 7.1 — forge init: Step Execution Engine (P0)
**As a** developer,
**I want** `forge init` to execute all 11 setup steps reliably and idempotently,
**so that** I can set up a new project in < 60 seconds.

**Acceptance Criteria:**
- [ ] `InitStep` interface implemented with `check()` (idempotency) and `run()` for all 11 steps
- [ ] Steps run sequentially; each step checks if already done before executing
- [ ] Progress output per UX spec (numbered steps, checkmarks, timing)
- [ ] Interrupted run can be resumed — completed steps are skipped
- [ ] `--dry-run` flag shows what would be done without executing
- [ ] `--skip=<step-id>` flag skips specific steps
- [ ] Total execution time < 60s on standard dev machine with good internet
- [ ] End-to-end test: run `forge init` on a temp directory, verify all artifacts created

**Estimate:** 1 day

---

### STORY 7.2 — GitHub Integration (P1)
**As a** developer,
**I want** `forge init --github` to create a GitHub repo with branch protection,
**so that** my project is properly set up on GitHub from day one.

**Acceptance Criteria:**
- [ ] Checks for `gh` CLI; prints install instructions if missing, exits gracefully
- [ ] `gh repo create <name> --[public|private] --source=. --remote=origin` executed
- [ ] Branch protection rules set via GitHub API:
  - Require PR review (1 reviewer minimum)
  - Require status checks to pass
  - Require branches up to date
  - Restrict direct push to main
- [ ] First commit pushed to remote on completion
- [ ] Repo URL printed at end of init output
- [ ] Works with both `--public` and `--private` flags (default: private)

**Estimate:** 4h

---

### STORY 7.3 — Pre-Commit Hook: Full Pipeline (P0)
**As a** developer,
**I want** the pre-commit hook to auto-fix, type-check, scan for secrets, and gate on tests,
**so that** every commit is clean, typed, tested, and secret-free.

**Acceptance Criteria:**
- [ ] Prettier auto-fixes staged `.ts/.tsx/.js` files; fixed files re-staged
- [ ] ESLint `--fix` runs on staged `.ts/.tsx/.js` files; fixed files re-staged
- [ ] Ruff `check --fix` + `format` runs on staged `.py` files; fixed files re-staged
- [ ] `tsc --noEmit --incremental` runs; blocks commit on type error with file:line message
- [ ] `mypy` runs on staged `.py` files; blocks commit on type error with file:line message
- [ ] `gitleaks protect --staged` runs; hard-blocks on any secret detection
- [ ] `detect-secrets scan` runs on staged files against `.secrets.baseline`
- [ ] `vitest run --changed HEAD` runs only changed-file tests; blocks on failure
- [ ] Coverage checked for changed files; blocks if below threshold
- [ ] All parallel groups run concurrently (Lefthook `parallel: true`)
- [ ] Total hook time < 30s on a standard project (50 files, 200 tests)

**Estimate:** 1 day

---

### STORY 7.4 — Commit-Msg Hook (P0)
**As a** developer,
**I want** every commit message validated against Conventional Commits,
**so that** the commit history is always parseable for changelogs and semantic versioning.

**Acceptance Criteria:**
- [ ] `commitlint --edit $1` runs on every commit message
- [ ] Valid types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
- [ ] Aigency scopes defined: router, tui, forge, cache, quota, classifier, intent, cascade,
  hooks, config, deps, security
- [ ] On failure: prints exact error, shows valid format, shows example, prints `forge commit` hint
- [ ] On success: silent (no output)
- [ ] Works for both `git commit -m` and interactive commits

**Estimate:** 2h

---

### STORY 7.5 — Pre-Push Hook: Full Gate (P1)
**As a** developer,
**I want** pre-push to run the full quality and security suite,
**so that** nothing broken, untested, or insecure reaches the remote.

**Acceptance Criteria:**
- [ ] Full project `tsc --noEmit` runs (not just staged files)
- [ ] Full `vitest run --coverage` runs; blocks if coverage < thresholds (lines 80%, functions 80%, branches 70%)
- [ ] `audit-ci --moderate` runs; blocks on moderate+ severity vulnerabilities
- [ ] `gitleaks detect` runs full history scan; blocks on any detection
- [ ] `semgrep --config=p/typescript src/` runs; blocks on ERROR severity findings
- [ ] PR size check: warns at > 20 files changed vs main, blocks at > 50 (with `--no-verify` override)
- [ ] Parallel execution (Lefthook) — all checks run concurrently
- [ ] Total time < 120s on a standard project

**Estimate:** 4h

---

### STORY 7.6 — Post-Commit Summary Hook (P1)
**As a** developer,
**I want** a non-blocking summary after every commit,
**so that** I can see the quality state of my commit and know what to do next.

**Acceptance Criteria:**
- [ ] Runs after every successful commit (non-blocking under all conditions)
- [ ] Output per UX spec: files changed, tests passed, coverage %, security status, time taken
- [ ] Suggests next action: `git push` or `forge pr`
- [ ] Never produces exit code != 0
- [ ] Runs in < 1 second (reads from last test run output, does not re-run tests)

**Estimate:** 2h

---

### STORY 7.7 — forge commit CLI Command (P1)
**As a** developer,
**I want** an interactive conventional commit builder,
**so that** I never have to remember commit message format.

**Acceptance Criteria:**
- [ ] `forge commit` launches interactive commitizen prompt
- [ ] Type selection from full list with descriptions (per UX spec)
- [ ] Scope selection from Aigency scope list + "other" + "none" options
- [ ] Subject prompt with imperative-tense hint
- [ ] Optional body, breaking change flag, issue reference fields
- [ ] Preview of final commit message before confirmation
- [ ] On confirm: runs `git commit` with generated message (which triggers commit-msg hook)
- [ ] On cancel: exits cleanly with no commit made

**Estimate:** 4h

---

### STORY 7.8 — forge pr CLI Command (P2)
**As a** developer,
**I want** `forge pr` to auto-generate a PR description from my commit history,
**so that** I can create a high-quality PR in seconds.

**Acceptance Criteria:**
- [ ] Reads commit log since branching from main with `git log main..HEAD`
- [ ] Parses conventional commit messages: type, scope, subject, body
- [ ] Groups commits by type: feat, fix, refactor, docs, chore
- [ ] Computes coverage delta: current vs main branch baseline
- [ ] Runs `forge audit --json` for security summary
- [ ] Generates PR description per UX spec template (summary, changes, coverage table, security,
  checklist)
- [ ] `--open` flag: calls `gh pr create --web` to open PR in browser
- [ ] `--draft` flag: creates as draft PR
- [ ] Works when there are 0 conventional commits (generates minimal description with warning)

**Estimate:** 1 day

---

### STORY 7.9 — forge check and forge audit Commands (P1)
**As a** developer,
**I want** `forge check` and `forge audit` as standalone commands,
**so that** I can run quality and security checks on demand without committing.

**Acceptance Criteria — forge check:**
- [ ] Runs identical pipeline to pre-commit hook
- [ ] Reports all findings (does not auto-fix in check mode — use `forge fix` for that)
- [ ] `--fix` flag enables auto-fix mode
- [ ] Exit code 0 on all passing, 1 on any failure
- [ ] `--json` flag outputs structured JSON report

**Acceptance Criteria — forge audit:**
- [ ] Runs: gitleaks full history, detect-secrets full scan, semgrep on src/, audit-ci
- [ ] Per-scanner results shown clearly (clean / N findings)
- [ ] Findings include: file, line, rule, severity, remediation hint
- [ ] Exit code 0 on clean, 1 on any finding
- [ ] `--json` flag outputs structured JSON report

**Estimate:** 4h

---

### STORY 7.10 — Shareable Configs Package (P1)
**As a** developer in any project,
**I want** to import forge-quality's configs individually,
**so that** I can adopt parts of the toolchain without running `forge init`.

**Acceptance Criteria:**
- [ ] `@aigency/forge-quality/eslint` exports a valid ESLint flat config
- [ ] `@aigency/forge-quality/prettier` exports a valid Prettier config object
- [ ] `@aigency/forge-quality/tsconfig` is a valid `tsconfig.json` (strict mode, modern targets)
- [ ] `@aigency/forge-quality/commitlint` exports a valid commitlint config with Aigency scopes
- [ ] `@aigency/forge-quality/vitest` exports a valid Vitest config with coverage settings
- [ ] `@aigency/forge-quality/ruff` is a valid `ruff.toml` (exported as file, not JS)
- [ ] Each config documented in README with extend/override examples
- [ ] All configs tested: a minimal project using each config passes its respective linter/tool

**Estimate:** 4h

---

## EPIC 8 — CI/CD + Open Source Readiness
**Goal:** GitHub Actions CI, READMEs, and open source packaging complete.
**Phase:** 4 | **Estimate:** 3 days

---

### STORY 8.1 — GitHub Actions: PR Quality Check (P1)
**As a** project maintainer,
**I want** every PR to run the full quality suite automatically,
**so that** no broken or insecure code can be merged.

**Acceptance Criteria:**
- [ ] `ci.yml` runs on every PR targeting `main`
- [ ] Runs `pnpm turbo check` (lint + typecheck + test) across all packages
- [ ] Runs router integration tests via Docker Compose
- [ ] Status check required for PR merge (configured via branch protection)
- [ ] Workflow completes in < 10 minutes
- [ ] Uses caching: pnpm store cache, Turborepo remote cache

**Estimate:** 4h

---

### STORY 8.2 — GitHub Actions: Security Audit on Push (P1)
**As a** project maintainer,
**I want** a security audit to run on every push to main,
**so that** any credential or vulnerability that slipped through is caught immediately.

**Acceptance Criteria:**
- [ ] `security.yml` runs on every push to `main`
- [ ] Runs `forge audit` (full history gitleaks, detect-secrets, semgrep, audit-ci)
- [ ] On finding: creates GitHub issue with finding details (using gh CLI)
- [ ] On finding: posts comment on the triggering commit
- [ ] Workflow uses `actions/checkout@v4` with `fetch-depth: 0` for full history scan

**Estimate:** 3h

---

### STORY 8.3 — SimpleLLMRouter README (P2)
**As a** potential open source user,
**I want** a comprehensive README for SimpleLLMRouter,
**so that** I can understand, install, and configure the router in under 15 minutes.

**Acceptance Criteria:**
- [ ] README includes: hero section with value proposition, quick-start (< 5 commands to running),
  architecture diagram (ASCII), provider configuration guide, all env vars documented,
  TUI screenshot/GIF, contributing guide link, license
- [ ] Quick-start uses Docker Compose as primary path
- [ ] All env vars have default values and descriptions
- [ ] "Built by Aigency" section explains the real-world use case (Meta Code Squad)
- [ ] Badges: license, build status, npm version (router-tui Python package)

**Estimate:** 4h

---

### STORY 8.4 — forge-quality README (P2)
**As a** potential open source user,
**I want** a comprehensive README for @aigency/forge-quality,
**so that** I can install it into my project in under 5 minutes.

**Acceptance Criteria:**
- [ ] README includes: hero with value prop, three install modes (project/global/bootstrap),
  all CLI commands documented with example output, hook behavior explained,
  config reference for all tools, contributing guide, license
- [ ] Demo section: shows `forge init` output (truncated), `forge commit` prompt,
  pre-commit hook output (happy path + secret blocked)
- [ ] "How it works" section explains the hook pipeline with a simple diagram
- [ ] Compatibility table: OS, Node.js version, Python version, package managers

**Estimate:** 4h

---

### STORY 8.5 — npm Package Publishing (P2)
**As an** open source user,
**I want** to install `@aigency/forge-quality` from npm,
**so that** I don't need to clone the monorepo to use it.

**Acceptance Criteria:**
- [ ] `package.json` configured correctly: `name`, `version`, `bin`, `exports`, `files`, `engines`
- [ ] `bin/forge` entry point works after `npm install -g @aigency/forge-quality`
- [ ] `npx @aigency/forge-quality init` works without global install
- [ ] Package published to npm under `@aigency` scope
- [ ] GitHub Actions release workflow: on tag push, run tests, publish to npm
- [ ] `.npmignore` excludes: `src/`, `tests/`, `*.test.ts`, dev configs — ships only `dist/`

**Estimate:** 4h

---

## EPIC 9 — Docker + Local Development
**Goal:** One-command startup for the full stack.
**Phase:** 1 | **Estimate:** 1 day

---

### STORY 9.1 — Docker Compose Full Stack (P1)
**As a** developer,
**I want** `docker compose up` to start the router and all dependencies,
**so that** I can run the full system without manual setup.

**Acceptance Criteria:**
- [ ] `docker-compose.yml` defines services: `router`, `router-tui` (opt-in profile), `optillm` (opt-in profile)
- [ ] `docker compose up` starts just the router by default
- [ ] `docker compose --profile tui up` starts router + TUI
- [ ] `docker compose --profile optillm up` starts router + OptiLLM
- [ ] Router container: reads API keys from host env, mounts `providers.json` read-only,
  persists data to named volume
- [ ] Router health check: `GET /health` returns 200
- [ ] `docker compose down -v` cleans up all state
- [ ] Router Docker image < 500MB
- [ ] `providers.example.json` checked into repo as reference (no real keys)

**Estimate:** 1 day

---

## Backlog Summary

| Epic | Stories | P0 | P1 | P2 | P3 | Phase | Est. |
|------|---------|----|----|----|----|-------|------|
| 1 — Monorepo Foundation | 3 | 2 | 1 | 0 | 0 | 1 | 3d |
| 2 — Router: Bug Fixes | 3 | 3 | 0 | 0 | 0 | 1 | 2d |
| 3 — Router: Cache | 4 | 2 | 2 | 0 | 0 | 1 | 3d |
| 4 — Router: Intelligence | 6 | 1 | 3 | 2 | 0 | 2 | 4d |
| 5 — Router: Templates | 1 | 0 | 0 | 1 | 0 | 2 | 1d |
| 6 — TUI: All Screens | 6 | 0 | 4 | 2 | 0 | 3 | 5d |
| 7 — forge-quality Core | 10 | 3 | 5 | 2 | 0 | 1-2 | 5d |
| 8 — CI + Open Source | 5 | 0 | 2 | 3 | 0 | 4 | 3d |
| 9 — Docker | 1 | 0 | 1 | 0 | 0 | 1 | 1d |
| **Total** | **39** | **11** | **18** | **10** | **0** | | **~27d** |

---

## Sprint Plan (2-week sprints)

### Sprint 1 — Foundation + Critical Fixes (Weeks 1-2)
Stories: 1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 3.4, 9.1
Goal: Monorepo set up, router bug-free, cache manager scaffolded, Docker working.

### Sprint 2 — Cache + forge-quality Hooks (Weeks 3-4)
Stories: 3.1, 3.2, 3.3, 4.5, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6
Goal: 3-layer cache operational, all 4 forge hooks working, forge init functional.

### Sprint 3 — Router Intelligence + forge CLI (Weeks 5-6)
Stories: 4.1, 4.2, 4.3, 4.4, 4.6, 5.1, 7.7, 7.8, 7.9, 7.10
Goal: Full intelligence layer, OptiLLM, all forge CLI commands done.

### Sprint 4 — TUI + Open Source Polish (Weeks 7-8)
Stories: 6.1, 6.2, 6.3, 6.4, 6.5, 6.6, 8.1, 8.2, 8.3, 8.4, 8.5
Goal: Full TUI, GitHub Actions CI, READMEs, npm publish — both packages open-source ready.
