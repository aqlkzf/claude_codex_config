# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **collection of guideline documents for Claude Code itself**, not an application. Each file under `claude_code/` is a self-contained playbook that another Claude Code instance is expected to read and execute against the user's local `~/.claude/` setup.

There is no build, no test suite, no runtime. The "code" is the markdown guides; the "deployment target" is whichever machine the user is running Claude Code on.

## How to operate in this repo

When the user references one of the guides (or asks for the behavior it describes), **follow the guide end-to-end against their machine** — don't just summarize it, don't just copy/paste the snippets back at them. Each guide is written so that you (Claude Code) are the executor.

Currently:

- `claude_code/claude-code-cache-statusline-guide.md` — installs `~/.claude/statusline-command.sh` and merges a `statusLine` block into `~/.claude/settings.json` so the user sees a two-line status line with cache-hit metrics.
- `claude_code/claude-code-add-opus46model.md` — merges three `ANTHROPIC_CUSTOM_MODEL_OPTION*` env vars into `~/.claude/settings.json` so a custom model entry (e.g. Opus 4.6 with 1M context) appears in the `/model` picker.

When the user says "add Opus 4.6 to my model picker" or "set up the cache status line", read the relevant guide first, then apply it. The `README.md` contains the canonical paste-and-go prompts that wrap each guide.

## Hard rules when applying a guide

- **Always merge into `~/.claude/settings.json`** — never overwrite. Preserve every existing key (`env`, `permissions`, `model`, `enabledPlugins`, `statusLine`, `theme`, `extraKnownMarketplaces`, etc.). Read the file, parse it, merge, write back.
- **Never hardcode the user's home path** in `settings.json` snippets. Use `~/.claude/...`. The reference user's `/home/shuai/...` paths in examples are illustrative only.
- **Don't change the user's default `"model"` key** unless they explicitly ask.
- After applying, run the guide's verification snippet and **tell the user to restart Claude Code** — `env` and `statusLine` are read at startup, not on `/clear`.

## When editing the guides themselves

- Each guide is a *standalone* doc — a fresh Claude Code reading just that one file via `WebFetch` must be able to execute it. Don't introduce cross-guide dependencies or assume shared context.
- Keep the structure: **When to Use → What to Do → Verification → Troubleshooting → Requirements**. The README's install prompt assumes this layout.

## Adding a new guide

The README is consumed by another Claude Code instance at install time — the install prompt fetches `README.md`, iterates every bullet under **Currently includes**, and applies each guide. To stay compatible with that contract:

1. Drop the new guide into `claude_code/` following the standard 5-section layout above.
2. Add **one** bullet under **Currently includes** in `README.md` linking to it (relative path, not raw URL — the install prompt knows the base).
3. Don't add prose, alternate prompts, or extra sections to the README. Anything that changes how Claude *behaves* during install belongs in this file (CLAUDE.md), not the README. The README is a manifest the AI iterates; this file is the operating manual.
4. Push to `main`. The install URL (`raw.githubusercontent.com/.../main/README.md`) picks it up immediately.

## Repo workflow

- Working tree: `/home/shuai/Datahdd/za/config_claudecodex` and `/workspace/shared/shuai/za/config_claudecodex` are the same directory (one mounts the other). Edit either; git operations only need to run once.
- Remote: `git@github.com:aqlkzf/claude_codex_config.git`, default branch `main`. The README's quick-install prompts fetch from `raw.githubusercontent.com/aqlkzf/claude_codex_config/main/...` — pushing to `main` is what makes a guide change live for users.
- `plans/` is the per-session plans directory (configured by the user's global `plansDirectory: ./plans`). Don't commit anything inside it.
