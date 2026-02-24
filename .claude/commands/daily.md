---
description: Extract daily engineering insights from logs or a markdown file
argument-hint: "[YYYY-MM-DD or /path/to/file.md]"
---

# Daily Insights Extraction → Knowledge Base

## Input Parsing

Parse `$ARGUMENTS`:
- **Date** (e.g., `YYYY-MM-DD`): Read logs from the logs directory for that date
- **File path** (e.g., `/path/to/file.md`): Read that specific file as input. Extract the date from the filename (e.g., `activity-log-YYYY-MM-DD.md` → `YYYY-MM-DD`). If no date can be extracted, ask the user.
- **Empty**: Use today's date

## Reading from Logs Directory

The logs directory is at `/Users/John/Desktop/logs` (also symlinked as `logs/` in the project root, but the symlink may not resolve for Glob). Always search using the absolute path.

When the input is a date:

1. Use Glob to find conversation files: `/Users/John/Desktop/logs/*/{date}/session_*/conversation-*.md` and `/Users/John/Desktop/logs/*/{date}/codex_session_*/conversation-*.md`
2. Read each `conversation-*.md` file found — these are the **primary source**. They contain human-readable markdown with `## 👤 User` / `## 🤖 Assistant` sections showing the full engineering conversation.
3. Check for activity logs using **both** methods (Glob can miss files under symlinked paths):
   - Glob: `/Users/John/Desktop/logs/*/activity-log-{date}.md`
   - If Glob returns nothing, fall back to Bash: `ls /Users/John/Desktop/logs/*/activity-log-{date}.md 2>/dev/null`
   - Read any activity logs found by either method
4. Only read `transcript.jsonl` from a session directory if the corresponding conversation file lacks sufficient detail for insight extraction

If no conversation files or activity logs are found for the date, tell the user and stop.

## Knowledge Base Integration

### Step 1: Read the Knowledge Base Index

Read `output/kb/index.md`. If it doesn't exist, this is the first run — treat the index as empty and proceed (you'll create it at the end).

### Step 2: Extract Insights

Extract non-obvious, specific technical insights from the logs. Critical rules:

- **NEVER pad** — if only 2 genuinely interesting things happened, return 2
- Every insight MUST reference CONCRETE technical decisions, specific files, specific tools, or specific interactions
- GOOD: "Chose binary search simulation over algebraic solution because median-based metrics are piecewise non-differentiable"
- BAD: "Implemented feature X" or "Fixed bug in component Y" or "Good debugging skills demonstrated"
- NEVER produce generic observations like "importance of testing" or "value of clean code"
- The quotable one-liner should be genuinely memorable, not corporate platitudes
- Lessons learned must be SPECIFIC and ACTIONABLE, not generic advice

### Insight Types

Classify each insight using one of the types below, or create a new type if the insight clearly doesn't fit any existing one (use PascalCase naming):
- **DecisionRationale** — why a specific technical choice was made over alternatives
- **FailureLesson** — what went wrong, what was learned from debugging or mistakes
- **PatternDiscovery** — a reusable pattern or technique discovered during work
- **ToolEvaluation** — assessment of a tool, library, or framework from hands-on use
- **ProcessEvolution** — how a workflow, process, or methodology evolved based on experience
- **TechnicalDepth** — deep technical exploration of a concept, algorithm, or system
- **PerformanceInsight** — measurement-driven optimization: profiling, benchmarking, and the specific change that moved the needle
- **Workaround** — hitting a platform/library limitation and finding a creative bypass
- **IntegrationLesson** — making two systems, APIs, or services work together: interop pain and solutions
- **DebugApproach** — the methodology of finding a bug, not just the failure itself: how you isolated the root cause
- **Tradeoff** — a conscious compromise with explicit costs on both sides, not just "I chose A over B"

### Step 3: Classify & Match Each Insight

For each extracted insight, determine:

1. **Domain category** — which folder it belongs to. Start with existing categories from `output/kb/index.md`, then use the known defaults below:

| Folder | Domain |
|---|---|
| `frontend` | React, Next.js, CSS, UI/UX, component design |
| `backend` | APIs, server logic, auth, middleware |
| `database` | PostgreSQL, Firestore, data modeling, queries |
| `architecture` | System design, patterns, dependency management, abstractions |
| `algorithms` | Algorithmic techniques, numerical methods, data structures |
| `ai-integration` | LLM pipelines, prompt engineering, structured extraction |
| `devprocess` | Tooling, workflows, CI/CD, git, DX |
| `debugging` | Debugging techniques, failure analysis, diagnostics |
| `general` | Catch-all — use only when no existing or new category fits |

