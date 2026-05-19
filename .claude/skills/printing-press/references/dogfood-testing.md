# Dogfood Testing

Supplementary guidance for Phase 4 (Shipcheck). The main skill SKILL.md contains the core protocol (steps 1–4, test lists, and reporting format). This reference provides additional context.

## Common Failure Patterns

| Symptom | Root Cause | Fix |
|---|---|---|
| Empty list results | Response envelope not unwrapped | Fix client or output helpers |
| `--select` filtering broken | `filterFields` can't parse envelope | Add `extractResponseData` |
| CSV output shows JSON | CSV check runs after JSON pipe check | Fix promoted template output path |
| Search returns no results | FTS table connection issue in `search.go` switch | Fix switch statement |
| `sync` endpoint 404 | API version header mismatch | Add per-path client headers |
| Mutation command naming wrong | `operationId` not cleaned up in Command Use field | Clean operationId |
| Help text with placeholders | Example field contains unresolved placeholder values | Fix Example field |
| "0 results" for `me` command | Provenance counter assumes arrays | Count single objects as 1 |
| Discoverable cancel/confirm paths | Subcommands hidden under operationId groups | Check available commands |

## What NOT to Test

Exclude: internal implementation details, performance benchmarks, concurrent access, edge cases requiring specific account configurations, endpoints beyond user permission levels.

## Test Data Consent

Before creating test data, confirm with users:

> "I'll create test data on your account (test bookings, test records, etc.) and clean up by cancelling/deleting them. OK to proceed?"
