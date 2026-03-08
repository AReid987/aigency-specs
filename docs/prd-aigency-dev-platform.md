# Product Requirements Document
## SimpleLLMRouter v2 + @aigency/forge-quality

**Version:** 1.0 | **Date:** 2026-03-05 | **Owner:** Antonio Reid
**Status:** Approved | **Monorepo:** aigency/ (Turborepo)

---

## 1. Overview

This PRD defines the complete product requirements for two packages developed together inside
the `aigency/` Turborepo:

- **SimpleLLMRouter v2** — intelligent LLM routing gateway with Textual TUI
- **@aigency/forge-quality** — plug-and-play commit quality and security enforcement package

Both packages are designed to be individually open-sourceable, independently installable, and
deeply integrated when used together inside the Aigency monorepo.

---

## 2. SimpleLLMRouter v2 — Product Requirements

### 2.1 User Personas

**P1 — The Aigency Orchestrator (primary)**
Antonio running the Meta Code Squad. Needs 24/7 autonomous operation across 8 providers
with zero manual intervention. Monitors via TUI. Cares about: uptime, token savings, routing
accuracy, visibility.

**P2 — The Self-Hosted Developer**
Solo developer running local AI workflows. Wants provider redundancy, free-tier optimization,
and a beautiful interface to show their setup. Cares about: easy setup, zero cost, documentation.

**P3 — The Open Source AI Builder**
Contributing to or forking the router for custom provider configs. Cares about: clean
architecture, extensibility, test coverage, MIT license.

---

### 2.2 Functional Requirements

#### FR-R01: OpenAI-Compatible Proxy
- The router MUST expose an HTTP API on port 8080 that is drop-in compatible with the OpenAI
  `/v1/chat/completions` endpoint
- Any client using the OpenAI SDK MUST be able to point `baseURL` at the router with zero
  code changes
- The router MUST pass through all standard request fields: messages, model hint, temperature,
  max_tokens, stream
- The router MAY override the `model` field based on routing decisions regardless of what
  the client requested

#### FR-R02: Provider Registry
- The router MUST support a minimum of 8 providers: Gemini, OpenAI, Anthropic, DeepSeek,
  Together.ai, Groq, Mistral, Cohere
- Provider configuration MUST be defined in a JSON/YAML file external to the source code
- Each provider config MUST include: name, base_url, api_key env var reference, models array,
  daily_token_limit, requests_per_minute, enabled flag, canary_weight
- Adding a new provider MUST require only a config file change — no code changes

#### FR-R03: Semantic Intent Detection
- The router MUST classify every incoming request into one of: CODE_REVIEW, BUG_FIX,
  ARCHITECTURE, DOCUMENTATION, SIMPLE_CHAT, REASONING, TOOL_CALL, CREATIVE, UNKNOWN
- Classification MUST use local embeddings (no external API call) to achieve < 100ms latency
- Classification confidence score MUST be logged and available in the SSE metrics stream
- Intent MUST map to a preferred provider tier and routing strategy

#### FR-R04: 14-Dimension Complexity Classifier
- The router MUST score every request on 14 dimensions: code, logic, context_length, reasoning,
  creative, safety, tool_calling, multilingual, math, vision, speed, cost, context_window,
  instruction_follow
- Each dimension MUST produce a float score 0.0–1.0
- The aggregate complexity tier MUST be one of: SIMPLE, MEDIUM, COMPLEX, REASONING
- Classifier MUST have > 90% unit test coverage with edge cases documented

#### FR-R05: Cascade Fallback Routing
- The router MUST support cascade routing: attempt cheapest qualifying model first, escalate
  to next tier if quality threshold not met
- Quality threshold MUST be configurable per routing strategy (default 0.75)
- Cascade chain MUST be configurable (default: Flash → Pro → GPT-4o-mini → GPT-4o)
- Cascade escalation events MUST be logged with reason

#### FR-R06: Five Routing Strategies
- The router MUST support five named strategies selectable at runtime:
  - `quality` — maximize response quality; prefer strongest models
  - `cost` — minimize token spend; prefer free tier, smallest models
  - `latency` — minimize TTFB; prefer fastest providers (Groq, Gemini Flash)
  - `balanced` — weighted composite of quality, cost, latency
  - `simple-shuffle` — round-robin across healthy providers (useful for load testing)
- Active strategy MUST be changeable via REST PATCH without router restart
- Active strategy MUST be visible and editable in the TUI Router Config screen

