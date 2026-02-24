---
description: Generate STAR-format interview stories about a topic or from a KB concept file
argument-hint: "[topic or /path/to/file.md]"
---

# Interview Story Generation (STAR Format)

## Input Parsing

Parse `$ARGUMENTS`:
- **File path** (contains `/` or ends in `.md`): Read that file directly as source material
- **Topic string** (anything else): Search the KB for matching concept(s):
  1. Read `output/kb/index.md`
  2. Match against concept titles, slugs, tags, and descriptions (case-insensitive, partial match)
  3. If match(es) found → read the matching concept file(s) from `output/kb/{category}/{slug}.md` — concept files with multiple dated entries are especially valuable here because they show evolution of a topic across multiple dates, providing richer STAR stories
  4. If no match → tell the user and list available topics from the index
- **Empty**: List available topics from `output/kb/index.md` and ask the user to pick one

## Generation Rules

Generate STAR-format interview stories covering different competencies. Critical rules:

### STAR Format (strict)
Each story MUST have all four parts with concrete details:

- **Situation**: Set the scene — project context, team size, stakes, timeline. Make the reader understand the environment.
- **Task**: What was YOUR specific responsibility or goal. Not "the team needed to..." but "I was responsible for..."
- **Action**: Detail SPECIFIC technical decisions you made. Not "I collaborated with the team" but "I designed a cache invalidation strategy using Redis TTL with a pub/sub channel for immediate invalidation of critical paths"
- **Result**: Measurable outcomes where possible — performance improvements, time saved, adoption metrics, reliability gains. What was delivered, what was learned.

### Competency Coverage
Stories should collectively demonstrate:
- **Technical depth** — deep system knowledge, complex debugging, architectural design
- **Leadership** — mentoring, influencing decisions, driving alignment
- **Problem-solving** — systematic approaches to ambiguous problems
- **Communication** — translating technical concepts, stakeholder management
- **System design** — end-to-end thinking, tradeoff analysis, scalability

### Story Quality
- Avoid stories that are just "I fixed a bug" — frame them as systematic approaches to complex problems
- Every story should have a "so what" — why does this matter beyond the immediate task
- Map each story to 2-4 common interview questions it answers well
- Include measurable outcomes wherever possible (even estimates are better than nothing)

### Common Interview Questions to Map To
- "Tell me about a time you debugged a complex issue"
- "Describe a system you designed from scratch"
- "Tell me about a time you had to make a difficult technical tradeoff"
- "Describe how you mentored or influenced others"
- "Tell me about a time a project didn't go as planned"
- "How do you approach performance optimization?"
- "Tell me about a time you had to learn a new technology quickly"
- "Describe your approach to technical debt"

## Output

Write the result to `output/posts/interviews/{slug}-stories.md` (derive slug from the matched topic or filename) using this format:

```markdown
# Interview Stories — [Topic Title]

## [Story Title — descriptive name for the story]

**Competencies:** Technical Depth, Problem-Solving (list 2-3)
**Answers:** "Tell me about a time you..." (list 2-4 interview questions)

### Situation
[Project context, team, stakes, timeline]

### Task
[Your specific responsibility]

### Action
[Detailed technical actions and decisions]

### Result
[Measurable outcomes and what was learned]

---

(repeat for each story)
```

After writing the file, confirm the path and list the story titles with their competency tags.
