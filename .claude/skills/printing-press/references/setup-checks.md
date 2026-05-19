# Setup Checks

Post-contract verification after running `printing-press preflight`. Capture `PRINTING_PRESS_BIN=<absolute-path>` from output before proceeding. All subsequent invocations use this absolute path.

## Six Conditional Checks

Apply each only when its trigger signal is present; skip otherwise.

### 1. Setup Errors
Trigger: `[setup-error]` in output.
Action: **Halt immediately.** Surface the message verbatim. The user must manually install missing prerequisites before re-invoking.

### 2. Repo Upgrades
Trigger: `[repo-upgrade-available]` in output.
Action: Prompt user whether to pull `origin/main`. On successful pull, stop the current run — user must reload. Cache the skip decision against the detected SHA to prevent re-prompting for the same target.

### 3. Minimum Version Compatibility
Trigger: binary version < `min-binary-version` frontmatter field.
Action: Query binary version via `printing-press --version`. Compare using semver rules. Incompatible binaries halt execution with upgrade instructions: `go install github.com/mvanhorn/cli-printing-press/v4/cmd/printing-press@latest`.

### 4. Standalone Binary Updates
Trigger: `[upgrade-available]` in output.
Action: Offer upgrade prompt. After accepting, re-resolve `PRINTING_PRESS_BIN` before continuing — `go install` may write to a different location than the original binary.

### 5. Browser Backend Installation
Trigger: `[browser-tools-missing]` in output.
Action: Prompt for installing `browser-use` and/or `agent-browser` based on what's missing. Failed installs **do not block** the run; proceed with `--docs` fallback if needed.

### 6. Shadow Advisory
Trigger: `[binary-shadow]` in output.
Action: Inform the user that a global binary differs from the local build being used. Take no action beyond informing.
