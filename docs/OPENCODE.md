# Using Printing Press CLIs in OpenCode

This guide explains how to use and develop CLI Printing Press skills inside an **OpenCode** terminal session.

## Session Setup

OpenCode picks up project rules automatically from `AGENTS.md` at the repo root and configuration from `opencode.json`. No extra flags are needed.

Start a session in the repository root:

```bash
opencode
```

Skills in `.claude/skills/` are discovered automatically by both OpenCode and Claude Code.

## Available Slash Commands

| Command | Purpose |
|---|---|
| `/printing-press <app>` | Full pipeline: resolve → research → generate → build → shipcheck |
| `/printing-press-polish <slug>` | QA pass on an existing CLI |
| `/printing-press-reprint <slug>` | Regenerate a published CLI under the current machine |
| `/printing-press-score <slug>` | Evaluate a CLI against the Steinberger scorecard |

## Prerequisites

Install the `printing-press` binary before invoking any skill:

```bash
go install github.com/mvanhorn/cli-printing-press/v4/cmd/printing-press@latest
```

Verify Go 1.26.3+ is installed and `$GOPATH/bin` is on `$PATH`:

```bash
go version
printing-press --version
```

Optional browser backends for sniff phases:

```bash
pip install browser-use
npm install -g agent-browser
```

## Authentication

Each generated CLI manages its own auth. Common patterns:

- Environment variables: `export NOTION_API_KEY=<key>`
- Built-in auth subcommand: `<api>-pp-cli auth login`
- MCP server integration: configure in your `opencode.json` `mcpServers` block

**Never commit secrets.** Use env vars or your OS secret store. See `.claude/skills/printing-press/references/secret-protection.md`.

## MCP Server Integration

Generated CLIs include MCP server stubs. Add them to `opencode.json` after generation:

```json
{
  "mcpServers": {
    "notion-pp": {
      "type": "stdio",
      "command": "notion-pp-mcp",
      "env": {
        "NOTION_API_KEY": "${NOTION_API_KEY}"
      }
    }
  }
}
```

## CLI vs MCP Trade-offs

| | CLI Skills | MCP Server |
|---|---|---|
| Context tokens | Lower (skill-driven, selective) | Higher (full tool surface) |
| Discovery | Slash command invocation | IDE-native tool catalog |
| Best for | Scripted workflows, pipelines | Interactive IDE use |

## Per-Repository Rules

OpenCode reads `AGENTS.md` in the repository where you are committing. Always follow the `AGENTS.md`, `CONTRIBUTING.md`, and any rules files in the target repository — not just this one.

## Troubleshooting

| Problem | Fix |
|---|---|
| `printing-press: command not found` | Add `$GOPATH/bin` to `$PATH` |
| Skill not found | Restart OpenCode after adding `.claude/skills/` |
| Binary version mismatch | Run `go install ...@latest` and restart |
| Browser tools missing | `pip install browser-use` or `npm install -g agent-browser` |
| Token auth failure | Run `printing-press auth doctor` |