#### FR-R07: Quota Tracker
- The router MUST track token consumption per provider per day
- QuotaTracker MUST be wired into the main request handler (currently unconnected — critical fix)
- When a provider reaches the warning threshold (default 80%), it MUST be deprioritized in routing
- When a provider reaches the hard cutoff (default 95%), it MUST be excluded from routing
- Quota state MUST persist across restarts (SQLite)
- Quota MUST reset on provider's reset schedule (default: daily at 00:00 UTC)

#### FR-R08: Circuit Breaker
- Each provider MUST have an independent circuit breaker
- Circuit opens after 3 consecutive failures within a 60-second window
- Open circuit MUST exclude the provider from routing for a configurable cooldown (default 120s)
- Circuit state: CLOSED (healthy), OPEN (excluded), HALF-OPEN (testing recovery)
- Circuit state MUST be visible per provider in the TUI Quota Manager screen

#### FR-R09: Three-Layer Cache
- **L1 In-Memory Cache:** exact match on `hash(model_tier + normalized_prompt)`, TTL 5 minutes,
  max 1000 entries (LRU eviction)
- **L2 Disk Cache:** exact match persisted to SQLite, TTL 24 hours, max 10GB
- **L3 Semantic Cache:** embedding-based similarity match using hnswlib, threshold 0.92,
  TTL 72 hours; embeddings computed locally using a lightweight model (all-MiniLM-L6-v2)
- Cache lookup MUST check L1 → L2 → L3 in order; first hit wins
- Cache hit MUST be logged with layer (L1/L2/L3) and similarity score
- All cache layers MUST be independently flushable via CLI and TUI

#### FR-R10: Memory-Augmented Routing
- The router MUST store a rolling history of routing decisions (last 10,000 requests)
- For each new request, router MUST check if a similar past request was routed successfully
- If a high-confidence match exists (similarity > 0.88), the historically successful provider
  MUST be weighted higher in the routing decision
- History MUST persist across restarts (SQLite)

#### FR-R11: OptiLLM Integration
- The router MUST support optional routing through an OptiLLM proxy for enhanced inference
- OptiLLM integration MUST be gated by: `OPTILLM_ENABLED=true` env var and complexity
  threshold (default: COMPLEX and REASONING tiers only)
- If OptiLLM is unreachable, the router MUST fall through to direct provider call
- OptiLLM technique selection MUST be automatic based on intent: MCTS for reasoning,
  self-consistency for code, chain-of-thought for architecture

#### FR-R12: Prompt Template System
- The router MUST support a template registry loaded from a `templates/` directory
- Templates MUST use `{{variable}}` Handlebars-style syntax
- Templates MUST support partials via `{{> partial-name}}` include syntax
- Template resolution MUST occur before the request is forwarded to the provider
- Built-in templates MUST include: code-review, bug-fix, architecture, documentation, chat

#### FR-R13: SSE Metrics Stream
- The router MUST expose a Server-Sent Events endpoint at `/metrics/stream`
- The stream MUST emit events for: every routing decision, cache hits/misses, quota updates,
  circuit breaker state changes, provider errors
- Event format MUST be JSON with fields: timestamp, event_type, provider, model, latency_ms,
  tokens, intent, complexity, cache_layer, error (if any)
- The TUI MUST consume this stream exclusively for all live data

---

### 2.3 Non-Functional Requirements

#### NFR-R01: Performance
- Routing decision (intent + classify + strategy + quota check) MUST complete in < 100ms p95
- L1 cache lookup MUST complete in < 1ms
- L2 cache lookup MUST complete in < 10ms
- L3 semantic cache lookup MUST complete in < 50ms
- SSE stream MUST support at least 10 concurrent TUI/client connections

#### NFR-R02: Reliability
- Router MUST continue serving requests if any single provider is down
- Router MUST continue serving requests if OptiLLM is down
- Router MUST continue serving requests if the TUI is disconnected
- Router MUST handle provider API errors gracefully with automatic retry (max 2 retries,
  exponential backoff)

#### NFR-R03: Security
- API keys MUST be loaded from environment variables — never hardcoded
- API keys MUST never appear in logs or SSE streams (masked as `sk-...xxxx`)
- The router MUST NOT log full request/response bodies at INFO level (only at DEBUG)
- The router SHOULD support an optional auth token on its own API for multi-user environments

