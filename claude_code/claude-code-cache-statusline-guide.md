# Cache Metrics Status Line

Set up a two-line status line showing model, CWD, git branch, context-usage bar, cache cw/cr/hit, session cost, and elapsed time.

## Config

Write `~/.claude/statusline-command.sh` with this content and make it executable (`chmod +x`):

```bash
#!/bin/bash
input=$(cat)

MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')

abbrev_path() {
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

ABBREV_DIR=$(abbrev_path "$DIR")

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

Merge into `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline-command.sh",
    "padding": 0
  }
}
```

Restart Claude Code for the status line to appear.

## Requirements

- `jq` on `$PATH`
- `bash` for the script
