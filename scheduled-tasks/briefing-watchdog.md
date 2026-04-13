# briefing-watchdog

- **taskId:** `briefing-watchdog`
- **cronExpression:** `25 6 * * *`
- **description:** Verify morning briefing, diagnose failures, attempt repairs, alert if needed

---

## Prompt

Verify today's morning briefing was generated. If not, diagnose the failure, attempt repairs, and alert.

Do NOT use the Agent tool anywhere in this task.

**Project root:** `{PROJECT_ROOT}`

### Architecture context

The morning briefing is a pipeline of scheduled tasks:
- gather-weather (5:50 AM) -> writes `{PROJECT_ROOT}/data/staging/weather.md`
- gather-news (5:50 AM) -> writes `{PROJECT_ROOT}/data/staging/news.md`
- gather-local (5:55 AM) -> writes `{PROJECT_ROOT}/data/staging/local.md`
- assemble-briefing (6:08 AM) -> reads staging, writes `{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md`, generates TTS, delivers

### Step 1: Check if briefing exists

Check if `{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md` exists (using today's date) and is more than 500 bytes.

**If YES:** Briefing delivered successfully. Note "Briefing verified for [date]" and stop.

**If NO:** Continue to diagnosis.

### Step 2: Diagnose -- check staging files

Check each staging file and classify its state:

| File | Check | Classification |
|------|-------|---------------|
| `{PROJECT_ROOT}/data/staging/weather.md` | Exists? >100 bytes? Has today's timestamp in status comment? | OK / MISSING / STALE |
| `{PROJECT_ROOT}/data/staging/news.md` | Exists? >100 bytes? Has today's timestamp? | OK / MISSING / STALE |
| `{PROJECT_ROOT}/data/staging/local.md` | Exists? >100 bytes? Has today's timestamp? | OK / MISSING / STALE |

Build a diagnosis string like: "weather: OK, news: MISSING, local: OK"

### Step 3: Attempt repairs based on diagnosis

#### Case A: All staging files exist but briefing is missing
The assembly task failed. Attempt repair:
1. Read all three staging files
2. Compose the briefing directly (same format as assemble-briefing task)
3. Write to `{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md`
4. Run TTS: `python "{PROJECT_ROOT}/scripts/generate_tts.py" "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md"`
5. Deliver via all channels:
   ```bash
   python "{PROJECT_ROOT}/scripts/send_discord.py" --title "Morning Briefing -- MONTH DD, YYYY [RECOVERED]" --file "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md" --attach "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.mp3"
   python "{PROJECT_ROOT}/scripts/send_email.py" --subject "Morning Briefing -- MONTH DD, YYYY [RECOVERED]" --body-file "{PROJECT_ROOT}/data/briefings/YYYY-MM-DD.md"
   bash "{PROJECT_ROOT}/scripts/send_push.sh" "Briefing Recovered -- Assembly had failed but watchdog recovered and delivered."
   ```
6. If MP3 generation fails, deliver without --attach flag.

#### Case B: Some staging files exist, some missing
Assemble a partial briefing from whatever staging files exist:
1. Read whichever staging files are available
2. For missing sections, use placeholder: "[Section unavailable -- gatherer failed]"
3. Write briefing, generate TTS, deliver with [PARTIAL] tag in subject/title
4. Send push alert listing which gatherers failed:
   ```bash
   bash "{PROJECT_ROOT}/scripts/send_push.sh" "Briefing Partial -- Delivered with missing sections: [list]. Check gatherer tasks."
   ```

#### Case C: ALL staging files missing
The entire pipeline failed (possibly machine was asleep, or all gatherers hung on permissions).
1. Attempt a quick recovery -- do a single WebSearch for "weather {CITY} {STATE} today" and gather local data from files
2. If WebSearch works, write a minimal briefing with weather + local data (skip news)
3. Deliver with [MINIMAL] tag
4. If even WebSearch fails, send alert only:
   ```bash
   bash "{PROJECT_ROOT}/scripts/send_push.sh" "Briefing FAILED -- All pipeline stages failed. No staging data available. Run /briefing manually."
   ```

### Step 4: Report

Summarize what happened:
- Which staging files were present/missing/stale
- Whether repair was attempted and succeeded
- Whether delivery succeeded
