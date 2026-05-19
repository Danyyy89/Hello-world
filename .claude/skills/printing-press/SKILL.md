---
description: "Generate ship-ready CLIs and MCP servers from API specs via a research-generate-build-shipcheck loop"
context: fork
user-invocable: true
min-binary-version: "4.0.0"
model: claude-sonnet-4-6
---

# /printing-press

Generates ship-ready CLIs for APIs through a lean research-generate-build-shipcheck loop. Eliminates pre-code documentation theater and late-stage validation surprises.

## Invocation Examples

```
/printing-press Notion
/printing-press Discord codex
/printing-press https://example.com
/printing-press --spec ./openapi.yaml
/printing-press --har ./capture.har --name MyAPI
```

## Operation Modes

**Default** — Claude handles research, generation orchestration, implementation, and verification.

**Codex** (`codex` or `--codex` argument) — Delegates pure code-writing (store/data-layer, workflow commands, README edits) to Codex CLI while keeping research, gap prioritization, and verification on Claude. Includes a 3-failure circuit breaker before reverting to local completion. See `references/codex-delegation.md`.

**Polish** — Standalone skill `/printing-press-polish` for second-pass improvements to existing CLIs.

## Preflight

Before user engagement, enforce mandatory preflight by running:

```bash
printing-press preflight
```

Read and follow `references/setup-checks.md` to handle all six conditional signals:
`[setup-error]`, `[repo-upgrade-available]`, `[upgrade-available]`, `[browser-tools-missing]`, `[binary-shadow]`, and min-version incompatibility.

Capture `PRINTING_PRESS_BIN=<absolute-path>` from preflight output. **Every subsequent `printing-press ...` call must use this absolute path** to eliminate PATH shadowing by stale global installs.

## User Flow

### No Arguments (Orientation)
Display process overview and example invocations, then ask whether to type an API target or browse the public library first.

### With Arguments (Briefing)
Confirm the process timeline, describe end deliverables, and collect upfront context (API keys, browser auth, vision/specific features). Route to **Multi-Source Priority Gate** if multiple distinct services are named.

### Multi-Source Priority Gate
For combo CLIs (e.g., "Google Flights + Kayak + FlightAware"), user confirms source ordering before Phase 1 research. Primary source gets headline commands and README prominence. Economics check surfaces free/paid scope conflicts.

## Run Initialization

Create scoped artifact paths at run start:

```bash
PRESS_RUNSTATE="$(printing-press runstate-path)"
PRESS_LIBRARY="$(printing-press library-path)"
PRESS_MANUSCRIPTS="$(printing-press manuscripts-path)"
RUN_ID="$(printing-press run-id)"
```

Write `state.json` with `api_name`, `run_id`, `working_dir`, `spec_path` so `/printing-press-score` can rediscover the run.

## Phase 0: Resolve & Reuse

**URL Detection** — Light GET fetch. Specs (OpenAPI/Swagger JSON/YAML) and HAR files route directly to `--spec`/`--har` paths. HTML or fetch failures trigger disambiguation: target the API or the website itself?

**Prior Research Reuse** — Check `$PRESS_MANUSCRIPTS/<api-slug>/` for existing research. Reuse if recent and good. On binary upgrades with version deltas affecting assumptions, re-validate prior research before reuse.

**Library Check** — Detect active builds (lock status), existing CLIs (`go.mod` presence), and incomplete debris. Present options: regenerate fresh, improve existing, or review prior research first. Reclaim stale locks automatically on `lock acquire`.

**Public-Library Check** — Scan `mvanhorn/printing-press-library/registry.json` for existing CLIs the user may have named differently. Classify at High (same product, different name) and Medium (adjacent/category) confidence. Display top 2 candidates per source within 4-option prompt limits. Combo CLIs surface component-level matches with "informational" framing.

**API Key Gate** — Determine if authentication is required. Detect existing tokens via environment or `gh auth` patterns. If required but absent, offer user option to provide key for Phase 5 live testing or proceed without. Public APIs skip this gate entirely.

