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
it "we're done," and it archives what it learned (what worked, what didn't) and
removes its own scaffolding — leaving your project as it was, plus the work it
produced. Next time you want something similar, it reads that archive and reuses
the lessons instead of starting from zero.

## What's in here

- **`START_HERE.md`** — the seed. Everything Claude needs to understand the job
  and assemble the system. This is the only file that matters at the start; the
  rest gets generated (and namespaced so it can be cleanly removed later).

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

From there, just talk it through and look over what it makes. Refine by
discussing ("did we cover X?", "have the roadmap revisit Y near the end") — you
shouldn't need to edit the plan by hand. When `HANDOFF.md` appears, you're ready
to run.

## What running it looks like

`HANDOFF.md` gives you the exact command. It loops one focused `claude -p` session
at a time, each doing a single roadmap step in a fresh context, with hard caps on
iterations and spend so the worst case is "it stopped and left a note," never a
runaway overnight loop. It runs on your subscription, so a long build makes the
most of the plan you're already paying for.

## One honest note

A self-assembling system is powerful but harder to debug than a fixed script.
Treat the first run as a supervised dry run: watch an iteration or two, read the
`bus/log/`, confirm the caps and the hand-off behave — then trust it overnight.
