# CLI Printing Press — Agent Rules

This repository hosts the **CLI Printing Press** skills: a research-generate-build-shipcheck system that turns API specifications (OpenAPI, HAR, or URL) into ship-ready CLIs and MCP servers optimized for AI agents.

## Available Skills

| Slash Command | Purpose |
|---|---|
| `/printing-press <app>` | Full pipeline: resolve → research → generate → build → shipcheck |
| `/printing-press-polish <slug>` | QA pass on an existing CLI: diagnose, fix, verify |
| `/printing-press-reprint <slug>` | Regenerate a published CLI under the current machine |
| `/printing-press-score <slug>` | Evaluate a CLI against the Steinberger scorecard |

## Environment Requirements

- **Go** 1.26.3 or newer
- **printing-press binary** v4.0.0+ (`go install github.com/mvanhorn/cli-printing-press/v4/cmd/printing-press@latest`)
- `$GOPATH/bin` on `$PATH`
- Optional: `browser-use`, `agent-browser` for browser-sniff phases

## Cardinal Rules

### Secret & PII Protection
API key *values*, tokens, passwords, and session cookies must **never** appear in any artifact, commit, or proof file. Only environment variable *names* (e.g. `STEAM_API_KEY`) and placeholders are safe. HAR files must have auth headers stripped before use — see `.claude/skills/printing-press/references/secret-protection.md`.

### Testing Requirement
Do not ship a CLI that hasn't been behaviorally tested against real targets. Build passes and verify rates measure structure, not correctness. Full dogfood testing is mandatory; gaps found during testing must be fixed before shipping, not deferred to v0.2.

### Feature Scope Lock
Features approved in Phase 1.5 are shipping scope. Mid-build downgrades to stubs require return to Phase 1.5 for explicit re-approval.

### No Time Estimates
Skip fabricated time ranges ("~15–30 min") in user-facing prompts. Describe scope instead (files touched, lines of code). Exceptions: whole-CLI runs (~30–60 min typical), tool installs (~10 s), network-bound operations (~5–10 min).

## Artifact Layout

Each run scopes its output to:

```
$PRESS_RUNSTATE/          mutable working state (research, proofs, pipeline artifacts)
$PRESS_LIBRARY/           published CLIs under <api-slug>/ subdirectories
$PRESS_MANUSCRIPTS/       archived run evidence
```

A `state.json` file persists run metadata (`api_name`, `run_id`, `working_dir`, `spec_path`) so `/printing-press-score` can rediscover the current run.

## Scoring Bar

A publishable CLI requires:
- Tier 1 Infrastructure score ≥ 25 / 50
- Tier 2 Domain Correctness score ≥ 25 / 50
- Combined ≥ 75 / 100 (grade B)
- Zero open blocking findings (PII, broken auth, missing MCP surface)

## Cross-Tool Compatibility

Skills in `.claude/skills/` are loaded by both **Claude Code** (`--plugin-dir .`) and **OpenCode** natively. The `opencode.json` at the repo root configures project-level context for OpenCode sessions.
