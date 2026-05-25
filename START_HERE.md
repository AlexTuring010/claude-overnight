# START HERE — bootstrap for a self-assembling overnight builder

You (Claude Code) are reading START_HERE.md, dropped into a repository. The
directory may be **empty** (a fresh project) or an **existing project** you
should work inside. This file is a **seed**, not a spec. Your job in this first
conversation is to understand the situation, agree on a bounded goal, then
assemble a small autonomous system that pursues that goal phase by phase,
overnight, on its own — and that cleans up after itself once the goal is met.

Think in **arcs**: one bounded objective at a time — a new project, or a single
big task inside an existing one (a feature, a migration, a large refactor, a
content push). You set the system up for *this* arc, the human runs it overnight,
reviews the results, adjusts, and re-runs the next night until the arc is done.
Then it archives what it learned and clears its own scaffolding back out of the
project (§10).

The work could be anything: code, data, prose, research. Do not assume a domain.
Do **not** start doing the work yourself. Build the *machine* that does it, scoped
to this arc.

---

## 0. The end state we are assembling

When you finish this bootstrap conversation, the apparatus should exist —
**contained so it can be cleanly removed later** (§3). Keep it under one
namespaced home (e.g. `.overnight/`) wherever the tool doesn't force a location:

1. `CLAUDE.md` (or a pointer appended to an existing one) — conventions every
   future session reads automatically.
2. `ROADMAP.md` — the arc's phased plan, a **floor, not a ceiling** (see §3).
3. `.claude/agents/overnight-*.md` — the agent roles, clearly named (see §4).
4. `bus/` — the file-based message board the agents coordinate through (§5).
5. `run.py` — the runner that drives `claude -p` sessions in a loop (§6).
6. `manifest.json` — every file/dir the system created, so teardown removes
   exactly its own scaffolding and nothing of the human's (§10).
7. `HANDOFF.md` — written last: what you built, what the human should check, and
   the single command to start the overnight run (§7).

Within the working area, everything is fair game for every agent to read and
edit — agents are not sandboxed from each other's work. But the system's own
apparatus stays namespaced and tracked, so the project it lives in is left exactly
as it was (plus the work produced, minus the scaffolding) when the arc ends.

---

## 1. How the human uses this (tell them this up front)

