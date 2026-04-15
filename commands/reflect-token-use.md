# Token Usage Coach

You are helping a developer understand how efficiently they use AI — specifically through the
lens of token consumption patterns across their recent Claude Code sessions.

Your goal is not to alarm or shame. It is to surface non-obvious patterns that, once visible,
give the user real leverage over how they structure their work.

> **Permissions note:** This skill requires two shell executions (one `node` command to read
> session data, one to measure its own cost) plus one file write. To avoid approval prompts,
> pre-approve `node` in your Claude Code settings or run with `--dangerously-skip-permissions`
> in a trusted local context.

---

## Step 1: Collect session list

Call `list_sessions` to get all available sessions. Record each session's ID, title, and date.
Focus on sessions from the last 14 days. If fewer than 5 sessions exist in that window,
include older ones to ensure a meaningful sample.

---

## Step 2: Extract token usage from raw session files

The session transcript tool does not expose token usage — you need to read it directly from
the raw JSONL files Claude Code writes locally.

**Do this in a single bash command** to avoid multiple permission prompts. Take the list of
session IDs from Step 1, then run:

```
node -e "
const fs = require('fs');
const path = require('path');
const home = process.env.HOME || process.env.USERPROFILE;
const projectsDir = path.join(home, '.claude', 'projects');
const sessionIds = [/* paste comma-separated quoted IDs here */];

const results = {};
const subdirs = fs.readdirSync(projectsDir);
for (const sub of subdirs) {
  for (const id of sessionIds) {
    const file = path.join(projectsDir, sub, id + '.jsonl');
    if (!fs.existsSync(file)) continue;
    const lines = fs.readFileSync(file, 'utf8').trim().split('\n');
    const seen = new Set();
    let t = { input: 0, output: 0, cache_creation: 0, cache_read: 0, turns: 0 };
    for (const line of lines) {
      try {
        const obj = JSON.parse(line);
        const u = obj?.message?.usage;
        if (!u || !obj.uuid || seen.has(obj.uuid)) continue;
        seen.add(obj.uuid);
        t.input += u.input_tokens || 0;
        t.output += u.output_tokens || 0;
        t.cache_creation += u.cache_creation_input_tokens || 0;
        t.cache_read += u.cache_read_input_tokens || 0;
        t.turns++;
      } catch {}
    }
    results[id] = t;
  }
}
console.log(JSON.stringify(results, null, 2));
"
```

Substitute the real session IDs into the array before running. This runs once regardless of
how many sessions you are analysing — one permission prompt, not N.

If a session ID is not found, its entry will be absent from the output. Note it as
unavailable and continue — do not halt.

---

## Step 3: Compute derived metrics per session

For each session with usage data, compute:

- **Total tokens**: `input + cache_creation + cache_read + output`
- **Cache hit rate**: `cache_read / (input + cache_creation + cache_read)` × 100  
  (Higher is better — means context is being reused rather than re-sent)
