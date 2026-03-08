# Aigency Core Org Chart — Human / Executive Layer

> Last updated: 2026-03-08
> This document defines the human command layer of the Aigency organization.
> For the AI agent network, see `org/aigency-agent-org-chart.md`.

---

## Executive Command Structure

```
Antonio Reid
Founder & CEO
@ReidTheArchitect
        |
        |---------------------------------------------|        |
   Product & Engineering                        Operations & Growth
   (AI-assisted via Meta Code Squad)            (AI-assisted via Aigency Agents)
        |                                             |
        |-- Platform Architecture                     |-- Marketing & GTM
        |-- Development Execution                     |-- Community & Partnerships
        |-- Infrastructure & DevOps                   |-- Revenue & Customer Success
        |-- Quality & Security                        |-- Finance & Legal
```

---

## Roles

### Antonio Reid — Founder & CEO

- **Authority:** Final decision on all product, architecture, and strategic decisions
- **Primary interface:** Telegram (@ReidTheArchitect), Nebula AI
- **Escalation target:** All agent blockers, ambiguities, and open questions route here
- **Tooling:** Nebula AI (orchestration), GitHub (code review), Linear (sprint tracking), Notion (strategy docs)

---

## Product Domains

| Domain | Description | Primary AI Agent Squad |
|--------|-------------|------------------------|
| **Aigency Core Platform** | The main dev platform powering all products | Meta Code Squad |
| **SimpleLLMRouter** | Multi-provider LLM routing layer | Meta Code Squad |
| **Forge Quality** | Code quality enforcement system | Meta Code Squad |
| **LP Generator** | Landing page generation pipeline | Landing Page Squad |
| **Project Blackout** | (Stealth — TBD) | TBD |
| **Learning Accelerator / Forge** | AI-powered learning platform | Forge Squad |
| **Hyperlocal** | Hyperlocal product (TBD) | TBD |
| **Aigency World** | 3D environment viewer for meta-code-squad | TBD |

---

## Decision Authority Matrix

| Decision Type | Owner | Approver |
|---------------|-------|----------|
| Architecture changes | Gemini CLI (drafts) | Antonio Reid |
| Sprint scope | Ruflo (proposes) | Antonio Reid |
| Production deploys | CI/CD pipeline | Antonio Reid (merge approval) |
| New agent creation | Nebula AI | Antonio Reid |
| External partnerships | TBD | Antonio Reid |
| Budget / spend | TBD | Antonio Reid |

---

## Communication Channels

| Channel | Purpose |
|---------|---------|  
| Telegram @ReidTheArchitect | Primary async communication with AI agents |
| GitHub PRs | Code review and merge decisions |
| Linear | Sprint tracking and backlog management |
| Notion | Strategy, OKRs, and long-form documentation |
| Nebula AI | AI orchestration and agent delegation |

---

## Escalation Protocol

All AI agent blockers follow this path:

```
Agent detects blocker
        |
Add to .planning/prp.md section 9 (Open Questions)
        |
Ruflo reviews and attempts resolution
        |
If unresolved: Ruflo notifies Antonio via Telegram
        |
Antonio provides direction within 24 hours
        |
Agent resumes execution
```
