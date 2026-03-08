# Memory Architecture — Aigency Platform

> Part of the Aigency Dev Platform architecture documentation.
> Implements the memory management principles from `constitutions/AI-CODER-CONSTITUTION.md` Section 5.
> See also: `architecture/agentic-tooling-integration-strategy.md` for tooling context.

---

## Overview

AI agents have a "Context Window" — a hard limit on how much they can hold in working memory at once. To build a large, coherent system across multiple agents and sessions, memory must be managed across three distinct tiers — analogous to CPU registers, RAM, and a hard drive.

```
Tier 1: Volatile (CONTINUITY)     — What am I doing right now?
Tier 2: Long-Term (LEDGERS)       — What decisions have been made?
Tier 3: On-Demand (SKILLS)        — What do I need to know for this specific task?
```

---

## Tier 1: Volatile Memory (CONTINUITY)

**Analogy:** CPU registers / RAM
**Lifespan:** Current session only
**Owner:** All agents (write); Ruflo (compaction)

### Purpose
Track immediate execution state so agents do not lose position mid-task or between turns.

### Implementation

| File | Contents | Updated By |
|------|----------|------------|
| `.planning/continuity.md` | Current task, last action, last error, next step | Every agent after each action |
| `.planning/handoffs/<timestamp>.md` | Inter-agent state transfer | Sending agent |
| Ruflo context compaction | Automatic summarization when context approaches limit | Ruflo daemon |

### Schema: continuity.md

```markdown
## Current Task
[What the agent is currently executing]

## Last Action
[What was just done — file written, command run, etc.]

## Last Output / Error
[Result or error from last action]

## Next Step
[What to do next]

## Blockers
[Anything blocking progress — empty if none]
```

### Rules
- Every agent MUST update `continuity.md` after each action.
- On session start, read `continuity.md` before reading anything else.
- Ruflo fires compaction automatically when context exceeds 80% of window.

---

## Tier 2: Long-Term Memory (LEDGERS)

**Analogy:** Hard drive / database
**Lifespan:** Permanent (git-tracked)
**Owner:** Letta Code (primary); all agents (append)

### Purpose
Maintain a permanent, queryable record of every architectural decision, so any new agent (or human) can be "briefed" on the full project history instantly.

### Implementation

| Store | Technology | Contents |
|-------|-----------|----------|
| `.planning/ledger.md` | Markdown (git-tracked) | Chronological decision log |
| `.letta/memory/` | Letta memory blocks | Semantic codebase knowledge |
| Letta server (:8283) | Letta stateful agent | Cross-session queryable memory |

### Schema: ledger.md entry

```markdown
## [YYYY-MM-DD HH:MM] Decision: [Short title]
**Agent:** [Who made the decision]
**Context:** [What problem was being solved]
**Decision:** [What was decided]
**Rationale:** [Why this approach over alternatives]
**Source Doc:** [Which spec doc guided this decision]
**Impact:** [What this decision affects downstream]
```

### Rules
- Every architectural decision MUST be logged to `ledger.md` before moving to the next task.
- Letta `/init deep` must be run at the start of a new project phase.
- On commit, Letta `/remember` fires automatically via git hook to sync codebase state.
- Ledger entries are append-only — never delete or edit past entries.

---

## Tier 3: On-Demand Skills (SKILL LOADING)

**Analogy:** Loading a program from disk into RAM
**Lifespan:** Task duration only
**Owner:** Ruflo (registry); all agents (consumers)

### Purpose
Avoid flooding agent context with irrelevant instructions. Agents load only the skill docs they need for the current task, then release them.

### Implementation

| Component | Location | Purpose |
|-----------|----------|---------|  
| Skills registry | `.claude/skills-index.json` | Maps skill names to file paths |
| Skill files | `.claude/skills/<skill-name>.md` | Detailed instructions for a specific capability |
| Skill loader | Ruflo daemon | Injects skill content into agent context on demand |

### Skills Registry Schema

```json
{
  "skills": [
    {
      "name": "typescript-module",
      "path": ".claude/skills/typescript-module.md",
      "description": "How to create a TypeScript module in this monorepo",
      "tags": ["typescript", "packages", "monorepo"]
    }
  ]
}
```

### Rules
- Do NOT load all skills at session start. Only load the skill relevant to the current task.
- After a skill is used, it can be evicted from context to make room for the next skill.
- Ruflo manages the skills registry. Other agents request skills by name.
- New skills are proposed by any agent but must be reviewed and merged by Ruflo.

---

## Memory Flow Diagram

```
Session Start
      |
Read continuity.md  (Tier 1 — what was I doing?)
      |
Query Letta memory  (Tier 2 — what decisions were made?)
      |
Load relevant skill (Tier 3 — what do I need for THIS task?)
      |
Execute task
      |
Write artifact to .planning/
      |
Update continuity.md  (Tier 1 — update state)
      |
Log decision to ledger.md  (Tier 2 — if architectural)
      |
Ruflo compaction  (Tier 1 — if context > 80%)
      |
Git commit -> Letta /remember hook  (Tier 2 — sync)
```

---

## Agent Memory Responsibilities

| Agent | Tier 1 (Volatile) | Tier 2 (Long-Term) | Tier 3 (Skills) |
|-------|------------------|-------------------|----------------|
| **Ruflo** | Owns compaction | Coordinates Letta syncs | Owns registry, loads on request |
| **Gemini** | Updates after synthesis tasks | Logs architecture decisions | Loads architecture skill docs |
| **Kimi** | Updates after each code task | Logs implementation decisions | Loads coding skill docs |
| **iFlow** | Updates after planning tasks | Logs planning decisions | Loads diagram skill docs |
| **Letta** | N/A (IS the long-term store) | Primary owner | Responds to skill queries |

---

## Integration with SimpleLLMRouter

All Letta queries and Ruflo compaction calls route through SimpleLLMRouter (:8080).
No agent calls LLM provider APIs directly — all memory operations that require inference go through the router.

See `docs/simplellmrouter-v2-spec.md` for router configuration.
