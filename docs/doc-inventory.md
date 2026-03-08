# Aigency Doc Inventory & Repo Assignment
Version: 1.4 | Date: 2026-03-08 | Owner: Antonio Reid
Status: Source of truth for the current initialization sprint.
Scope: aigency-specs, meta-code-squad, and simplellmrouter only.

---

## REPO MODEL

THREE REPOS IN SCOPE FOR THIS SPRINT ONLY.

  Repo 1: aigency-specs       DOES NOT EXIST. Create first.
  Repo 2: meta-code-squad     DOES NOT EXIST. Create after Repo 1.
  Repo 3: simplellmrouter     EXISTS at AReid987/simplellmrouter. Push docs only.

aigency-specs is the foundation. It is NOT a product repo.
It holds constitutions, doc templates, shared configs, org charts, and justfile
bootstrap commands. Any new repo runs one command and gets scaffolded correctly:
CLAUDE.md in root, justfile, coding standards, shared arch docs.

All other repos (aigency platform turborepo, learning-coach, etc.) are out of
scope for this sprint and are not listed here.

---

## ORG CHARTS (two distinct files, both canonical)

  docs/AIGENCY-CORE-ORG-CHART.md
    The human/executive command layer.
    Antonio Reid (THE ARCHITECT) + 10 executive personas.
    Brand colors, callsigns, domains of ownership.
    -> aigency-specs (shared reference for all repos)

  docs/aigency-agent-org-chart.md
    The AI agent squads network.
    Aigency Agile Squad, Forge Squad, GSD Squad, Landing Page Squad,
    Claude Code Hive Mind, Core Platform.
    Each squad has commander + specialists + optional drones.
    -> aigency-specs (shared reference for all repos)

---

## REPO 1: aigency-specs (SHARED BOOTSTRAP REPO)

Purpose: constitutions, templates, shared configs, justfile commands
that new repos pull on init.

Constitutions & Standards
  docs/CLAUDE.md
    AI coding agent constitution. Rules, patterns, anti-patterns.
    This is the canonical CLAUDE.md pulled into every repo on setup.

  docs/meta-code-squad-master-system-spec.md (28KB)
    Master system specification for the Meta-Code-Squad orchestration layer.

  docs/meta-code-squad-addendum-v2.md (27KB)
    Addendum v2 with updated patterns and toolchain details.

  docs/meta-code-squad-tool-inventory.md (9KB)
    Full inventory of tools available to coding agents.

  docs/meta-code-squad-launch-guide.md (15KB)
    Launch checklist and setup guide for the meta-code-squad system.

Org Charts (shared across all products)
  docs/AIGENCY-CORE-ORG-CHART.md
    Executive/human command layer. Antonio + 10 execs.

  docs/aigency-agent-org-chart.md
    AI agent squads network. All squads and their structures.

Agent Personas (shared across all products)
  docs/aigency-agent-personas.md
    Full persona definitions, bios, and avatar specs for all squad agents.
    Includes Newton "Nexus" Chen and full Aigency Agile Squad.

  docs/LEARNING_ACCELERATOR_PERSONAS.md  <-- AUTHORITATIVE
    Learning Accelerator agent team personas.
    Newer file (7 days old). Use this one.

  docs/AIGENCY-LEARNING-ACCELERATOR-PERSONAS.md  <-- SUPERSEDED
    Older version (8 days old). Do not copy. Archive or delete.

Shared Architecture & Integration Docs
  docs/memory-architecture.md
    Multi-Tier Memory Architecture. Four-tier cognitively-inspired memory
    for autonomous coding agents. Shared pattern used across systems.

  docs/integrations-spec.md
    Aigency Integrations Spec v1.0. Sugar AI execution layer replacement,
    MCP server, Ralph Mode, GitHub-native task queue. Shared across systems.

  docs/agentic-tooling-integration-strategy.md (23KB)
    Strategy doc for integrating agentic tooling. Shared reference.

Templates & Justfile
  docs/justfile.txt  (actually at code/justfile.txt, 15KB)
    Master justfile with all just commands including repo bootstrap commands.
    This ships with aigency-specs so new repos can pull it.

---

## REPO 2: meta-code-squad

New repo. Create after aigency-specs exists.
Run just setup after clone to pull shared files from aigency-specs.
Meta-orchestrator, forge-quality CLI, Ruflo/Letta integration, agor.live canvas.

System-specific docs to place in docs/:

  docs/ruflo-master-integration-guide.md  (30KB)
    Ruflo (claude-flow v3.5) master integration guide.
    Memory, persistence, and orchestration engine under meta-code-squad.

  code/ruflow-repo.md  (15KB)
    RuFlow repo structure and setup reference.
    Copy to docs/ruflow-repo.md on placement.

  docs/execution-plan-aigency-dev-platform.md  (30KB)
    Execution plan. Despite the name, content is System A / meta-code-squad
    implementation specifics. Belongs here.

  docs/tech-stack.md
    Tech stack reference for meta-code-squad and SimpleLLMRouter. Confirmed.

Note: SimpleLLMRouter v2 spec lives in Repo 3 (its own repo), not here.
The router is a dependency of meta-code-squad but owns its own repo.

---

## REPO 3: simplellmrouter

Already exists at: github.com/AReid987/simplellmrouter
Action: push docs only. No repo creation needed.

  docs/simplellmrouter-v2-spec.md  (55KB)
    Comprehensive spec for SimpleLLMRouter v2.
    Architecture, TUI, semantic cache, OptiLLM, 3-layer routing, provider matrix.
    Destination: simplellmrouter/docs/simplellmrouter-v2-spec.md

---

## OUT OF SCOPE FOR THIS SPRINT

The following repos and their associated docs are NOT part of this initialization.
They will be set up in future sprints using aigency-specs as the bootstrap source.

  aigency (turborepo)   -- public platform, System C, Next.js at apps/web
  learning-coach        -- Socratic Learning Coach Bot, Telegram-based
  + all other products in the workspace

All AIGENCY-* prefixed docs, Learning Coach 01-06 numbered docs,
SOL-ARCADE, CLAWVAULT, NEXUS, SoundGrid, LP-GEN-SYSTEM, and
aigency-core-* v1 family docs are out of scope and should not be touched.

---

## FILES PULLED AUTOMATICALLY VIA just setup

Every new repo gets these on init, pulled from aigency-specs:

  constitutions/CLAUDE.md              placed at: CLAUDE.md (repo root)
  justfile                             placed at: justfile (repo root)
  architecture/memory-architecture.md  placed at: docs/
  architecture/integrations-spec.md    placed at: docs/

---

## SUPERSEDED FILES -- DO NOT COPY TO ANY REPO

  docs/AIGENCY-LEARNING-ACCELERATOR-PERSONAS.md
    8 days old. Superseded by LEARNING_ACCELERATOR_PERSONAS.md (7 days old).
    Review for unique content before deleting.

---

## CREATION ORDER

  Step 1: Create aigency-specs. Populate constitutions/, org/, architecture/. Push.
  Step 2: Create meta-code-squad. Run just setup. Copy system-specific docs. Push.
  Step 3: Push simplellmrouter-v2-spec.md to existing simplellmrouter repo.

---

## STATUS SUMMARY

  aigency-specs      DOES NOT EXIST   create first
  meta-code-squad    DOES NOT EXIST   create after aigency-specs
  simplellmrouter    EXISTS           push docs only
