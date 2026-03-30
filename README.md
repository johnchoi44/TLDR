# TLDR — Engineering Content Engine

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill-based system that reads engineering logs and generates professional content. No external APIs, no build tools — just `/slash-commands` that read markdown and write markdown.

## Commands

### `/daily [YYYY-MM-DD]`

Extracts engineering insights from a day's logs and writes them into a topic-based knowledge base.

- Reads conversation files and activity logs from the `logs/` directory
- Classifies insights by domain (frontend, backend, architecture, ai-integration, etc.)
- Creates or appends to concept files under `output/kb/{category}/{slug}.md`
- Maintains a searchable index at `output/kb/index.md`

### `/linkedin [topic or /path/to/file.md]`

Generates a LinkedIn post about a specific topic from the knowledge base.

- Looks up the topic in `output/kb/index.md` and reads matching concept files
- Produces a 200-300 word post with a scroll-stopping hook, concrete technical story, and discussion question
- Output: `output/posts/linkedin/{slug}-post.md`

### `/blogs [YYYY-MM-DD, topic, or /path/to/file.md]`

Generates a blog post from engineering activity logs, a KB topic, or a file.

- Accepts a date (reads logs directly), a topic (looks up KB concepts), or a file path
- Produces a 600-1200 word post with code snippets, technical depth, and actionable takeaways
- Output: `output/posts/blogs/{slug}-blog.json` (JSON array with one blog post object)

### `/interview [topic or /path/to/file.md]`

Generates STAR-format interview stories about a specific topic from the knowledge base.

- Looks up the topic in `output/kb/index.md` and reads matching concept files
- Produces stories with Situation, Task, Action, Result — mapped to common interview questions
- Output: `output/posts/interviews/{slug}-stories.md`

## Workflow

```
Engineering logs ──► /daily ──► Knowledge Base ──► /linkedin
                                                ──► /interview
               ──► /blogs
```

1. Work on projects as usual — Claude Code logs conversations automatically
2. Run `/daily 2026-02-24` to extract insights into the KB
3. Run `/linkedin react-state-batching` (or any topic) to generate a post from KB concepts
4. Run `/blogs 2026-02-24` to generate a blog post from that day's engineering logs

## Setup

Requires [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed. The `.claude/commands/` directory contains the skill definitions — Claude Code picks them up automatically.

**Local directories** (not tracked in git):
- `logs/` — symlink to your Claude Code logs directory
- `output/` — generated content (KB, posts, articles)
