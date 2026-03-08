# Architecture Document
## SimpleLLMRouter v2 + @aigency/forge-quality

**Version:** 1.0 | **Date:** 2026-03-05 | **Owner:** Antonio Reid
**Status:** Approved | **Monorepo:** aigency/ (Turborepo)

---

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         aigency/ monorepo                               │
│                                                                         │
│  ┌──────────────────────┐      ┌──────────────────────────────────────┐ │
│  │   apps/              │      │   packages/                          │ │
│  │                      │      │                                      │ │
│  │  simplellmrouter/    │◄─────┤  forge-quality/                      │ │
│  │  (TypeScript/Node)   │      │  (@aigency/forge-quality)            │ │
│  │                      │      │                                      │ │
│  │  router-tui/         │      │  tsconfig/                           │ │
│  │  (Python/Textual)    │      │  (shared TS config)                  │ │
│  │                      │      │                                      │ │
│  └──────────────────────┘      └──────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

The two primary artifacts are:

1. **SimpleLLMRouter v2** (`apps/simplellmrouter` + `apps/router-tui`) — a runtime service
   deployed locally or in a container. TypeScript core, Python TUI as a separate process.

2. **@aigency/forge-quality** (`packages/forge-quality`) — a dev-time package installed into
   projects. Runs at commit time, not at application runtime.

They are co-developed in the same monorepo but independently deployable and open-sourceable.

---

## 2. Monorepo Structure

