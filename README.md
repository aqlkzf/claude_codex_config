# claude_codex_config

Personal Claude Code / Codex configuration snippets I reuse across machines.

> **TL;DR for everyone else:** open Claude Code anywhere and paste one of the prompts under [Quick install](#quick-install-let-claude-code-do-it) — Claude reads the guide from this repo and applies it to your `~/.claude/` for you.

Currently includes:

- [`claude_code/claude-code-cache-statusline-guide.md`](claude_code/claude-code-cache-statusline-guide.md) — two-line status line showing model, abbreviated CWD, git branch, context-usage bar, cache write/read/hit ratio, session cost, and elapsed time.
- [`claude_code/claude-code-add-opus46model.md`](claude_code/claude-code-add-opus46model.md) — register a custom model entry (e.g. Opus 4.6 with 1M context) in the `/model` picker via three env vars in `settings.json`.

## Quick install (let Claude Code do it)

Open Claude Code in any directory and paste **one** of the prompts below. Claude fetches the guides from this repo and applies them to your local `~/.claude/` — no manual copy/paste needed.

### One-liner (simplest)

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/README.md and install every config it lists.
```

### Install everything (with guardrails)

Same idea as the one-liner, but spells out the safety rules. Use this if the one-liner produces an unsafe merge on your setup.

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/README.md
and apply every guide listed under "Currently includes" to my machine end-to-end.

For each guide:
  1. Fetch the raw URL (https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/<path>).
  2. Follow its "What to Do" section against my local files.
  3. Run its Verification step and show me the output.

Hard rules across all guides:
  - Always MERGE into ~/.claude/settings.json — never overwrite. Preserve every
    existing key (env, permissions, model, enabledPlugins, statusLine, theme,
    extraKnownMarketplaces, etc.).
  - Don't change my default top-level "model" key.
  - Use ~/.claude/<filename> in any settings paths — no hardcoded /home/<me>/ paths.
  - If a guide says it needs /<slash-command> (e.g. /statusline), invoke that first
    and feed the guide's spec to it; fall back to direct execution only if the
    command is unavailable.

At the end, list which guides you applied, what changed in settings.json (keys added,
keys preserved), and remind me to fully restart Claude Code.
```

### Install just one

Prefer the combined prompt above. Use these only if you want a single config in isolation.

<details>
<summary>Status line only</summary>

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/claude_code/claude-code-cache-statusline-guide.md
and follow it end-to-end. Merge into ~/.claude/settings.json without touching other keys;
use ~/.claude/statusline-command.sh in the command path; run the verification snippet;
remind me to restart Claude Code.
```

</details>

<details>
<summary>Custom model in `/model` picker only</summary>

```text
Read https://raw.githubusercontent.com/aqlkzf/claude_codex_config/main/claude_code/claude-code-add-opus46model.md
and follow it end-to-end. Merge the three ANTHROPIC_CUSTOM_MODEL_OPTION* env vars into
~/.claude/settings.json's "env" block without touching other keys; don't change my
default "model" key; run the verification jq snippet; remind me to restart Claude Code.
```

</details>

## Adding a new guide (for future me)

To stay compatible with the "Install everything" prompt above:

1. Drop the new guide into `claude_code/` following the standard layout (`When to Use → What to Do → Verification → Troubleshooting → Requirements`).
2. Add one bullet under **Currently includes** linking to it — that list is what the bootstrap prompt iterates.
3. Push to `main`. The prompt's URL (`raw.githubusercontent.com/.../main/README.md`) picks it up immediately; no prompt edits needed.

## Manual install

If you'd rather do it yourself, open the linked guide directly — each one has the snippet, the `settings.json` block, and a verification section.

## Requirements

- `jq` on `$PATH`
- `bash` (the script's shebang; your interactive shell can be bash or zsh)
- A Claude Code build that supports the `statusLine` setting
