# Cache Metrics Status Line — Setup Guide for Claude Code

Instructions for Claude Code to configure cache monitoring (cw/cr/hit) on a user's status line.

## When to Use

User asks to monitor cache performance, add cache metrics to their status line, or mentions wanting to see cw/cr/hit.

## What to Do

### 1. Write the script to `~/.claude/statusline-command.sh`

```bash
#!/bin/bash
input=$(cat)

MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')

fish_path() {
  local path="$1"
  local home="$HOME"
  if [[ "$path" == "$home"* ]]; then
    path="~${path#$home}"
  fi
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

ABBREV_DIR=$(fish_path "$DIR")

COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
CACHE_READ=$(echo "$input" | jq -r '.context_window.current_usage.cache_read_input_tokens // 0')
CACHE_WRITE=$(echo "$input" | jq -r '.context_window.current_usage.cache_creation_input_tokens // 0')
DURATION_MS=$(echo "$input" | jq -r '.cost.total_duration_ms // 0')

CYAN='\033[36m'; GREEN='\033[32m'; YELLOW='\033[33m'; RED='\033[31m'; MAGENTA='\033[35m'; RESET='\033[0m'

if [ "$PCT" -ge 90 ]; then BAR_COLOR="$RED"
elif [ "$PCT" -ge 70 ]; then BAR_COLOR="$YELLOW"
else BAR_COLOR="$GREEN"; fi

FILLED=$((PCT / 10)); EMPTY=$((10 - FILLED))
BAR=""
for ((i=0; i<FILLED; i++)); do BAR+=$'\xe2\x96\x88'; done
for ((i=0; i<EMPTY; i++)); do BAR+=$'\xe2\x96\x91'; done

MINS=$((DURATION_MS / 60000)); SECS=$(((DURATION_MS % 60000) / 1000))

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

CACHE_HIT_PCT=""
CACHE_TOTAL=$((CACHE_READ + CACHE_WRITE))
if [ "$CACHE_TOTAL" -gt 0 ]; then
  CACHE_HIT_PCT=$(awk "BEGIN { printf \"%.0f\", ($CACHE_READ / $CACHE_TOTAL) * 100 }")
fi

BRANCH=""
git rev-parse --git-dir > /dev/null 2>&1 && BRANCH=" | 🌿 $(git branch --show-current 2>/dev/null)"

echo -e "${CYAN}[$MODEL]${RESET} 📁 ${ABBREV_DIR}$BRANCH"
COST_FMT=$(printf '$%.2f' "$COST")

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

### 3. Add `statusLine` to `~/.claude/settings.json`

Merge this block into the user's existing settings (do not overwrite other keys):

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline-command.sh",
    "padding": 0
  }
}
```

The default interpreter is bash — no need to prefix `bash` in the command field. The `#!/bin/bash` shebang in the script is sufficient.

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

## Diagnostic Reference

If the user asks what the numbers mean:

- **cw high, cr low, hit <50%**: Cache is not being reused. Likely waited >5min (TTL expired) or prompt structure changed significantly between turns.
- **cw low, cr high, hit >80%**: Healthy — most of the context prefix is served from cache.
- **Both 0**: First turn of session, no cache data yet.
- **hit drops after compaction**: Expected — context compression rewrites the conversation prefix.

## Requirements

- `jq` must be installed on the user's system
- `bash` (script uses bashisms: `read -ra`, `[[ ]]`, arithmetic)
- Claude Code version with `statusLine` support in settings.json