```
aigency/
├── apps/
│   ├── simplellmrouter/              TypeScript — LLM routing core
│   │   ├── src/
│   │   │   ├── server.ts             HTTP server, OpenAI-compat proxy
│   │   │   ├── router/
│   │   │   │   ├── index.ts          Main routing orchestrator
│   │   │   │   ├── classifier.ts     14-dimension complexity scorer
│   │   │   │   ├── intent-detector.ts Semantic intent classification
│   │   │   │   ├── cascade.ts        Cascade fallback routing
│   │   │   │   ├── memory.ts         History-augmented routing
│   │   │   │   └── strategies/
│   │   │   │       ├── quality.ts
│   │   │   │       ├── cost.ts
│   │   │   │       ├── latency.ts
│   │   │   │       ├── balanced.ts
│   │   │   │       └── shuffle.ts
│   │   │   ├── cache/
│   │   │   │   ├── manager.ts        Cache orchestrator (L1→L2→L3)
│   │   │   │   ├── l1-memory.ts      node-lru-cache, 5min TTL
│   │   │   │   ├── l2-disk.ts        better-sqlite3, 24h TTL
│   │   │   │   └── l3-semantic.ts    hnswlib + MiniLM embeddings, 72h TTL
│   │   │   ├── quota/
│   │   │   │   ├── tracker.ts        Per-provider token tracking
│   │   │   │   └── circuit-breaker.ts Per-provider circuit breaker
│   │   │   ├── providers/
│   │   │   │   ├── registry.ts       Load + validate provider configs
│   │   │   │   └── client.ts         Generic provider HTTP client
│   │   │   ├── templates/
│   │   │   │   ├── registry.ts       Template loader + renderer
│   │   │   │   ├── code-review.md
│   │   │   │   ├── bug-fix.md
│   │   │   │   ├── architecture.md
│   │   │   │   └── partials/
│   │   │   │       ├── system-base.md
│   │   │   │       └── code-context.md
│   │   │   ├── metrics/
│   │   │   │   ├── collector.ts      Aggregates all runtime metrics
│   │   │   │   └── sse-stream.ts     SSE endpoint /metrics/stream
│   │   │   └── optillm/
│   │   │       └── client.ts         Optional OptiLLM proxy client
│   │   ├── tests/
│   │   │   ├── classifier.test.ts
│   │   │   ├── intent-detector.test.ts
│   │   │   ├── cascade.test.ts
│   │   │   ├── cache/
│   │   │   │   ├── l1.test.ts
│   │   │   │   ├── l2.test.ts
│   │   │   │   └── l3.test.ts
│   │   │   ├── quota/
│   │   │   │   ├── tracker.test.ts
│   │   │   │   └── circuit-breaker.test.ts
│   │   │   └── strategies/
│   │   │       └── *.test.ts
│   │   ├── providers.json            Provider config (gitignored keys)
│   │   ├── providers.example.json    Safe example config for repo
│   │   ├── package.json
│   │   ├── tsconfig.json             extends @aigency/tsconfig/base
│   │   └── Dockerfile
│   │
│   └── router-tui/                   Python — Textual TUI
│       ├── app.py                    Main Textual Application class
│       ├── screens/
│       │   ├── boot.py               Boot animation screen
│       │   ├── dashboard.py          Main dashboard
│       │   ├── live_inference.py     Real-time decision stream
│       │   ├── quota.py              Quota manager
│       │   ├── metrics.py            Metrics deep dive
│       │   ├── provider_config.py    Provider config editor
│       │   ├── router_config.py      Router settings editor
│       │   ├── cache_inspector.py    Cache browser + flush
│       │   └── logs.py               Log viewer
│       ├── widgets/
│       │   ├── sparkline.py          ASCII sparkline chart
│       │   ├── quota_bar.py          Colored progress bar
│       │   ├── request_tree.py       Collapsible decision tree
│       │   ├── dimension_table.py    14-dimension data table
│       │   └── animated_logo.py      Boot animation widget
│       ├── client/
│       │   └── router_client.py      SSE consumer + REST client
│       ├── pyproject.toml
│       └── Dockerfile
│
├── packages/
│   ├── forge-quality/                @aigency/forge-quality
│   │   ├── src/
│   │   │   ├── cli/
│   │   │   │   ├── index.ts          CLI entry point (forge binary)
│   │   │   │   ├── init.ts           forge init command
│   │   │   │   ├── commit.ts         forge commit (commitizen)
│   │   │   │   ├── pr.ts             forge pr (PR generator)
│   │   │   │   ├── check.ts          forge check (manual run)
│   │   │   │   └── audit.ts          forge audit (security)
│   │   │   ├── init/
│   │   │   │   ├── detect-context.ts  Monorepo vs standalone detection
│   │   │   │   ├── git-setup.ts       git init + .gitignore
│   │   │   │   ├── github-setup.ts    gh repo create + branch protection
│   │   │   │   ├── hooks-setup.ts     Lefthook + Husky installation
│   │   │   │   ├── linters-setup.ts   ESLint + Prettier + Ruff install
│   │   │   │   ├── typecheck-setup.ts tsc + mypy install
│   │   │   │   ├── test-setup.ts      Vitest + pytest + coverage
│   │   │   │   ├── security-setup.ts  gitleaks + detect-secrets + semgrep
│   │   │   │   └── commit-setup.ts    commitlint + commitizen
│   │   │   ├── hooks/
│   │   │   │   ├── pre-commit.ts      Pre-commit hook definition
│   │   │   │   ├── commit-msg.ts      Commit-msg hook definition
│   │   │   │   ├── pre-push.ts        Pre-push hook definition
│   │   │   │   └── post-commit.ts     Post-commit summary hook
│   │   │   └── pr/
│   │   │       ├── generator.ts       PR description generator
│   │   │       └── templates/
│   │   │           └── pr-description.md
│   │   ├── configs/
│   │   │   ├── eslint.js              Shareable ESLint config
│   │   │   ├── prettier.js            Shareable Prettier config
│   │   │   ├── tsconfig.json          Strict TypeScript config
│   │   │   ├── commitlint.js          Commitlint config + Aigency scopes
│   │   │   ├── vitest.ts              Vitest config with coverage
│   │   │   ├── ruff.toml              Ruff config for Python
│   │   │   └── .gitleaks.toml         gitleaks allowlist base
│   │   ├── templates/
│   │   │   ├── ts/                    TypeScript project templates
│   │   │   ├── py/                    Python project templates
│   │   │   └── fullstack/             Both language templates
│   │   ├── lefthook.yml               Lefthook hook definitions
│   │   ├── package.json               name: "@aigency/forge-quality"
│   │   └── README.md
│   │
│   └── tsconfig/                      @aigency/tsconfig
│       ├── base.json
│       ├── nextjs.json
│       └── react-library.json
│
├── docker-compose.yml                 Router + OptiLLM + Redis (optional)
├── turbo.json                         Turborepo pipeline config
├── pnpm-workspace.yaml
├── package.json
└── .github/
    └── workflows/
        ├── ci.yml                     PR checks (forge check)
        └── security.yml               Push to main (forge audit)
```

