# Add Opus 4.6 (1M) to `/model` Picker

Merge into `~/.claude/settings.json` under the `"env"` key:

```json
{
  "env": {
    "ANTHROPIC_CUSTOM_MODEL_OPTION": "claude-opus-4-6[1m]",
    "ANTHROPIC_CUSTOM_MODEL_OPTION_NAME": "Opus 4.6 (1M context)",
    "ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION": "High-capability model with extended context window"
  }
}
```

Restart Claude Code. The entry appears at the bottom of `/model`.

## Requirements

- All three vars must be set together
- Claude Code build that supports custom-model env vars
