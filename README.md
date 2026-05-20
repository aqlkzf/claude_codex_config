# claude_codex_config

Personal Claude Code / Codex configuration snippets I reuse across machines.

Currently includes:

- [`claude_code/claude-code-cache-statusline-guide.md`](claude_code/claude-code-cache-statusline-guide.md) — two-line status line showing model, abbreviated CWD, git branch, context-usage bar, cache write/read/hit ratio, session cost, and elapsed time.

## Quick install (let Claude Code do it)

Open Claude Code in any directory and paste the prompt below. Claude will read the guide from this repo and apply it to your local `~/.claude/` — no manual copy/paste of the script needed.

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/claude_code/claude-code-cache-statusline-guide.md
and follow it end-to-end to set up the cache-metrics status line on this machine:

1. Write the script verbatim to ~/.claude/statusline-command.sh (create the directory if missing).
2. chmod +x it.
3. Merge the statusLine block into ~/.claude/settings.json — preserve every other key
   (env, permissions, model, enabledPlugins, theme, etc.). If settings.json doesn't
   exist, create it with just the statusLine block.
4. Run the verification snippet from the guide and show me the rendered output.
5. Remind me to restart Claude Code for the status line to take effect.

Use ~/.claude/statusline-command.sh in the settings command (not a hardcoded /home/<me>/ path).
Don't touch any other settings.
```

## Manual install

If you'd rather do it yourself, follow [the guide](claude_code/claude-code-cache-statusline-guide.md) directly — it has the script, the `settings.json` snippet, and a verification section.

## Requirements

- `jq` on `$PATH`
- `bash` (the script's shebang; your interactive shell can be bash or zsh)
- A Claude Code build that supports the `statusLine` setting
