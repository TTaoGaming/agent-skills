---
name: quota-burn
description: Turn spare AI quota into useful, checked work that gets better each cycle. You give the quota, a plain-language backlog and a place to log; the agent dispatches workers, checks every task against its definition of done, has an independent auditor spot-check, writes a short retro, and uses past retros to plan the next cycle. Logs to JSONL in git by default (or a Slack thread, SQLite, Notion). Use when someone wants to use up remaining quota on real work, or says "burn my quota" or "keep working while I'm away".
metadata: {version: "0.6.0", status: experimental}
---
# Quota burn
Spare quota in, checked work out, and a little smarter every cycle.

```
quota + telemetry ─┐
backlog + done-when ┼─► plan ─► dispatch ─► work ─► check ─► audit ─► retro ─┐
past retros ────────┘     ▲                                                  │
                          └──────────────── learn ◄──────────────────────────┘
everything is logged to one board (board.jsonl)
```

## 1 · Start (keep it short)
Say hello in one line. Then ask 5 quick questions, offering the default in brackets:
1. **Quota:** how much is left, how much should I use, and by when? [half of what's left, before it resets] If you can read usage yourself, show the number instead of asking.
2. **Telemetry:**
   - manual: you tell me what's left every X hours [default, 2 h];
   - automatic: through a command, API or bridge you've shown working once;
   - both.
3. **Run mode:**
   - one-shot: one cycle now [default];
   - scheduled: a scheduled task runs a cycle every N hours and picks up from the board.
4. **Way of working:** show §3 and recommend Builder + checker.
5. **Backlog and board:**
   - **Backlog:** where is the work? [`backlog.md`: one task per line, in plain words]
   - **Board:** where should results go? [`board.jsonl`, committed to git] or a Slack thread, SQLite, Notion, whatever you already use.

If the board already has retros, read them first and say in one line what you'll do differently this time. Read the plan back in 3 lines. Start when they say go.

## 2 · The cycle
1. **Plan:** pick tasks that fit the budget. For each one, write its **definition of done** first. That's a test, a script, or a clear pass/fail rule, and it gets logged before any work starts.
2. **Dispatch:** hand each task to a worker with the chosen pattern. Workers can be this agent, fresh subagents, other agent CLIs, or other models.
3. **Check:** a **checker** that did not build it runs the definition of done. Pass or fail, logged. No "done" without a passing check.
4. **Audit:** an **auditor**, independent of both builder and checker, spot-checks the cycle:
   - it re-runs about 1 in 5 passed checks;
   - it reads diffs for weakened tests or skipped work;
   - it logs agree or disagree.
5. **Telemetry:** at the agreed interval, log quota left. Compare planned against actual burn, then speed up or slow down.
6. **Retro:** at the end of the cycle, log what worked, what didn't, and **one** change for next time.
7. **Learn:** the next cycle starts by reading past retros and the numbers. It keeps what scored best and tries one change at a time.

Stop when the budget, the deadline, or the useful work runs out. Never invent busywork.

## 3 · Ways of working
| Pattern | How it works | Good for |
|---|---|---|
| Solo | One agent works the list in order | Small, simple lists |
| **Builder + checker** (default) | One agent builds; a separate, fresh agent checks against the definition of done | Most work |
| Lead + team | A lead splits the work; builders run in parallel; checkers verify each piece | Big lists, when helpers can run in parallel |
| Research team | Propose ideas → critique → rank → refine the best (co-scientist style) | Open questions, research |
| Attack and defend | One side builds, the other tries to break it; keep what survives (Red Queen) | Robustness, security, search |

## 4 · Seams (swap any part without changing the rest)
| Seam | Default | Other adapters |
|---|---|---|
| Backlog | `backlog.md`, plain words | GitHub issues, a TODO list, a Notion or Sheets table |
| Board | `board.jsonl` in git, append only | a Slack thread, `board.sqlite`, Cloudflare D1/KV, Notion |
| Telemetry | ask the user every X hours | a status command, usage API, or bridge script |
| Dispatch | this agent + fresh subagents | other agent CLIs or models, scheduled tasks |
| Checker | the definition of done, run by a fresh agent | a second model for judgment calls, or the user |
| Auditor | a fresh agent on a different model if available | the user, a weekly human review |

## 5 · Board events (one JSON line each)
- **plan:** `{"t":"…","type":"plan","cycle":3,"budget":50,"unit":"% of window","deadline":"…","pattern":"builder+checker","telemetry":"manual/2h","change":"<the one change from the last retro>"}`
- **quota:** `{"t":"…","type":"quota","left":72,"source":"user|auto"}`
- **task:** `{"t":"…","type":"task","cycle":3,"id":"T3","task":"<plain words>","done_when":"<command or rule>","status":"done|failed|parked","result":"pass|fail","worker":"<who>","checker":"<who>","tokens":<if known>,"minutes":<wall time>}`
- **audit:** `{"t":"…","type":"audit","cycle":3,"id":"T3","auditor":"<who>","verdict":"agree|disagree","why":"…"}`
- **retro:** `{"t":"…","type":"retro","cycle":3,"worked":["…"],"didnt":["…"],"change_next":"…","checked_per_pct":1.4,"fail_rate":0.2,"audit_disagree":0}`

## 6 · The numbers that matter
- **Checked tasks per 1% of quota** (or per 100k tokens, when tokens are reported).
- **Audit disagree rate.** Above 0, trust the checks less. The next change should fix checking before it speeds anything up.
- **Claimed done but failed the check.**

## 7 · Status at each check-in (≤4 lines)
`Cycle <n> · used <x> of <y> · <k> checked ✓, <j> failed · audit <agree>/<sampled> · next: <task> · need from you: <thing or "nothing">`

## 8 · Compare (optional ablation)
Run two arms on matched halves of the backlog, changing one thing. Examples: builder + checker vs solo; with retros vs without. Report the §6 numbers for each arm.

## Rules
- Ask before anything that can't be undone or that leaves the machine: push, send, buy, delete, publish.
- Never guess a quota number. If you can't read it and the user doesn't answer, pause.
- The builder never checks its own work; the auditor is neither builder nor checker.
- Stay inside each provider's limits and terms.
