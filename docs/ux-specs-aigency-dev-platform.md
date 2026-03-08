# UI/UX Specification
## SimpleLLMRouter v2 TUI + @aigency/forge-quality CLI

**Version:** 1.0 | **Date:** 2026-03-05 | **Owner:** Antonio Reid
**Status:** Approved | **Applies to:** router-tui (Textual) + forge CLI

---

## 1. Design Principles

### 1.1 Core Principles

**Information density over decoration.** Every pixel (character cell) earns its place.
The TUI is a monitoring tool, not a landing page. Show the data.

**Reactive, not polled.** UI state updates the moment data changes. No manual refresh.
The user should never wonder "is this current?" — if it's on screen, it's live.

**Errors are actionable.** Every error state tells the user exactly what went wrong and
what to do next. "Error connecting to provider" is not acceptable. "Gemini: 429 rate limit
— circuit open for 120s, routing to alternatives" is.

**Speed as a feature.** Boot < 3s. Hook < 30s. Navigation instantaneous. Users feel
the tool respecting their time.

**Brand without noise.** Aigency's purple/violet color identity appears in key moments
(boot, headers, highlights) without overwhelming the functional interface.

### 1.2 Color System

```
Background:     #1a1a2e  (deep navy — primary bg)
Surface:        #16213e  (card/panel bg)
Surface-raised: #0f3460  (elevated panels, active states)
Border:         #2d2d4a  (subtle panel borders)

Brand-primary:  #7c3aed  (Aigency violet — logo, key headers)
Brand-accent:   #a855f7  (lighter purple — highlights, hover)

Status-green:   #22c55e  (healthy, success, online)
Status-yellow:  #f59e0b  (warning, approaching limit)
Status-red:     #ef4444  (error, circuit open, blocked)
Status-blue:    #3b82f6  (info, cache hit, neutral)
Status-cyan:    #06b6d4  (active, processing, live)

Text-primary:   #f1f5f9  (main content)
Text-secondary: #94a3b8  (labels, metadata)
Text-muted:     #475569  (timestamps, less important)

Code-highlight: #fbbf24  (amber — provider names, model names)
```

### 1.3 Typography Rules

- **Headers:** Bold, brand-primary or text-primary
- **Labels:** text-secondary, no bold
- **Values/data:** text-primary, sometimes code-highlight for provider/model names
- **Timestamps:** text-muted, right-aligned
- **Errors:** status-red, preceded by `[!]`
- **Success:** status-green, preceded by `[✓]`
- **Warnings:** status-yellow, preceded by `[⚠]`
- **Processing:** status-cyan, animated spinner `⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏`

---

## 2. TUI — Global Layout

