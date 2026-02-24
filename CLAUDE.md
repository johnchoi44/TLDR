# TLDR — Engineering Content Engine

A Claude Code skill-based system that reads engineering logs and generates professional content. No external APIs, no build tools — just `/slash-commands` that read markdown and write markdown.

## Engineer Profile

**John** — AI/Full-Stack Engineer building SingleCase AI. Penn State IST graduate student. Core stack: TypeScript, React/Next.js, Python, Node.js, PostgreSQL, LLM integration (Claude, Gemini). Works across frontend, backend, AI pipelines, and infrastructure. Writes daily engineering logs across multiple projects.

## Logs Directory Structure

The `logs/` symlink points to `/Users/John/Desktop/logs`, which contains per-project subdirectories:

```
logs/
├── ExcelsiorProduction/
│   ├── activity-log-YYYY-MM-DD.md              # Optional — not every project/date has one
│   ├── YYYY-MM-DD/
│   │   ├── sessions.md                         # Session metadata
│   │   ├── session_{8hex}/
│   │   │   ├── conversation-{slug}.md          # PRIMARY readable source
│   │   │   ├── transcript.jsonl                # Deep detail fallback
│   │   │   └── plans/
│   │   └── session_{8hex}/
│   │       ├── conversation-{slug}.md
│   │       ├── transcript.jsonl
│   │       └── plans/
│   └── ...
├── portfolio/                                  # Some projects have no activity-log files
│   └── YYYY-MM-DD/
│       ├── sessions.md
│       └── session_{8hex}/
│           ├── conversation-{slug}.md
│           └── transcript.jsonl
├── TLDR/
│   └── YYYY-MM-DD/
│       ├── sessions.md
│       ├── codex_session_{id}/                 # Codex (automated) sessions
│       │   ├── conversation-{id}.md
│       │   └── transcript.jsonl
│       └── session_{8hex}/
│           ├── conversation-{slug}.md
│           └── transcript.jsonl
```

### Reading logs for a date

1. **Conversation files** (primary source): `logs/*/{date}/session_*/conversation-*.md` — human-readable markdown with `## 👤 User` / `## 🤖 Assistant` sections. Session dirs may be `session_{8hex}/` or `codex_session_{id}/`.
2. **Activity logs** (secondary, if present): `logs/*/activity-log-{date}.md` — structured daily logs with `### [CATEGORY] Title` headings. Not every project has these.
3. **Transcripts** (deep detail fallback): `logs/*/{date}/session_*/transcript.jsonl` — only consult when conversations lack sufficient detail. Filter to `type=user` and `type=assistant`, skip `thinking` and `tool_use` blocks.

Use Glob patterns to discover what's available. Not every project will have logs for every date.

## Output Conventions

All output goes into the `output/` directory:

| Content Type | Path | Filename Pattern |
|---|---|---|
| Knowledge base index | `output/kb/` | `index.md` |
| KB concept files | `output/kb/{category}/` | `{slug}.md` |
| Daily insights | `output/insights/` | `YYYY-MM-DD-insights.md` |
| LinkedIn posts | `output/posts/linkedin/` | `{slug}-post.md` |
| Medium articles | `output/posts/medium/` | `{slug}-article.md` |
| Interview stories | `output/posts/interviews/` | `{slug}-stories.md` |

### Knowledge Base Structure

The `/daily` skill writes insights into a topic-based knowledge base under `output/kb/`. Each concept gets its own markdown file that grows over time as the same topic recurs. Default categories:

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
| `general` | Catch-all — only when no existing or new category fits |

This list is extensible — new categories (e.g., `security`, `performance`, `infrastructure`, `testing`) are created when insights clearly belong to a domain not covered above. Check `output/kb/index.md` for the current full list.

Downstream skills (`/linkedin`, `/medium`, `/interview`) read topic-based concept files from `output/kb/{category}/` and the KB index at `output/kb/index.md`. They fall back to reading a user-provided file path directly.

## Universal Quality Rules

These apply to ALL content generation:

1. **No corporate buzzwords** — never use: synergy, leverage, empower, ecosystem, paradigm shift, game-changer, best-in-class, holistic, thought leader
2. **Every insight must reference concrete technical details** — specific tools, files, libraries, metrics, decisions, or code patterns. "Improved performance" is banned; "Reduced Firestore reads from 47 to 3 per page load by batching user queries" is good.
3. **No generic observations** — "testing is important" and "clean code matters" are not insights. Every claim needs a specific story behind it.
4. **No padding** — if the day was light, produce fewer insights rather than stretching thin material. Quality over quantity always.
5. **First person, technically credible** — write as a senior engineer sharing real experience, not a content marketer producing "thought leadership."