## Phase 1: Research Brief

**When `BROWSER_SNIFF_TARGET_URL` is set** (website-target mode from Phase 0): skip catalog/spec searches and focus on feature, users, workflows, and competitors.

**Built-in Catalog Check** — For spec-based entries, offer user choice between catalog config (skips discovery) or full discovery. For wrapper-only entries (no spec), explain that hand-written module or browser-sniffing is required; record choice in `state.json`.

**Single Research Brief Output** — `research/<stamp>-feat-<api>-pp-cli-brief.md` answers six questions:

1. What is this API actually used for?
2. Top 3–5 power-user workflows?
3. Table-stakes competitor features?
4. What data deserves local store?
5. Why choose this CLI over incumbents?
6. Product name and thesis?

Sections: API Identity, Reachability Risk, Top Workflows, Table Stakes, Data Layer, Codebase Intelligence (DeepWiki), User Vision (if provided), Source Priority (for combos), Build Priorities.

## Phase 1.6: Pre-Browser-Sniff Auth Intelligence

**Skip if briefing already captured `AUTH_CONTEXT`.** Proactively assess what auth context user could provide:

- API key auth → ask for key for read-only smoke testing
- Browser session auth → ask if user is logged in (disclose CLI will support browser-based auth)
- Dual auth → ask about both in sequence
- No auth needed → skip silently

Capture responses in `AUTH_CONTEXT` so API Key Gate does not re-ask.

## Phase 1.7: Browser-Sniff Gate

**Mandatory hard stop.** Evaluate whether browser-sniffing would improve spec:

| Condition | Decision |
|---|---|
| Spec exists + gaps | Offer enrichment |
| No spec + community docs available | Offer primary discovery |
| No signals | Skip silently |

**Marker File Contract** — Every source named in briefing must have exactly one entry in `$PRESS_RUNSTATE/runs/$RUN_ID/browser-browser-sniff-gate.json` before Phase 1.5 proceeds. Entries contain: `source_name`, `decision` (approved/declined/skip-silent/pre-approved), `reason`, `asked_at`. Phase 1.5 **halts** with no marker file.

**Banned Skip Reasons** — Do not skip for: "client-rendered needs Playwright," "direct HTTP got 403," "time budget looks tight," "substitute data source exists," "documentation looks thorough." These are exactly when browser-sniff adds value. Use `AskUserQuestion`.

**Direct HTTP Challenge Rule** — When bot-protection signals appear (403, 429, WAF, DataDome), run `printing-press probe-reachability` before announcing browser escalation. Settle runtime (`standard_http` | `browser_http` | `browser_clearance_http` | `unknown`) separately from discovery decision.

**Approval Disclosure** — Enumerate possibilities (browser-use, agent-browser, manual HAR, chrome-MCP, computer-use screenshot guidance).

**Time Budget** — Browser-sniff approval-to-results: 3 minutes max (applied AFTER user approves, not as a reason to skip).

If approved: write marker entry, then read and follow `references/browser-sniff-capture.md`.
If declined: write marker entry and proceed with existing spec or `--docs` fallback.

## Phase 1.8: Crowd-Sniff Gate

**Skip entirely if user passed `--spec`.** Evaluate whether npm SDKs and GitHub code mining would improve spec. Time budget: 10 minutes max.

Decision matrix: spec exists + gaps → enrich; no spec + SDKs available → primary discovery; no signals → skip silently.

If approved: follow `references/crowd-sniff.md`.
If declined: proceed with existing spec or `--docs`.

## Phase 1.5: Ecosystem Absorb Gate

**Mandatory stop gate before generation.** Requires valid `browser-browser-sniff-gate.json` marker file with entry for every briefing source.

**Step 1.5a** — Search for every tool touching the API: Claude Code plugins, MCP servers, Claude skills, competing CLIs, npm packages, official plugins, MCP registries, automation scripts, SDKs, client libraries.

