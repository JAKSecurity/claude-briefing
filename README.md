# claude-briefing

An automated morning briefing pipeline for Claude Code. Gathers weather, news, and project status, then delivers a compiled briefing via Discord, email, push notification, and TTS audio.

## Architecture

```
5:50 AM  ─┬─ gather-weather  ─→ data/staging/weather.md
           ├─ gather-news     ─→ data/staging/news.md
5:55 AM  ──── gather-local    ─→ data/staging/local.md
6:08 AM  ──── assemble-briefing ─→ data/briefings/YYYY-MM-DD.md + .mp3
                                    └─→ Discord, Email, Push
6:25 AM  ──── briefing-watchdog (recovery if assembly failed)
```

Each stage is a Claude Code scheduled task that runs independently. The pipeline is resilient — each gatherer writes its own staging file, the assembler reads whatever is available, and the watchdog recovers from partial failures.

## Pipeline Stages

### Gatherers (independent, parallel)
- **gather-weather** — Fetches NWS forecast via WebFetch, with WebSearch fallback
- **gather-news** — Searches for headlines and a deep-dive topic (rotates by day of week)
- **gather-local** — Reads projects.yaml, TELOS goals, shopping list, and generates project status

### Assembly
- **assemble-briefing** — Combines staging files into a clean markdown briefing, generates TTS audio, delivers via all channels, then cleans up staging

### Recovery
- **briefing-watchdog** — Runs 17 minutes after assembly; if briefing is missing or empty, diagnoses which stage failed and attempts minimal recovery

## Setup

1. Copy `config.example.yaml` to `config.yaml` and customize
2. Replace `{PROJECT_ROOT}`, `{CITY}`, `{STATE}`, etc. in scheduled task prompts
3. Install [claude-notify](https://github.com/JAKSecurity/claude-notify) for delivery scripts
4. Register scheduled tasks with your Claude Code scheduler

## Dependencies

- [claude-notify](https://github.com/JAKSecurity/claude-notify) — delivery scripts (Discord, email, push, TTS)
- Claude Code scheduled tasks (Anthropic API)
- WebSearch + WebFetch tools (available in Claude Code)

## Configuration

See `config.example.yaml` for all options. Key settings:
- **Location** — NWS coordinates for weather
- **Weekly focus** — deep-dive topic rotation
- **Delivery channels** — which notifications to send
- **Project root** — where data/ directory lives

## License

MIT
