# claude_codex_config

Personal Claude Code / Codex configuration snippets I reuse across machines.

## Currently includes

- [`claude_code/claude-code-cache-statusline-guide.md`](claude_code/claude-code-cache-statusline-guide.md) — two-line status line with model, CWD, git branch, context-usage bar, cache cw/cr/hit, session cost, and elapsed time.
- [`claude_code/claude-code-add-opus46model.md`](claude_code/claude-code-add-opus46model.md) — register a custom model entry (e.g. Opus 4.6 with 1M context) in the `/model` picker.

## Install (paste into Claude Code)

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/README.md and install every config it lists. For each bullet under "Currently includes", fetch the guide at https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/<path> and apply it end-to-end. Merge into ~/.claude/settings.json (never overwrite), don't change my default "model" key, then remind me to restart Claude Code.
```

That's it. The prompt reads this README, walks every bullet under **Currently includes**, fetches the guide, and applies it.

## Adding a new guide

1. Drop the guide into `claude_code/` using the standard layout: `When to Use → What to Do → Verification → Troubleshooting → Requirements`.
2. Add one bullet under **Currently includes** linking to it.
3. Push to `main`. The install prompt picks it up automatically — no prompt edits needed.

## Requirements

- `jq` on `$PATH`
- `bash` for the status-line script's shebang (your interactive shell can be bash or zsh)
- A Claude Code build that supports the `statusLine` setting and `ANTHROPIC_CUSTOM_MODEL_OPTION*` env vars