---

## 3. SimpleLLMRouter v2 — Architecture

### 3.1 Request Lifecycle

```
Incoming Request (OpenAI-compatible)
         │
         ▼
┌─────────────────┐
│   server.ts     │  HTTP POST /v1/chat/completions
│   Express/Hono  │  Auth check (optional bearer token)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Cache Manager  │  L1 lookup (< 1ms)
│  (check first)  │  L2 lookup (< 10ms)
│                 │  L3 semantic lookup (< 50ms)
└────────┬────────┘
         │ MISS
         ▼
┌─────────────────┐
│ Intent Detector │  Local embeddings → intent classification
│                 │  Output: intent + confidence (< 100ms)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Classifier    │  14-dimension scoring
│                 │  Output: complexity tier (SIMPLE/MED/COMPLEX/REASONING)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Memory Router  │  Check routing history for similar past requests
│                 │  Boost historically successful providers
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Strategy Engine │  Apply active strategy (quality/cost/latency/balanced/shuffle)
│                 │  Generate ranked provider list
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Quota + CB     │  Filter providers: quota exceeded? circuit open?
│  Filter         │  Output: eligible provider list
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Cascade Router  │  Try top provider → if quality threshold not met → escalate
│                 │  Max 2 escalations per request
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ OptiLLM Client  │  If COMPLEX/REASONING: route through OptiLLM proxy
│  (optional)     │  If disabled or unreachable: skip
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Provider Client  │  HTTP call to selected provider
│                 │  Retry: max 2, exponential backoff
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Cache Writer    │  Write response to L1 + L2 + L3 async (non-blocking)
│ Metrics Emitter │  Emit SSE event with full routing decision data
│ Quota Updater   │  Increment provider token counter
└────────┬────────┘
         │
         ▼
    Response returned to client
```

### 3.2 Routing Decision Data Flow

Every routing decision produces a `RoutingDecision` object that flows through the entire
pipeline and is emitted on the SSE stream:

```typescript
interface RoutingDecision {
  request_id: string;
  timestamp: string;                    // ISO 8601
  intent: IntentType;
  intent_confidence: number;            // 0.0-1.0
  complexity: ComplexityTier;
  dimensions: Record<DimensionKey, number>; // all 14 scores
  cache_result: CacheResult | null;     // null = miss
  strategy: RoutingStrategy;
  candidates: ProviderCandidate[];      // ranked list before quota filter
  eligible: ProviderCandidate[];        // after quota + circuit filter
  selected: string;                     // provider id
  model: string;                        // actual model used
  escalated: boolean;
  escalation_reason?: string;
  optillm_used: boolean;
  optillm_technique?: string;
  latency_ms: number;
  ttfb_ms: number;
  tokens_prompt: number;
  tokens_completion: number;
  tokens_total: number;
  success: boolean;
  error?: string;
}
```

### 3.3 Cache Architecture

