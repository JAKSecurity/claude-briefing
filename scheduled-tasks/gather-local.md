# gather-local

- **taskId:** `gather-local`
- **cronExpression:** `55 5 * * *`
- **description:** Gather local data (tasks, TELOS, projects) for morning briefing -- writes to staging/local.md

---

## Prompt

Gather local data for the morning briefing.

**Project root:** `{PROJECT_ROOT}`
**Output file:** `{PROJECT_ROOT}/data/staging/local.md`

### Instructions

Read the following files and compose the local data section. This task uses NO web searches -- only file reads.

#### Files to read
1. `{PROJECT_ROOT}/data/projects.yaml` -- project statuses, next actions, due dates
2. `{TELOS_PATH}` -- TELOS goals (including Live Pulse section if present)
3. `{PROJECT_ROOT}/data/changelog.md` -- recent project status changes
4. `{PROJECT_ROOT}/data/shopping-list.md` -- shopping list
5. `{PROJECT_ROOT}/data/staging/nudges.json` -- nudges from reconciliation engine (if present)

#### Output format

Write to `{PROJECT_ROOT}/data/staging/local.md`:

```markdown
## Project Updates

[Entries from changelog.md since yesterday. If none, say "No project changes since yesterday."]

## Goal Progress & Deadlines

[For each TELOS goal, show related projects from projects.yaml with:
- Current status and days since last_touched (flag >7 days with WARNING)
- Due dates (flag overdue or due within 7 days)
- Next action and next_action_due if set]

## Nudges & Recommendations

[If nudges.json exists and has entries:
- Show the "recommendation" field as "What to work on: ..."
- List each nudge with its type and message
- Group by type: deadlines first, then staleness, then delegatable, then other
If nudges.json is missing or empty, say "No nudges today."]

## Shopping Reminder

[If unchecked items exist in shopping-list.md, mention the count. If empty, say "Shopping list is empty."]
```

Use ONLY ASCII-safe characters. No em dashes, curly quotes, or emoji.

#### Write status
```
<!-- status: success | timestamp: YYYY-MM-DDTHH:MM:SS -->
```