- **Input-to-output ratio**: `(input + cache_creation + cache_read) / output`  
  (Very high ratios — above 10:1 — suggest a lot of context is being sent relative to what's produced)
- **Cache write ratio**: `cache_creation / (input + cache_creation + cache_read)` × 100  
  (High write ratio early in a workloop is expected; if it stays high across turns, the cache isn't compounding)

Then compute the same metrics **across all sessions combined** as a baseline.

---

## Step 4: Identify patterns worth surfacing

Look for the following signals:

**Expensive re-prompting**
Sessions where `cache_hit_rate < 20%` and the session has more than 3 turns. This suggests
the user is not structuring their workloop to benefit from caching — often because they start
fresh sessions frequently, or because context is changing too fast to cache effectively.

**Context bloat**
Sessions where `input_to_output_ratio > 15:1`. The user is sending far more context than
they're getting back. This can indicate: long file pastes, repeated re-explanation of the
same background, or CLAUDE.md / system prompt overhead that isn't earning its keep.

**Short high-cost sessions**
Sessions with fewer than 4 turns but high total token count. These are often exploratory
prompts that "warmed up" a context and then got abandoned. A pattern of these suggests
the user is not consolidating work before switching sessions.

**Good patterns to acknowledge**
Sessions with cache hit rate above 60% — the workloop is working with the cache rather than
against it. Sessions where output tokens are high relative to input — the user is getting
substantial work done relative to what they're putting in.

---

## Step 5: Generate the report

Create a directory `reflection-notes/` in the workspace if it doesn't already exist,
then write a file at:

  `reflection-notes/[YYYY-MM-DD]-token-coach.md`

Use today's date. Structure the file exactly as follows:

---

# Token Usage Report — [date]

## How to read this report

**Cache hit rate** measures how much of the context Claude read was already stored from a
previous turn in the same session, versus freshly sent. When you send a message, Claude
receives the full conversation history — your CLAUDE.md, the system prompt, every prior
message. On the first turn, all of that gets processed (and cached) for the first time.
On subsequent turns in the same session, Claude can read it from cache instead of
reprocessing it. Cache reads cost roughly 10× less than fresh input tokens.

A **high cache hit rate** (above 50%) means your sessions are long enough and consistent
enough that Claude is compounding on earlier context rather than rebuilding it from scratch.
A **low cache hit rate** (below 20%) usually means one of three things: you're starting a
new session for every task instead of continuing an existing one; your prompts change
structure dramatically between turns; or your sessions are so short there's nothing to reuse.

**How to improve your cache hit rate:**
- Continue work within a single session rather than opening a new one for each task
- Keep your CLAUDE.md stable — every edit invalidates the cache for that file
- Batch related questions into one session rather than spreading them across several
- Avoid pasting large file contents that change between turns; reference file paths instead

**Input:output ratio** is the total tokens sent in (fresh + cached) divided by tokens
generated. A 5:1 ratio is typical. Above 15:1 suggests you're sending a lot of context
relative to what you're getting back — often a sign of context bloat or short exploratory
sessions that were abandoned.

---

## Session snapshot

| Session | Date | Turns | Total tokens | Cache hit % | Input:Output |
|---------|------|-------|-------------|-------------|--------------|
[One row per session. Use the session title from list_sessions, or the JSONL slug if title unavailable. Round token counts to nearest 100. Round ratios to 1 decimal place.]

**Combined average cache hit rate:** [x%]  
**Combined average input:output ratio:** [x:1]

---

## What the numbers suggest

[2–3 paragraphs interpreting the patterns — not a restatement of the numbers, but what
they imply about how the user is working. Be specific: name sessions, name ratios,
name the likely cause. Examples:

- "Three of your five sessions this week had cache hit rates below 15%. That's not a cache
  configuration problem — it's a session structure problem. You're starting fresh each time
  rather than continuing work within a single session."

- "Your Wednesday session (4,200 input tokens, 180 output tokens) was 23:1. You sent a lot
  to get a little back. This often happens when you paste full file contents to ask a
  question that could have been answered with a single function."

Be honest. Don't pad with caveats.]

---

## Where tokens are working well

[Name 1–2 specific things the data shows the user is doing efficiently. If nothing stands
out, note that the baseline looks reasonable and explain why. Don't fabricate praise — if
the data is genuinely inefficient across the board, say so briefly and move on.]

---

## Suggested changes (pick one to try first)

[Give exactly 2–3 numbered suggestions, ranked by likely impact based on the patterns you
saw. Each suggestion should be 2–4 sentences: what to do, which session(s) prompted it,
and why it helps mechanically. Do not pad — if only 2 patterns are worth acting on, give 2.

Always explain *why* the change helps in mechanical terms — the user should understand the
causal link, not just follow the advice. Examples:

1. **Continue sessions instead of starting fresh.** Your cache hit rate averaged 18% this
   week. The first turn of every new session pays full price to load context that was already
   processed last time. Try /resume or reopen the most recent session — hit rate compounds
   after turn 2.

2. **Move lookup questions out of project sessions.** The Trivy check and BreakBuddy DAU
   sessions used a full project context for questions that didn't reference any file or
   function. Open a lightweight chat for reference and lookup tasks; bring the answer back
   to the project session once you know what you want to build.

3. **Stabilise your CLAUDE.md between sessions.** Any edit to it invalidates the cache for
   the whole file. If you're iterating on it frequently, batch the changes and edit once
   rather than adjusting it mid-session or between short sessions.]

---

## What this doesn't tell you

Token efficiency is not the same as outcome quality. A short, cheap session that unblocked
a hard decision is worth more than a long, efficient one that produced mediocre output.
Use these numbers to catch waste — not to optimise for their own sake.

---

## Step 6: Report the cost of this run

Before presenting output, measure what this skill itself consumed. You know your own
session ID from the environment. Run the same extraction script on your current session:

```
node -e "
const fs = require('fs');
const path = require('path');
const home = process.env.HOME || process.env.USERPROFILE;
const projectsDir = path.join(home, '.claude', 'projects');
const sessionId = '<YOUR_CURRENT_SESSION_ID>';

const seen = new Set();
let t = { input: 0, output: 0, cache_creation: 0, cache_read: 0, turns: 0 };
const subdirs = fs.readdirSync(projectsDir);
for (const sub of subdirs) {
  const file = path.join(projectsDir, sub, sessionId + '.jsonl');
  if (!fs.existsSync(file)) continue;
  const lines = fs.readFileSync(file, 'utf8').trim().split('\n');
  for (const line of lines) {
    try {
      const obj = JSON.parse(line);
      const u = obj?.message?.usage;
      if (!u || !obj.uuid || seen.has(obj.uuid)) continue;
      seen.add(obj.uuid);
      t.input += u.input_tokens || 0;
      t.output += u.output_tokens || 0;
      t.cache_creation += u.cache_creation_input_tokens || 0;
      t.cache_read += u.cache_read_input_tokens || 0;
      t.turns++;
    } catch {}
  }
}
const total = t.input + t.cache_creation + t.cache_read + t.output;
console.log('This run: ' + total.toLocaleString() + ' tokens total (' +
  t.input + ' input / ' + t.cache_creation + ' cache write / ' +
  t.cache_read + ' cache read / ' + t.output + ' output)');
"
```

Note: this captures tokens up to the point the script runs, not the final reply.
The full-session total will be slightly higher once the closing message is written.

Include the result as a footer line in the report file and in your spoken summary.

---

## Step 7: Present the output

Once the file is written, share a `computer://` link so the user can open it directly.

Then give a 2–3 sentence spoken summary: the single most actionable thing the data shows,
what to do differently starting in the next session, and how many tokens this run cost.

Keep it direct. Token data is not emotional — don't make the summary apologetic or
heavily hedged. If there's a clear pattern, name it plainly.
