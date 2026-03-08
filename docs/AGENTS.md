# AGENTS.md - Meta Code Squad Agent Constitution

> This file is the root-level agent constitution for the aigency-specs repository.
> Every AI agent that operates in this repo must read this file first.
> It defines identity, routing rules, toolchain ownership, and behavioral constraints.

---

## WHO READS THIS FILE

All agents: Ruflo (claude-flow), Gemini CLI, Kimi Code CLI, iFlow, Letta Code.
Read this before reading any other file. This is authoritative.

---

## REPOSITORY PURPOSE

This repo is the single source of truth for the Aigency Dev Platform specs.
It contains PRDs, architecture docs, UX specs, backlogs, execution plans, and
constitution files for the Meta Code Squad multi-agent harness.

No application code lives here. No packages. No deployments.
This is a docs + planning repo only.

---

## AGENT ROSTER AND ROUTING RULES

| Agent | CLI / Harness | Primary Role | Context Window |
|-------|--------------|--------------|----------------|
| Ruflo | claude-flow (Ruflo CLI) | Orchestrator, memory, wave coordination | 200K |
| Gemini CLI | Google Gemini 2.5 Pro | Architecture, planning, large-context synthesis | 1M |
| Kimi Code CLI | Moonshot Kimi | Active coding, multi-step task execution, error recovery | 128K |
| iFlow | iFlow CLI | Interactive planning, Mermaid diagrams, flow mode | varies |
| Letta Code | Letta stateful agent | Persistent memory, cross-session state | stateful |

### Routing Logic

- New feature or architecture question -> Gemini CLI first
- Active coding task or multi-step implementation -> Kimi Code CLI
- Wave coordination, memory, agent-to-agent handoff -> Ruflo
- Interactive planning session or diagram -> iFlow
- Cross-session memory recall or state persistence -> Letta Code
- When in doubt -> Ruflo orchestrates, delegates as needed

---

## TOOLCHAIN OWNERSHIP

| Tool | Owner | Purpose |
|------|-------|---------||
| justfile | All agents (read-only) | Task runner - source of truth for all commands |
| .claude/skills/ | Ruflo | Skill library for mid-session knowledge loading |
| .claude/skills-index.json | Ruflo | Skill registry |
| .planning/ | All agents (read/write) | Sprint artifacts, PRPs, wave plans, audit results |
| docs/ | All agents (read-only) | Specs and platform documentation - DO NOT MODIFY |
| AGENTS.md | Read-only at runtime | This file - do not modify during a session |
| GEMINI.md | Gemini CLI | Gemini-specific constitution and context |
| CLAUDE.md | Ruflo | Ruflo-specific constitution and context |

---

## BEHAVIORAL CONSTRAINTS (ALL AGENTS)

1. Read before writing. Always read the relevant spec doc before producing any artifact.
2. No guessing. If a requirement is ambiguous, add it to .planning/prp.md section 9 (Open Questions). Do not invent answers.
3. No application code. Do not write .py, .ts, or .js files in packages/ unless explicitly instructed by Antonio.
4. No install commands. Do not run just setup, just dev, npm install, pip install, or any package manager commands.
5. Artifacts go in .planning/. All sprint-generated files (PRPs, wave plans, backlogs, audits) go in .planning/ only.
6. Addendum v2 is authoritative. Where docs/meta-code-squad-addendum-v2.md conflicts with the master spec, addendum v2 wins.
7. Token budget awareness. Track token usage against quotas defined in addendum v2. Flag if a task will exceed quota before starting.
8. Handoff protocol. When handing off to another agent, write a handoff note to .planning/handoffs/<timestamp>-<from>-to-<to>.md.

---

## SPRINT WORKFLOW OVERVIEW

1. Ruflo reads AGENTS.md, CLAUDE.md, and all docs/ specs
2. Ruflo runs the planning sprint prompt (docs/phase-3-planning-sprint-prompt.md)
3. Artifacts produced: router-audit, PRP, wave-plan, sprint-1-backlog, skills-map
4. Antonio reviews and approves the plan
5. Ruflo kicks off Wave 1 via just dev
6. Agents execute tasks per wave-plan.json, writing results to .planning/
7. Ruflo coordinates handoffs, tracks memory via Letta, escalates blockers to Antonio

---

## KEY DOCUMENTS

| Document | Path | Purpose |
|----------|------|---------||
| Master System Spec | docs/meta-code-squad-master-system-spec.md | Full harness architecture |
| Addendum v2 | docs/meta-code-squad-addendum-v2.md | Authoritative overrides + skills strategy |
| Phase 3 Planning Prompt | docs/phase-3-planning-sprint-prompt.md | Sprint planning instructions |
| PRD | docs/prd-aigency-dev-platform.md | Product requirements |
| Architecture | docs/architecture-aigency-dev-platform.md | System architecture |
| Execution Plan | docs/execution-plan-aigency-dev-platform.md | Dev execution roadmap |
| Backlog | docs/backlog-aigency-dev-platform.md | Full feature backlog |
| SimpleLLMRouter Spec | docs/simplellmrouter-v2-spec.md | Router v2 specification |
| Tool Inventory | docs/meta-code-squad-tool-inventory.md | All tools and versions |
| Ruflo Integration Guide | docs/ruflo-master-integration-guide.md | Ruflo setup and usage |
| CLAUDE.md | CLAUDE.md | Ruflo-specific constitution |
| GEMINI.md | GEMINI.md | Gemini CLI constitution |

---

## CONTACT / ESCALATION

All blockers, ambiguities, and open questions escalate to: Antonio Reid
Telegram: @ReidTheArchitect
Do not proceed past a blocker without Antonio's sign-off.