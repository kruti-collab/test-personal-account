---
description: Pick the single highest priority work item and suggest exact SDLC command
argument-hint: "[--claim] [--team] [--assigned-only] [--all]"
allowed-tools: Bash, Read, Write
---

# Work Prioritizer - Single Action Recommendation

**Purpose**: Analyze all GitHub issues/PRs, pick the ONE highest priority item, and suggest the exact SDLC slash command to run next.

## Collision Prevention

**How this command prevents team collisions:**

1. **Default: Available items only** - Only recommends unassigned items OR items already assigned to you. Items assigned to other team members are skipped.

2. **`--claim` flag** - Automatically assigns the recommended item to you via GitHub, signaling to the team that you're working on it.

3. **GitHub as source of truth** - All coordination happens through GitHub assignments, visible to entire team (not just Claude sessions).

## Usage

```bash
# Get highest priority AVAILABLE work (default - skips items assigned to others)
/work-prioritizer

# Get available work AND claim it (auto-assign to yourself)
/work-prioritizer --claim

# Get only YOUR currently assigned work
/work-prioritizer --assigned-only

# Show ALL items including those assigned to others (original behavior)
/work-prioritizer --all

# Show team workload distribution
/work-prioritizer --team
```

## Implementation

### Step 1: Parse Arguments and Get User

```bash
ASSIGNED_ONLY=false
TEAM_VIEW=false
CLAIM_WORK=false
SHOW_ALL=false

for arg in $ARGUMENTS; do
  case "$arg" in
    --assigned-only) ASSIGNED_ONLY=true ;;
    --team) TEAM_VIEW=true ;;
    --claim) CLAIM_WORK=true ;;
    --all) SHOW_ALL=true ;;
  esac
done

CURRENT_USER=$(gh api user --jq '.login')

echo "🎯 GAL Work Prioritizer"
echo "═══════════════════════════════════════════════════════════"
if [ "$SHOW_ALL" = false ]; then
  echo "📋 Mode: Available work only (use --all to see everything)"
else
  echo "📋 Mode: Showing ALL items (including assigned to others)"
fi
echo ""
```

### Step 2: Helper Function - Filter Available Items

```bash
# Filter function: keep only unassigned OR assigned to current user
filter_available() {
  local json_input="$1"
  if [ "$SHOW_ALL" = true ]; then
    echo "$json_input"
  else
    echo "$json_input" | jq --arg user "$CURRENT_USER" '
      [.[] | select(
        (.assignees | length == 0) or
        (.assignees[]? | .login == $user)
      )]'
  fi
}
```

### Step 3: Gather and Filter GitHub Data

```bash
# Infrastructure blockers
RAW_BLOCKERS=$(gh issue list --state open \
  --label "Infrastructure" \
  --json number,title,labels,assignees 2>/dev/null || echo "[]")
BLOCKERS=$(filter_available "$RAW_BLOCKERS")

# Quick wins (auto-approve eligible)
RAW_QUICK_WINS=$(gh pr list --state open \
  --search "label:auto-approve-eligible" \
  --json number,title,labels,assignees,headRefName 2>/dev/null || echo "[]")
QUICK_WINS=$(filter_available "$RAW_QUICK_WINS")

# Blocked work
RAW_BLOCKED=$(gh pr list --state open \
  --search "label:sdlc:5-review-blocked OR label:sdlc:6-verify-blocked" \
  --json number,title,labels,assignees,headRefName 2>/dev/null || echo "[]")
BLOCKED_WORK=$(filter_available "$RAW_BLOCKED")

# High-risk PRs
RAW_HIGH_RISK=$(gh pr list --state open \
  --search "label:risk:high" \
  --json number,title,labels,assignees,headRefName 2>/dev/null || echo "[]")
HIGH_RISK=$(filter_available "$RAW_HIGH_RISK")

# Production bugs
RAW_BUGS=$(gh issue list --state open --label "bug" \
  --json number,title,labels,assignees --limit 10 2>/dev/null || echo "[]")
PROD_BUGS=$(filter_available "$RAW_BUGS")
```

### Step 4: Display Categories

