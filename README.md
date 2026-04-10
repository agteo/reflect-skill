# reflect-skill

A [Claude Code](https://claude.ai/code) slash command that acts as a weekly reflection coach.

When you run `/reflect` inside a Claude Code session, it reads your recent conversation transcripts and git commit history, then writes a structured coaching notes file that surfaces patterns, friction points, and honest questions — the kind of things a good coach would say out loud.

## What it does

1. Reads your recent Claude sessions (last 14 days by default) and cross-references them with any other AI tool usage it can detect — Codex, ChatGPT, Gemini, and others leave traces in how sessions start, what context gets re-explained, and what work appears in git without a corresponding conversation
2. Pulls recent `git log` from the active project, including files with heavy churn
3. Writes a coaching notes file to `reflection-notes/YYYY-MM-DD-coach-notes.md`
4. Gives you a brief spoken summary of the single most important takeaway

The output is deliberately uncomfortable in places. It looks for patterns you might not see yourself, decisions that are stalling, topics you keep circling without resolution, and — critically — whether your use of AI across tools is intentional or reactive.

One section specifically analyses **tool-switching behaviour**: signs that you moved between Claude, Codex, ChatGPT, or other tools mid-task, whether that switching was planned or forced (by token limits, rate limits, habit), and whether continuity was lost in the handoff. This is designed for people who use more than one AI tool regularly and want an honest read on whether they're in control of that workflow.

## Installation

Claude Code loads skills from a `commands/` directory. You can install this skill either globally (available in all projects) or locally (available only in one project).

### Global install

```bash
# Copy into your global Claude Code commands directory
cp commands/reflect.md ~/.claude/commands/reflect.md
```

### Local install (per project)

```bash
# From the root of the project you want to use it in
mkdir -p .claude/commands
cp /path/to/reflect-skill/commands/reflect.md .claude/commands/reflect.md
```

Then invoke it inside any Claude Code session:

```
/reflect
```

## Output format

Each run produces a Markdown file in `reflection-notes/` with these sections:

- **Themes I noticed** — 2–3 dominant topics with specific evidence and a probing question
- **Decisions in motion** — what reached resolution vs. what's still open
- **Friction points** — where you got stuck, with evidence not just description
- **What you might be avoiding** — gaps, dropped threads, things said but not followed up
- **How you're using AI** — quality of collaboration, prompt intentionality, tool-switching patterns
- **Momentum check** — what's moving forward vs. recurring without progress
- **One question to sit with** — a single question to carry into the week

## Requirements

- [Claude Code](https://claude.ai/code) with access to `list_sessions` and `read_transcript` tools
- Git (optional — used to analyse commit history alongside session data)
