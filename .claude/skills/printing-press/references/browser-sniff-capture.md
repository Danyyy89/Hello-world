# Browser-Sniff Capture

Invoked only after user approval at Phase 1.7. **Replayability is the success criterion** — captures must produce shippable surfaces (replayable API calls, persisted query registries, importable browser-clearance cookies, or structured HTML/SSR/RSS extraction targets).

## Step 1: Tool Selection

Priority order:
1. `browser-use` CLI mode — stable, version-locked, no API key required
2. `agent-browser` — fallback when browser-use unavailable
3. Chrome-MCP — failure-recovery fallback only
4. Manual HAR — user exports from DevTools (optionally with computer-use screenshot guidance)

Detect available tools via agent tool catalog inspection. Set `CHROME_MCP_AVAILABLE` and `COMPUTER_USE_AVAILABLE` flags for later recovery menu composition.

## Step 1d: Session Transfer (Authenticated Targets Only)

- Chrome running: save cookies via `agent-browser`, close Chrome, restart `browser-use` with Chrome profile.
- Chrome offline: load `browser-use` directly with the Chrome profile containing saved credentials.

## Step 2: Capture Execution

| Tool | Capture Method |
|---|---|
| browser-use | Performance API collection via CLI (open/eval/scroll/close) |
| agent-browser | HAR-based capture with enriched response bodies |
| Chrome-MCP | Drive user's existing Chrome session via extension |
| Manual HAR | User exports from DevTools |

**Critical Rules:**

1. Use click-based SPA navigation after installing interceptors; `browser-use open` resets the JS context and destroys fetch/XHR interceptors.
2. Never skip auth discovery when sessions expire. If login page loads instead of account pages, offer headed login as fallback.
3. Adaptive request pacing: start at 1 s, ramp down 20% per 5 consecutive successes (minimum 0.3 s), double on HTTP 429, hard-stop 30 s after 3 consecutive 429s.
4. Detect GraphQL BFF patterns by checking whether >50% of XHR/fetch POST URLs resolve to the same endpoint; extract operation names from request bodies.
5. Install response-body interceptors **before** navigation to avoid re-firing requests against WAFs.

## Step 2c.5: Challenge-Only Capture Safety

When capture returns only challenge pages, login redirects, or access-denied responses, **do not proceed to Phase 2.** Present recovery menu:

- Try cleared-browser capture again
- Try chrome-MCP (badge as Recommended when anti-bot block detected, if `CHROME_MCP_AVAILABLE`)
- I'll provide a HAR from DevTools
- Discuss alternate CLI scope

## Credential Protection

Scrub at write time before persisting to disk:
- Headers: `Authorization`, `Cookie`, `Set-Cookie`, `Proxy-Authorization`, `X-Api-Key`, `X-Auth-Token`, `X-Session-Id`, and any matching `/^x-.*-(token|key|auth|session|secret)$/i`
- Query params: `access_token=`, `api_key=` → `REDACTED`

## Replayability Validation

After capture:
- Cookie auth: replay test endpoints with and without cookies to confirm cookies produce authenticated responses
- All endpoints: confirm they can round-trip through Surf with Chrome TLS fingerprint without resident browser execution
- Thin results (<5 endpoints): re-sniff with explicit interaction; compare against community wrappers and Phase 1 research

## Report

Write `$DISCOVERY_DIR/browser-sniff-report.md` covering: user goal flow, pages and interactions, configuration, endpoints discovered (method/path/status/auth), traffic analysis, coverage gaps, response samples, rate-limit events, authentication context, bundle extraction results.

Phase 5.5 cleanup removes ephemeral aids: `rm -f "$DISCOVERY_DIR"/devtools-help-*.png`.
