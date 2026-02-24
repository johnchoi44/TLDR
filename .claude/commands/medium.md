---
description: Generate a Medium article about a topic or from a KB concept file
argument-hint: "[topic or /path/to/file.md]"
---

# Medium Article Generation

## Input Parsing

Parse `$ARGUMENTS`:
- **File path** (contains `/` or ends in `.md`): Read that file directly as source material
- **Topic string** (anything else): Search the KB for matching concept(s):
  1. Read `output/kb/index.md`
  2. Match against concept titles, slugs, tags, and descriptions (case-insensitive, partial match)
  3. If match(es) found → read the matching concept file(s) from `output/kb/{category}/{slug}.md` — concept files with multiple dated entries are especially useful for showing evolution over time
  4. If no match → tell the user and list available topics from the index
- **Empty**: List available topics from `output/kb/index.md` and ask the user to pick one

## Generation Rules

Generate an **800-1500 word Medium article**. Critical rules:

### Title & Subtitle
- Title should promise SPECIFIC value — "How I Solved X" beats "Thoughts on Software Engineering"
- Subtitle provides additional context and draws the reader in
- Never use clickbait or vague titles

### Structure
- **Introduction** (2-3 paragraphs): Establish the problem and why it matters. Hook the reader with a relatable scenario.
- **3-6 content sections**: Each with a clear heading and purpose. Each section should advance the narrative.
- **Conclusion**: Actionable takeaways, not platitudes

### Content Rules
- Include at least **2 code snippets or technical diagrams** (described in markdown code blocks)
- Mix personal experience ("I discovered...") with technical explanation ("This works because...")
- Reference specific technologies, libraries, patterns by name
- Tone: **"senior engineer teaching from experience"** — not academic, not casual

### Banned
- Generic advice without backing stories
- Corporate buzzwords (see CLAUDE.md universal rules)
- Sections that don't advance the narrative

### Tags
- Include 5-7 Medium tags for discoverability at the end

## Output

Write the result to `output/posts/medium/{slug}-article.md` (derive slug from the matched topic or filename) using this format:

```markdown
# [Article Title]
*[Subtitle]*

[Introduction — 2-3 paragraphs establishing the problem]

## [Section 1 Heading]

[Content with technical details, code snippets where relevant]

## [Section 2 Heading]

[Content continuing the narrative]

(continue for 3-6 sections)

## Key Takeaways

- [Actionable takeaway 1]
- [Actionable takeaway 2]
- [Actionable takeaway 3]

---

*Tags: tag1, tag2, tag3, tag4, tag5*
```

After writing the file, confirm the path and show the title and a one-sentence summary.