### 2.1 Chrome (persistent across all screens)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ◆ SimpleLLMRouter  ●LIVE  8 providers  ▲1,247 req  ⏱ 4h32m  [?] [q]   │  ← Header bar
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                    [ SCREEN CONTENT AREA ]                               │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│  [1]Dashboard  [2]Live  [3]Quota  [4]Metrics  [5]Config  [6]Cache  [9]Log│  ← Nav bar
└──────────────────────────────────────────────────────────────────────────┘
```

**Header bar fields:**
- `◆ SimpleLLMRouter` — logo mark + app name (brand-primary)
- `●LIVE` — pulsing dot (status-cyan when connected, status-red when SSE disconnected)
- `8 providers` — count of healthy providers
- `▲1,247 req` — total requests this session
- `⏱ 4h32m` — uptime
- `[?]` — opens keyboard shortcuts overlay
- `[q]` — quit with confirmation

**Nav bar:** Active screen highlighted with surface-raised bg + brand-accent text.
All other items: text-secondary. Keyboard shortcut number shown inline.

### 2.2 Keyboard Map (global)

| Key | Action |
|-----|--------|
| `1`–`9` | Jump to screen directly |
| `Tab` / `Shift+Tab` | Cycle screens forward/back |
| `?` | Toggle keyboard help overlay |
| `q` | Quit (shows confirmation: "Quit SimpleLLMRouter? [y/N]") |
| `Ctrl+R` | Force refresh all data |
| `Esc` | Close modal / cancel / back |
| `Space` | Pause/resume (on Live Inference screen) |
| `/` | Focus search/filter (on screens with filter) |
| `↑↓` | Scroll content |
| `Enter` | Select / expand item |

---

## 3. TUI — Screen Specifications

### Screen 0: Boot Sequence

**Trigger:** App launch. Auto-transitions to Dashboard when all systems ready.
**Duration:** 2–3 seconds (real boot time, no artificial delay)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│                                                                          │
│              ░░░   ░░░░  ░░░░  ░░░░  ░░░  ░░░  ░░░  ░░░░               │
│             ░░ ░░ ░░    ░░    ░░  ░░ ░░░░ ░░░░ ░░░░ ░░                  │
│             ░░░░░ ░░░░  ░░ ░░ ░░░░░ ░░ ░░░░ ░░ ░░░  ░░░░               │
│             ░░ ░░ ░░    ░░ ░░ ░░  ░░ ░░  ░░  ░░ ░░░░    ░░             │
│             ░░ ░░ ░░░░░ ░░░░  ░░  ░░ ░░  ░░  ░░ ░░░  ░░░░              │
│                                                                          │
│                    SimpleLLMRouter  v2.0.0                               │
│             Free-tier intelligent routing for autonomous AI              │
│                                                                          │
│         ████████████████████████████████████████░░░░░░░░  84%          │
│                                                                          │
│         [✓] Provider registry loaded          (8 providers)             │
│         [✓] Quota tracker initialized         (247M tokens used)        │
│         [✓] Cache L1 ready                    (warm: 0 entries)         │
│         [✓] Cache L2 ready                    (warm: 234 entries)       │
│         [✓] Cache L3 semantic ready           (warm: 89 vectors)        │
│         [→] Building intent detection index...  ⠙                       │
│                                                                          │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Animation sequence:**
1. `(0ms)` Black screen
2. `(0–800ms)` ASCII logo reveals left-to-right, character by character — brand-primary color
3. `(800–1200ms)` Version + tagline fade in — text-secondary
4. `(1200ms)` Progress bar appears at 0%
5. `(1200–Nms)` Each subsystem initializes; spinner → checkmark transition per line
6. `(N+200ms)` Progress bar hits 100%, brief flash (white → brand-primary → normal)
7. `(N+400ms)` Screen fades out, Dashboard fades in

**Error state during boot:**
```
│         [!] Provider: Groq — API key missing (GROQ_API_KEY not set)     │
│             Continuing with 7 providers                                  │
```
Yellow `[!]` warning, not a blocker — boot continues.

**Fatal error (no providers at all):**
```
│         [✗] FATAL: No providers configured or reachable.                │
│             Set at least one API key in .env and restart.                │
```
Red `[✗]`, boot halts, press any key to exit.

---

### Screen 1: Main Dashboard

**Purpose:** Runtime home screen. All critical metrics visible without drilling down.
**Auto-refresh:** Every 2 seconds (reactive via SSE, not polling)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ◆ SimpleLLMRouter  ●LIVE  8 providers  ▲1,247 req  ⏱ 4h32m  [?] [q]   │
├───────────────────────────┬───────────────────────┬──────────────────────┤
│  SYSTEM STATUS            │  QUOTA OVERVIEW        │  ROUTING             │
│                           │                        │                      │
│  Router    ● ACTIVE       │  Gemini Flash ████░ 73%│  Mode:  BALANCED     │
│  Cache     ● WARM         │  Gemini Pro   ██░░░ 41%│  Strat: SEMANTIC     │
│  OptiLLM  ● ENABLED       │  GPT-4o-mini  █░░░░ 18%│  Cache: L1+L2+L3    │
│  Queue     12 pending     │  Claude Haiku ░░░░░  2%│  OptiL: COMPLEX+     │
│  Workers   4 active       │  DeepSeek     ░░░░░  1%│  Casc:  ON (0.75)   │
│                           │  Together     ████░ 68%│                      │
│  Errors    0.2%           │  Groq         █████ 99%│  [Edit Strategy]     │
│  P50       89ms           │  Mistral      ░░░░░  8%│                      │
│  P95       340ms          │                        │  ⚠ Groq near limit   │
├───────────────────────────┴───────────────────────┴──────────────────────┤
│  ACTIVITY — LAST 60 SECONDS                                              │
│                                                                          │
│  Req/s  ▁▁▂▃▃▅▃▂▄▆▅▃▂▁▂▃▄▅▆▄▃▂▁▂▃▄▅▆▇▆▅▄▃▂▃▄▃▂▁▂▃▄▅▃▂▁▂▃▂▁▁▂▃▄▅      │
│         0s                          30s                         60s      │
│                                                                          │
│  Cache hit rate: 61%  │  Tokens saved today: 48,320  │  Err rate: 0.2%  │
├──────────────────────────────────────────────────────────────────────────┤
│  RECENT REQUESTS                                                         │
│                                                                          │
│  14:32:07  [✓] CODE_REVIEW    gemini-2.0-flash   89ms   1,247 tok  L2↑  │
│  14:32:06  [↑] CACHE HIT L2   —                   2ms     890 tok  —    │
│  14:32:04  [✓] ARCHITECTURE   deepseek-r1      1,203ms  4,891 tok  —    │
│  14:32:01  [✓] BUG_FIX        gpt-4o-mini       203ms    782 tok  —    │
│  14:31:58  [!] REASONING      gpt-4o (cascade)   891ms  2,341 tok  esc  │
│  14:31:55  [✓] SIMPLE_CHAT    gemini-2.0-flash    44ms    123 tok  L1↑  │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Widget details:**

*System Status panel:*
- Each status dot pulses subtly when its subsystem is active
- Queue depth turns yellow > 20, red > 50
- Errors % turns yellow > 1%, red > 5%

*Quota Overview panel:*
- Progress bars: green < 80%, yellow 80-95%, red > 95%
- Providers sorted by usage % descending
- Warning badge appears for any provider > 90%

*Routing panel:*
- `[Edit Strategy]` button opens Router Config screen (Screen 6)
- All values are live; strategy change reflects immediately

*Activity sparkline:*
- 60 data points, one per second
- Bar height proportional to req/s (auto-scaled)
- Color: cyan normally, yellow on high load, red on errors

*Recent Requests feed:*
- Icons: `[✓]` success, `[↑]` cache hit, `[!]` fallback/cascade used, `[✗]` error
- `L1↑` / `L2↑` / `L3↑` badge on cache hits
- `esc` badge = cascade escalation occurred
- Click any row → opens detail view overlay

---

### Screen 2: Live Inference Stream

**Purpose:** Real-time window into every routing decision. The most compelling screen.
**Key metaphor:** Watching the router think out loud.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ◆ SimpleLLMRouter  ●LIVE  8 providers  ▲1,247 req  ⏱ 4h32m  [?] [q]   │
├──────────────────────────────────────────────────────────────────────────┤
│  LIVE INFERENCE STREAM              [SPACE: Pause]  [/: Filter▼]  [↑↓]  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  14:32:07.441  ● NEW REQUEST  ▸ (click to expand)                       │
│  ├─ Intent:     CODE_REVIEW       confidence: 0.94  ████████████████░░░ │
│  ├─ Complexity: MEDIUM            score: 0.61       ████████████░░░░░░░ │
│  ├─ Dimensions: code↑ logic↑ context↑  creative↓ safety↓               │
│  ├─ Cache:      MISS              semantic sim: 0.71 (< 0.92 threshold) │
│  ├─ Strategy:   BALANCED → candidates: gemini-pro, gpt-4o-mini          │
│  ├─ Quota:      gemini-pro OK(41%)  gpt-4o-mini OK(18%)                 │
│  ├─ Selected:   gemini-2.0-pro    score: 0.87                           │
│  └─ Result:     ✓ SUCCESS  89ms TTFB  1,247 tokens                      │
│                                                                          │
│  ──────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  14:32:06.102  ◈ CACHE HIT L2  ▸ (collapsed)                           │
│                                                                          │
│  ──────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  14:32:04.891  ● NEW REQUEST  ▸ (click to expand)                       │
│  ├─ Intent:     ARCHITECTURE      confidence: 0.89                      │
│  ├─ Complexity: REASONING         score: 0.88  ████████████████████░░░  │
│  ├─ OptiLLM:    ENABLED           technique: mcts                       │
│  ├─ Cache:      MISS                                                     │
│  ├─ Selected:   deepseek-r1       (reasoning specialist)                │
│  └─ Result:     ✓ SUCCESS  1,203ms TTFB  4,891 tokens                   │
│                                                                          │
│  ──────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  14:32:01.334  ⚠ CASCADE ESCALATION  ▸ (click to expand)               │
│  ├─ Intent:     REASONING         confidence: 0.91                      │
│  ├─ Attempt 1:  gemini-2.0-flash  quality score: 0.61 < 0.75 threshold  │
│  ├─ Escalating: → gpt-4o                                                │
│  └─ Result:     ✓ SUCCESS (via cascade)  891ms  2,341 tokens            │
│                                                                          │
│  ─── 4 requests shown — 1,243 in history ──── [Load more ↑] ────────── │
└──────────────────────────────────────────────────────────────────────────┘
```

