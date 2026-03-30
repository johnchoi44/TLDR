---
description: Generate a blog post from activity logs, a KB topic, or a file
argument-hint: "[YYYY-MM-DD, topic, or /path/to/file.md]"
---

# Blog Post Generation

## Input Parsing

Parse `$ARGUMENTS`:
- **Date** (matches `YYYY-MM-DD`): Read logs from the logs directory for that date
- **File path** (contains `/` or ends in `.md`): Read that file directly as source material. Extract a date from the filename if possible; if no date can be extracted, use today's date.
- **Topic string** (anything else): Search the KB for matching concept(s):
  1. Read `output/kb/index.md`
  2. Match against concept titles, slugs, tags, and descriptions (case-insensitive, partial match)
  3. If match(es) found → read the matching concept file(s) from `output/kb/{category}/{slug}.md` — concept files with multiple dated entries are especially useful for showing evolution over time
  4. If no match → tell the user and list available topics from the index
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
4. Only read `transcript.jsonl` from a session directory if the corresponding conversation file lacks sufficient detail for generating a blog post

If no conversation files or activity logs are found for the date, tell the user and stop.

## Generation Rules

Generate **exactly one blog post** per invocation. The post should synthesize the source material into a cohesive article about what was learned.

If the source material lacks substance for a full post, tell the user and stop. Do not pad.

### Title
- Specific and value-promising — "How I Solved X" or "Why Z Breaks When You W"
- Never clickbait, vague, or generic ("Thoughts on Engineering")

### Content Structure (600-1200 words)
- **Opening** (2-3 paragraphs): Establish the problem or discovery. Start with a concrete scenario, not an abstract statement.
- **3-5 content sections**: Each with a `##` heading advancing the narrative. Include at least **1 code snippet or concrete technical example**.
- **Closing**: Actionable takeaway — what should the reader do differently?

### Content Rules
- Mix personal experience ("I discovered...") with technical explanation ("This works because...")
- Reference specific technologies, libraries, patterns by name
- Tone: **"senior engineer teaching from experience"** — not academic, not casual

### Excerpt
- 1-2 sentences capturing the core insight
- Must include at least one specific technical detail
- Should work as a standalone preview/teaser

### Banned
- Generic advice without backing stories
- Corporate buzzwords (see CLAUDE.md universal rules)
- Sections that don't advance the narrative
- Padding or stretching thin material

## Output

Derive a kebab-case `slug` from the title.

Derive the `date` field in "Mon YYYY" format from the input date. If no date was provided, use today's date.

Ensure the `output/posts/blogs/` directory exists (use `mkdir -p`).

Write the result to `output/posts/blogs/{slug}-blog.md` as a markdown file with YAML frontmatter:

```markdown
---
title: "<title>"
slug: "<kebab-case-string>"
date: "<Mon YYYY>"
excerpt: "<1-2 sentence preview with concrete technical detail>"
---

# <title>

<Full blog post content with ## section headings, code blocks, and all formatting. 600-1200 words.>

---

*Written with the help of AI*
```

After writing the file, confirm the path and show the title and a one-sentence summary.
