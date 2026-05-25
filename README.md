# Overnight Builder

You've got a goal — a personal blog, a careful rewrite of your study notes, a
feature in an existing app — and you don't want to sit at the screen telling
Claude "next step, next step" all day. This repo is a **seed**: drop it into
your project, talk through the goal with Claude, and it assembles the roadmap,
agents, and runner needed to execute it autonomously, phase by phase, while
you sleep.

You never hand-write the plan — you discuss it. The system handles one goal
end-to-end, then cleans itself back out when you're done.

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

3. Talk it through. Claude asks a few questions, plays back what it understood,
   and assembles the system. Refine by talking — "did we cover X?", "make it
   revisit Y at the end."
4. When **`HANDOFF.md`** appears, you're ready. It contains the single command
   to start the overnight run.

You don't need to understand the rest of this README before starting — Claude
explains as it goes and offers a sensible default for every choice.

---

## After setup: two ways to interact

- **Run it (headless):** `python .overnight/run.py` — the autonomous overnight
  loop. Start it and walk away.
- **Talk to it (interactive):** open `claude` in the repo and run the
  **`/overnight`** slash command. That loads the system's context; after that
  you just talk. "How's it going?", "bias toward more detail", "I think we're
  done with this goal" — it understands all of it. No other commands to remember.

  *If `/overnight` isn't available* (or your editor doesn't show it), paste this
  instead: `Read .overnight/orientation.md and .overnight/bus/state.json, then
  act as the overnight system.` Same effect — the command is only a shortcut.

The system stays dormant between sessions — no persistent process holds a
conversation in memory. State lives in files; a session reads them to know
where things stand.

---

## One goal at a time

You come to this with a goal — a blog, a rewrite, a feature in an existing app.
You don't have to break it into pieces up front; the conversation with Claude
figures out the shape, and the system handles one goal end-to-end.

A goal usually takes a few overnight runs to finish: set it up, let it run,
review in the morning, adjust by talking, run again. When you tell Claude the
goal is done (just say so), it archives what it learned — what worked, what
didn't — and resets, leaving your project as it was plus the work it produced.
Next time you bring a new goal, it reads that archive and reuses the lessons
instead of starting from zero.

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
