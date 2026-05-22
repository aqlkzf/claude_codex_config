# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of config snippets for Claude Code. Each file under `claude_code/` states what to set in `~/.claude/settings.json` (and optionally what files to write). The executing Claude Code instance reads a guide and applies it.

## How to apply a guide

Read the guide, then set the config it describes. Merge into `~/.claude/settings.json` — never overwrite. Preserve all existing keys. Don't change the user's `"model"` key unless asked. Tell the user to restart Claude Code afterward.

## Adding a new guide

1. Drop the file into `claude_code/`. State the config to set — nothing more.
2. Add one bullet under **Currently includes** in `README.md`.
3. Push to `main`.

## Repo workflow

- Working tree: `/home/shuai/Datahdd/za/config_claudecodex` and `/workspace/shared/shuai/za/config_claudecodex` are the same directory.
- Remote: `git@github.com:aqlkzf/claude_codex_config.git`, default branch `main`.
- `plans/` is the per-session plans directory. Don't commit it.
