# claude_codex_config

Personal Claude Code / Codex configuration snippets I reuse across machines.

## Currently includes

- [`claude_code/claude-code-cache-statusline-guide.md`](claude_code/claude-code-cache-statusline-guide.md) — two-line status line with model, CWD, git branch, context-usage bar, cache cw/cr/hit, session cost, and elapsed time.
- [`claude_code/claude-code-add-opus46model.md`](claude_code/claude-code-add-opus46model.md) — register a custom model entry (e.g. Opus 4.6 with 1M context) in the `/model` picker.
- [`claude_code/claude-code-skip-webfetch-preflight.md`](claude_code/claude-code-skip-webfetch-preflight.md) — skip the preflight confirmation on WebFetch so URLs are fetched without prompting.

## Install (paste into Claude Code)

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/README.md and install every config it lists. For each bullet under "Currently includes", fetch the guide at https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/<path> and apply it end-to-end. Merge into ~/.claude/settings.json (never overwrite), don't change my default "model" key, then remind me to restart Claude Code.
```

That's it. The prompt reads this README, walks every bullet under **Currently includes**, fetches the guide, and applies it.

## Requirements

- `jq` on `$PATH`
- `bash` for the status-line script's shebang (your interactive shell can be bash or zsh)
- A Claude Code build that supports the `statusLine` setting and `ANTHROPIC_CUSTOM_MODEL_OPTION*` env vars
