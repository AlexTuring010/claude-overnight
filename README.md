# Overnight Builder — a self-assembling agent system

This repo doesn't contain a build system. It contains a **seed** that builds one
*for you*, by talking to you.

Drop it into a project — empty *or* one you're already working in — open it in
Claude Code, and have a conversation. Claude gets its bearings on what's there,
works out the one bounded goal you want tackled (a new project, or a big task
inside an existing one — a feature, a migration, a refactor, a content push),
then assembles the whole machine: a phased roadmap, a set of agents (including
one that carries your intent and makes decisions on your behalf), a file-based
message bus they coordinate through, and a runner. You start one command and walk
away, and it works the goal phase by phase, overnight, on its own.

You never hand-write the plan. You discuss it. The system sets itself up — and
cleans itself back out when the job's done.

## Think in arcs

The system is scoped to one **arc** at a time. You set it up, run it overnight,
review, adjust, and re-run the next night until the arc is complete. Then you tell
it you're done (just say so — see below), and it archives what it learned (what
worked, what didn't) and resets for the next arc — leaving your project as it was,
plus the work it produced. Next time you want something similar, it reads that
archive and reuses the lessons instead of starting from zero.

## How it stays out of your project

It's built to live inside a project you care about without contaminating it:

- Everything the system owns lives in a single `.overnight/` folder (roadmap,
  runner, bus, logs, manifest). During setup Claude asks how you want it handled
  in git — **gitignored** (the default; your commits only ever contain real work,
  state stays on this machine) or **committed** (survives across machines, at the
  cost of some git noise).
- The few files Claude Code requires elsewhere (`agents`, slash `commands`) are
  `overnight-` prefixed and recorded in a manifest.
- Your `CLAUDE.md` is **never touched** — the system stays dormant until you
  summon it, so its orientation lives in its own folder, not in your project's
  instructions.
- **Archives live outside the repo** (default `~/.overnight/archive/<project>/<arc>/`,
  or wherever you choose at setup), so past arcs never enter any project's git and
  can be reused across projects.

When an arc ends you can keep the system installed for the next one, or have it
uninstall completely — using its manifest to delete precisely what it added, so
your project returns to exactly its prior state, plus the work.

## Two ways to interact with it

- **Run it (headless):** `python .overnight/run.py` — the autonomous overnight
  loop. Start it and walk away.
- **Talk to it (interactive):** open `claude` in the repo and run one command —
  **`/overnight`**. That loads the system's context so the session knows what's
  going on; after that you just talk. "How's it going?", "bias toward more
  detail", "I think we're done with this" — it understands all of it. No other
  commands to remember.
  - *If `/overnight` isn't set up* (or your editor doesn't show it), just paste:
    "Read `.overnight/orientation.md` and `.overnight/bus/state.json`, then act as
    the overnight system." Same effect — the command was only a shortcut for this.

The system stays dormant until you run `/overnight` — it never touches your
`CLAUDE.md` or interferes with normal Claude Code use. There's no persistent
process holding a conversation in memory; the state lives in files, and a session
reads them (via `/overnight`, or the runner's own prompt) to know where things
stand.

## What's in here at the start

- **`START_HERE.md`** — the seed. The only file that matters at the start;
  everything else is generated during setup (and namespaced so it removes cleanly).

## Before you start

- Install Claude Code and run `claude login` with your Pro/Max plan.
- Make sure `ANTHROPIC_API_KEY` is **not** set in your shell — if it is, the CLI
  bills per token instead of using your plan.
- Work on a fresh git branch. The system edits files autonomously, so you want a
  clean branch to review before merging.

## How to start

Put `START_HERE.md` in the repo, open it in Claude Code, and send this:

```
Read START_HERE.md. Don't start the work yet — follow it: get your bearings on
what's here, talk with me to scope the arc, play back what you understood, then
assemble the system. Let's begin.
```

Talk it through, look over what it makes, refine by discussing. When `HANDOFF.md`
appears, you're ready to run.

You don't need to understand how any of this works going in — Claude starts by
explaining what it is and how you'll work together, then walks you through setup.
If a choice comes up (like the git question above), it'll offer a sensible default
you can just accept.

## One honest note

A self-assembling system is powerful but harder to debug than a fixed script.
Treat the first run as a supervised dry run: watch an iteration or two, read the
`.overnight/bus/log/`, confirm the caps and the hand-off behave — then trust it
overnight.