#### NFR-R04: Observability
- Every routing decision MUST be logged with: timestamp, intent, complexity, provider selected,
  latency, token count, cache result
- Structured JSON logging at all levels
- Log level configurable via `LOG_LEVEL` env var

#### NFR-R05: Portability
- MUST run with `docker-compose up` with zero additional setup
- MUST run with `npm start` for local development
- Docker image MUST be < 500MB
- MUST work on Linux, macOS, and Windows (WSL2)

---

### 2.4 Textual TUI Requirements

#### FR-T01: Boot Screen
- MUST display Aigency ASCII logo with animated reveal (< 2 seconds)
- MUST show real-time initialization progress for each subsystem
- MUST display provider connection status during boot
- MUST auto-transition to Dashboard on successful boot
- MUST be skippable with `--no-animation` flag

#### FR-T02: Main Dashboard
- MUST show system status (router, cache, OptiLLM, queue depth)
- MUST show per-provider quota bars (color-coded: green/yellow/red)
- MUST show 60-second rolling request rate sparkline
- MUST show p50/p95 latency, cache hit rate, error rate, tokens saved today
- MUST show last 10 requests with provider, latency, token count
- MUST refresh automatically every 2 seconds

#### FR-T03: Live Inference Stream
- MUST show every routing decision as a collapsible tree in real time
- Tree nodes: Intent → Complexity → Dimensions → Cache check → Strategy → Quota → Selected → Result
- MUST support pause/resume of the stream
- MUST support filter by: All / Errors Only / Cache Hits / By Intent / By Provider
- MUST retain last 500 requests in scrollable history

#### FR-T04: Quota Manager
- MUST show per-provider: daily usage bar, % used, absolute tokens used/limit, status badge
- MUST show monthly aggregate: total capacity, used, saved via cache, projected end-of-month
- MUST show circuit breaker state per provider
- MUST support: add provider, edit provider, disable provider, test provider connection

#### FR-T05: Metrics Deep Dive
- MUST show request volume over time (hourly/daily toggle)
- MUST show per-dimension routing analysis table (match rate, avg score, impact weight)
- MUST show per-provider performance: avg latency, error rate, token throughput
- MUST show cache ROI: tokens saved, estimated cost savings, hit rate by layer

#### FR-T06: Router Config Screen
- MUST allow live switching of routing strategy (no restart)
- MUST allow adjustment of cascade quality threshold (slider)
- MUST allow toggling OptiLLM on/off and threshold
- MUST allow toggling cache layers independently
- MUST apply changes via REST PATCH to router immediately

#### FR-T07: Navigation
- Number keys 1-9 jump to screens directly
- Tab/Shift+Tab cycle through screens
- `?` opens keyboard shortcut help overlay
- `q` quits with confirmation prompt

---

## 3. @aigency/forge-quality — Product Requirements

### 3.1 User Personas

**P1 — The Monorepo Maintainer**
Running a Turborepo with 5+ packages. Wants shared quality config, consistent hooks across all
packages, and a single `pnpm add` to wire everything up. Cares about: zero duplication, easy
updates, speed.

**P2 — The Solo Developer**
Starting a new TypeScript or Python project. Wants professional hygiene in 60 seconds without
reading 10 different tool docs. Cares about: just works, auto-fixes things, doesn't slow them down.

**P3 — The Team Lead**
Enforcing standards across a team that uses AI-assisted development. Wants commits to be
reviewable, small, and trustworthy. Cares about: conventional commits, security scanning,
PR size enforcement, conventional coverage gates.

---

### 3.2 Functional Requirements

#### FR-Q01: CLI — forge init
- `forge init` MUST execute the full 11-step setup sequence (see architecture doc)
- MUST detect project context: Turborepo package vs standalone project
- MUST detect language: TypeScript, Python, or both
- MUST support flags: `--github`, `--public`/`--private`, `--template=ts|py|fullstack`
- MUST be idempotent: running on an existing project MUST skip already-configured steps
- MUST complete in < 60 seconds on a standard development machine
- MUST provide clear progress output for each step with success/failure indication

#### FR-Q02: GitHub Integration
- When `--github` flag is passed, MUST use GitHub CLI (`gh`) to create remote repository
- MUST check for `gh` installation and prompt user to install if missing
- MUST create repository with specified visibility (default: private)
- MUST configure branch protection rules via GitHub API:
  - Require pull request reviews before merging (minimum 1)
  - Require status checks to pass before merging
  - Require branches to be up to date before merging
  - Restrict direct pushes to `main`/`master`
