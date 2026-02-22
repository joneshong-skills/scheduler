[English](README.md) | [繁體中文](README.zh.md)

# scheduler

macOS launchd-based task scheduler with natural language configuration.

## 說明

Scheduler translates natural language scheduling requests into macOS launchd jobs, managed via a centralized Python registry — supporting one-time, periodic, and cron-style schedules.

## 功能特色

- Natural language scheduling ('every day at 9am', 'in 30 minutes')
- macOS launchd integration — no cron required
- Centralized job registry with list, pause, and delete operations
- Supports periodic intervals, exact times, and cron expressions
- Jobs persist across reboots via launchd plist management
- Human-readable job status and next-run time display

## 使用方式

透過以下觸發語句呼叫 Claude Code 來使用此技能：

- "schedule a task"
- "set up a cron job"
- "run something periodically"
- "排程"
- "定時任務"
- "每天幾點執行"

## 相關技能

- [`macos-ui-automation`](https://github.com/joneshong-skills/macos-ui-automation)
- [`executor`](https://github.com/joneshong-skills/executor)

## 安裝

將技能目錄複製到 Claude Code 技能資料夾：

```
cp -r scheduler ~/.claude/skills/
```

放置在 `~/.claude/skills/` 的技能會被 Claude Code 自動發現，無需額外註冊。