```
Cache Manager
├── L1: MemoryCache
│   ├── Store: node-lru-cache (Map-backed, O(1) get/set)
│   ├── Key: SHA256(model_tier + normalize(prompt))
│   ├── TTL: 300s (5 min)
│   └── Max: 1000 entries (LRU eviction)
│
├── L2: DiskCache
│   ├── Store: SQLite via better-sqlite3
│   ├── Table: cache_entries (key TEXT, value TEXT, expires_at INTEGER)
│   ├── Key: SHA256(model_tier + normalize(prompt))
│   ├── TTL: 86400s (24h)
│   └── Index: expires_at for efficient expiry cleanup
│
└── L3: SemanticCache
    ├── Store: hnswlib (HNSW index, cosine similarity)
    ├── Embeddings: all-MiniLM-L6-v2 via @xenova/transformers (local, no API)
    ├── Threshold: 0.92 cosine similarity
    ├── TTL: 259200s (72h)
    └── Persistence: index saved to disk on write, loaded on startup

Cache lookup order: L1 → L2 → L3
Cache write order: async, non-blocking, write to all three on miss+response
Normalization: strip whitespace, lowercase, remove punctuation from prompt before hashing
```

### 3.4 Quota + Circuit Breaker State Machine

```
Provider State:
  HEALTHY ──[3 failures in 60s]──► CIRCUIT_OPEN
  CIRCUIT_OPEN ──[120s cooldown]──► HALF_OPEN
  HALF_OPEN ──[1 success]──► HEALTHY
  HALF_OPEN ──[1 failure]──► CIRCUIT_OPEN

Quota State:
  AVAILABLE ──[usage > 80%]──► WARNING (deprioritized)
  WARNING ──[usage > 95%]──► HARD_STOP (excluded)
  HARD_STOP ──[daily reset]──► AVAILABLE

Combined routing eligibility:
  eligible = state == HEALTHY && quota_state != HARD_STOP
```

### 3.5 SSE Metrics Stream

```
GET /metrics/stream
Content-Type: text/event-stream
Connection: keep-alive

Event types emitted:
  routing_decision  — full RoutingDecision object (every request)
  cache_hit         — {layer, similarity, tokens_saved}
  quota_update      — {provider, tokens_used, tokens_limit, pct}
  quota_warning     — {provider, pct, threshold}
  circuit_change    — {provider, from_state, to_state, reason}
  provider_error    — {provider, error, retry_count}
  system_status     — {uptime, req_total, req_per_sec, p50_ms, p95_ms} (every 5s)
```

---

## 4. @aigency/forge-quality — Architecture

### 4.1 Package Boundaries

```
@aigency/forge-quality
├── Runtime boundary: ZERO — runs only at dev time (git hooks, CLI)
├── Production bundle: NOT included in any app bundle
├── Distribution: npm package + global install
└── Exports:
    ├── /bin/forge          CLI binary
    ├── /configs/*          Shareable tool configs
    └── /hooks/*            Hook script templates
```

### 4.2 forge init — Step Execution Engine

```typescript
// Execution model: sequential steps, each idempotent, resumable
interface InitStep {
  id: string;
  name: string;
  check: () => Promise<boolean>;    // is this step already done?
  run: () => Promise<void>;         // execute the step
  rollback?: () => Promise<void>;   // undo if needed
}

const INIT_STEPS: InitStep[] = [
  detectContext,      // monorepo? standalone? language?
  gitInit,            // git init + .gitignore + branch config
  githubSetup,        // gh repo create + branch protection (if --github)
  installHookchain,   // lefthook + husky
  installLinters,     // eslint + prettier OR ruff
  installTypecheckers, // tsc OR mypy
  installTestRunner,  // vitest OR pytest + coverage
  installSecurity,    // gitleaks + detect-secrets + semgrep + audit-ci
  installCommitTools, // commitlint + commitizen
  writeConfigs,       // write all config files
  firstCommit,        // git add . && git commit && git push
];
```

### 4.3 Hook Execution Architecture

Hooks are defined in `lefthook.yml` and installed into `.husky/` for compatibility.
Lefthook runs groups in parallel; steps within a group run sequentially.

