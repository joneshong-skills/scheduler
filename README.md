[English](README.md) | [繁體中文](README.zh.md)

# scheduler

macOS launchd-based task scheduler with natural language configuration.

## Description

Scheduler translates natural language scheduling requests into macOS launchd jobs, managed via a centralized Python registry — supporting one-time, periodic, and cron-style schedules.

## Features

- Natural language scheduling ('every day at 9am', 'in 30 minutes')
- macOS launchd integration — no cron required
- Centralized job registry with list, pause, and delete operations
- Supports periodic intervals, exact times, and cron expressions
- Jobs persist across reboots via launchd plist management
- Human-readable job status and next-run time display

## Usage

Invoke by asking Claude Code with trigger phrases such as:

- "schedule a task"
- "set up a cron job"
- "run something periodically"
- "排程"
- "定時任務"
- "每天幾點執行"

## Related Skills

- [`macos-ui-automation`](https://github.com/joneshong-skills/macos-ui-automation)
- [`executor`](https://github.com/joneshong-skills/executor)

## Install

Copy the skill directory into your Claude Code skills folder:

```
cp -r scheduler ~/.claude/skills/
```

Skills placed in `~/.claude/skills/` are auto-discovered by Claude Code. No additional registration is needed.
