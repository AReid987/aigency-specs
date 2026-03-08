# Aigency Integrations Spec v1.0

> Part of the Aigency Dev Platform architecture documentation.
> Defines all external service integrations, API patterns, and authentication strategies.
> See also: `architecture/agentic-tooling-integration-strategy.md` for the full tooling strategy.

---

## Overview

The Aigency platform integrates with external services across four categories:

1. **AI / LLM Providers** — All routed through SimpleLLMRouter
2. **Developer Tooling** — GitHub, Linear, Notion
3. **Communication** — Telegram, Discord, Slack, Gmail
4. **Data & Infrastructure** — Supabase, Google Workspace, Airtable

All integrations follow a consistent pattern:
- OAuth connections managed by Nebula AI
- API calls proxy through SimpleLLMRouter where applicable
- Credentials stored as environment variables, never hardcoded
- Agents never call provider APIs directly — always through the router or a dedicated agent

---

## 1. AI / LLM Provider Integrations

All LLM calls in the Meta Code Squad harness route through SimpleLLMRouter v2 at `http://localhost:8080`.

| Provider | Models | Usage | Auth |
|----------|--------|-------|------|
| Anthropic | claude-3-5-sonnet, claude-3-opus | Complex logic, state machines (Ruflo/CLAUDE) | API key via env |
| Google Gemini | gemini-2.5-pro | Architecture, large-context synthesis (Gemini CLI) | API key via env |
| Moonshot Kimi | kimi-latest | Active coding, code review sweeps | API key via env |
| OpenAI | gpt-4o, gpt-4o-mini | Overflow routing, embeddings | API key via env |
| Qwen / Roo | qwen-coder | Quota overflow buffer | API key via env |
| Letta | Internal | Stateful memory agent at :8283 | Local server |

### SimpleLLMRouter v2 Routing Rules

See `docs/simplellmrouter-v2-spec.md` for full routing configuration. Summary:

```
Complex logic / security / state machines  -->  Anthropic (Claude)
Architecture / planning / synthesis        -->  Google Gemini 2.5 Pro
Active coding / multi-step execution       -->  Moonshot Kimi
Quota overflow                             -->  Qwen / Roo / iFlow
Embeddings / fast lookups                  -->  OpenAI
```

---

## 2. Developer Tooling Integrations

### GitHub

| Property | Value |
|----------|-------|
| Account | AReid987 |
| Auth | OAuth (connected via Nebula) |
| Primary repos | aigency-specs, meta-code-squad, simplellmrouter |
| Usage | Source control, CI/CD, issue tracking, PR reviews |
| Agent | github-agent (Nebula) |

Key workflows:
- All code changes via PR — no direct pushes to main
- CI runs on every PR: lint, typecheck, CodeQL security scan
- Dependabot monitors dependency updates
- Kimi Code CLI performs pre-merge code review sweeps

### Linear

| Property | Value |
|----------|-------|
| Account | read.musik@gmail.com |
| Auth | OAuth (connected via Nebula) |
| Usage | Sprint tracking, backlog management, issue lifecycle |
| Agent | linear-oauth-agent (Nebula) |

Integration pattern:
- Sprint backlog synced from `.planning/sprint-1-backlog.md`
- Issue status updates driven by kanban state in Ruflo/Loki
- Blockers automatically create Linear issues via Ruflo hook

### Notion

| Property | Value |
|----------|-------|
| Account | read.musik@gmail.com |
| Auth | OAuth (connected via Nebula) |
| Usage | Strategy docs, OKRs, long-form documentation |
| Agent | notion-agent (Nebula) |

---

## 3. Communication Integrations

### Telegram

| Property | Value |
|----------|-------|
| Handle | @ReidTheArchitect |
| Auth | Bot token via env |
| Usage | Primary async interface between Antonio and AI agents |
| Agent | telegram-agent (Nebula) |

Key patterns:
- Agent blockers -> Telegram message to @ReidTheArchitect
- NEXUS trading alerts -> Telegram notifications
- Sprint status updates -> Telegram digest

### Discord

| Property | Value |
|----------|-------|
| Server | Aigency |
| Auth | Bot token via env |
| Usage | Community communication, alpha signal monitoring |
| Agent | discord-solana-alpha-monitor (NEXUS) |

### Gmail

| Property | Value |
|----------|-------|
| Account | read.musik@gmail.com |
| Auth | OAuth (connected via Nebula) |
| Usage | External communications, verification emails |
| Agent | gmail-agent (Nebula) |

---

## 4. Data & Infrastructure Integrations

### Supabase

| Property | Value |
|----------|-------|
| Account | Aigency |
| Auth | Management API OAuth |
| Usage | Primary database for all Aigency platform products |
| Agent | Create via Nebula when needed |

Database conventions:
- Row-level security (RLS) enabled on all tables
- All migrations version-controlled in repo
- No direct DB access from agent code — always via API layer

### Google Workspace

| Service | Usage | Agent |
|---------|-------|-------|
| Google Sheets | NEXUS trading journal, data exports | google-sheets-agent |
| Google Docs | Documentation drafts, collaborative editing | google-docs-agent |
| Google Drive | File storage, asset management | google-drive-agent |
| Google Calendar | Sprint planning, scheduling | google-calendar-agent |

### Airtable

| Property | Value |
|----------|-------|
| Account | read.musik@gmail.com |
| Auth | OAuth |
| Usage | Structured data management (TBD — not yet in active use) |

---

## 5. Authentication Strategy

### Principles

1. **No hardcoded credentials.** All secrets stored as environment variables.
2. **OAuth preferred.** Use OAuth over API keys wherever the provider supports it.
3. **Nebula manages connections.** All OAuth connections are managed through Nebula AI's connected apps system.
4. **Agents never store credentials.** Agents receive credentials via environment injection at runtime.
5. **Rotate regularly.** API keys should be rotated every 90 days.

### Environment Variable Conventions

```
# LLM Providers
ANTHROPIC_API_KEY=
GEMINI_API_KEY=
OPENAI_API_KEY=
MOONSHOT_API_KEY=

# Infrastructure
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Communication
TELEGRAM_BOT_TOKEN=
DISCORD_BOT_TOKEN=

# SimpleLLMRouter
ROUTER_BASE_URL=http://localhost:8080
LETTA_BASE_URL=http://localhost:8283
```

All env vars are defined in `.env.example` at repo root. Never commit `.env`.

---

## 6. Integration Testing

Each integration has a health check command in the justfile:

```bash
just check-router    # Verify SimpleLLMRouter is responding
just check-letta     # Verify Letta server is up
just check-providers # Ping all LLM provider APIs
just status          # Full stack health check
```

Integration tests live in `packages/llm-router/tests/integration/`.