**Interaction details:**

*Collapsible trees:*
- Default: most recent request expanded, all others collapsed
- Click header row or press `Enter` to toggle expand/collapse
- Expanded = shows all decision tree nodes
- Collapsed = shows one-line summary: timestamp + type + provider + result

*Row icons:*
- `●` = normal routed request (cyan)
- `◈` = cache hit (blue)
- `⚠` = cascade escalation or warning (yellow)
- `✗` = error (red)

*Pause mode:*
- `Space` pauses new events from appearing (buffer continues filling)
- Header shows `[PAUSED — N buffered]` in yellow
- `Space` again resumes and flushes buffer
- Useful for inspecting a specific request tree

*Filter dropdown (`/`):*
```
┌─────────────────────────┐
│  Filter requests:        │
│  ○ All                  │
│  ○ Errors only          │
│  ○ Cache hits only      │
│  ○ Cascades only        │
│  ─────────────────────  │
│  By Intent:             │
│  ○ CODE_REVIEW          │
│  ○ ARCHITECTURE         │
│  ○ BUG_FIX              │
│  ○ REASONING            │
│  ○ SIMPLE_CHAT          │
│  ─────────────────────  │
│  By Provider:           │
│  ○ gemini-2.0-flash     │
│  ○ gpt-4o-mini          │
│  ...                    │
└─────────────────────────┘
```

---