```bash
echo "🚨 INFRASTRUCTURE BLOCKERS"
echo "──────────────────────────────────────────────────────────"
if [ "$BLOCKERS" != "[]" ] && [ -n "$BLOCKERS" ]; then
  echo "$BLOCKERS" | jq -r '.[] |
    "🔴 #\(.number): \(.title)" +
    (if .assignees | length > 0 then " [@\(.assignees[0].login)]" else " [AVAILABLE]" end)'
else
  echo "✅ No available infrastructure blockers"
fi
echo ""

echo "⚡ QUICK WINS (Auto-Approve Eligible)"
echo "──────────────────────────────────────────────────────────"
if [ "$QUICK_WINS" != "[]" ] && [ -n "$QUICK_WINS" ]; then
  echo "$QUICK_WINS" | jq -r '.[] |
    "🟢 PR #\(.number): \(.title)" +
    (if .assignees | length > 0 then " [@\(.assignees[0].login)]" else " [AVAILABLE]" end)'
else
  echo "ℹ️ No available quick wins"
fi
echo ""

echo "⏸️ BLOCKED SDLC WORK"
echo "──────────────────────────────────────────────────────────"
if [ "$BLOCKED_WORK" != "[]" ] && [ -n "$BLOCKED_WORK" ]; then
  echo "$BLOCKED_WORK" | jq -r '.[] |
    "🟡 PR #\(.number): \(.title)" +
    (if .assignees | length > 0 then " [@\(.assignees[0].login)]" else " [AVAILABLE]" end)'
else
  echo "✅ No blocked work"
fi
echo ""

echo "🔶 HIGH-RISK PRs"
echo "──────────────────────────────────────────────────────────"
if [ "$HIGH_RISK" != "[]" ] && [ -n "$HIGH_RISK" ]; then
  echo "$HIGH_RISK" | jq -r '.[] |
    "🔶 PR #\(.number): \(.title)" +
    (if .assignees | length > 0 then " [@\(.assignees[0].login)]" else " [AVAILABLE]" end)'
else
  echo "✅ No available high-risk PRs"
fi
echo ""

echo "🐛 PRODUCTION BUGS"
echo "──────────────────────────────────────────────────────────"
if [ "$PROD_BUGS" != "[]" ] && [ -n "$PROD_BUGS" ]; then
  echo "$PROD_BUGS" | jq -r '.[] |
    "🔴 #\(.number): \(.title)" +
    (if .assignees | length > 0 then " [@\(.assignees[0].login)]" else " [AVAILABLE]" end)'
else
  echo "✅ No available production bugs"
fi
echo ""
```

### Step 5: Team View (if --team)

```bash
if [ "$TEAM_VIEW" = true ]; then
  echo "👥 TEAM WORKLOAD"
  echo "──────────────────────────────────────────────────────────"

  TEAM_DATA=$(gh issue list --state open --json assignees --limit 100 | jq -r '
    [.[] | .assignees[]? | .login] | group_by(.) | map({user: .[0], count: length}) | sort_by(-.count) | .[]')

  if [ -n "$TEAM_DATA" ]; then
    echo "$TEAM_DATA" | jq -rs '
      .[] | "👤 @\(.user): \(.count) items" +
      (if .user == "'"$CURRENT_USER"'" then " (you)" else "" end)'
  else
    echo "ℹ️ No assigned work found"
  fi
  echo ""
fi
```

### Step 6: Pick Single Next Action