- MUST push initial commit to remote on completion

#### FR-Q03: Pre-Commit Hook
- MUST run in parallel where dependencies allow (Lefthook parallel groups)
- MUST auto-fix: Prettier formatting, ESLint fixable violations, Ruff lint, Ruff format
- MUST re-stage auto-fixed files before proceeding
- MUST type-check staged TypeScript files (tsc --noEmit --incremental)
- MUST type-check staged Python files (mypy)
- MUST run only tests related to staged files (not full suite — preserves speed)
- MUST check coverage delta for changed files; block if coverage drops below threshold
- MUST scan staged files for secrets (gitleaks --staged); MUST hard-block if found
- MUST NOT auto-fix secret scan violations under any circumstances
- MUST complete in < 30 seconds on a standard development machine

#### FR-Q04: Commit-Msg Hook
- MUST validate commit message against Conventional Commits specification
- Valid format: `type(scope): subject` where type is one of:
  `feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert`
- On failure: MUST print exact validation error, show valid format example, exit 1
- MUST support Aigency-specific scopes: router, tui, forge, cache, quota, classifier,
  intent, cascade, hooks, config, deps, security

#### FR-Q05: Pre-Push Hook
- MUST run full project type check (not just staged files)
- MUST run full test suite with coverage
- MUST enforce coverage thresholds: lines 80%, functions 80%, branches 70%
- MUST run full dependency audit (audit-ci --moderate)
- MUST run full gitleaks history scan
- MUST run semgrep SAST on `src/` directory
- MUST check PR size: warn at > 20 files changed vs main, block at > 50 files
  (overridable with `--no-verify` for emergencies with warning message)
- MUST complete in < 120 seconds

#### FR-Q06: Post-Commit Hook (non-blocking)
- MUST print a human-readable commit summary after every successful commit:
  files changed count, tests passed count, coverage %, security status
- MUST suggest next action: `git push` or `forge pr`
- MUST NOT block on any condition (informational only)

#### FR-Q07: CLI — forge commit
- MUST launch an interactive commitizen prompt for constructing conventional commit messages
- MUST guide user through: type selection, scope selection, subject, body (optional),
  breaking change flag (optional), issue references (optional)
- MUST use Aigency-specific scope list from FR-Q04

#### FR-Q08: CLI — forge pr
- MUST generate a pull request description from the commit history since branching from main
- Generated PR description MUST include:
  - Summary section (auto-generated from commit messages)
  - Changes section (grouped by commit type: feat, fix, refactor, etc.)
  - Test coverage delta (before/after comparison)
  - Security scan summary (clean / N issues found)
  - Checklist: [ ] Tests pass, [ ] Coverage maintained, [ ] No secrets, [ ] Docs updated
- MUST support opening the PR in browser with `--open` flag
- MUST use `gh pr create` for PR submission

#### FR-Q09: CLI — forge check
- MUST run all quality checks without committing
- Equivalent to manually triggering the pre-commit pipeline
- Useful for: CI pipelines, pre-flight checks before pushing

#### FR-Q10: CLI — forge audit
- MUST run full security audit on demand:
  - gitleaks full history scan
  - detect-secrets scan of entire repository
  - semgrep SAST on src/
  - audit-ci dependency vulnerability check
- MUST produce a structured report: clean/issues-found per scanner

#### FR-Q11: Shareable Configs
- MUST export shareable configurations for all tools:
  - `@aigency/forge-quality/eslint` — ESLint config with TypeScript + quality rules
  - `@aigency/forge-quality/prettier` — Prettier config
  - `@aigency/forge-quality/tsconfig` — TypeScript strict config
  - `@aigency/forge-quality/commitlint` — Commitlint conventional config + Aigency scopes
  - `@aigency/forge-quality/ruff` — Ruff config for Python
  - `@aigency/forge-quality/vitest` — Vitest config with coverage settings
- Each config MUST be individually importable for teams that only want parts of the toolchain

#### FR-Q12: Install Modes
- **Per-project:** `pnpm add -D @aigency/forge-quality` then `forge init`
- **Global:** `npm install -g @aigency/forge-quality` for developer machine-wide install
- **Bootstrap:** `npx @aigency/forge-quality init --name <project> --github` creates directory,
  inits git, creates GitHub repo, installs all tooling from scratch