### Screen 3: Quota Manager

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ◆ SimpleLLMRouter  ●LIVE  8 providers  ▲1,247 req  ⏱ 4h32m  [?] [q]   │
├──────────────────────────────────────────────────────────────────────────┤
│  QUOTA MANAGER                              Resets: daily at 00:00 UTC  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Provider          Model Tier    Used Today     Limit     % Used  State  │
│  ──────────────────────────────────────────────────────────────────────  │
│  Gemini Flash      free         ████████░░░  812K/1M      73%    ● OK   │
│  Gemini Pro        free         ████░░░░░░░  201K/1M      41%    ● OK   │
│  GPT-4o-mini       free         ██░░░░░░░░░  184K/1M      18%    ● OK   │
│  Claude Haiku      free         ░░░░░░░░░░░   89K/∞        —     ● OK   │
│  DeepSeek Chat     free         ░░░░░░░░░░░   12K/∞        —     ● OK   │
│  Together.ai       free         ███████░░░░  512K/1M      68%    ● OK   │
│  Groq Llama        free         ██████████░  987K/1M      99%    ⚠ WARN │
│  Mistral           free         ░░░░░░░░░░░    8K/∞        —     ● OK   │
│                                                                          │
│  Circuit Breakers:  All CLOSED ● (healthy)                              │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│  MONTHLY AGGREGATE                                                       │
│                                                                          │
│  Total capacity:     ~1.04B tokens/month                                 │
│  Used this month:    247M tokens  (23.8%)   ████░░░░░░░░░░░░░░░░░░░    │
│  Saved via cache:     89M tokens  ( 8.6%)                               │
│  Effective used:     158M tokens  (15.2%)                               │
│  Projected EOM:      891M / 1.04B (85.7%)  — SAFE                       │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│  [A: Add Provider]  [E: Edit Selected]  [D: Disable]  [T: Test]  [R: Reset] │
└──────────────────────────────────────────────────────────────────────────┘
```

**Provider actions:**
- `A` — opens Add Provider modal (name, base_url, api_key env var, models, limits)
- `E` — opens Edit modal for selected provider (arrow keys to select row)
- `D` — toggles provider enabled/disabled (confirmation prompt)
- `T` — fires a test request to selected provider, shows latency + status
- `R` — manual quota reset for selected provider (confirmation: "Reset Groq quota? [y/N]")

**Circuit breaker detail (expand any provider):**
```
│  Groq Llama  ▸ expanded                                                  │
│  ├─ Circuit:     CLOSED (healthy)                                        │
│  ├─ Failures:    0 in last 60s                                           │
│  ├─ Quota:       987K / 1M (99%) — ⚠ routing deprioritized             │
│  ├─ Reset in:    6h 14m                                                  │
│  └─ Models:      llama-3.1-70b, llama-3.1-8b, mixtral-8x7b             │
```

---

### Screen 4: Metrics Deep Dive

Four tabs, navigable with `←→` or `Tab`:

**Tab A: Overview**
```
  Request volume (last 24h — hourly bars):
  ▁▁▂▃▄▅▆▇█▇▆▅▄▃▂▁▁▂▃▄▅▆▇▆▅▄▃▂▃▄▅▆▇█▇▆▅▄▃▂▁▁▂▃▄
  00  02  04  06  08  10  12  14  16  18  20  22  00

  Success rate:  99.8%   Error rate:  0.2%   Fallback rate: 3.1%
  Cache hit L1:   12%    Cache hit L2:  31%  Cache hit L3:  18%
  Total cache:    61%    Tokens saved: 48,320 today  (est. $0.97 saved)
```

**Tab B: Routing Dimensions**
```
  14-DIMENSION ANALYSIS  (last 1,000 requests)

  Dimension          Triggered   Avg Score  Influence
  ────────────────────────────────────────────────────
  code                  87%        0.81       ████████  HIGH
  logic                 74%        0.69       ███████   HIGH
  context_length        92%        0.91       ████████  MED
  reasoning             61%        0.58       ██████    HIGH
  instruction_follow    83%        0.79       ███████   HIGH
  cost                  91%        0.88       ████████  HIGH
  speed                 78%        0.74       ███████   MED
  context_window        67%        0.62       ██████    MED
  tool_calling          55%        0.51       █████     MED
  creative              43%        0.39       ████      LOW
  math                  38%        0.34       ███       MED
  safety                99%        0.99       CRITICAL  CRIT
  multilingual          12%        0.09       █         LOW
  vision                 8%        0.06       █         LOW
```

**Tab C: Provider Performance**
```
  Provider           Avg Lat  P95 Lat  Error%  Tok/s    Quality
  ─────────────────────────────────────────────────────────────
  Gemini Flash         44ms    180ms    0.1%   2,400     0.82
  Gemini Pro          189ms    520ms    0.3%   1,800     0.91
  GPT-4o-mini         203ms    610ms    0.2%   1,200     0.88
  Claude Haiku        167ms    490ms    0.1%   1,600     0.89
  DeepSeek Chat       312ms    890ms    0.4%     900     0.87
  Together.ai         156ms    450ms    0.5%   2,100     0.84
  Groq Llama           38ms    120ms    0.8%   3,200     0.79
  Mistral             201ms    590ms    0.2%   1,400     0.85
