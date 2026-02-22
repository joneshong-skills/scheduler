---
name: scheduler
description: >-
  This skill should be used when the user asks to "schedule a task",
  "set up a cron job", "run something periodically", "remind me every",
  "create a scheduled job", "排程", "定時任務", "每天幾點執行",
  "幾分鐘後執行", "定期跑", mentions scheduling, timers, periodic tasks,
  or discusses automating recurring operations on macOS.
version: 0.2.0
tools: Bash, Read, Write, sandbox_execute
---

# Scheduler

macOS launchd-based task scheduler with centralized registry. Translate natural language
scheduling requests into launchd jobs, managed via a single Python script.

## Agent Delegation

Delegate cron job setup to `worker` agent.

## Core Concepts

- **Backend**: macOS `launchd` (plist files in `~/Library/LaunchAgents/`)
- **Registry**: All jobs tracked in `~/Claude/skills/scheduler/registry.json`
- **Logs**: stdout/stderr captured in `~/Claude/skills/scheduler/logs/`
- **Label prefix**: `com.joneshong.scheduler.<name>`
- **Shell**: Commands run via `/bin/zsh -lc` (inherits full login environment)

## Script Path

```bash
SC="python3 ~/.claude/skills/scheduler/scripts/scheduler.py"
```

## Workflow

### Step 1: Parse the User's Request

Extract from natural language:
- **What** to run (command/script)
- **When** to run it (interval, specific time, day of week)
- **Name** for the job (kebab-case, descriptive)
- **Description** (brief purpose)

### Step 2: Build the Schedule

Map the request to a schedule dict:

| User says | Schedule dict |
|-----------|--------------|
| "every 5 minutes" | `{"interval": 300}` |
| "every hour" | `{"interval": 3600}` |
| "daily at 9:30" | `{"calendar": {"Hour": 9, "Minute": 30}}` |
| "every Monday at 10:00" | `{"calendar": {"Weekday": 1, "Hour": 10}}` |
| "every day at midnight" | `{"calendar": {"Hour": 0, "Minute": 0}}` |
| "1st of every month at 8:00" | `{"calendar": {"Day": 1, "Hour": 8, "Minute": 0}}` |
| "run now and then every 10 min" | `{"interval": 600, "run_at_load": true}` |

**Calendar keys** (all optional, omit = wildcard):
- `Hour` (0-23), `Minute` (0-59), `Weekday` (0=Sun, 1=Mon...6=Sat), `Day` (1-31), `Month` (1-12)

### Step 3: Execute

```bash
# Add a job
$SC add <name> "<command>" '<schedule_json>' "description"

# List all jobs
$SC list

# View logs
$SC logs <name> [lines]

# Disable (keep in registry, unload from launchd)
$SC disable <name>

# Re-enable
$SC enable <name>

# Remove completely
$SC remove <name>
```

### Step 4: Confirm to User

After adding a job, report:
1. Job name and label
2. Schedule in human-readable form
3. Whether launchd loaded successfully
4. Log file locations

## Quick Reference

### Common Patterns

```bash
# Backup script every 6 hours
$SC add backup-docs "tar czf ~/backups/docs-\$(date +%Y%m%d).tar.gz ~/Documents" \
  '{"interval": 21600}' "Backup Documents folder"

# Daily git pull at 08:00
$SC add daily-pull "cd ~/project && git pull" \
  '{"calendar": {"Hour": 8, "Minute": 0}}' "Morning git pull"

# Health check every 2 minutes, start immediately
$SC add health-check "curl -s https://example.com/health" \
  '{"interval": 120, "run_at_load": true}' "API health monitor"

# Weekly Monday 09:00 report
$SC add weekly-report "python3 ~/scripts/report.py" \
  '{"calendar": {"Weekday": 1, "Hour": 9, "Minute": 0}}' "Weekly status report"
```

### Weekday Numbers

| 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| Sun | Mon | Tue | Wed | Thu | Fri | Sat |

### Troubleshooting

- **Job not running?** Check `$SC logs <name>` for errors
- **Permission denied?** Ensure the command script is executable (`chmod +x`)
- **launchctl load failed?** Verify plist syntax: `plutil ~/Library/LaunchAgents/com.joneshong.scheduler.<name>.plist`
- **List launchd status**: `launchctl list | grep com.joneshong.scheduler`

## Sandbox Optimization

Batch operations benefit from `sandbox_execute`:

- **Batch schedule queries**: Import `scripts/scheduler.py` in sandbox to list/query multiple jobs at once, returning structured JSON instead of CLI output
- **Registry analysis**: Read and aggregate `registry.json` data in sandbox for status dashboards

Principle: **Deterministic batch work → sandbox; reasoning/presentation → LLM.**

## Additional Resources

### Scripts
- **`scripts/scheduler.py`** — Core scheduler: add/remove/enable/disable/list/logs commands
