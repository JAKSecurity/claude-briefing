# assemble-briefing

- **taskId:** `assemble-briefing`
- **cronExpression:** `8 6 * * *`
- **description:** Assemble morning briefing from staging files, generate TTS, and deliver via all channels

---

## Prompt

Assemble the morning briefing from staging files and deliver via all channels.

**Project root:** `{PROJECT_ROOT}`

### Step 1: Read staging files

Read these files:
- `{PROJECT_ROOT}/data/staging/weather.md` -- weather forecast
- `{PROJECT_ROOT}/data/staging/news.md` -- headlines and deep-dive
- `{PROJECT_ROOT}/data/staging/local.md` -- tasks, TELOS, projects, shopping

If any staging file is missing or empty, note the gap in the briefing but continue with what's available.

### Step 2: Check staleness

For each staging file that exists, check the `<!-- status: ... -->` comment at the end. If the timestamp is more than 24 hours old, prepend `[STALE DATA]` to that section.

### Step 3: Compose the briefing

Combine into a clean markdown briefing. Save to `{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md` (today's date) with these sections:
- Date & Weather (3-day forecast)
- Top 10 Headlines
- Deep-Dive Focus (per weekly_focus schedule in {PROJECT_ROOT}/data/config.yaml)
- Project Updates
- Goal Progress & Deadlines
- Nudges & Recommendations
- Shopping Reminder

Use ONLY ASCII-safe characters. No em dashes (use --), no curly quotes, no emoji.

### Step 4: Generate TTS audio

Run:
```bash
python "{PROJECT_ROOT}/scripts/generate_tts.py" "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md"
```
This produces the .mp3 alongside the .md file. If this fails, log the error and continue -- audio is optional.

### Step 5: Deliver via all channels

**Discord** -- IMPORTANT: Use the --file and --attach flags:
```bash
python "{PROJECT_ROOT}/scripts/send_discord.py" --title "Morning Briefing -- MONTH DD, YYYY" --file "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md" --attach "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.mp3"
```
Do NOT pass the file path as a positional argument -- that sends the path as raw text instead of the file contents. If no MP3 was generated, omit --attach.

**Email:**
```bash
python "{PROJECT_ROOT}/scripts/send_email.py" --subject "Morning Briefing -- MONTH DD, YYYY" --body-file "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md"
```

**Push notification:**
```bash
bash "{PROJECT_ROOT}/scripts/send_push.sh" "Morning Briefing ready -- [1-line summary of key items]"
```

### Step 6: Clean up staging

Delete the staging files after successful delivery:
```bash
rm -f "{PROJECT_ROOT}/data/staging/weather.md" "{PROJECT_ROOT}/data/staging/news.md" "{PROJECT_ROOT}/data/staging/local.md"
```

### Error handling
- If Discord delivery fails, still attempt email and push
- If TTS fails, still deliver the text briefing
- Log any errors clearly so the watchdog task can diagnose
- Do NOT use the Agent tool anywhere in this task
