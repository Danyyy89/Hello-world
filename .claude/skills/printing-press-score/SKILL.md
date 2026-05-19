---
description: "Evaluate a generated CLI against the Steinberger scorecard; supports single-score and comparison modes"
context: fork
user-invocable: true
min-binary-version: "4.0.0"
model: claude-sonnet-4-6
---

# /printing-press-score

Evaluates generated CLIs against the Steinberger bar. Supports three modes: rescore the current CLI, score a single CLI by name or path, and compare two CLIs side-by-side.

## Invocation

```
/printing-press-score                           # rescore current CLI
/printing-press-score notion                    # score by slug
/printing-press-score ./path/to/cli             # score by path
/printing-press-score notion linear             # compare two CLIs
```

## Preflight

Locate the `printing-press` binary; prefer local build before global PATH. Export `PRINTING_PRESS_BIN=<absolute-path>`. Enforce `min-binary-version: 4.0.0`.

## CLI Resolution

Resolve the CLI identifier via:
1. Direct path (if argument looks like a path)
2. Exact library match in `$PRESS_LIBRARY/`
3. Suffix variations (`-pp-cli`, `-cli`)
4. Glob patterns
5. Archived state files in `$PRESS_MANUSCRIPTS/`

Halt with a clear error if no match found.

## Spec Discovery

Locate the OpenAPI spec at:
1. `<cli-dir>/spec.json`
2. Archived `state.json` metadata from `$PRESS_MANUSCRIPTS/<slug>/`

## Execution

```bash
# Single score
$PRINTING_PRESS_BIN scorecard --dir "$CLI_DIR" --spec "$SPEC_PATH"

# Comparison (run in parallel)
$PRINTING_PRESS_BIN scorecard --dir "$CLI_DIR_A" --spec "$SPEC_PATH_A" &
$PRINTING_PRESS_BIN scorecard --dir "$CLI_DIR_B" --spec "$SPEC_PATH_B" &
wait
```

## Output Rendering

Display rich markdown tables showing:
- Dimension scores with file-level evidence
- Tier 1 and Tier 2 subtotals
- Combined total and letter grade
- Identified gaps with fix suggestions
- Delta columns (comparison mode only)

Mark unscored dimensions as N/A, not zero.

For scoring dimension details and fix patterns, consult `.claude/skills/printing-press/references/scorecard-patterns.md`.