> Open this repo in Claude Code and say: "Read START_HERE.md and let's set this
> up." Talk with me about what you're building, look over what I produce, and
> refine by talking ("did we cover X?", "make the roadmap revisit Y at the
> end"). You never have to edit the plan yourself unless you want to. When
> HANDOFF.md appears, you're ready to run.

---

## 2. Your job, in order

0. **Get your bearings.** If the directory isn't empty, survey it first — read
   the README, manifests, structure, recent git history — and form an honest
   picture of the existing project before proposing anything. Also check the
   archive (§10): if a similar arc was done before, it's reference material.
1. **Understand the arc (§8)** — through conversation, not a form. Don't write
   anything until you genuinely understand the goal and have played your
   understanding back to the human.
2. **Assemble the apparatus** (§0), contained and tracked in `manifest.json`.
3. **Walk the human through it** and revise on their feedback, conversationally.
4. **Write HANDOFF.md** with the run command and a checklist.

---

## 3. Principles the assembled system MUST embody

These are hard-won; encode them into `CLAUDE.md`, `ROADMAP.md`, and the agent
prompts so every future session inherits them.

- **Intelligence lives in the agents, not the runner.** Keep `run.py` as dumb as
  the project allows. It runs a session, reads the decision the session handed
  back, and mechanically does that — call this agent, poll that source, loop,
  enforce caps. Judgment calls ("fix this now or make it a roadmap task?", "what
  do we work on next?", "which agent should handle this?") belong to agents and
  are expressed through the control protocol, never hardcoded as Python logic.
  When in doubt about where a decision goes: if it needs judgment, an agent makes
  it.
- **Contain the apparatus; never clobber the human's work.** In an existing
  project you are a guest. Keep everything the system adds under one namespaced
  home (`.overnight/` for roadmap, bus, runner, logs; clearly-named
  `.claude/agents/overnight-*.md` for agents), and record every file you create
  in `manifest.json`. Never overwrite an existing `CLAUDE.md` — append a short
  pointer instead. Do the actual work on a dedicated git branch. This separation
  is what lets the system clear itself later (§10) without disturbing anything
  that was here before, or the work it produced.
- **One task per session.** Each `claude -p` run does ONE roadmap step in a
  fresh context. Batching many steps into one session measurably lowers quality
  because attention spreads thin. Fresh context each time keeps it sharp; the
  state lives in files, not in conversation memory.
- **Roadmap is a floor, not a ceiling.** A session is allowed to: do more than
  one step if they're tightly coupled, split a step that turned out too big,
  add new steps it discovers are needed, reorder what's left, or **revisit and
  redo an earlier step** if reviewing later work exposed a problem. The roadmap
  is a living document the agents maintain, not a fixed track.
- **Each session leaves the repo in a coherent state** (builds/lints/tests if
  applicable) so the next amnesiac session can pick up safely.
- **Stay grounded in the project's source of truth** — its spec, requirements,
  and any material the human provided. Don't fabricate. An admitted gap is
  better than a confident error; wrong output erodes trust faster than missing
  output does.
- **Separate doing from reviewing.** The session that produced something must
  not be the one that signs off on it; a fresh reviewer catches what the maker
  rationalised.
- **Stop safely, never spin.** On ambiguity, exhausted budget, or repeated
  non-progress, halt and leave a clear message for the human rather than looping.

---

## 4. Agent roles to define (`.claude/agents/`)

Define these as Claude Code subagents. Tune the exact set with the human — some
projects want more, some fewer — but the spine is:

- **principal** — carries the human's intent and taste. Its system prompt is
  built from your conversation: what they're making, who it's for, their quality
  bar, their preferences and red lines. The runner consults this agent when a
  *project decision* comes up so choices match what the human would have said,
  without waking them. (Name it for the human's role if that fits better, e.g.
  "the founder", "the author", "the lead".)
- **builder** — does one roadmap step: produces/edits the actual project files,
  grounded in the source of truth, then leaves the repo coherent.
- **reviewer** — read-only. Checks the last step against the project's
  requirements and source of truth, and for build/correctness problems; files
  issues to the bus, never edits.
- **planner** — owns `ROADMAP.md`. Adds/splits/reorders/reopens steps per §3
  based on what the others report.
- **orchestrator** *(add for complex projects)* — decides what to work on next
  and which agent should do it. Useful when the roadmap has several parallel
  tracks rather than one line: the runner asks the orchestrator "what now?" and
  it picks the track, the step, and the agent. For a simple linear project you
  don't need this — the next step is just the next unchecked item.

Scale the set to the project: a small job might only need builder + reviewer; a
large, high-quality one benefits from all of the above. Decide with the human.

Give every agent permission to read the whole repo. Builder may edit; reviewer
is read-only; planner edits only the roadmap and the bus.

---

## 5. The message bus (`bus/`)

The agents are amnesiac across sessions, so they coordinate through files. Keep
it dead simple and human-readable:

- `bus/inbox/` — append-only messages, one markdown file per message, named
  `NNN-from-<agent>-to-<agent>.md`, each with a short header (from, to, topic,
  timestamp) and a body.
- `bus/state.json` — the small shared state the runner reads between sessions:
  current roadmap cursor, last status, cumulative cost, open review items.
- `bus/log/` — one file per session: the prompt it got and the control block it
  returned, so any run is replayable and debuggable.

The runner's job is to tell each session, in its prompt, **where to look**:
"read bus/state.json and any unread messages in bus/inbox/ addressed to you
before you start." Agents leave replies the same way. That's the whole protocol
— no framework, just a convention every agent prompt describes.

---

## 6. The runner (`run.py`) — keep it thin

`run.py` is a dumb dispatcher, not a brain (see §3). It drives the CLI the human
already pays for and does only mechanical work; every real decision comes from an
agent via the control block.

The loop:

- Each iteration runs **one** `claude -p "<prompt>" --output-format json
  --permission-mode acceptEdits` session, in the repo, on a dedicated git branch.
- The prompt points the session at `bus/state.json` and its inbox and tells it
  what to do (per §3). Which session you run can itself be an agent's decision —
  e.g. ask the orchestrator "what next?" and run whatever it names.
- Each session ends its reply with a fenced ` ```control ` block. Make it
  **action-oriented**, so the agent tells the runner what to do next rather than
  the runner deciding. For example:
  `{"status": "continue|done|blocked", "action": "next_step|delegate|poll|consult_principal",
  "agent": "builder", "next_prompt": "...", "summary": "...", "reason": "..."}`.
  The runner just executes the named action — run that agent, poll that source,
  consult the principal — carries any `next_prompt` to the next fresh session
  (the runner is the memory across the context boundary), and loops.
- It stops only on `done`, `blocked`, a stall (repeated/empty next step), or a cap.
- **Caps, always:** `MAX_ITERATIONS` and `MAX_BUDGET_USD`, with running cost
  printed each loop. Worst case must be "it stopped and left a note," never "it
  ran all night in circles."

**External inputs / live feedback (optional).** A project may have a source of
new work the system should watch — a database of user comments, an issue
tracker, an inbox. If so, the runner can periodically check whether there's new
content (a quick read, or a small `claude` skill/command the project sets up with
its own API keys), and when there is, hand it to an agent to **triage**. The
agent decides — not Python — whether each item is a quick fix to do now or a new
roadmap task for the planner to schedule. Small things may get fixed on the spot;
larger ones become tracked steps. Python only ferries the items to the agent and
does what the agent's control block says.

---

## 7. HANDOFF.md (write this last)

- One paragraph: what got assembled.
- A checklist for the human: skim `ROADMAP.md`, the agent prompts, `CLAUDE.md`.
- The single command to start the overnight run, and how to stop it.
- A note that they can keep discussing with you to refine any of it before running.

---

## 8. Understand the project — through conversation, not a checklist

Do **not** fire a fixed list of questions and tick boxes. Have a real
conversation. Your goal is genuine understanding of what the human wants and how
they'd judge it, and *you* decide when you have it: ask follow-ups, dig into
vague answers, surface tradeoffs, propose options and react to their taste, let
them lead and follow the threads that matter to them. A short, unusual project
needs different questions than a large one — adapt.

You are ready to move on only when you could honestly answer, in the human's own
voice, things like:

- What are we building or doing, and what do "done" and "good" look like to them?
- **What is the bounded goal of this arc, and how will we know it's complete?**
  (This scopes the roadmap and tells the system when to stop and archive.)
- If this lives in an existing project: what's already there, what may the system
  touch, and what is off-limits?
- What is the source of truth the work must stay grounded in, and where is it?
- Which decisions should the principal agent make on their behalf, and which
  must always come back to them?
- Where does the output live — stack, repo, fresh or existing?
- How autonomous do they want it — full runs within caps, or pauses at checkpoints?

Treat that as the understanding you need, not a script to read aloud — and
expect to need more for any project that doesn't fit the mould. When you believe
you understand, play back a one-paragraph summary and let them correct it before
you build anything.

---

## 9. Hard constraints (bake these in; they protect the human)

- **Run on their subscription, not API billing.** The CLI uses their Claude plan
  as long as they're logged in (`claude login`) and `ANTHROPIC_API_KEY` is NOT
  set in the shell. Put this check in `run.py` and in HANDOFF.md.
- **Mind the limits.** Today this draws from interactive plan limits (the
  five-hour / weekly windows); from 15 June 2026 it draws from the separate
  monthly Agent SDK credit instead. Either way the budget/iteration caps keep it
  inside what they're paying for — "make the most of the plan," not blow past it.
- **Always run on a dedicated git branch** so auto-accept edits can't clobber
  anything; the human reviews diffs before merging.
- **Respect third-party rights.** If the project uses material, assets, data, or
  code the human doesn't own, make sure its licence or permission covers the
  intended use — especially if the output will be sold or published. Flag any
  uncertainty in HANDOFF rather than quietly assuming it's fine.
- **For others, not just them:** if the human ever points this at running on
  behalf of paying customers, that crosses into territory their plan may not
  cover — surface it, don't silently do it.

---

*Begin by greeting the human and getting into the conversation in §8. Build the
machine, not the project.*

---

## 10. Lifecycle: run nightly, then archive and clear

This apparatus is meant to live only as long as the arc.

**Re-running.** Running `run.py` again resumes the arc — the roadmap cursor and
state persist in the bus, so a second night picks up where the first stopped.
Between runs the human can adjust: edit `ROADMAP.md`, drop notes in `bus/inbox/`,
or just talk to you ("last night's output was too terse — bias toward more
detail"). Re-runs absorb those adjustments. The human reviews each night's branch
diffs before trusting the next run.

**Teardown — when the human says the arc is done** ("we're finished, clear
yourself"), do this in order:

1. **Archive a retrospective.** Write to the archive: what the arc was, the
   roadmap as it actually ended up (not just as planned), key decisions, what
   worked, and — if the human tells you — what didn't. Include the final agent
   prompts. This is the arc's memory.
2. **Leave the work.** The actual output (the branch / merged changes) stays —
   that's the deliverable. Teardown removes scaffolding, not results.
3. **Remove the apparatus** using `manifest.json`: exactly the files the system
   created, nothing else. Delete the working branch only if the human has merged
   it. The project is left as it was, plus the work, minus the scaffolding.

**The archive.** Choose a location with the human, ideally *outside* this repo so
it survives across arcs and projects (e.g. `~/overnight-archive/<arc-name>/`, or a
small dedicated repo). One folder per arc, each holding its retrospective.

**Reuse across arcs.** When the human later says "remember that arc we did? let's
do something similar, but this time…", read the relevant archived retrospective
first and treat it as reference — repeat what worked, avoid what they flagged as
not working — then set up the new arc informed by it. The archive is how the
system improves over time instead of starting from zero each time. Surfacing
"last time you said X didn't work, so I'll do Y instead" is exactly the behaviour
to aim for.
