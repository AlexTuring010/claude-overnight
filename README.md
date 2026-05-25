# Overnight Builder — a self-assembling agent system

This repo doesn't contain a build system. It contains a **seed** that builds one
*for you*, by talking to you.

You drop it into a project, open it in Claude Code, and have a conversation.
Claude works out what you're building — anything: a web app, a dataset pipeline,
a book, a research codebase, a game — then assembles the whole machine: a phased
roadmap, a set of agents (including one that carries your intent and makes
decisions on your behalf), a file-based message bus they coordinate through, and
a runner. When it's done, you start one command and walk away, and it builds your
project phase by phase, overnight, on its own.

The point: you never hand-write the plan. You discuss it. The system sets
itself up.

## What's in here

- **`START_HERE.md`** — the seed. Everything Claude needs to understand your
  project and assemble the system. This is the only file that matters at the
  start; the rest gets generated.

After the bootstrap conversation, Claude will have created `CLAUDE.md`, a living
`ROADMAP.md`, `.claude/agents/`, a `bus/` message board, `run.py`, and a
`HANDOFF.md` telling you exactly how to run it.

## Before you start

- Install Claude Code and run `claude login` with your Pro/Max plan.
- Make sure `ANTHROPIC_API_KEY` is **not** set in your shell — if it is, the CLI
  bills per token instead of using your plan.
- Start in a fresh git repo / branch. The system edits files autonomously, so
  you want a clean branch you can review before merging.

## How to start

Put `START_HERE.md` in the repo, open it in Claude Code, and send this:

```
Read START_HERE.md. Don't build my project yet — follow it: talk with me to
understand what I'm making, play back what you understood, then assemble the
system. Let's begin.
```

From there, just talk it through and look over what it makes. You can refine
anything by discussing it ("did we cover X?", "have the roadmap revisit Y near
the end") — you shouldn't need to edit the plan by hand. When `HANDOFF.md`
appears, you're ready to run.

## What running it looks like

`HANDOFF.md` gives you the exact command. It loops one focused `claude -p`
session at a time, each doing a single roadmap step in a fresh context, with hard
caps on iterations and spend so the worst case is "it stopped and left a note,"
never a runaway overnight loop. It runs on your subscription, so a long build
makes the most of the plan you're already paying for.

## One honest note

A self-assembling system is powerful but harder to debug than a fixed script.
Treat the first run as a supervised dry run: watch an iteration or two, read the
`bus/log/`, confirm the caps and the hand-off behave — then trust it overnight.
