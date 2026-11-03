---
name: quota-burn
description: Turn spare AI quota into useful, checked work. Asks how much quota you have and how fast to use it, takes a plain-language backlog, runs an orchestrator → workers → checker loop, and posts every result to a coordination board (JSONL in git by default; SQLite, Slack or others if you prefer). Use when someone wants to use up remaining quota on real work, or says "burn my quota" or "keep working while I'm away".
metadata: {version: "0.5.0", status: experimental}
---
# Quota burn
Spare quota in, checked work out. You write the backlog in plain words; the agent does the rest and logs everything.

## 1 · Start (keep it short)
Say hello in one line. Then ask 5 quick questions, offering the default in brackets:
1. **Quota:** how much is left, how much should I use, and by when? [half of what's left, before it resets] If you can read usage yourself, show the number instead of asking.
2. **Quota checks:**
   - manual: you tell me what's left every X hours [default, 2 h];
   - automatic: through a tool or bridge you've shown working once;
   - both.
3. **Run mode:**
   - one-shot: work now, in this session [default];
   - scheduled: a scheduled task every N hours continues from the board.
4. **Way of working:** show §2 and recommend Builder + checker.
5. **Backlog and board:**
   - **Backlog:** where is the work? [`backlog.md`: one task per line, in plain words]
   - **Board:** where should results go? [`board.jsonl`, committed to git after each check-in] Or SQLite, a Slack thread, Cloudflare, Notion: whatever you already use.

Read the plan back in 3 lines. Start when they say go.

## 2 · Ways of working
| Pattern | How it works | Good for |
|---|---|---|
| Solo | One agent works the list in order | Small, simple lists |
| **Builder + checker** (default) | One agent builds; a separate, fresh agent checks the result against a test written first | Most work |
| Lead + team | A lead splits the work; several builders run in parallel; checkers verify each piece | Big lists, when you can run helpers in parallel |
| Research team | Propose ideas → critique → rank → refine the best (co-scientist style) | Open questions, research |
| Attack and defend | One side builds, the other tries to break it; keep what survives (Red Queen) | Robustness, security, search |

## 3 · Seams (swap any part without changing the rest)
| Seam | Default | Other adapters |
|---|---|---|
| Backlog | `backlog.md`, plain words; the agent adds each task's check | GitHub issues, a TODO list, a Notion or Sheets table |
| Board | `board.jsonl` in git: one JSON line per event, append only | `board.sqlite`, a Slack thread, Cloudflare D1/KV, Notion |
| Quota | ask the user every X hours | a status command, usage API, or small bridge script |
| Workers | this agent + fresh subagents | other agent CLIs or models |
| Checker | the task's test or script, written first | a second model for judgment calls, or the user |

## 4 · Board events (`board.jsonl`, one line each)
- **Plan:** `{"t":"<time>","type":"plan","budget":50,"unit":"% of window","deadline":"<time>","pattern":"builder+checker","quota_check":"manual/2h","run":"one-shot"}`
- **Quota:** `{"t":"<time>","type":"quota","left":72,"source":"user"}`
- **Task:** `{"t":"<time>","type":"task","id":"T3","task":"<plain words>","check":"<command or rule>","status":"done|failed|parked","result":"pass|fail","worker":"<who>","checker":"<who>","tokens":<if known>}`
- **Note:** `{"t":"<time>","type":"note","text":"started / paused / stopped / why"}`

Anyone can read the status from the board. Several agents can append to it safely.

## 5 · Work loop
1. **Pick** the next backlog line. Write its check first: a test, a script, or a clear pass/fail rule.
2. **Do** it with the chosen pattern. The checker is never the agent that built it.
3. **Post** a `task` event with the check's result. Commit at each check-in.
4. **Check quota** at the agreed interval:
   - post a `quota` event;
   - compare planned vs actual burn;
   - slow down or speed up.
5. **Stop** when the budget, the deadline, or the useful work runs out. Never invent busywork.

**The number that matters:** checked tasks per 1% of quota (or per 100k tokens, when the tool reports tokens).

## 6 · Scheduled mode (optional)
Set the scheduled task only after the user agrees. Each run:
1. read the backlog and the board;
2. check quota if due;
3. do the next tasks;
4. post events;
5. stop.

The schedule ends itself when the budget, the deadline or the work runs out.

## 7 · Compare (optional ablation)
Run two arms on matched halves of the backlog, changing one thing: e.g. builder + checker vs solo, or with vs without this skill. Report checked tasks per 1% of quota and "claimed done but failed the check" for each arm.

## 8 · Status at each check-in (≤4 lines)
`Used <x> of <y> · <n> tasks checked ✓ · next: <task> · need from you: <thing or "nothing">`

## Rules
- Ask before anything that can't be undone or that leaves the machine: push, send, buy, delete, publish.
- Never guess a quota number. If you can't read it and the user doesn't answer, pause.
- Stay inside each provider's limits and terms.
