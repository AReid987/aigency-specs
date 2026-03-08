# aigency-specs

Shared bootstrap repository for all Aigency products.

## Purpose

This repo is the foundation every other Aigency repo pulls from on init via `just setup`. It holds:

- **constitutions/** — CLAUDE.md, coding standards, AI agent rules
- **org/** — Executive org chart, agent squads network
- **architecture/** — Shared architecture patterns and integration specs
- **templates/** — Justfile, turbo.json, .claude/settings.json templates

## Usage

When setting up a new repo, run:
```bash
just setup
```

This pulls the canonical CLAUDE.md, justfile, and shared architecture docs into the new repo automatically.

## Contents

| Path | Description |
|------|-------------|
| `constitutions/CLAUDE.md` | AI coding agent constitution — rules, patterns, anti-patterns |
| `org/AIGENCY-CORE-ORG-CHART.md` | Executive/human command layer |
| `org/aigency-agent-org-chart.md` | AI agent squads network |
| `architecture/memory-architecture.md` | Multi-tier memory architecture pattern |
| `architecture/integrations-spec.md` | Aigency integrations spec v1.0 |
| `templates/justfile` | Master justfile with all bootstrap commands |