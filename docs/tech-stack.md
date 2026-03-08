# Tech Stack - Aigency Dev Platform

> Authoritative reference for all technologies, versions, and toolchain choices.
> Last updated: 2026-03-08

---

## Core Platform

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| Runtime | Node.js | 20 LTS | Primary backend runtime |
| Language | TypeScript | 5.x | All packages |
| Package Manager | pnpm | 9.x | Workspace-level |
| Monorepo | Turborepo | latest | Task orchestration |
| Task Runner | Just | latest | justfile commands |

---

## Frontend

| Technology | Version | Notes |
|-----------|---------|-------|
| Next.js | 14.x | App router |
| React | 18.x | UI framework |
| Tailwind CSS | 3.x | Styling |
| shadcn/ui | latest | Component library |

---

## Backend / API

| Technology | Version | Notes |
|-----------|---------|-------|
| Hono | latest | Edge-compatible API framework |
| tRPC | 11.x | Type-safe API layer |
| Zod | 3.x | Schema validation |
| Prisma | 5.x | ORM |

---

## Database / Storage

| Technology | Usage | Notes |
|-----------|-------|-------|
| Supabase (Postgres) | Primary DB | Auth, row-level security |
| Redis | Caching, queues | Via Upstash or Railway |
| S3-compatible | File storage | Supabase Storage or Cloudflare R2 |

---

## AI / LLM Infrastructure

| Component | Technology | Notes |
|-----------|-----------|-------|
| LLM Router | SimpleLLMRouter v2 | Routes to Gemini, Claude, Kimi, etc. |
| Orchestration | Ruflo (claude-flow) | Multi-agent wave coordination |
| Persistent Memory | Letta Code | Cross-session agent state |
| Interactive Planning | iFlow CLI | Flow mode, Mermaid diagrams |
| Primary Coder Agent | Kimi Code CLI | 128K window, multi-step tasks |
| Architecture Agent | Gemini CLI | 1M window, large-doc synthesis |

---

## DevOps / Infrastructure

| Technology | Usage | Notes |
|-----------|-------|-------|
| GitHub Actions | CI/CD | Lint, test, build, deploy |
| Vercel | Frontend hosting | Next.js deployments |
| Railway / Fly.io | Backend services | API, workers |
| Docker | Containerization | Local dev and prod |
| Loki | Log aggregation | Observability |

---

## Testing

| Technology | Usage | Notes |
|-----------|-------|-------|
| Vitest | Unit tests | Fast, ESM-native |
| Playwright | E2E tests | Browser automation |
| MSW | API mocking | Test isolation |

---

## Key Constraints

- All new packages must be TypeScript-first
- No CommonJS - ESM only
- All environment config via .env files validated with Zod
- No direct DB access from frontend - always go through API layer
- All AI calls route through SimpleLLMRouter - no direct provider SDK calls in app code