# Phase 3 — Planning Sprint Prompt
## Paste this into Claude Code at the root of `meta-code-squad/`
**Version:** 1.0 | **Date:** 2026-03-07 | **Owner:** Antonio Reid
**Mode:** Plan only — NO code execution until explicitly authorized

---

## HOW TO USE THIS PROMPT

1. Open Claude Code in the `meta-code-squad/` repo root
2. Make sure `just setup` has completed successfully (`just doctor` should pass)
3. Paste the entire block below the horizontal rule as your first message
4. Claude will read all docs, produce the planning artifacts, and stop
5. You review the artifacts before any code runs

---

---

# PLANNING SPRINT — META CODE SQUAD

You are Claude Code, operating as the lead architect and planning agent for the Meta Code Squad system. This is a **planning-only sprint**. Your job is to read everything, understand everything, and produce a complete set of planning artifacts. You will NOT write application code, run installs, or modify any existing files during this sprint. You will ONLY create new files inside `.planning/`.

---

## STEP 1 — READ ALL CONTEXT DOCUMENTS

Read every document in `docs/` in this order. Do not summarize yet — just read and build your internal model.

```
docs/meta-code-squad-master-system-spec.md      ← The bible. Read this first and most carefully.
docs/meta-code-squad-addendum-v2.md             ← Corrections. These OVERRIDE the master spec where they conflict.
docs/meta-code-squad-tool-inventory.md          ← Every CLI and API in the stack.
docs/meta-code-squad-launch-guide.md            ← Onboarding sequence and first-session flow.
docs/agentic-tooling-integration-strategy.md    ← How all harnesses connect together.
docs/simplellmrouter-v2-spec.md                 ← Router design — routing logic, provider configs, TUI.
docs/prd-aigency-dev-platform.md                ← What we are building. This becomes the PRP.
docs/architecture-aigency-dev-platform.md       ← System architecture of the platform being built.
docs/backlog-aigency-dev-platform.md            ← Raw backlog. You will convert this to a structured sprint.
```

Also read these if present:
```
AGENTS.md           ← Root agent context rules (may or may not exist yet)
CLAUDE.md           ← Claude-specific context (may or may not exist yet)
packages/llm-router/    ← Scan the router codebase — list all files, identify what exists vs what is missing
justfile            ← Understand the full command surface
.planning/          ← Read any existing files here (stub structure may already be present)
```

---

## STEP 2 — AUDIT THE LLM ROUTER

After reading `docs/simplellmrouter-v2-spec.md` and `docs/meta-code-squad-addendum-v2.md`, scan `packages/llm-router/` and produce a router audit. Answer these specific questions:

1. Does `providers.yaml` (or equivalent) exist? List all providers currently configured.
2. Which providers are MISSING vs the addendum v2 list: NVIDIA NIM, Bonsai, Giga AI, GitHub Models, HuggingFace, Cloudflare Workers?
3. Does Kimi's context window say 256K anywhere? (It should be 128K — flag if wrong)
4. Is there a startup call to `/api/v1/models` for VoidAI? (Required — flag if missing)
5. Are there any hardcoded OpenAI or Anthropic API keys or endpoints? (Must remove — flag all instances)
6. Is Cerebras configured as the primary overflow provider? (Should be — 57,600 req/day, 1M tokens/hr)
7. Does the OpenAI-compatible proxy exist on port 8080?
8. Does the 3-layer semantic cache exist?
9. Does the Textual TUI exist?
10. Does OptiLLM inference boosting exist?

Write your findings to `.planning/router-audit.md`. Format: each item as PASS / FAIL / MISSING with one line of detail.

---

## STEP 3 — PRODUCE THE PROJECT REQUIREMENTS PLAN (PRP)

Using `docs/prd-aigency-dev-platform.md` as the source, produce a Project Requirements Plan at `.planning/prp.md`.

The PRP must include:

### PRP Structure