```yaml
# lefthook.yml (generated by forge init)
pre-commit:
  parallel: true
  jobs:
    format-fix:
      glob: "*.{ts,tsx,js,jsx}"
      run: prettier --write {staged_files} && eslint --fix {staged_files}
      stage_fixed: true          # re-stage auto-fixed files

    format-fix-py:
      glob: "*.py"
      run: ruff check --fix {staged_files} && ruff format {staged_files}
      stage_fixed: true

    typecheck-ts:
      run: tsc --noEmit --incremental
      fail_text: "TypeScript errors found. Fix manually — cannot auto-fix."

    typecheck-py:
      glob: "*.py"
      run: mypy {staged_files}
      fail_text: "Mypy errors found. Fix manually — cannot auto-fix."

    secret-scan:
      run: gitleaks protect --staged --config=.gitleaks.toml
      fail_text: "SECRET DETECTED. Remove credential before committing."

    detect-secrets:
      run: detect-secrets scan --baseline .secrets.baseline {staged_files}

    semgrep:
      glob: "*.{ts,tsx,py}"
      run: semgrep --config=p/typescript --config=p/python {staged_files} --quiet

    test-changed:
      run: vitest run --changed HEAD --passWithNoTests

    coverage-check:
      run: vitest run --coverage --changed HEAD --passWithNoTests

commit-msg:
  jobs:
    commitlint:
      run: commitlint --edit {1}
      fail_text: "Invalid commit message format.\nUse: type(scope): subject\nRun: forge commit"

pre-push:
  parallel: true
  jobs:
    full-typecheck:
      run: tsc --noEmit
    full-test:
      run: vitest run --coverage
    full-audit:
      run: audit-ci --moderate
    security-scan:
      run: gitleaks detect && semgrep --config=p/typescript src/
    pr-size-check:
      run: forge _check-pr-size     # internal CLI command

post-commit:
  jobs:
    summary:
      run: forge _post-commit-summary
```

### 4.4 forge pr — PR Description Generator

```typescript
// Algorithm:
// 1. Get commit log since branching from main
//    git log main..HEAD --format="%H|%s|%b"
// 2. Parse conventional commits: extract type, scope, subject, body
// 3. Group by type: feat[], fix[], refactor[], docs[], chore[]
// 4. Get coverage delta: run vitest --coverage --reporter=json, compare to main branch baseline
// 5. Run forge audit --json for security summary
// 6. Render PR description template with all data
// 7. forge pr --open: gh pr create --title "<first feat>" --body "<rendered>" --web

interface PRData {
  branch: string;
  base: string;
  commits: ParsedCommit[];
  grouped: Record<ConventionalType, ParsedCommit[]>;
  coverage: { before: number; after: number; delta: number };
  security: { clean: boolean; findings: SecurityFinding[] };
  stats: { files_changed: number; insertions: number; deletions: number };
}
```

### 4.5 Security Scanning Stack

```
Tool            Purpose                     Block on    Auto-fix
──────────────────────────────────────────────────────────────────
gitleaks        Secret/credential detection  ALWAYS      NEVER
detect-secrets  Secret baseline tracking     ALWAYS      NEVER
semgrep         SAST (code vulnerabilities)  pre-push    NEVER
audit-ci        npm/pip dependency CVEs      pre-push    NEVER (use npm audit fix separately)

Secret patterns covered by gitleaks default rules:
  - AWS access keys, secret keys, session tokens
  - GitHub tokens (classic + fine-grained)
  - OpenAI API keys (sk-...)
  - Anthropic API keys
  - Google API keys
  - Private keys (RSA, EC, PGP)
  - JWT tokens
  - Slack tokens
  - Discord bot tokens
  - Stripe keys
  - Database connection strings with passwords
  - Generic high-entropy strings in .env files
```

---

## 5. Monorepo Build Pipeline