**Step 1.5a.5** — For top 1–2 MCP repos with public source, extract ground-truth API usage (endpoint paths, auth patterns, response field selections). Max 3 minutes. Feed into absorb manifest with "(source)" attribution.

**Step 1.5a.6** — Query DeepWiki for semantic codebase understanding on discovered GitHub repos. Time budget: 2 minutes. Skip on unavailability. See `references/deepwiki-research.md`.

**Step 1.5b** — Create absorb manifest cataloging EVERY feature from EVERY tool. Each row = a feature the CLI must match or beat. Stubs must be explicit with status and reason. No quiet stubs for shipping-scope features.

**Step 1.5c** — Identify transcendence features via customer model and use-case brainstorming. Minimum 5 transcendence features that only this approach can deliver.

**Step 1.5c.5** — Always spawn the subagent for novel-feature suggestion (customer model → 2× candidates → adversarial cut). See `references/novel-features-subagent.md`. No manual fallback; subagent is the only valid path.

**Step 1.5d** — Present Phase Gate 1.5 prose: research summary, absorb manifest (absorbed + transcendence tables), hand-code feature count, build priorities, and explicit stub list. User must approve before generation.

## Phase 2: Generate

Run:
```bash
$PRINTING_PRESS_BIN generate --spec "$SPEC_PATH" --name "$API_SLUG" --output "$PRESS_LIBRARY/$API_SLUG"
```

Review generated scaffolding for structural completeness. Identify high-value gaps from absorb manifest not covered by generation.

## Phase 3: Build GOAT

Implement high-priority hand-code features identified in Phase 1.5:
- Workflow commands (compound queries using local store)
- Data pipeline enhancements
- Auth edge cases
- MCP surface completeness
- Novel/transcendence features

Lock heartbeat:
```bash
$PRINTING_PRESS_BIN lock heartbeat --run-id "$RUN_ID"
```

## Phase 4: Shipcheck

Read and follow `references/dogfood-testing.md` for the full dogfood protocol. Run the unified shipcheck block:

```bash
$PRINTING_PRESS_BIN verify --dir "$PRESS_LIBRARY/$API_SLUG"
$PRINTING_PRESS_BIN scorecard --dir "$PRESS_LIBRARY/$API_SLUG" --spec "$SPEC_PATH"
$PRINTING_PRESS_BIN dogfood --dir "$PRESS_LIBRARY/$API_SLUG"
```

Apply `references/scorecard-patterns.md` to interpret scorecard output and prioritize fixes.

**Ship Decision:**
- `ship` — Verify ≥ 80%, scorecard ≥ 75, zero blocking findings. Run `/printing-press-polish` before publish.
- `ship-with-gaps` — Verify ≥ 65%, scorecard ≥ 65, README documents known gaps.
- `hold` — Below thresholds or unresolved blockers.

## Phase 5: Live Smoke Tests (Optional)

When an API key is available and user approves, run read-only smoke tests against live API. See `references/per-source-rate-limiting.md` for adaptive pacing.

Proof artifact: `proofs/<stamp>-fix-<api>-pp-cli-live-smoke.md`.

## Phase 5.5: Publish

When ship decision is `ship` or `ship-with-gaps`:

1. Secret scan: follow `references/secret-protection.md` before writing any artifact.
2. Archive run artifacts to `$PRESS_MANUSCRIPTS/<api-slug>/<run-id>/`.
3. Publish CLI to library.

## Outputs (up to 5 durable artifacts per run)

1. `research/<stamp>-feat-<api>-pp-cli-brief.md`
2. `research/<stamp>-feat-<api>-pp-cli-absorb-manifest.md`
3. `proofs/<stamp>-fix-<api>-pp-cli-build-log.md`
4. `proofs/<stamp>-fix-<api>-pp-cli-shipcheck.md`
5. `proofs/<stamp>-fix-<api>-pp-cli-live-smoke.md` (optional)
