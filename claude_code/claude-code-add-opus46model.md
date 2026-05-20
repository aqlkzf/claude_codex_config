# Add a Custom Model to Claude Code's `/model` Picker

Guideline for Claude Code to follow when the user asks to register an extra entry (e.g. `claude-opus-4-6[1m]`) in the `/model` menu. Claude Code reads this file and then performs the steps below against the user's local `~/.claude/settings.json` — the user is not expected to edit anything by hand.

## When to Use

The user asks to:
- Add Opus 4.6, Opus 4.7, a 1M-context variant, or any non-default model ID to the `/model` picker
- Use a model ID exposed by a proxy/relay (`ANTHROPIC_BASE_URL`) that the official picker doesn't list
- Make a custom model selectable across sessions instead of typing `/model <id>` every time

If the user only wants a one-off switch, point them at `/model <id>` — no config change needed; do not apply this guide.

## What Claude Code Should Do

### 1. Merge the three env vars into `~/.claude/settings.json`

Read the file (create it if missing), parse the JSON, **merge** the keys below into the `"env"` block, write it back. Preserve every other key in `env` (`ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, etc.) and every other top-level key (`permissions`, `model`, `statusLine`, `enabledPlugins`, `theme`, …). All three vars are required as a set — partial config does not render an entry.

```json
{
  "env": {
    "ANTHROPIC_CUSTOM_MODEL_OPTION": "claude-opus-4-6[1m]",
    "ANTHROPIC_CUSTOM_MODEL_OPTION_NAME": "Opus 4.6 (1M context)",
    "ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION": "High-capability model with extended context window"
  }
}
```

| Variable | Purpose |
|---|---|
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | The model ID sent to the API. Must match what your endpoint accepts. |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | Display label in the `/model` picker. |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | Subtitle shown under the label. |

The `[1m]` suffix is a routing hint for 1M-context variants, recognized by Anthropic's API and most relays — keep it if the user wants the extended context window, drop it for the 200k default.

### 2. Restart prompt

The `env` block is read at startup, not on `/clear`. After writing the file, tell the user to fully restart Claude Code; the new entry will appear at the bottom of the `/model` menu.

## Full Example (with proxy + default model preserved)

The merged `settings.json` should look like this — note that `ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, and the top-level `"model"` key are **not** introduced by this guide; they are left untouched if already present.

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://your-proxy:8080",
    "ANTHROPIC_AUTH_TOKEN": "sk-xxx",
    "ANTHROPIC_CUSTOM_MODEL_OPTION": "claude-opus-4-6[1m]",
    "ANTHROPIC_CUSTOM_MODEL_OPTION_NAME": "Opus 4.6 (1M context)",
    "ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION": "High-capability model with extended context window"
  },
  "model": "sonnet[1m]"
}
```

Result:
- Default on launch: `sonnet[1m]` (unchanged)
- `/model` picker: now lists **Opus 4.6 (1M context)** as a selectable entry alongside the built-ins

## Other Ways to Switch Models

| Method | Use it for |
|---|---|
| `/model` menu | Persistent selection within the session (and the only way that surfaces the custom entry) |
| `/model <id>` | One-off switch mid-session — works without any config |
| `claude --model <id>` | Override on launch |
| `"model": "<id>"` in `settings.json` | Make `<id>` the default for every session |

## Verification

After restart:

```bash
# 1. The three vars are present
jq '.env | with_entries(select(.key | startswith("ANTHROPIC_CUSTOM_MODEL_OPTION")))' ~/.claude/settings.json

# 2. Inside Claude Code, run:
#    /model
#    -> the custom entry should appear at the bottom of the list
```

## Troubleshooting

- **Entry doesn't appear**: confirm all three vars are set and Claude Code was fully restarted (not just `/clear`-ed).
- **Entry appears but selecting it errors**: the model ID is wrong for your endpoint. Check what the proxy / Anthropic API actually accepts and update `ANTHROPIC_CUSTOM_MODEL_OPTION`.
- **Want multiple custom models**: only one custom slot exists via env vars. For more, use `/model <id>` per session, or switch the default via the top-level `"model"` key.

## Requirements

- Claude Code build that supports custom-model env vars (recent versions; pre-released builds may not)
- A reachable endpoint (default Anthropic API or `ANTHROPIC_BASE_URL` proxy) that recognizes the chosen model ID
