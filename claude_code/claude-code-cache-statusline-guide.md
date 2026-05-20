# Cache Metrics Status Line — Setup Guide for Claude Code

Instructions for Claude Code to configure a cache-monitoring status line (cw/cr/hit) that matches the reference user's setup exactly.

## When to Use

User asks to monitor cache performance, add cache metrics to their status line, mentions wanting to see cw/cr/hit, or says "set up the same status line as mine / as the reference".

## What to Do

### 1. Write the script to `~/.claude/statusline-command.sh`

Write this file verbatim — comments included. This is the exact script the reference user runs.

```bash
#!/bin/bash
input=$(cat)

MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')

# Path abbreviation: shorten all parent dirs to their first letter
abbrev_path() {
  local path="$1"
  local home="$HOME"
  # Replace $HOME prefix with ~
  if [[ "$path" == "$home"* ]]; then
    path="~${path#$home}"
  fi
  # Split into parts, abbreviate all but the last
  local IFS='/'
  read -ra parts <<< "$path"
  local result=""
  local count=${#parts[@]}
  for ((i=0; i<count-1; i++)); do
    local part="${parts[$i]}"
    if [ -z "$part" ]; then
      result+="/"
    elif [ "$part" = "~" ]; then
      result+="~/"
    else
      result+="${part:0:1}/"
    fi
  done
  result+="${parts[$((count-1))]}"
  echo "$result"
}

ABBREV_DIR=$(abbrev_path "$DIR")

COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
CACHE_READ=$(echo "$input" | jq -r '.context_window.current_usage.cache_read_input_tokens // 0')
CACHE_WRITE=$(echo "$input" | jq -r '.context_window.current_usage.cache_creation_input_tokens // 0')
DURATION_MS=$(echo "$input" | jq -r '.cost.total_duration_ms // 0')

CYAN='\033[36m'; GREEN='\033[32m'; YELLOW='\033[33m'; RED='\033[31m'; MAGENTA='\033[35m'; RESET='\033[0m'

# Pick bar color based on context usage
if [ "$PCT" -ge 90 ]; then BAR_COLOR="$RED"
elif [ "$PCT" -ge 70 ]; then BAR_COLOR="$YELLOW"
else BAR_COLOR="$GREEN"; fi

FILLED=$((PCT / 10)); EMPTY=$((10 - FILLED))
BAR=""
for ((i=0; i<FILLED; i++)); do BAR+=$'\xe2\x96\x88'; done
for ((i=0; i<EMPTY; i++)); do BAR+=$'\xe2\x96\x91'; done

MINS=$((DURATION_MS / 60000)); SECS=$(((DURATION_MS % 60000) / 1000))

# Format token counts (e.g., 120000 -> 120k, 1500 -> 1.5k)
fmt_tokens() {
  local t="$1"
  if [ "$t" -ge 1000 ]; then
    awk "BEGIN { v=$t/1000; if (v >= 10) printf \"%.0fk\", v; else printf \"%.1fk\", v }"
  else
    echo "$t"
  fi
}

CW_FMT=$(fmt_tokens "$CACHE_WRITE")
CR_FMT=$(fmt_tokens "$CACHE_READ")

# Calculate cache hit ratio: cr / (cr + cw)
CACHE_HIT_PCT=""
CACHE_TOTAL=$((CACHE_READ + CACHE_WRITE))
if [ "$CACHE_TOTAL" -gt 0 ]; then
  CACHE_HIT_PCT=$(awk "BEGIN { printf \"%.0f\", ($CACHE_READ / $CACHE_TOTAL) * 100 }")
fi

BRANCH=""
git rev-parse --git-dir > /dev/null 2>&1 && BRANCH=" | 🌿 $(git branch --show-current 2>/dev/null)"

echo -e "${CYAN}[$MODEL]${RESET} 📁 ${ABBREV_DIR}$BRANCH"
COST_FMT=$(printf '$%.2f' "$COST")
# Line 2: bar, usage%, cache write/read/hit, cost, time
CACHE_INFO="${YELLOW}cw:${CW_FMT}${RESET} ${GREEN}cr:${CR_FMT}${RESET}"
HIT_INFO=""
if [ -n "$CACHE_HIT_PCT" ]; then
  if [ "$CACHE_HIT_PCT" -lt 50 ]; then
    HIT_INFO=" ${RED}hit:${CACHE_HIT_PCT}%${RESET}"
  else
    HIT_INFO=" ${MAGENTA}hit:${CACHE_HIT_PCT}%${RESET}"
  fi
fi
echo -e "${BAR_COLOR}${BAR}${RESET} ${PCT}% | ${CACHE_INFO}${HIT_INFO} | ${YELLOW}${COST_FMT}${RESET} | ⏱️ ${MINS}m ${SECS}s"
```

