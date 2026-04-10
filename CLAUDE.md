# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A Claude Code skill repository. It contains a single custom slash command — `/reflect` — that acts as a weekly reflection coach. When invoked, it reads Claude conversation transcripts (via `list_sessions` / `read_transcript` tools), inspects recent git history, and writes a structured coaching notes file to `reflection-notes/[YYYY-MM-DD]-coach-notes.md`.

## Repository structure

```
commands/
  reflect.md    ← the skill prompt (invoked as /reflect in Claude Code)
reflection-notes/   ← created at runtime; not committed
```

Skill prompts live in `commands/` as Markdown files. Claude Code picks them up automatically as slash commands using the filename (without extension) as the command name.

## How skills work

A skill file is a plain Markdown prompt. It has no build step, no dependencies, and no tests. The "code" is the prompt text itself.

- To add a new skill: create `commands/<name>.md`
- To modify behavior: edit the Markdown directly
- To invoke locally: run `/reflect` (or `/<name>`) inside a Claude Code session

## Key design decisions in `reflect.md`

- **Two data sources:** session transcripts (Claude's own conversation history) + `git log` output. The git data is meant to surface work that happened *outside* Claude sessions.
- **Coaching stance, not summarisation:** the prompt explicitly instructs Claude to interpret and challenge rather than recap — this is load-bearing; be careful not to soften it.
- **Tool-switching analysis:** one section specifically looks for signs the user switched between AI tools mid-task (Codex, ChatGPT, etc.) and whether that switching is intentional or reactive.
- **Output format is fixed:** the section headers and order in the coach notes file are intentional. Don't reorder or rename them without updating the prompt.
