---
description: Generate a LinkedIn post about a topic or from a KB concept file
argument-hint: "[topic or /path/to/file.md]"
---

# LinkedIn Post Generation

## Input Parsing

Parse `$ARGUMENTS`:
- **File path** (contains `/` or ends in `.md`): Read that file directly as source material
- **Topic string** (anything else): Search the KB for matching concept(s):
  1. Read `output/kb/index.md`
  2. Match against concept titles, slugs, tags, and descriptions (case-insensitive, partial match)
  3. If match(es) found → read the matching concept file(s) from `output/kb/{category}/{slug}.md`
  4. If no match → tell the user and list available topics from the index
- **Empty**: List available topics from `output/kb/index.md` and ask the user to pick one

## Generation Rules

Generate a **200-300 word LinkedIn post**. Critical rules:

### Hook (opening 1-2 sentences)
- MUST be a surprising observation, counterintuitive finding, or bold technical statement
- NEVER start with "I'm excited to share...", "Today I learned...", "Just finished...", or any variation
- The hook should stop the scroll — make the reader think "wait, what?"

**Good hook examples:**
- "Spent 3 hours debugging a race condition only to discover the real bug was in my mental model of how React batches state updates."
- "The hardest part of building an AI pipeline isn't the AI — it's deciding what NOT to send to the model."
- "I rewrote the same function 4 times today. Each version was 'correct'. Only the last one was right."

### Body (200-250 words)
- Tell a SPECIFIC story with technical details — a real decision, a real debugging session, a real architectural choice
- Include at least one concrete technical detail (a tool name, a specific metric, a code pattern)
- Write in first person, conversational but technically credible
- No abstract advice — everything should come from a concrete experience

### Banned Words
Never use: synergy, leverage, empower, ecosystem, paradigm shift, game-changer, best-in-class, holistic, thought leader, innovative, disruptive

### Call to Action
- Invite genuine discussion — ask a question that engineers actually want to answer
- NEVER use "like and share" or "follow for more"

### Hashtags
- Include 3-5 relevant hashtags at the end

## Output

Write the result to `output/posts/linkedin/{slug}-post.md` (derive slug from the matched topic or filename) using this format:

```markdown
# LinkedIn Post — [Topic Title]

[Hook — opening 1-2 sentences]

[Body — the story with technical details, 200-250 words]

[Call to action — genuine question for discussion]

#Hashtag1 #Hashtag2 #Hashtag3
```

After writing the file, confirm the path and show a brief preview of the hook.