```markdown
# Project Requirements Plan
## Meta Code Squad — Aigency Dev Platform Sprint 1

### 1. Mission Statement (2-3 sentences max)
### 2. What We Are Building (this sprint only — not the full roadmap)
### 3. Acceptance Criteria (the definition of done for Sprint 1)
### 4. Out of Scope (explicit list of what we are NOT building this sprint)
### 5. Technical Constraints
   - Provider constraints (no OpenAI key, no Anthropic key)
   - Context window facts (Gemini = 1M-2M, Kimi = 128K, iFlow = 128K)
   - Harness constraints (which tools are available, which are future)
   - Token budget (estimate total tokens for this sprint at current free quotas)
### 6. Quality Gates (from Loki's 9-gate system — list all 9, mark which apply to Sprint 1)
### 7. Agent Assignment Map (which agent handles which tasks)
   - Claude Code: core logic, security, state machines, router fixes
   - Gemini CLI: tests, config files, docs, boilerplate
   - Kimi Code CLI: active coding, multi-step task execution, error recovery (128K window)
   - iFlow: architecture mapping, scaffold generation, diagram generation
   - Ruflo (claude-flow): memory, context, skill learning
### 8. Risk Register (top 5 risks with mitigation)
### 9. Open Questions (things that need Antonio's decision before code runs)
```

---

## STEP 4 — PRODUCE THE WAVE PLAN

Based on `docs/meta-code-squad-master-system-spec.md` section on phased build sequence, and the PRP you just wrote, produce `.planning/wave-plan.json`.

Structure:
```json
{
  "project": "meta-code-squad",
  "sprint": 1,
  "total_waves": 4,
  "waves": [
    {
      "wave": 1,
      "name": "Foundation",
      "description": "...",
      "packages": ["llm-router"],
      "tasks": [...],
      "agents": ["claude-code", "gemini-cli"],
      "gate": "router passes all 10 audit items",
      "estimated_tokens": 0,
      "status": "pending"
    }
  ]
}
```

Wave breakdown guidance:
- **Wave 1 — Router Fix:** Fix all FAIL/MISSING items from router audit. Get the proxy live on :8080. All 14 providers configured. Cerebras as primary overflow. VoidAI dynamic model fetch.
- **Wave 2 — Forge Quality:** `@aigency/forge-quality` package — commit hooks, security scanner, quality gates.
- **Wave 3 — Memory Layer:** Letta Code init, AgentDB config, Ruflo skill registration, `.letta/memory/` git sync setup.
- **Wave 4 — Orchestration:** GSD plan files, Loki kanban, Sugar task queue, full `just dev` end-to-end test.

---

## STEP 5 — PRODUCE THE SPRINT 1 BACKLOG

Convert `docs/backlog-aigency-dev-platform.md` into a structured sprint backlog at `.planning/sprint-1-backlog.md`.

Include ONLY Wave 1 and Wave 2 tasks (router fix + forge quality). Waves 3-4 go in a separate file `.planning/sprint-2-backlog-draft.md` — just save them there for now.

Sprint 1 backlog format:
```markdown
# Sprint 1 Backlog

## Wave 1 — Router Fix
| ID | Task | Agent | Estimated Tokens | Priority | Depends On |
|----|------|-------|-----------------|----------|------------|
| W1-01 | Add NVIDIA NIM provider to providers.yaml | gemini-cli | ~2K | P0 | — |
| W1-02 | ... | ... | ... | ... | ... |

## Wave 2 — Forge Quality
| ID | Task | Agent | Estimated Tokens | Priority | Depends On |
|----|------|-------|-----------------|----------|------------|
| W2-01 | ... | ... | ... | ... | ... |

## Sprint 1 Token Budget Summary
| Provider | Daily Quota | Sprint Allocation | % of Quota |
|----------|------------|-------------------|------------|
| Cerebras | 57,600 req/day | ~X req | ~Y% |
| ...
```

Generate ALL router fix tasks from your audit in Step 2. Do not skip tasks because they seem small — every FAIL and MISSING item from the audit becomes a backlog task.

---

## STEP 6 — PRODUCE THE AGENTS.md ROOT CONTEXT FILE

Write `AGENTS.md` at the repo root. This is the single-source context file every agent reads at the start of every session. It must be concise — agents will read this every time, so every word must earn its place.

