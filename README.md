# Skills to give you Feedback on how you use AI

Two [Claude Code](https://claude.ai/code) slash commands for reflecting on how you work with AI.

---

## `/reflect` — Weekly coaching notes

When you run `/reflect`, it reads your recent conversation transcripts and git commit history, then writes a structured coaching notes file that surfaces patterns, friction points, and honest questions — the kind of things a good coach would say out loud.

### What it does

1. Reads your recent Claude sessions (last 14 days by default) and cross-references them with any other AI tool usage it can detect — Codex, ChatGPT, Gemini, and others leave traces in how sessions start, what context gets re-explained, and what work appears in git without a corresponding conversation
2. Pulls recent `git log` from the active project, including files with heavy churn
3. Writes a coaching notes file to `reflection-notes/YYYY-MM-DD-coach-notes.md`
4. Gives you a brief spoken summary of the single most important takeaway

The output is deliberately uncomfortable in places. It looks for patterns you might not see yourself, decisions that are stalling, topics you keep circling without resolution, and — critically — whether your use of AI across tools is intentional or reactive.

One section specifically analyses **tool-switching behaviour**: signs that you moved between Claude, Codex, ChatGPT, or other tools mid-task, whether that switching was planned or forced (by token limits, rate limits, habit), and whether continuity was lost in the handoff. This is designed for people who use more than one AI tool regularly and want an honest read on whether they're in control of that workflow.

### Output format

Each run produces a Markdown file in `reflection-notes/` with these sections:

- **Themes I observed** — 2–3 dominant topics with specific evidence and a probing question
- **Decisions in motion** — what reached resolution vs. what's still open
- **Friction points** — where you got stuck, with evidence not just description
- **What you might be avoiding** — gaps, dropped threads, things said but not followed up
- **How you're using AI** — quality of collaboration, prompt intentionality, tool-switching patterns
- **Momentum check** — what's moving forward vs. recurring without progress
- **One question for self-reflection** — a single question to carry into the week

---

## `/reflect-token-use` — Token efficiency analysis

When you run `/reflect-token-use`, it reads the raw session files Claude Code writes locally, extracts token usage data, and produces a report on how efficiently you're using context across your recent sessions.

> **Scope:** This skill only covers Claude Code sessions. Token usage from other AI tools (Cursor, Codex, ChatGPT, Gemini, etc.) is not captured — those tools don't expose usage data in a format this skill can read. If you use multiple tools, the report reflects your Claude usage only.

### What it does

1. Reads token usage directly from Claude Code's local JSONL session files (the transcript tool doesn't expose this data — the skill goes to the source)
2. Computes per-session metrics: total tokens, cache hit rate, input-to-output ratio
3. Identifies patterns like context bloat, expensive re-prompting, and abandoned high-cost sessions
4. Writes a report to `reflection-notes/YYYY-MM-DD-token-coach.md`
5. Reports how many tokens the skill itself consumed to run

The report includes a plain-English explanation of what cache hit rate means and how to improve it — it doesn't assume you already know how LLM caching works.

### Output format

Each run produces a Markdown file in `reflection-notes/` with these sections:

- **How to read this report** — explains cache hit rate, input:output ratio, and how to improve them
- **Session snapshot** — table of all analysed sessions with token counts and efficiency metrics
- **What the numbers suggest** — interpretation of patterns, with specific session references
- **Where tokens are working well** — what the data shows you're doing efficiently
- **Suggested changes** — 2–3 numbered, ranked recommendations with mechanical explanations of why each helps
- **What this doesn't tell you** — reminder that efficiency is not the same as outcome quality
- **This run cost** — token count for the skill run itself

### Permissions

This skill runs two shell commands (one `node` script to read session data, one to measure its own cost) and writes one file. To avoid approval prompts, pre-approve `node` in your Claude Code settings.

---

## Installation

Claude Code loads skills from a `commands/` directory. You can install either or both skills globally (available in all projects) or locally (available only in one project).

### Global install

```bash
cp commands/reflect.md ~/.claude/commands/reflect.md
cp commands/reflect-token-use.md ~/.claude/commands/reflect-token-use.md
```

### Local install (per project)

```bash
mkdir -p .claude/commands
cp /path/to/reflect-skill/commands/reflect.md .claude/commands/reflect.md
cp /path/to/reflect-skill/commands/reflect-token-use.md .claude/commands/reflect-token-use.md
```

Then invoke inside any Claude Code session:

```
/reflect
/reflect-token-use
```

## Requirements

- [Claude Code](https://claude.ai/code) with access to `list_sessions` and `read_transcript` tools
- Node.js — required by `/reflect-token-use` to parse session files
- Git (optional — used by `/reflect` to analyse commit history alongside session data)
