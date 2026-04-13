# gather-news

- **taskId:** `gather-news`
- **cronExpression:** `50 5 * * *`
- **description:** Gather news across all topics for morning briefing -- writes to staging/news.md

---

## Prompt

Gather news for the morning briefing.

**Project root:** `{PROJECT_ROOT}`
**Output file:** `{PROJECT_ROOT}/data/staging/news.md`

### Instructions

You MUST complete quickly. Do NOT use the Agent tool. Do NOT launch subagents. Use WebSearch and WebFetch directly. Search topics SEQUENTIALLY -- do not try to parallelize.

#### Step 0: Read news history for deduplication

Read `{PROJECT_ROOT}/data/staging/news-history.md` if it exists. This file contains headlines, source URLs, and one-line summaries from the past 5 days. Use it to avoid repeating stories:

- **SKIP** stories where the same source URL appeared in the last 3 days AND there is no meaningful new development
- **Tag as [UPDATE]** stories where the same topic was covered recently BUT there is a genuine new development (new data, new decision, escalation, reversal). Lead with what's new, don't repeat background.
- **Include normally** stories that are genuinely new (no matching URLs or headlines in history)

If the history file doesn't exist, proceed normally -- all stories are new.

#### Step 1: Determine today's focus

Read `{PROJECT_ROOT}/data/config.yaml` and check the `weekly_focus` schedule for today's day of week:
- Monday: World news + New AI models (model size, parameters, MoE details, benchmarks esp. coding)
- Tuesday: Agentic AI (frameworks, tools, security, industry developments)
- Wednesday: Iran/US-Iran war + Ukraine-Russia war
- Thursday: Cybersecurity (CVEs, threats, tools, policy, breaches)
- Friday: Weekly rollup of all focus areas

#### Step 2: Gather headlines

Search for "top US and world news headlines today" using WebSearch. Extract the top 10 headlines with 1-2 sentence summaries and source attribution.

#### Step 3: Gather deep-dive content

For today's focus topic(s), perform 3-4 targeted WebSearches to gather detailed content. Write 5-10 paragraphs per focus topic with specific details, names, numbers, dates, sources. Target 15-20 minutes total reading time.

After EACH successful search, IMMEDIATELY append results to `{PROJECT_ROOT}/data/staging/news.md`. This way, if a later search hangs, earlier results are already saved.

If a WebSearch fails for a topic, try ONE more time. If it fails again, write `[Topic unavailable -- search failed]` and move to the next topic.

#### Step 4: Output format

The file should start fresh each run:

```markdown
## Top 10 Headlines

1. **Headline** -- Summary. (Source)
...

## Deep Dive -- [Today's Focus Topic]

[5-10 paragraphs per topic with specific details]
```

Use ONLY ASCII-safe characters. No em dashes, curly quotes, or emoji. Use -- for dashes, straight quotes.

#### Step 5: Update news history

After writing all news, update `{PROJECT_ROOT}/data/staging/news-history.md`:

1. Read the existing file (if it exists)
2. Append today's entries:
```
## YYYY-MM-DD
- HEADLINE | source_url | one-line summary
```
3. Remove any entries older than 5 days
4. Write the updated file back

#### Step 6: Write status

Append at end of news.md:
```
<!-- status: success | topics_found: N | stories_new: N | stories_updated: N | timestamp: YYYY-MM-DDTHH:MM:SS -->
```
