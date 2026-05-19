---
description: "QA pass on an existing generated CLI: diagnose, fix, verify, then ship or hold"
context: fork
user-invocable: true
min-binary-version: "4.0.0"
model: claude-sonnet-4-6
---

# /printing-press-polish

Standalone QA skill for generated CLIs. Runs diagnostics and automated fixes to achieve publish-readiness. Operates in a forked context to isolate diagnostic noise from the parent workflow.

## Invocation

```
/printing-press-polish <slug>
/printing-press-polish --standalone <slug>
```

## Preflight

Enforce `min-binary-version: 4.0.0` before proceeding. Locate the `printing-press` binary; prefer local build within the printing-press repository before falling back to global PATH. Export `PRINTING_PRESS_BIN=<absolute-path>`.

## Caller Modes

**Standalone** (invoked directly or with `--standalone` flag): publishes when ship decision is reached.

**Mid-pipeline** (invoked from main SKILL.md Phase 4): defers publish to parent workflow.

## Phase 1: Baseline Diagnostics

Capture before-state:

```bash
$PRINTING_PRESS_BIN verify --dir "$CLI_DIR"
$PRINTING_PRESS_BIN scorecard --dir "$CLI_DIR" --spec "$SPEC_PATH"
$PRINTING_PRESS_BIN dogfood --dir "$CLI_DIR"
$PRINTING_PRESS_BIN audit-tools --dir "$CLI_DIR"
$PRINTING_PRESS_BIN pii-scan --dir "$CLI_DIR"
```

Also run: `go vet ./...` from the CLI directory.

Perform a divergence check against the public library to prevent silent overwrites of upstream fixes.

## Phase 2: Fix

Prioritized remediation order:

1. MCP surface migration (blocking if incomplete)
2. Verify failures (exact path/auth mismatches)
3. Dead code removal
4. CLI command descriptions
5. README completeness (missing standard sections)
6. SKILL static checks
7. PII and tool audit findings

Maximum **3 total polish invocations per session** (initial + 2 retries). Do not re-run research or generation — polish fixes existing CLIs only.

Do not modify files outside the target CLI directory.

## Phase 3: Re-Diagnose

Re-run all Phase 1 commands and compare against baseline. Validate that fixes resolved the underlying issues before emitting ship recommendation.

## Ship Decision

| Decision | Criteria |
|---|---|
| `ship` | Verify ≥ 80%, scorecard ≥ 75, zero SKILL/workflow/PII/tools-audit blocking findings; publish-validate passes (standalone mode) |
| `ship-with-gaps` | Verify ≥ 65%, scorecard ≥ 65, README documents known gaps, same blocking-findings gates |
| `hold` | Below thresholds or unresolved blockers |

Report before/after deltas with structured results in `proofs/<stamp>-fix-<slug>-polish.md`.