```

**Tab D: Cache ROI**
```
  Cache Performance:
  L1 hits today:  1,247  avg saved: 1,100 tok each  →  1.37M tokens saved
  L2 hits today:  3,891  avg saved:   890 tok each  →  3.46M tokens saved
  L3 hits today:  1,103  avg saved: 1,240 tok each  →  1.37M tokens saved

  Total tokens saved:  6.2M today  /  89M this month
  Est. cost savings:   $0.97 today  /  $14.20 this month
  (at GPT-4o-mini pricing $0.15/1M tokens)

  Cache efficiency over time (last 7 days):
  Day      Hits    Miss    Rate
  Mon      4,891   2,103   70%   ██████████████
  Tue      5,102   2,891   64%   █████████████
  Wed      6,234   1,891   77%   ████████████████
  Thu      7,891   1,203   87%   █████████████████
  ...
```

---

### Screen 5: Provider Config (Modal)

Opens as a centered modal overlay on any screen.

```
┌──────────────────────────────────────────────┐
│  EDIT PROVIDER: Gemini Flash                 │
├──────────────────────────────────────────────┤
│                                              │
│  Display name:   [Gemini Flash           ]   │
│  Provider ID:    gemini-flash                │
│  Base URL:       [https://generativelang]    │
│  API Key Env:    [GEMINI_API_KEY          ]  │
│                                              │
│  Models (one per line):                      │
│  [gemini-2.0-flash-exp                   ]   │
│  [gemini-2.0-flash                       ]   │
│                                              │
│  Daily token limit:  [1000000            ]   │
│  Requests/minute:    [60                 ]   │
│  Canary weight:      [0                  ]   │
│  (0 = not in canary; 0.1 = 10% of traffic)  │
│                                              │
│  Circuit breaker:                            │
│  Failure threshold:  [3] within [60]s        │
│  Cooldown:           [120]s                  │
│                                              │
│  Enabled:  [✓] Yes                          │
│                                              │
│  [Test Connection]    [Save]    [Cancel]      │
└──────────────────────────────────────────────┘
```

---

### Screen 6: Router Config

```
┌──────────────────────────────────────────────────────────────────────────┐
│  ROUTER CONFIGURATION                              Changes apply live    │
├─────────────────────────────────┬────────────────────────────────────────┤
│  ROUTING STRATEGY               │  CACHE SETTINGS                        │
│                                 │                                        │
│  Active:  ◉ balanced            │  L1 Memory:   [✓] ON   TTL: [300]s    │
│           ○ quality             │  L2 Disk:     [✓] ON   TTL: [86400]s  │
│           ○ cost                │  L3 Semantic: [✓] ON   TTL: [259200]s │
│           ○ latency             │  L3 threshold:[0.92                 ]  │
│           ○ shuffle             │                                        │
│                                 │  [Flush L1]  [Flush L2]  [Flush All]  │
│  CASCADE FALLBACK               ├────────────────────────────────────────┤
│  Enabled:   [✓] ON              │  OPTILLM                               │
│  Threshold: [0.75           ]   │                                        │
│  (0.0–1.0 quality score)        │  Enabled:   [✓] ON                    │
│                                 │  Threshold: ◉ COMPLEX                 │
│  Cascade chain:                 │             ○ REASONING only          │
│  1. gemini-2.0-flash            │             ○ ALL requests            │
│  2. gemini-2.0-pro              │                                        │
│  3. gpt-4o-mini                 │  URL: [http://localhost:8000       ]   │
│  4. gpt-4o                      │                                        │
│  [Edit chain]                   │  Status:  ● Connected  (42ms)         │
│                                 ├────────────────────────────────────────┤
│  QUOTA GUARDS                   │  MEMORY ROUTING                        │
│  Warning:    [80] %             │                                        │
│  Hard stop:  [95] %             │  Enabled:    [✓] ON                   │
│  Reset time: [00:00] UTC        │  Similarity: [0.88              ]      │
│                                 │  History:    9,847 decisions stored    │
│                                 │  [Clear History]                       │
│                                 │                                        │
│  [Save All Changes]             │                  [Reset to Defaults]   │
└─────────────────────────────────┴────────────────────────────────────────┘
```

All changes dispatch a REST PATCH to `/config` immediately on field change (debounced 500ms).
A `[✓ Saved]` flash appears in top-right for 2s after each successful save.

---

### Screen 7: Cache Inspector

```
┌──────────────────────────────────────────────────────────────────────────┐
│  CACHE INSPECTOR                                                         │
├──────────────────────────────────┬───────────────────────────────────────┤
│  LAYER STATS                     │  BROWSE ENTRIES  [/: search]          │
│                                  │                                       │
│  L1 Memory:  187 entries / 1000  │  Key (truncated)       TTL   Hits    │
│  Size: 2.3MB                     │  ──────────────────────────────────  │
│  Oldest: 4m 12s ago              │  sha256:a3f2...b891  4m12s    3      │
│  Hit rate today: 12%             │  sha256:c891...a234  3m44s    1      │
│  [Flush L1]                      │  sha256:f234...d567  2m01s    7      │
│                                  │  sha256:9abc...e012  1m23s    2      │
│  L2 Disk:  3,891 entries         │  sha256:1234...5678  0m44s   12      │
│  Size: 48MB / 10GB               │                                       │
│  Oldest: 23h 44m ago             │  [Enter: view entry detail]          │
│  Hit rate today: 31%             │                                       │
│  [Flush L2]                      │  ENTRY DETAIL (selected):            │
│                                  │  ────────────────────────────────    │
│  L3 Semantic: 1,103 vectors      │  Intent:     CODE_REVIEW             │
│  Index size: 12MB                │  Complexity: MEDIUM                  │
│  Oldest: 71h 12m ago             │  Model tier: mid                     │
│  Hit rate today: 18%             │  Prompt:     "Review this TypeSc..."  │
│  Threshold: 0.92                 │  Response:   "The code looks good..." │
│  [Flush L3]                      │  Created:    2026-03-04 18:32:07     │
│                                  │  Expires:    2026-03-07 18:32:07     │
│  [Flush ALL Caches]              │  Hit count:  7                        │
│                                  │  [Delete Entry]                      │
└──────────────────────────────────┴───────────────────────────────────────┘
```

---

### Screen 8: Log Viewer

```
┌──────────────────────────────────────────────────────────────────────────┐
│  LOGS    Level: [INFO▼]  Component: [ALL▼]  [/: search]  [Auto-scroll✓] │
├──────────────────────────────────────────────────────────────────────────┤
│  14:32:07.441  INFO   router      REQUEST code_review → gemini-2.0-pro  │
│  14:32:07.441  INFO   cache       L2 MISS sha256:a3f2...b891            │
│  14:32:07.441  INFO   classifier  MEDIUM (0.61) — code↑ logic↑          │
│  14:32:07.531  INFO   provider    gemini-2.0-pro dispatched              │
│  14:32:07.620  INFO   provider    TTFB 89ms                              │
│  14:32:07.821  INFO   router      SUCCESS 1247 tokens → caching L1+L2   │
│  14:32:07.821  INFO   quota       gemini-pro: 201001/1000000 (20.1%)    │
│  14:32:08.102  INFO   cache       L2 HIT sha256:c891 → served 2ms       │
│  14:32:08.209  WARN   quota       groq: 987000/1000000 (98.7%) — WARN   │
│  ...                                                                     │
│                                                                          │
│  [E: Export to file]           Showing: 47 / 12,847 total log lines     │
└──────────────────────────────────────────────────────────────────────────┘
```

Log level filter: DEBUG / INFO / WARN / ERROR (cumulative — ERROR shows only errors, etc.)
Component filter: ALL / router / cache / classifier / provider / quota / optillm / sse

---

## 4. forge CLI — UX Specification

### 4.1 Output Design Principles

**Structured, not chatty.** Every line is signal. No filler like "Great! Now I'll..." or
"Almost done...". Show what happened, clearly.

**Color codes meaning:**
- Green `✓` = success / completed
- Yellow `⚠` = warning / skippable issue
- Red `✗` = error / blocked
- Cyan `→` = in progress / next step
- Blue `ℹ` = informational / tip
- Bold white = section headers

**Width:** Designed for 80-column terminals. Works at any width ≥ 80.

---

### 4.2 forge init — Full Output

```
$ forge init --github --template=ts

  forge-quality v1.0.0 — Project Quality Toolkit
  ─────────────────────────────────────────────────

  Detecting project context...
  → TypeScript project detected
  → Standalone project (not inside a monorepo)

  Step 1/11  Git Setup
  ✓ git init (already initialized)
  ✓ .gitignore written (243 patterns — env, secrets, build artifacts)
  ✓ Default branch set to main
  ✓ GPG signing: not configured (skip)

  Step 2/11  GitHub Repository
  → Checking for GitHub CLI... ✓ gh 2.40.0
  → Creating repository: my-project (private)
  ✓ Repository created: github.com/AReid987/my-project
  ✓ Remote origin set
  ✓ Branch protection enabled:
    • Require PR review (1 reviewer)
    • Require status checks to pass
    • Restrict direct push to main

  Step 3/11  Hook Toolchain
  ✓ lefthook 1.5.2 installed
  ✓ husky 9.0.0 installed (fallback)
  ✓ .husky/ initialized

  Step 4/11  Linters
  ✓ eslint 8.56.0 installed
  ✓ @typescript-eslint/parser installed
  ✓ prettier 3.2.0 installed
  ✓ .eslintrc.json written (extends @aigency/forge-quality/eslint)
  ✓ .prettierrc written

  Step 5/11  Type Checking
  ✓ typescript 5.3.3 installed
  ✓ tsconfig.json written (extends @aigency/forge-quality/tsconfig)

  Step 6/11  Test Runner
  ✓ vitest 1.2.0 installed
  ✓ @vitest/coverage-v8 installed
  ✓ vitest.config.ts written
  ✓ Coverage thresholds: lines 80%, functions 80%, branches 70%

  Step 7/11  Security Scanners
  ✓ gitleaks 8.18.0 detected (system install)
  ✓ detect-secrets 1.4.0 installed (pip)
  ✓ semgrep 1.50.0 installed (pip)
  ✓ audit-ci 6.6.0 installed
  ✓ .gitleaks.toml written
  ✓ .secrets.baseline created (0 known secrets)

  Step 8/11  Commit Toolchain
  ✓ @commitlint/cli installed
  ✓ @commitlint/config-conventional installed
  ✓ commitizen installed
  ✓ commitlint.config.js written
  ✓ .cz-config.js written (15 Aigency scopes)

  Step 9/11  Hook Definitions
  ✓ lefthook.yml written (pre-commit, commit-msg, pre-push, post-commit)
  ✓ Hooks installed into .husky/

  Step 10/11  Configuration Files
  ✓ All configs written and validated

  Step 11/11  First Commit
  ✓ git add . (47 files staged)
  ✓ git commit: "chore: initialize forge-quality toolchain"
  ✓ git push origin main

  ─────────────────────────────────────────────────
  ✓ Done in 34s. Your project is ready.

  What's next:
  • Write code, then:  forge commit
  • Check quality:     forge check
  • Create a PR:       forge pr
  • Security scan:     forge audit

  ℹ Hooks are active. Your next commit will run the full quality pipeline.
```

---

### 4.3 Pre-Commit Hook — Output Examples

**Happy path (all passing):**
```
[forge] Running pre-commit checks...

  ✓ Format (prettier + eslint --fix)    2 files auto-fixed, re-staged
  ✓ Type check (tsc)                    0 errors
  ✓ Secret scan (gitleaks)              clean
  ✓ Tests (changed files)               14 tests passed
  ✓ Coverage (changed files)            91% (threshold: 80%) ✓

  3 files committed
```

**Type error (blocks commit):**
```
[forge] Running pre-commit checks...

  ✓ Format (prettier + eslint --fix)    0 changes needed
  ✗ Type check (tsc)                    1 error

  src/router/classifier.ts:47:12
  Type 'string' is not assignable to type 'ComplexityTier'.

  Commit blocked. Fix the error above and try again.
  Cannot auto-fix type errors — fix manually.
```

**Secret detected (hard block):**
```
[forge] Running pre-commit checks...

  ✓ Format (prettier + eslint --fix)    0 changes needed
  ✗ SECRET DETECTED — commit blocked

  File:    .env.local
  Line:    3
  Pattern: generic-api-key
  Match:   OPENAI_API_KEY=sk-proj-...

  Remove the secret before committing.
  Add to .gitignore if this file should never be committed.
  Never use --no-verify to bypass secret scanning.
```

**Coverage drop (blocks commit):**
```
[forge] Running pre-commit checks...

  ✓ Format                              clean
  ✓ Type check                          0 errors
  ✓ Secrets                             clean
  ✓ Tests                               8 tests passed
  ✗ Coverage dropped below threshold

  Before: 87%  After: 74%  Threshold: 80%
  Uncovered lines in: src/router/cascade.ts (lines 34-67)

  Add tests for the new code paths and try again.
```

---

### 4.4 commit-msg Hook — Output Example

**Valid message:** (silent, no output)

**Invalid message:**
```
[forge] Validating commit message...

  ✗ Invalid commit message format

  Your message:  "fixed the bug with routing"
  Problem:       Missing type prefix

  Required format:  type(scope): subject
  Valid types:      feat fix docs style refactor perf test build ci chore revert
  Example:          fix(router): resolve cascade fallback threshold comparison

  Run `forge commit` for an interactive prompt.
```

---

### 4.5 forge commit — Interactive Prompt

```
$ forge commit

  forge commit — Conventional Commit Builder
  ───────────────────────────────────────────

  ? Commit type:
  ❯ feat      — A new feature
    fix       — A bug fix
    docs      — Documentation only
    refactor  — Code change (no feature/fix)
    test      — Adding or fixing tests
    perf      — Performance improvement
    chore     — Build/tooling/dependencies
    ci        — CI configuration
    revert    — Revert a previous commit

  ? Scope (optional):
  ❯ router
    tui
    forge
    cache
    quota
    classifier
    intent
    cascade
    hooks
    config
    deps
    security
    (other — type manually)
    (none)

  ? Short description (imperative, present tense):
  > add cascade fallback routing with quality threshold

  ? Longer description (optional, press Enter to skip):
  > Implements RouteLLM-style cascade: tries cheapest model first,
  > escalates to next tier if quality score < configurable threshold.

  ? Breaking change? (y/N): N

  ? Issues closed (e.g. #123, press Enter to skip): #42

  ───────────────────────────────────────────
  Preview:

  feat(router): add cascade fallback routing with quality threshold

  Implements RouteLLM-style cascade: tries cheapest model first,
  escalates to next tier if quality score < configurable threshold.

  Closes #42

  ───────────────────────────────────────────
  ? Confirm and commit? (Y/n): Y

  [forge] Running pre-commit checks...
  ✓ Format    ✓ Types    ✓ Secrets    ✓ Tests    ✓ Coverage

  ✓ Committed: feat(router): add cascade fallback routing with quality threshold
  Files: 4 changed | Tests: 23 passed | Coverage: 87% | Security: clean
  Next: git push  or  forge pr
```

---

### 4.6 forge pr — Output

```
$ forge pr

  forge pr — Pull Request Generator
  ────────────────────────────────────

  Branch:  feat/cascade-routing → main
  Commits: 7 (since branching 2 days ago)

  Analyzing commits...
  ✓ Commits parsed
  ✓ Coverage delta computed: 81% → 87% (+6%)
  ✓ Security scan: clean

  ────────────────────────────────────
  Generated PR Description:

  ## feat(router): Cascade Fallback Routing

  ### Summary
  Implements RouteLLM-style cascade routing with configurable quality
  thresholds. The router now attempts the cheapest qualifying model
  first and escalates to stronger models only when needed.

  ### Changes
  **Features**
  - feat(router): add cascade fallback routing with quality threshold
  - feat(router): add 5 routing strategy modes (quality/cost/latency/balanced/shuffle)
  - feat(tui): add live inference stream with collapsible decision trees

  **Fixes**
  - fix(quota): wire QuotaTracker into server.ts request handler

  **Refactors**
  - refactor(classifier): extract 14-dimension scorer into standalone module

  ### Test Coverage
  | | Before | After |
  |--|--|--|
  | Lines | 81% | 87% |
  | Functions | 78% | 84% |
  | Branches | 71% | 76% |

  ### Security
  ✓ No secrets detected (gitleaks + detect-secrets)
  ✓ No vulnerabilities found (semgrep + audit-ci)

  ### Checklist
  - [x] Tests pass
  - [x] Coverage maintained (87% > 80% threshold)
  - [x] No secrets or credentials committed
  - [ ] Documentation updated
  - [ ] Breaking changes documented

  ────────────────────────────────────
  ? Open PR in browser? (Y/n): Y
  → Opening: github.com/AReid987/simplellmrouter/compare/feat/cascade-routing
```

---

### 4.7 forge audit — Output

```
$ forge audit

  forge audit — Security Scan
  ────────────────────────────

  → Running gitleaks (full history)...
  ✓ gitleaks: 0 secrets found in 247 commits

  → Running detect-secrets...
  ✓ detect-secrets: 0 new secrets (2 known in baseline)

  → Running semgrep (SAST)...
  ✓ semgrep: 0 findings (scanned 47 files)

  → Running audit-ci (dependencies)...
  ✓ audit-ci: 0 vulnerabilities (moderate+ severity)

  ────────────────────────────────────
  ✓ All scans clean. No security issues found.

  Last full audit: now
  ────────────────────────────────────
```

**With findings:**
```
  → Running semgrep (SAST)...
  ⚠ semgrep: 2 findings

  src/providers/client.ts:89
  Rule: javascript.express.security.audit.xss.direct-response-write
  Severity: WARNING
  Message: Unsanitized input passed to response.write()
  Fix: Sanitize user input before writing to response

  src/cache/l2-disk.ts:134
  Rule: typescript.sequelize.security.audit.raw-sql
  Severity: WARNING
  Message: Raw SQL query with string interpolation
  Fix: Use parameterized queries

  ────────────────────────────────────
  ⚠ 2 warnings found. Review before merging to main.
  ℹ Run with --fix to attempt auto-remediation where possible.
```

---

## 5. Post-Commit Summary — Full Spec

```
[forge] ✓ Committed: feat(router): add cascade fallback routing

  ┌─────────────────────────────────────────────────┐
  │  Commit Summary                                 │
  │                                                 │
  │  Files:      4 changed (+234 −12 lines)         │
  │  Tests:      23 passed  0 failed  0 skipped     │
  │  Coverage:   87%  (+2% vs previous)             │
  │  Security:   clean  (staged files)              │
  │  Time:       12.4s                              │
  │                                                 │
  │  Next:  git push                                │
  │         forge pr  (generate PR description)     │
  └─────────────────────────────────────────────────┘
```

---

## 6. Accessibility & Compatibility

### TUI
- All information conveyed by color MUST also be conveyed by symbol (`●`, `⚠`, `✗`, `✓`)
- TUI MUST render correctly at 80×24 minimum terminal size
- `--no-animation` flag disables all animations for CI and screen readers
- `--no-color` flag renders in monochrome for terminals without color support
- TUI MUST work in: iTerm2, Terminal.app, Windows Terminal, GNOME Terminal, tmux, screen

### forge CLI
- All output MUST be readable without color (symbols carry meaning)
- `--json` flag on any command outputs structured JSON for programmatic use
- `--quiet` flag suppresses all output except errors (for CI pipelines)
- `--verbose` flag shows full command output for debugging
