# Secret & PII Protection

Applied during Phase 5.5 before archiving manuscripts, and at any point an artifact is written to disk.

## API Key Scanning

Use fixed-string matching (`grep -F`), not regex — API keys often contain regex metacharacters (`+`, `/`, `.`, `=`). Skip keys shorter than 16 characters to prevent over-redaction. Replace detected keys using Python's literal string replacement:

```python
content = content.replace(actual_key_value, "REDACTED")
```

## HAR File Auth Stripping

Strip credentials from four locations using `jq` before archiving:
1. Request headers: `Authorization`, `Cookie`
2. Response headers: `Set-Cookie`
3. Query string parameters with token-like names
4. Cookie entries

## Runtime Key Handling

- Keys remain in shell variables only — never written to disk
- Passed via environment variables to prevent visibility in process lists
- Dry-run output may contain keys in query parameters for debugging; must **not** be captured in proof artifacts

## PII in Workspace Data

Live testing produces org names, team names, email addresses. Use generic descriptions in artifacts:
- "the workspace" instead of actual org names
- "5 overloaded members" instead of listing names

## Two-Tier Pattern Scanning

**Tier 1 (Auto-Redact):** Vendor-anchored patterns with near-zero false-positive rate (e.g. `sk-or-v1-` for OpenRouter, `ghp_` for GitHub tokens). Redact automatically without user prompt.

**Tier 2 (Warn-by-Default):** Generic patterns — emails, unanchored bearer tokens, capitalized name patterns. Prompt user with context before redacting. False-positive auto-redaction permanently corrupts a manuscript artifact.

**Allowlist:** Suppress matches against spec-derived terms, CLI help output, and universal terms like "Pull Request" and "Service Account".

## Session State Isolation

Browser session files are stored outside discovery directories by design, preventing accidental archive inclusion regardless of operator actions.