Structure:
```markdown
# AGENTS.md — Meta Code Squad

## What This Repo Is
[2-3 sentences]

## Current Sprint
Sprint 1, Wave [current wave]. Status: [status].

## Agent Roles
| Agent | Handles | Context Window | Daily Quota |
|-------|---------|----------------|------------|
| Claude Code | ... | 200K | Unlimited (local) |
| Gemini CLI | ... | 1M–2M | 1,500 RPD |
| Kimi Code CLI | active coding, multi-step tasks, error recovery | 128K | ~3x per 5hr |
| iFlow | ... | 128K | Unknown |

## Routing Rules (SimpleLLMRouter)
- No OpenAI API key. No Anthropic API key.
- Cerebras = primary overflow (57,600 req/day, 1M tok/hr)
- Gemini = large context tasks (>100K tokens)
- Kimi = active coding, multi-step task execution, error recovery
- VoidAI = model variety, call /api/v1/models at startup

## File Conventions
- All planning artifacts: .planning/
- All system docs: docs/
- All packages: packages/
- Memory exports: .planning/memory/letta-notes.md (run: just memory-dump)

## Quality Gates (Loki — 9 gates)
[List all 9 gates from the master spec]

## Key Commands
| Command | Does |
|---------|------|
| just dev | Start full stack |
| just doctor | Verify all CLIs |
| just understand | iFlow /understand (plan mode) |
| just diagram | iFlow /mermaid architecture |
| just memory-dump | Export Letta memory to .planning/memory/ |
| just sync-context | Regenerate CLAUDE.md, GEMINI.md, .kiro/steering/ |

## Open Decisions (needs Antonio)
[Pull from PRP section 9 — open questions]

## Harness Status
| Harness | Status | Notes |
|---------|--------|-------|
| Ruflo (claude-flow) | ... | |
| Letta Code | ... | |
| iFlow CLI | ... | |
| SimpleLLMRouter | ... | |
| Loki | ... | |
| Sugar AI | ... | |
```

---

## STEP 7 — FINAL SUMMARY REPORT

After producing all artifacts, write a summary to the terminal (not a file). Include:

1. **Router Audit Result:** X/10 passing. List the FAILs and MISSINGs.
2. **PRP written:** Yes/No. Key open questions (list them).
3. **Wave plan written:** Yes/No. Total estimated sprint tokens.
4. **Sprint 1 backlog written:** Yes/No. Total task count (Wave 1: X tasks, Wave 2: Y tasks).
5. **AGENTS.md written:** Yes/No.
6. **Blockers before Sprint 1 can start:** List anything that needs Antonio's decision.
7. **Recommended first command when Antonio approves:** (probably `just dev` or a specific Wave 1 task)

---

## STEP 8 - PRODUCE THE SKILLS MAP

Read `docs/meta-code-squad-addendum-v2.md` section 5 (Skills Strategy). Using that structure, produce `.planning/skills-map.md`.

The skills map defines which Ruflo skills exist, which need to be authored, and which agent owns each skill.

Template for `.planning/skills-map.md`:

    # Skills Map - Meta Code Squad Sprint 1

    ## What Skills Are
    Skills are reusable knowledge units agents load mid-session via Ruflo (claude-flow).
    Each skill is a .md file stored in .claude/skills/ and registered in .claude/skills-index.json.

    ## Skill Inventory
    | Skill Name | File | Status | Owner Agent | Trigger Condition |
    |------------|------|--------|-------------|-------------------|
    | router-provider-add | router-provider-add.md | NEEDS AUTHORING | gemini-cli | Adding a new provider |
    | ... | ... | ... | ... | ... |

    ## Skills Needed Before Sprint 1 Starts
    List any skills that Wave 1 tasks depend on. These must be authored BEFORE the wave runs.

    ## Skills Authored DURING Sprint 1
    List skills that agents will create as part of their wave work.

    ## Skill Authoring Rules (from addendum v2 section 5)
    [Pull the key rules verbatim from addendum v2]

---

## CONSTRAINTS FOR THIS SPRINT

- **DO NOT** run `just setup`, `just dev`, or any install commands
- **DO NOT** modify any existing files (justfile, docs/, packages/)
- **DO NOT** write application code (no .py, .ts, .js files in packages/)
- **DO** create files only inside `.planning/` and the root `AGENTS.md`
- **DO** flag any ambiguity rather than guess — add it to PRP section 9 (Open Questions)
- **DO** be specific about token estimates — use the quota numbers from addendum v2
- **DO** treat addendum v2 as authoritative wherever it conflicts with the master spec

---

## OUTPUT CHECKLIST

Before finishing, confirm you have created ALL of these:

- [ ] `.planning/router-audit.md`
- [ ] `.planning/prp.md`
- [ ] `.planning/wave-plan.json`
- [ ] `.planning/sprint-1-backlog.md`
- [ ] `.planning/sprint-2-backlog-draft.md`
- [ ] `.planning/skills-map.md`
- [ ] `AGENTS.md` (repo root)

If any of these is missing, produce it before reporting done.

---

**Begin with Step 1. Read all docs before producing any artifacts.**