**This list is extensible.** If an insight clearly belongs to a domain not covered by any existing category, create a new one:
- Use a kebab-case folder name (e.g., `security`, `performance`, `infrastructure`, `testing`, `data-engineering`)
- The new category will be added to `index.md` and its folder created under `output/kb/`
- Prefer creating a specific new category over dumping into `general` — `general` is the last resort, not the default for anything new
- Check the current `index.md` first — someone may have already created the category you need

2. **Concept matching** — compare against existing index entries by title, description, and tags:
   - If clearly about an existing concept → **APPEND** a new dated entry to that concept file
   - If new concept → **CREATE** a new concept file

3. **Slug** — for new concepts, generate a kebab-case slug (e.g., `cookie-scoping-localhost`, `react-state-batching`)

4. **Tags** — 3-6 specific, lowercase tags for the concept

### Step 4: Write Concept Files

**Directory creation:** Only create directories that are needed for the current run's output. Before writing any file, ensure its parent directory exists (use `mkdir -p` for just that one path). Do NOT pre-create the full category tree — only create `output/kb/{category}/` directories that will actually receive a file this run. Same for `output/kb/daily/` and `output/insights/`.

**For NEW concepts** — create `output/kb/{category}/{slug}.md`:

```markdown
# [Concept Title]

> [2-3 sentence description of what this concept covers — broad enough
> to encompass future entries on the same topic]

**Tags:** tag1, tag2, tag3, tag4

---

## YYYY-MM-DD — [Entry Title — specific to this date's work]

**Type:** DecisionRationale | FailureLesson | etc. | **Project:** [ProjectName]

[3-5 sentence narrative explaining the insight with concrete technical details]

**Technical Detail:** [1-2 sentence deep technical explanation for fellow engineers]

**Lessons:**
- [specific, actionable takeaway]
- [another if warranted]

> "[quotable one-liner]"
```

**For APPENDING to existing concepts** — read the existing concept file, then add a new dated section at the bottom (before any trailing whitespace):

```markdown

---

## YYYY-MM-DD — [Entry Title]

**Type:** DecisionRationale | **Project:** [ProjectName]

[narrative...]

**Technical Detail:** [...]

**Lessons:**
- [...]

> "[one-liner]"
```

Important: When appending, do NOT modify the existing header or previous entries. Only add the new `---` separator and dated section.

### Step 5: Update the Index

Update `output/kb/index.md` with the full current state of the knowledge base:

```markdown
# Knowledge Base Index

## frontend
- [Concept Title](frontend/slug.md) — Short description. Tags: tag1, tag2. Dates: YYYY-MM-DD

## backend
- [Concept](backend/slug.md) — Description. Tags: tag1, tag2. Dates: YYYY-MM-DD

## new-category-name
- [Concept](new-category-name/slug.md) — Description. Tags: tag1, tag2. Dates: YYYY-MM-DD

(... all existing categories preserved ...)
```

Rules for the index:
- All existing category sections in `index.md` MUST be preserved (even if empty — just the `## category` heading)
- When creating a new category, add its `## category-name` heading in alphabetical order among existing headings
- For new concepts, add a new entry under the appropriate category
- For appended concepts, add the new date to the existing entry's `Dates:` list
- Keep entries alphabetically sorted within each category
- The description and tags should match the concept file's header

### Step 6: Write the Insights File

Write `output/insights/YYYY-MM-DD-insights.md`:

```markdown
# Daily Insights — YYYY-MM-DD

> [2-3 sentence summary of the day's overall engineering work]

## [Insight Title — specific, referencing the actual technical decision or discovery]
**Type:** DecisionRationale | FailureLesson | PatternDiscovery | ToolEvaluation | ProcessEvolution | TechnicalDepth
**Category:** [Use the KB category name for this insight, e.g., Architecture, Debugging, AIIntegration, DevProcess, Frontend, or any new category]

[3-5 sentence narrative explaining the insight with concrete technical details]

**Technical Detail:** [1-2 sentence deep technical explanation for fellow engineers]

**Lessons Learned:**
- [specific, actionable takeaway]
- [another if warranted]

> "[quotable one-liner]"

---

(repeat for each insight)

## Emerging Threads
- [cross-day patterns or threads that may connect to other days' work]
- [themes worth watching]
```

## Confirmation

After writing all files, report:
- How many concept files were **created** (new) vs **appended** (existing)
- List each concept file path and whether it was new or updated
- The insights file path
