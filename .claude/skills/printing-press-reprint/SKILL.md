---
description: "Regenerate a published CLI under the current Printing Press machine, preserving prior research and patches"
context: fork
user-invocable: true
min-binary-version: "4.0.0"
model: claude-sonnet-4-6
---

# /printing-press-reprint

Regenerates an existing published CLI under the current Printing Press machine. Preserves prior research, novel features, and post-publish patches as reconciliation context rather than discarding them.

## When to Use

- After a Printing Press upgrade (new MCP surface, auth modes, scoring rubric) that would benefit the CLI more than manual polish
- When the user wants prior features re-evaluated against current personas rather than carried forward verbatim

## Invocation

```
/printing-press-reprint <slug>
/printing-press-reprint notion
/printing-press-reprint discord
```

## Phase A: Reconcile Library Presence

Match the user's API argument (slug, brand name, or fuzzy match) against `registry.json`. Compare local and public `run_id` and `generated_at` timestamps:

- Local only → import from public library if available
- Public only → import to local
- Both present, same `run_id` → continue
- Both present, different `run_id` → surface discrepancy and ask user which to use as base

Halt if no match found at any confidence level.

## Phase B: Verify Reconcilable Context

Locate prior research files at `$PRESS_MANUSCRIPTS/<api-slug>/<run-id>/`. Refresh patches from the public library.

If research predates the `research.json` contract (old format), surface this degradation to the user and offer: continue with degraded context, or redo research from scratch.

## Phase C: Research Strategy Recommendation

Based on research age:

| Age | Recommendation |
|---|---|
| < 30 days | Reuse appears safe |
| 30–120 days | Reuse plausible; user should flag known API churn |
| > 120 days | Redo recommended |

Ask user to choose: reuse prior research, redo from scratch, or preview prior research first.

## Phase D: Handoff to /printing-press

Invoke `/printing-press` with:
- The API target (resolved slug/name)
- User's regeneration reason
- Research mode selection (reuse/redo/preview)
- Prior patches (when present) framed as an informational watch-list to prevent silent regression of live-validated fixes

The handed-off `/printing-press` run proceeds from Phase 1 with the enriched context.
