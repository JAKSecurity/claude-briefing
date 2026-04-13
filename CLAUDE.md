# claude-briefing

## Project Overview
Automated morning briefing pipeline for Claude Code. Multi-stage: gather weather + news + local project status, assemble into markdown briefing, deliver via Discord/email/push/TTS.

## Pipeline Architecture
- `scheduled-tasks/gather-weather.md` — NWS forecast fetch (WebFetch primary, WebSearch fallback)
- `scheduled-tasks/gather-news.md` — Headlines + deep-dive (rotates by day of week)
- `scheduled-tasks/gather-local.md` — Project status, TELOS goals, shopping, nudges
- `scheduled-tasks/assemble-briefing.md` — Combine staging, generate TTS, deliver, cleanup
- `scheduled-tasks/briefing-watchdog.md` — Recovery if assembly failed

## Key Design Decisions
- Each gatherer is independent — they can fail without blocking others
- Staging files use HTML comment metadata for status tracking
- Assembly reads whatever staging files exist (partial briefing > no briefing)
- Watchdog runs 17 minutes after assembly — only triggers if briefing is missing
- Delivery via external claude-notify scripts (Discord, email, push, TTS)

## Configuration
- `config.example.yaml` — template with all configurable options
- Scheduled task prompts use `{VARIABLE}` placeholders for per-user values
- `{PROJECT_ROOT}` — path to the project data directory
- `{CITY}`, `{STATE}`, `{LAT}`, `{LON}`, `{NWS_SITE}` — weather location

## Dependencies
- `claude-notify` — delivery scripts (separate repo)
- Claude Code scheduled tasks (Anthropic API cron)
- WebSearch + WebFetch tools
