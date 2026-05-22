# Skip WebFetch Preflight

## When to Use

When you want Claude Code to skip the preflight confirmation before fetching URLs with the WebFetch tool, allowing it to fetch web content without prompting each time.

## What to Do

Merge the following key into `~/.claude/settings.json`:

```json
{
  "skipWebFetchPreflight": true
}
```

### Steps

1. Read `~/.claude/settings.json` (create `{}` if it doesn't exist).
2. Merge `"skipWebFetchPreflight": true` into the top-level object.
3. Write the file back, preserving all existing keys.

One-liner with `jq`:

```bash
tmp=$(mktemp) && jq '. + {"skipWebFetchPreflight": true}' ~/.claude/settings.json > "$tmp" && mv "$tmp" ~/.claude/settings.json
```

## Verification

```bash
jq '.skipWebFetchPreflight' ~/.claude/settings.json
# Expected: true
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Still getting preflight prompts | Restart Claude Code — settings are read at startup |
| `settings.json` doesn't exist | Create it with `echo '{}' > ~/.claude/settings.json` first |

## Requirements

- `jq` on `$PATH`
- Claude Code build that supports the `skipWebFetchPreflight` setting
