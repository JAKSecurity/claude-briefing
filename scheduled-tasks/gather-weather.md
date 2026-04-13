# gather-weather

- **taskId:** `gather-weather`
- **cronExpression:** `50 5 * * *`
- **description:** Gather weather data for morning briefing -- writes to staging/weather.md

---

## Prompt

Gather today's weather for the morning briefing.

**Project root:** `{PROJECT_ROOT}`
**Output file:** `{PROJECT_ROOT}/data/staging/weather.md`

### Instructions

You MUST complete quickly. Do NOT use the Agent tool. Do NOT launch subagents. Use WebSearch and WebFetch directly.

#### Step 0: Diagnostic
First, write a one-line diagnostic file to confirm the task is running:
```
Write to {PROJECT_ROOT}/data/staging/weather-diag.txt:
"gather-weather ran at [current timestamp]"
```

#### Step 1: Try WebFetch (preferred)
Fetch the live forecast from the National Weather Service:
```
https://forecast.weather.gov/MapClick.php?CityName={CITY}&state={STATE}&site={NWS_SITE}&textField1={LAT}&textField2={LON}
```
Extract current conditions and the full forecast for the next 3 days (6 forecast periods). Include day names, temperatures, wind, precipitation chances, and any alerts.

#### Step 2: Fallback -- WebSearch
If WebFetch fails, search for "weather {CITY} {STATE} today" using WebSearch.
If you get results, extract the forecast (temps, wind, precipitation, warnings).

#### Step 3: Fallback -- Stale cache
If both fail, check if yesterday's staging file exists at `{PROJECT_ROOT}/data/staging/weather.md`. If it does, read it and prepend this banner:
```
[STALE DATA - from previous day, today's weather fetch failed]
```
If no previous file exists either, write:
```
[WEATHER UNAVAILABLE - all data sources failed]
```

### Output format
Write to `{PROJECT_ROOT}/data/staging/weather.md`:
```markdown
## Weather -- {CITY}, {STATE}

[forecast content here with temps, wind, precipitation]

[Any warnings or alerts]
```

Use ONLY ASCII-safe characters. No em dashes, curly quotes, or emoji. Use -- for dashes, "degrees" or "F" for temperature.

#### Step 4: Write status
Append a status line at the very end of the file:
```
<!-- status: success | source: webfetch | timestamp: YYYY-MM-DDTHH:MM:SS -->
```
Use "websearch", "stale", or "unavailable" for the source field if fallbacks were used.