### 2. Make it executable

```bash
chmod +x ~/.claude/statusline-command.sh
```

### 3. Merge `statusLine` into `~/.claude/settings.json`

Merge this block into the user's existing settings (preserve all other keys — do not overwrite `env`, `permissions`, `model`, `enabledPlugins`, etc.):

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash /home/shuai/.claude/statusline-command.sh",
    "padding": 0
  }
}
```

Notes:
- Replace `/home/shuai/` with the target user's actual `$HOME` path (or use `~/.claude/statusline-command.sh` — Claude Code expands `~`). The reference setup uses the explicit absolute path with a leading `bash` invocation; either form works as long as the script is executable.
- `padding: 0` removes the blank line above the status line so it sits flush against the prompt.

### 4. Tell the user to restart Claude Code

The status line loads on startup. Changes take effect after restarting the CLI session.

## Metric Definitions

| Metric | Source field | Meaning | Color |
|--------|-------------|---------|-------|
| **cw** | `context_window.current_usage.cache_creation_input_tokens` | Tokens written to cache (costs 1.25x input price) | Yellow |
| **cr** | `context_window.current_usage.cache_read_input_tokens` | Tokens read from cache (costs 0.1x input price) | Green |
| **hit** | `cr / (cr + cw) * 100` | Cache hit ratio | Magenta (normal), Red (<50%) |

## JSON Input Schema (stdin to script)

```json
{
  "model": { "display_name": "Sonnet 4.6 (1M)" },
  "workspace": { "current_dir": "/home/user/project" },
  "cost": { "total_cost_usd": 1.23, "total_duration_ms": 135000 },
  "context_window": {
    "used_percentage": 42.5,
    "current_usage": {
      "input_tokens": 50000,
      "output_tokens": 3000,
      "cache_read_input_tokens": 120000,
      "cache_creation_input_tokens": 45000
    }
  }
}
```

## Output Example

```
[Sonnet 4.6 (1M)] 📁 ~/P/myproject | 🌿 main
████████░░ 80% | cw:45k cr:120k hit:73% | $1.23 | ⏱️ 2m 15s
```

Line 1: model · abbreviated CWD (parent dirs collapsed to first letter) · git branch (if inside a repo).
Line 2: context-usage bar · usage% · cache write/read/hit · session cost · elapsed time.

## Diagnostic Reference

If the user asks what the numbers mean:

- **cw high, cr low, hit <50%**: Cache is not being reused. Likely waited >5min (TTL expired) or prompt structure changed significantly between turns.
- **cw low, cr high, hit >80%**: Healthy — most of the context prefix is served from cache.
- **Both 0**: First turn of session, no cache data yet.
- **hit drops after compaction**: Expected — context compression rewrites the conversation prefix.

## Verification

After restart, the status line should render two lines under the prompt. Quick sanity checks:

```bash
# 1. Script is executable and runs without error on sample input
echo '{"model":{"display_name":"test"},"workspace":{"current_dir":"/tmp"},"cost":{"total_cost_usd":0,"total_duration_ms":0},"context_window":{"used_percentage":0,"current_usage":{"cache_read_input_tokens":0,"cache_creation_input_tokens":0}}}' \
  | ~/.claude/statusline-command.sh

# 2. statusLine block is present in settings
jq '.statusLine' ~/.claude/settings.json
```

If the second line renders without colors, the terminal isn't interpreting `\033[` escapes — confirm `echo -e` is the bash builtin (the `#!/bin/bash` shebang ensures this).

## Requirements

- `jq` installed on the user's system
- `bash` or `zsh` to run the script (the `#!/bin/bash` shebang is what executes — your interactive shell doesn't matter). Uses `read -ra`, `[[ ]]`, and `(( ))`, all supported by both.
- Claude Code version with `statusLine` support in `settings.json`