### 5.1 turbo.json

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "lint": {
      "outputs": []
    },
    "typecheck": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "check": {
      "dependsOn": ["lint", "typecheck", "test"],
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 5.2 Package Dependency Graph

```
@aigency/tsconfig
       │
       ├──► @aigency/forge-quality (devDependency)
       │
       └──► simplellmrouter (app)
                │
                └──► router-tui (Python — no JS deps, Docker only)

forge-quality is used BY simplellmrouter as a devDependency,
and is also used by itself (dogfooding).
```

### 5.3 Docker Compose

```yaml
# docker-compose.yml
services:
  router:
    build: ./apps/simplellmrouter
    ports:
      - "8080:8080"
    environment:
      - PROVIDERS_FILE=/config/providers.json
      - LOG_LEVEL=info
      - OPTILLM_URL=http://optillm:8000
    volumes:
      - ./providers.json:/config/providers.json:ro
      - router_data:/data          # SQLite cache + quota + history
    depends_on:
      - optillm

  router-tui:
    build: ./apps/router-tui
    environment:
      - ROUTER_URL=http://router:8080
    network_mode: host             # needed for terminal rendering
    profiles: ["tui"]              # opt-in: docker compose --profile tui up

  optillm:
    image: ghcr.io/codelion/optillm:latest
    ports:
      - "8000:8000"
    environment:
      - OPTILLM_API_KEY=${OPENAI_API_KEY}
    profiles: ["optillm"]          # opt-in: docker compose --profile optillm up

volumes:
  router_data:
```

---

## 6. Technology Decisions

### 6.1 Router Core

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | TypeScript/Node.js | Existing codebase; excellent async I/O; rich LLM ecosystem |
| HTTP framework | Hono | Faster than Express; TypeScript-native; edge-compatible |
| SQLite client | better-sqlite3 | Synchronous API; best performance for small datasets |
| LRU cache | node-lru-cache | Battle-tested; O(1) operations; TTL support |
| Vector index | hnswlib-node | Local HNSW; no external service; < 50ms lookup |
| Embeddings | @xenova/transformers (WASM) | Runs locally; no API; all-MiniLM-L6-v2 is 23MB |
| Test runner | Vitest | Fast; TypeScript-native; compatible with Turborepo |
| Process manager | Docker / PM2 | Docker for prod; PM2 for local dev |

### 6.2 TUI

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | Python | Textual requires Python; separate process is correct architecture |
| TUI framework | Textual | Best-in-class; reactive; CSS styling; browser mode |
| SSE client | httpx + anyio | Async streaming; Textual-compatible event loop |
| REST client | httpx | Single dependency for both SSE + REST |
| Python version | 3.11+ | Match Textual requirements; performance improvements |

### 6.3 forge-quality

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | TypeScript | Same ecosystem as projects it manages; single language |
| Hook runner | Lefthook | Parallel execution; faster than Husky alone; no Node in hooks |
| Fallback hooks | Husky | Universal compatibility; good Node.js project DX |
| TS linter | ESLint + @typescript-eslint | Standard; most plugin ecosystem; auto-fixable |
| Formatter | Prettier | De facto standard; integrates with ESLint |
| Python linter | Ruff | Replaces flake8+isort+black; 10-100x faster; single tool |
| Python formatter | Ruff format | Built into Ruff; no separate black dependency |
| Python types | Mypy | Gold standard; strict mode support |
| TS test runner | Vitest | Fast; TypeScript-native; coverage built-in |
| Python test runner | pytest + pytest-cov | Standard; flexible; coverage integration |
| Commit standard | Conventional Commits | Universal; enables CHANGELOG generation; semantic versioning |
| Commit UX | commitizen | Interactive; enforces format; familiar to teams |
| Secret scanner | gitleaks + detect-secrets | Complementary; gitleaks = broad rules; detect-secrets = baseline tracking |
| SAST | semgrep | Fastest open-source SAST; excellent TypeScript + Python rulesets |
| Dep audit | audit-ci | Wraps npm/pip audit; configurable severity threshold; CI-friendly |

---

## 7. Data Storage

### 7.1 Router Data (SQLite, at `/data/router.db`)

```sql
-- Quota tracking
CREATE TABLE quota (
  provider TEXT NOT NULL,
  date TEXT NOT NULL,           -- YYYY-MM-DD
  tokens_used INTEGER DEFAULT 0,
  requests_used INTEGER DEFAULT 0,
  PRIMARY KEY (provider, date)
);

-- L2 cache
CREATE TABLE cache_entries (
  key TEXT PRIMARY KEY,
  response TEXT NOT NULL,       -- JSON encoded
  model_tier TEXT NOT NULL,
  created_at INTEGER NOT NULL,
  expires_at INTEGER NOT NULL,
  hit_count INTEGER DEFAULT 0
);
CREATE INDEX idx_cache_expires ON cache_entries(expires_at);

-- Routing history (memory-augmented routing)
CREATE TABLE routing_history (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  request_hash TEXT NOT NULL,   -- hash of normalized prompt
  intent TEXT NOT NULL,
  complexity TEXT NOT NULL,
  provider TEXT NOT NULL,
  model TEXT NOT NULL,
  success INTEGER NOT NULL,     -- 0 or 1
  latency_ms INTEGER,
  tokens INTEGER,
  created_at INTEGER NOT NULL
);
CREATE INDEX idx_history_hash ON routing_history(request_hash);
CREATE INDEX idx_history_created ON routing_history(created_at);

-- Circuit breaker state
CREATE TABLE circuit_state (
  provider TEXT PRIMARY KEY,
  state TEXT NOT NULL,          -- CLOSED, OPEN, HALF_OPEN
  failure_count INTEGER DEFAULT 0,
  last_failure_at INTEGER,
  open_until INTEGER
);
```

### 7.2 L3 Semantic Cache (hnswlib, at `/data/semantic_cache/`)

```
/data/semantic_cache/
├── index.bin          HNSW index (cosine similarity)
├── metadata.json      Mapping: vector_id → {key, response_ref, created_at, expires_at}
└── responses/
    └── {key}.json     Cached responses (referenced from metadata)
```

### 7.3 forge-quality State Files

```
project root/
├── .secrets.baseline      detect-secrets baseline (committed, tracks known non-secrets)
├── .gitleaks.toml         gitleaks config + allowlist (committed)
├── .husky/
│   ├── pre-commit
│   ├── commit-msg
│   ├── pre-push
│   └── post-commit
└── lefthook.yml           parallel hook definitions (committed)
```

---

## 8. GitHub Actions CI

### ci.yml — runs on every PR
```yaml
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm install
      - run: pnpm turbo check   # runs lint + typecheck + test across all packages

  router-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker compose up -d router
      - run: pnpm --filter simplellmrouter test:integration
```

### security.yml — runs on push to main
```yaml
jobs:
  security-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }     # full history for gitleaks
      - run: pnpm dlx @aigency/forge-quality audit
```

---

## 9. Environment Variables

### Router (.env)
```bash
# Provider API Keys (never committed)
GEMINI_API_KEY=
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
DEEPSEEK_API_KEY=
TOGETHER_API_KEY=
GROQ_API_KEY=
MISTRAL_API_KEY=
COHERE_API_KEY=

# Router Config
PORT=8080
LOG_LEVEL=info                        # debug | info | warn | error
PROVIDERS_FILE=./providers.json
DATA_DIR=./data

# Cache Config
CACHE_L1_ENABLED=true
CACHE_L2_ENABLED=true
CACHE_L3_ENABLED=true
CACHE_L3_THRESHOLD=0.92               # semantic similarity threshold

# Routing Config
DEFAULT_STRATEGY=balanced             # quality | cost | latency | balanced | shuffle
CASCADE_ENABLED=true
CASCADE_THRESHOLD=0.75

# OptiLLM (optional)
OPTILLM_ENABLED=false
OPTILLM_URL=http://localhost:8000
OPTILLM_THRESHOLD=COMPLEX             # SIMPLE | MEDIUM | COMPLEX | REASONING

# Security
ROUTER_AUTH_TOKEN=                    # optional: require bearer token on router API
```

### TUI (.env)
```bash
ROUTER_URL=http://localhost:8080
ROUTER_AUTH_TOKEN=                    # if router auth is enabled
TUI_REFRESH_INTERVAL=2000             # ms
TUI_THEME=dark                        # dark | light
```
