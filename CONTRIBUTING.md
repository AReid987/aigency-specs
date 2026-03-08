# Contributing to Aigency

> This guide applies to all repositories in the Aigency organization: `aigency-specs`, `meta-code-squad`, `simplellmrouter`, and all product repos.
> All contributors (human and AI) must follow these conventions.

---

## Guiding Principles

All contributions are governed by the [AI Coder Constitution](constitutions/AI-CODER-CONSTITUTION.md). Key mandates:

- **Atomicity:** One logical change per PR. Do not bundle unrelated changes.
- **Verification:** All Quality Gates must pass before merge. No exceptions.
- **Evidence over Assertions:** "It works on my machine" is not sufficient — CI must be green.

---

## Branch Strategy

We use **trunk-based development** with short-lived feature branches.

```
main          — production-ready, always deployable
  |
  |-- feat/<scope>/<short-description>     e.g. feat/router/add-kimi-provider
  |-- fix/<scope>/<short-description>      e.g. fix/letta/memory-sync-on-commit
  |-- chore/<scope>/<short-description>    e.g. chore/deps/bump-anthropic-sdk
  |-- docs/<scope>/<short-description>     e.g. docs/arch/add-memory-architecture
```

### Rules

- Branch off `main` always — never branch off another feature branch.
- Branch names must be lowercase, hyphen-separated, no spaces.
- Delete branches after merge.
- Feature branches should live no longer than 3 days. If longer, break the work into smaller PRs.

---

## Commit Conventions

We follow [Conventional Commits](https://www.conventionalcommits.org/).

### Format

```
<type>(<scope>): <short description>

[optional body]

[optional footer: BREAKING CHANGE, Closes #issue]
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `chore` | Maintenance, deps, config |
| `refactor` | Code change with no behavior change |
| `test` | Adding or fixing tests |
| `ci` | CI/CD workflow changes |
| `perf` | Performance improvements |

### Examples

```
feat(router): add Moonshot Kimi as a provider

fix(letta): resolve memory sync failure on large commits

docs(arch): add memory architecture spec

chore(deps): bump anthropic-sdk to 0.39.0

ci: add CodeQL security scan workflow
```

### Rules

- Subject line: 72 characters max, imperative mood ("add" not "added")
- No period at end of subject line
- Body: explain WHY, not WHAT (the diff shows what)
- Reference issues: `Closes #123` or `Relates to #456`

---

## Pull Request Process

### Before Opening a PR

- [ ] Branch is up to date with `main`
- [ ] All Quality Gates pass locally (see below)
- [ ] PR description is filled out using the PR template
- [ ] Self-review completed — read your own diff

### PR Requirements

- **Title:** Must follow Conventional Commits format
- **Description:** Use the PR template (`.github/pull_request_template.md`)
- **Size:** Keep PRs small — aim for under 400 lines changed. Large PRs get deprioritized.
- **Reviewer:** Assign at least one reviewer (human or Kimi code sweep via `just review`)

### PR Template Sections

1. **Summary** — What does this PR do? (2-3 sentences)
2. **Motivation** — Why is this change needed? Link to issue or spec.
3. **Changes** — Bullet list of files changed and why.
4. **Testing** — How was this tested? What Quality Gates passed?
5. **Screenshots / Artifacts** — For UI changes or planning artifacts.

### Merge Strategy

- **Squash and merge** for feature branches (keeps main history clean)
- **Merge commit** for release branches only
- Delete branch after merge

---

## Quality Gates

A PR cannot be merged until all gates are green:

| Gate | Check | Tool |
|------|-------|------|
| **Gate 1 (Syntax)** | Lint passes, no type errors | ESLint, TypeScript, Ruff |
| **Gate 2 (Logic)** | All tests pass | pytest, vitest |
| **Gate 3 (Security)** | No new vulnerabilities | CodeQL, Snyk |
| **Gate 4 (Consensus)** | PR approved by reviewer | GitHub review or `just review` |

Run all gates locally before pushing:

```bash
just lint        # ESLint + Ruff
just typecheck   # TypeScript + mypy
just test        # All unit tests
just review      # Kimi full-codebase sweep
just ci          # All of the above in sequence
```

---

## For AI Agents

AI agents contributing to this repo must additionally:

1. **Read AGENTS.md first** — every session, before any other file.
2. **Write artifacts to `.planning/`** — not to `docs/` or root.
3. **Follow the Decide-Act-Verify loop** — no action without verification.
4. **Log decisions to `.planning/ledger.md`** — before moving to next task.
5. **Write a handoff note** when passing work to another agent.
6. **Never modify docs/ directly** — docs are source of truth, not output.
7. **Never run install commands** — `just setup` is for humans only.

---

## Issue Labels

| Label | Meaning |
|-------|---------|  
| `bug` | Something is broken |
| `feat` | New feature request |
| `chore` | Maintenance task |
| `docs` | Documentation improvement |
| `blocked` | Cannot proceed — needs input |
| `agent-task` | Assigned to an AI agent |
| `human-required` | Requires human judgment |
| `priority:high` | Must be in current sprint |
| `priority:low` | Nice to have |

---

## Getting Help

- **Blockers:** Post in `.planning/prp.md` section 9, then notify Ruflo
- **Questions about specs:** Read `docs/` first, then ask Antonio via Telegram (@ReidTheArchitect)
- **CI failures:** Run `just doctor` to verify toolchain, then `just ci` locally