```bash
echo "🎯 NEXT ACTION RECOMMENDATION"
echo "═══════════════════════════════════════════════════════════"

NEXT_ACTION=""
NEXT_COMMAND=""
NEXT_REASON=""
ITEM_TYPE=""  # "issue" or "pr"
ITEM_NUM=""

# P0: Infrastructure blockers
if [ "$BLOCKERS" != "[]" ] && [ -n "$BLOCKERS" ] && [ "$(echo "$BLOCKERS" | jq 'length')" -gt 0 ]; then
  PICKED=$(echo "$BLOCKERS" | jq -r '.[0] | "\(.number):\(.title)"')
  ITEM_NUM=$(echo "$PICKED" | cut -d':' -f1)
  ITEM_TITLE=$(echo "$PICKED" | cut -d':' -f2-)
  NEXT_ACTION="🔴 Fix Infrastructure Blocker #$ITEM_NUM: $ITEM_TITLE"
  NEXT_COMMAND="/sdlc:4-implement:run $ITEM_NUM"
  NEXT_REASON="Infrastructure blockers prevent ALL other work from proceeding"
  ITEM_TYPE="issue"

# P1: Quick wins
elif [ "$QUICK_WINS" != "[]" ] && [ -n "$QUICK_WINS" ] && [ "$(echo "$QUICK_WINS" | jq 'length')" -gt 0 ]; then
  PICKED=$(echo "$QUICK_WINS" | jq -r '.[0] | "\(.number):\(.title)"')
  ITEM_NUM=$(echo "$PICKED" | cut -d':' -f1)
  ITEM_TITLE=$(echo "$PICKED" | cut -d':' -f2-)
  NEXT_ACTION="🟢 Deploy Quick Win PR #$ITEM_NUM: $ITEM_TITLE"
  NEXT_COMMAND="/sdlc:7-deploy:run $ITEM_NUM"
  NEXT_REASON="Quick wins provide immediate velocity"
  ITEM_TYPE="pr"

# P2: High-risk PRs (not blocked)
elif [ "$HIGH_RISK" != "[]" ] && [ -n "$HIGH_RISK" ]; then
  AVAILABLE=$(echo "$HIGH_RISK" | jq '[.[] | select(.labels[]? | select(.name | contains("blocked")) | not)]')
  if [ "$AVAILABLE" != "[]" ] && [ "$(echo "$AVAILABLE" | jq 'length')" -gt 0 ]; then
    PICKED=$(echo "$AVAILABLE" | jq -r '.[0] | "\(.number):\(.title)"')
    ITEM_NUM=$(echo "$PICKED" | cut -d':' -f1)
    ITEM_TITLE=$(echo "$PICKED" | cut -d':' -f2-)
    NEXT_ACTION="🔶 Review High-Risk PR #$ITEM_NUM: $ITEM_TITLE"
    NEXT_COMMAND="/sdlc:5-review:run $ITEM_NUM"
    NEXT_REASON="High-risk PRs need review before they become blockers"
    ITEM_TYPE="pr"
  fi

# P3: Production bugs
elif [ "$PROD_BUGS" != "[]" ] && [ -n "$PROD_BUGS" ] && [ "$(echo "$PROD_BUGS" | jq 'length')" -gt 0 ]; then
  PICKED=$(echo "$PROD_BUGS" | jq -r '.[0] | "\(.number):\(.title)"')
  ITEM_NUM=$(echo "$PICKED" | cut -d':' -f1)
  ITEM_TITLE=$(echo "$PICKED" | cut -d':' -f2-)
  NEXT_ACTION="🐛 Fix Production Bug #$ITEM_NUM: $ITEM_TITLE"
  NEXT_COMMAND="/sdlc:bug-fix $ITEM_NUM"
  NEXT_REASON="Production bugs directly impact customers"
  ITEM_TYPE="issue"
fi

# Fallback
if [ -z "$NEXT_ACTION" ]; then
  NEXT_ACTION="🆕 No available work - all items assigned to others"
  NEXT_COMMAND="/work-prioritizer --all  # See all items"
  NEXT_REASON="Use --all to see items assigned to other team members"
fi

echo "➤ $NEXT_ACTION"
echo ""
echo "📋 COMMAND TO RUN:"
echo "   $NEXT_COMMAND"
echo ""
echo "💡 WHY: $NEXT_REASON"
echo ""
```

### Step 7: Auto-Claim (if --claim)

```bash
if [ "$CLAIM_WORK" = true ] && [ -n "$ITEM_NUM" ] && [ -n "$ITEM_TYPE" ]; then
  echo "🔒 CLAIMING WORK..."
  echo "──────────────────────────────────────────────────────────"

  if [ "$ITEM_TYPE" = "issue" ]; then
    gh issue edit "$ITEM_NUM" --add-assignee "@me" 2>/dev/null
    CLAIM_RESULT=$?
  else
    gh pr edit "$ITEM_NUM" --add-assignee "@me" 2>/dev/null
    CLAIM_RESULT=$?
  fi

  if [ $CLAIM_RESULT -eq 0 ]; then
    echo "✅ Assigned #$ITEM_NUM to @$CURRENT_USER"
    echo "   Other team members will not see this item in their recommendations"
  else
    echo "⚠️ Could not auto-assign. Please run:"
    echo "   gh $ITEM_TYPE edit $ITEM_NUM --add-assignee @me"
  fi
  echo ""
fi
```

### Step 8: Footer

```bash
echo "═══════════════════════════════════════════════════════════"
if [ "$CLAIM_WORK" = false ] && [ -n "$ITEM_NUM" ]; then
  echo "💡 TIP: Run with --claim to auto-assign this item to yourself"
fi
echo "📖 Collision prevention: Items assigned to others are hidden by default"
echo ""
```

## Priority Logic

| Priority | Criteria | Command |
|----------|----------|---------|
| **P0** | Infrastructure blockers | `/sdlc:4-implement:run` |
| **P1** | Auto-approve eligible PRs | `/sdlc:7-deploy:run` |
| **P2** | High-risk PRs (not blocked) | `/sdlc:5-review:run` |
| **P3** | Production bugs | `/sdlc:bug-fix` |
| **P4** | No available work | Suggest `--all` |

## Collision Prevention Summary

| Mechanism | How It Works |
|-----------|--------------|
| **Default filtering** | Only shows unassigned items or items assigned to you |
| **--claim flag** | Auto-assigns via `gh issue/pr edit --add-assignee @me` |
| **GitHub source of truth** | All team members see same assignment state |
| **--all escape hatch** | View everything when needed for triage |

This ensures multiple team members or Claude sessions won't pick the same work item.
