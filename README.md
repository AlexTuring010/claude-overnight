# Overnight Builder

You've got a goal — a personal blog, a careful rewrite of your study notes, a
feature in an existing app — and you don't want to sit at the screen telling
Claude "next step, next step" all day. This repo is a **seed**: drop it into
your project, talk through the goal with Claude, and it assembles the
roadmap, agents, and runner needed to execute it autonomously — step by step,
on one or more parallel tracks where that's safe, while you sleep.

You never hand-write the plan — you discuss it. The system handles one goal
end-to-end, holds the line on quality (never ships shallow to look
productive), and cleans itself back out when you're done.

---

## Quick start

**Prerequisites**

1. Install Claude Code and run `claude login` with your Pro/Max plan.
2. Make sure `ANTHROPIC_API_KEY` is **not** set in your shell — if it is, the
   CLI bills per token instead of using your plan.
3. Switch to a fresh git branch in the project you want to work in (or start
   from an empty repo). The system edits files autonomously, so you want a
   clean branch to review before merging.

**Start it**

1. Put `START_HERE.md` at the root of your repo.
2. Open the repo in Claude Code and send:

   ```
   Read START_HERE.md. Don't start the work yet — follow it: get your bearings on
   what's here, talk with me to scope the goal, play back what you understood, then
   assemble the system. Let's begin.
   ```

3. Talk it through. Setup happens across **2-3 short sessions** with a
   `/clear` between each — first you scope the goal with Claude (one
   conversation, fine for it to be long), then `/clear` and Claude
   plans the build in a fresh session, then `/clear` and Claude builds
   the system using fresh subagents per piece. Claude explains the
   why and tells you exactly what to paste at each handoff. The reason
   is the same one that makes the overnight loop work: fresh context
   per step. Refine by talking at any phase — "did we cover X?", "make
   it revisit Y at the end."
4. When **`HANDOFF.md`** appears, you're ready. It contains the single command
   to start the overnight run.

You don't need to understand the rest of this README before starting — Claude
explains as it goes and offers a sensible default for every choice.

---

## After setup: three ways to interact

- **Run it (headless):** `python .overnight/run.py` — the autonomous overnight
  loop. Start it and walk away. Nothing waits on your approval mid-run.
- **Read what happened:** open `.overnight/wakeup.html` in your browser each
  morning. A self-contained read-only page with per-track status, steps closed
  overnight, the decisions Claude made on your behalf, open blockers, and
  cost-per-step. Scannable in a minute — no log archaeology.
- **Talk to it (interactive):** open `claude` in the repo and run the
  **`/overnight`** slash command. That loads the system's context; after that
  you just talk. "How's it going?", "bias toward more detail", "the call on Y
  was wrong, correct it", "I think we're done with this goal" — it
  understands all of it. No other commands to remember.

  *If `/overnight` isn't available* (or your editor doesn't show it), paste
  this instead: `Read .overnight/orientation.md and .overnight/bus/state.json,
  then act as the overnight system.` Same effect — the command is only a
  shortcut.

The system stays dormant between sessions — no persistent process holds a
conversation in memory. State lives in files; a session reads them to know
where things stand.

---

## Operational notes

- **Stop it cleanly while it's running.** `touch .overnight/STOP` from any
  terminal, or Ctrl-C in the runner's terminal, or tell `/overnight` "stop
  after this iteration." All three finish the in-flight iteration, write
  the morning digest, and exit cleanly. If graceful stop ever hangs:
  `touch .overnight/KILL` or hit Ctrl-C twice within 5 seconds — partial
  steps recover on the next run.
- **No idea what state things are in?** Laptop died overnight, terminal
  crashed, woke up to an ambiguous repo: run
  `python .overnight/run.py --inspect`. It's a read-only diagnostic — no
  mutations, no sessions spawned — that reports what's definitely done,
  what's partial, what's orphaned, and what the recovery plan is. Same
  thing conversationally: ask `/overnight` "what state are we in?"
- **Rate limits.** If the runner hits the rolling 5-hour usage window, it
  parses the reset time from the error and waits idle until the window
  reopens (plus a small buffer), then resumes automatically — better than
  wasting a night because the limit was hit an hour before reset. If it
  hits the weekly cap (where waiting would be days), it halts with a
  clear note in `wakeup.html` so you can restart when the window opens.
  Either way, you'll see the state if you check. STOP and KILL still work
  during the wait.

You don't have to inspect before re-running. A normal
`python .overnight/run.py` also recovers from partial state on its own
(detects orphans, marks them, asks the planner to re-plan). `--inspect`
is for when you want to see the diagnosis *before* committing to another
overnight run.

---

## One goal at a time

You come to this with a goal — a blog, a rewrite, a feature in an existing
app. You don't have to break it into pieces up front; the conversation with
Claude figures out the shape, including whether the work naturally splits
into parallel tracks (e.g. infra / content / design) that can advance
independently overnight. The system handles one goal end-to-end, on one
track or many, whichever quality calls for.

A goal usually takes a few overnight runs to finish: set it up, let it run,
read `wakeup.html` in the morning, adjust by talking, run again. When you
tell Claude the goal is done (just say so), it archives what it learned —
what worked, what didn't — and resets, leaving your project as it was plus
the work it produced. Next time you bring a new goal, it reads that archive
and reuses the lessons instead of starting from zero.

## How it stays out of your project

It's built to live inside a project you care about without contaminating it:

- Everything the system owns lives in a single **`.overnight/`** folder
  (roadmap, runner, bus, logs, manifest). During setup Claude asks how to
  handle it in git — **gitignored** (the default; commits only ever contain
  real work) or **committed** (survives across machines, some git noise).
- The few files Claude Code requires elsewhere (`agents`, slash `commands`)
  are `overnight-` prefixed and recorded in a manifest.
- Your **`CLAUDE.md` is never touched** — orientation lives in the system's
  own folder, not in your project's instructions.
- **Archives live in one central spot across projects** (default
  `~/.overnight/archive/<project>/<goal>/`; setup asks where you want it). That
  keeps git noise out of any single project and lets lessons from one archive
  inform a future project's setup. If you want archives to sync across
  machines, point the directory at Dropbox / iCloud / OneDrive or a separate
  git repo you keep just for archives — both work better than burying it
  inside the project repo.

When a goal is done you can keep the system installed for the next one, or have
it uninstall completely — the manifest lets it delete precisely what it added,
so your project returns to its prior state, plus the work.

---

## One honest note

A self-assembling system is powerful but harder to debug than a fixed script.
Treat the first run as a supervised dry run: watch an iteration or two, read
`.overnight/bus/log/`, confirm the caps and the hand-off behave — then trust
it overnight.