#### FR-Q13: Language Support
- TypeScript/JavaScript: ESLint, Prettier, tsc, Vitest, gitleaks, detect-secrets, semgrep,
  audit-ci (npm audit)
- Python: Ruff, Mypy, pytest, pytest-cov, gitleaks, detect-secrets, semgrep, audit-ci (pip audit)
- Fullstack (both): all of the above, coordinated in a single pre-commit pipeline

---

### 3.3 Non-Functional Requirements

#### NFR-Q01: Speed
- pre-commit hook MUST complete in < 30 seconds (target: < 15 seconds on warm runs)
- pre-push hook MUST complete in < 120 seconds
- `forge init` MUST complete in < 60 seconds

#### NFR-Q02: Reliability
- All hooks MUST be idempotent — running twice on the same state MUST produce the same result
- Hook failures MUST produce clear, actionable error messages with file + line number
- `forge init` MUST be resumable — if interrupted, re-running MUST skip completed steps

#### NFR-Q03: Compatibility
- MUST work with Node.js >= 18 and pnpm >= 8
- MUST work with Python >= 3.10
- MUST work on Linux, macOS, Windows (WSL2)
- MUST work in GitHub Actions CI environment
- MUST work with both npm and pnpm package managers

#### NFR-Q04: Security
- Secret scanning MUST run before any other hook group — fail fast
- MUST maintain a `.secrets.baseline` file for detect-secrets to track known non-secrets
- gitleaks MUST be configured to scan `.env.*` files with high priority
- MUST support `.gitleaks.toml` allowlist for project-specific false positives
- MUST NEVER auto-fix or auto-suppress security findings

#### NFR-Q05: Transparency
- Every hook action MUST print what it is doing and why
- Auto-fix operations MUST print: "Auto-fixed: <file> — <what changed>"
- Block operations MUST print: "Blocked: <reason> at <file>:<line> — <how to fix>"
- The post-commit summary MUST show all quality dimensions in one glance

---

## 4. Integration Requirements

### INT-01: forge-quality inside the aigency monorepo
- The `simplellmrouter` app and `router-tui` app MUST both use `@aigency/forge-quality`
- The `forge-quality` package MUST use itself (dogfooding) — its own commits enforced by its
  own hooks
- Turborepo `turbo.json` MUST include forge-quality check tasks in the build pipeline

### INT-02: Router + forge-quality in CI
- GitHub Actions MUST run `forge check` on every PR
- GitHub Actions MUST run `forge audit` on every push to main
- CI failure on security scan MUST block merge

### INT-03: Cross-package conventional commits
- All packages in the monorepo MUST use the same commitlint config from forge-quality
- Scope in commit messages MUST identify the affected package: `feat(router):`, `fix(tui):`,
  `chore(forge):`

---

## 5. Acceptance Criteria Summary

### SimpleLLMRouter v2 — Done When:
- [ ] All existing tests pass + new tests for classifier (14 dimensions), cache (3 layers),
      quota tracker, circuit breaker, cascade routing, intent detection
- [ ] QuotaTracker wired into server.ts — quota persists and gates routing
- [ ] 3-layer cache operational — L1/L2/L3 all returning hits in integration tests
- [ ] Semantic intent detection returning correct intent for 10 canonical test prompts
- [ ] All 5 routing strategies selectable and producing different routing outcomes
- [ ] SSE stream emitting events that the TUI can consume in real time
- [ ] TUI boots in < 3s and all 9 screens render without errors
- [ ] Live inference stream shows collapsible decision trees for each request
- [ ] Docker Compose `up` starts router + OptiLLM with zero manual config
- [ ] README with quickstart, architecture diagram, config reference

### @aigency/forge-quality — Done When:
- [ ] `npx @aigency/forge-quality init --name test-project --github` completes < 60s
- [ ] All 4 hooks install and run correctly on TypeScript project
- [ ] All 4 hooks install and run correctly on Python project
- [ ] Secret in staged file is caught and commit blocked (verified with test credential)
- [ ] Commit with non-conventional message is blocked with clear error
- [ ] Large commit (> 50 files) blocked on pre-push with clear message
- [ ] `forge pr` generates correct PR description from commit history
- [ ] Package published to npm as `@aigency/forge-quality`
- [ ] Package works when installed globally on a fresh machine
- [ ] README with quickstart, all CLI commands documented, config reference
