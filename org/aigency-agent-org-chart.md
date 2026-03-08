# Aigency Agent Org Chart — AI Agent Network

> Last updated: 2026-03-08
> This document maps the full AI agent network for the Aigency platform.
> For the human command layer, see `org/AIGENCY-CORE-ORG-CHART.md`.
> All agents operate under the AI Coder Constitution: `constitutions/AI-CODER-CONSTITUTION.md`.

---

## Agent Hierarchy Overview

```
Antonio Reid (Human — Final Authority)
        |
   Nebula AI (Master Orchestrator — nebula.gg)
        |
        |-- Meta Code Squad (Development Harness)
        |       |-- Ruflo (Orchestrator)
        |       |-- Gemini CLI (Architect)
        |       |-- Kimi Code CLI (Engineer)
        |       |-- iFlow (Planner)
        |       |-- Letta Code (Librarian / Memory)
        |
        |-- Aigency Specialized Agents
                |-- Landing Page Squad
                |-- Forge Learning Squad
                |-- NEXUS Trading Network
                |-- Telegram Scout Agents
```

---

## Meta Code Squad — Core Development Harness

| Agent | Constitutional Role | Actual Tool | Strengths | Context |
|-------|-------------------|-------------|-----------|---------|  
| **Ruflo** | Orchestrator | claude-flow (Ruflo CLI) | Wave coordination, memory, MCP tools, agent-to-agent handoff | 200K |
| **Gemini CLI** | Architect | Google Gemini 2.5 Pro | 1M context, architecture reasoning, large-doc synthesis | 1M |
| **Kimi Code CLI** | Engineer | Moonshot Kimi | Active coding, multi-step execution, error recovery | 128K |
| **iFlow** | Planner | iFlow CLI | Interactive planning, Mermaid diagrams, flow mode | varies |
| **Letta Code** | Librarian | Letta stateful agent | Persistent memory, cross-session state, codebase knowledge | stateful |

### Constitutional Role Definitions

| Role | Responsibilities |
|------|------------------|
| **Orchestrator (Ruflo)** | Manages task list, remembers project state, assigns jobs to agents, coordinates waves, does NOT do implementation work |
| **Architect (Gemini)** | Designs systems, synthesizes large docs, produces architecture artifacts, does NOT write production code |
| **Engineer (Kimi)** | Executes coding tasks, follows the plan, runs quality gates, reports results |
| **Planner (iFlow)** | Produces visual plans, diagrams, and flow artifacts for human review |
| **Librarian (Letta)** | Manages documentation state, cross-session memory, ensures context remains accurate |

### Routing Rules

```
New feature / architecture question  -->  Gemini CLI
Active coding / multi-step task      -->  Kimi Code CLI
Wave coordination / memory           -->  Ruflo
Interactive planning / diagrams      -->  iFlow
Cross-session memory recall          -->  Letta Code
Blocker / escalation                 -->  Ruflo --> Antonio
```

---

## Aigency Specialized Agents (Nebula Network)

### Landing Page Squad

| Agent | Role |
|-------|------|
| Landing Page Discovery Specialist | Uncovers conversion requirements and user goals |
| Market Research Specialist | Competitive and market analysis |
| Landing Page Strategy Director | Converts research into strategy blueprints |
| Design System Architect | Design token and component system generation |
| Landing Page Copy Optimizer | Section-by-section copy engineering |
| Landing Page Architect | Production-ready implementation |

### Product & Agile Squad

| Agent | Role |
|-------|------|
| Michael "Meridian" Park | Product Manager — PRDs, requirements |
| Ava "Atlas" Patel | Architect — system blueprints |
| Clara "Cipher" Rodriguez | Analyst — research and project briefs |
| Fiona "Flux" Rivera | Scrum Master — sprint ceremonies |
| Phoenix "Prism" Kim | Design Architect — UI/UX blueprints |
| Beacon Backlog Prioritizer | Backlog management |
| Full-Stack Development Executor | Code implementation |
| Infrastructure & Deployment Automation | DevOps and CI/CD |
| Newton "Nexus" Chen | Agile Squad Orchestrator |

### NEXUS Trading Intelligence Network

| Agent | Role |
|-------|------|
| Meme Coin Intel Orchestrator | Master orchestrator for trading signals |
| Solana On-Chain Scanner | Real-time token monitoring |
| Alpha Signal Scorer | Signal scoring and tier classification |
| NEXUS Paper Trader | Simulated trade execution |
| NEXUS Strategy Evaluator | Performance analysis |
| NEXUS Market Regime Agent | Market regime classification |
| Discord Solana Alpha Monitor | Discord signal monitoring |
| Telegram Alpha Scout | Telegram signal monitoring |
| Reddit Solana Meme Scout | Reddit signal monitoring |
| Solana Social Scout | Twitter/X signal monitoring |

---

## Cross-Agent Communication Protocol

All agents follow the handoff protocol defined in `AGENTS.md`:

1. Complete task, write artifact to `.planning/`
2. Write handoff note to `.planning/handoffs/<timestamp>-<from>-to-<to>.md`
3. Handoff note includes: what was produced, decisions made, open questions, recommended next action
4. Notify Ruflo (or Nebula for specialized squads)

---

## Quality Gate Enforcement

Each agent enforces gates appropriate to their role:

| Agent | Gate 1 (Syntax) | Gate 2 (Logic) | Gate 3 (Security) | Gate 4 (Consensus) |
|-------|----------------|----------------|-------------------|--------------------|
| Ruflo | Validates task completeness | Checks against wave plan | Reviews for scope creep | Signs off on wave completion |
| Gemini | Validates artifact structure | Checks citation coverage | Reviews for spec conflicts | Checks against PRD |
| Kimi | Runs lint/typecheck | Checks acceptance criteria | Runs security scan | Requests Ruflo sign-off |
| Letta | Validates memory blocks | Checks state accuracy | No credentials in memory | Ruflo confirms sync |
