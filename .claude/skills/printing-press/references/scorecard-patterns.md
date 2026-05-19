# Scorecard Patterns

Interpretation guide for `printing-press scorecard` output. Grades: A ≥ 80%, B ≥ 65%, C ≥ 50%, D ≥ 35%, F < 35%.

## Scoring Dimensions (0–10 each, total 0–100)

### Tier 1: Infrastructure (50 pts)

| Dimension | What to Check | Max |
|---|---|---|
| Output Modes | `internal/cli/root.go`: json, plain, select, table, csv patterns (+2 each) | 10 |
| Auth | `os.Getenv` calls in `config.go`, `auth.go` file existence | 10 |
| Error Handling | `hint:` / `Hint:` strings and `code:` occurrences in `helpers.go` | 10 |
| Terminal UX | `colorEnabled`, `NO_COLOR`, `isatty` implementations | 10 |
| README | Exact sections: "Quick Start," "Output Formats," "Agent Usage," "Troubleshooting," "Doctor" | 10 |

### Tier 2: Domain Correctness (50 pts)

| Dimension | What to Check | Max |
|---|---|---|
| Doctor | HTTP patterns: `http.Get`, `http.NewRequest`, `http.Client` (×2 each) | 10 |
| Agent Native | `root.go` + `helpers.go`: json, select, dry-run, non-interactive, stdin, conflict handling | 10 |
| Local Cache | Cache implementation patterns; optional backends (sqlite/bolt/badger) | 10 |
| Breadth | CLI command file count (excluding standard ones); thresholds scale 0–10 | 10 |
| Path Validity | Endpoint paths match spec | 5 |
| Type Fidelity | Response field types match spec | 5 |

## Critical Notes

- Exact string matching: presence of `"json"` in root.go counts differently than in a comment
- File-specific scoping: checks apply to specific files, not the whole repo
- Breadth rewards architectural diversity, not endpoint proliferation
- DeadCode dimension uses negative scoring: each dead function found deducts points

## Common Score Gaps and Fixes

| Low Score Area | Typical Fix |
|---|---|
| Output Modes < 6 | Add `--csv`, `--select`, `--plain` flags to root command |
| Auth < 6 | Ensure `auth.go` exists; add `os.Getenv` for token in `config.go` |
| README < 6 | Add missing standard sections verbatim |
| Agent Native < 6 | Add `--dry-run`, `--yes`, `--no-input` flags; handle stdin |
| Breadth < 6 | Add domain-specific workflow commands (list, search, sync, me) |
