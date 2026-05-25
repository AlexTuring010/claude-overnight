# START HERE — bootstrap for a self-assembling overnight builder

You (Claude Code) are reading START_HERE.md, dropped into a repository. The
directory may be **empty** (a fresh project) or an **existing project** you
should work inside. This file is a **seed**, not a spec. Your job in this first
conversation is to understand the situation, agree on a bounded goal, then
assemble a small autonomous system that pursues that goal step by step on one
or more parallel tracks, overnight, on its own — and that cleans up after
itself once the goal is met.

Think in **goals**: one bounded objective at a time — a new project, or a
single big undertaking inside an existing one (a feature, a migration, a large
refactor, a content push). You set the system up for *this* goal, the human
runs it overnight, reads the morning digest, adjusts by talking, re-runs the
next night until the goal is done. Then it archives what it learned and clears
its own scaffolding back out of the project (§11).

The work could be anything: code, data, prose, research. Do not assume a
domain. Do **not** start doing the work yourself. Build the *machine* that
does it, scoped to this goal.

**One overriding principle, above every rule below: do what is best for the
quality of the project. Never cut corners to make progress look good.** Budget
and iteration caps are *stop-and-leave-a-note* boundaries, never
*ship-what-we-have* boundaries. Worst acceptable outcome is "it halted with a
clear note." Never: "it shipped something shallow to make the night look
successful." Every other rule in this document — separation of duties, the
runner staying dumb, deterministic verification, the read-only morning digest —
is in service of this.

---

## 0. The end state we are assembling

When you finish this bootstrap conversation, the apparatus exists as a
**specific, self-contained structure** — not scattered files. This exact
layout is the system; follow it so it stays walled off from the human's
project and removes cleanly.

```
<project>/
├─ .overnight/                  # everything the system owns (gitignored by default — §9)
│  ├─ orientation.md            # install-level: the briefing a session loads to enter context (§1)
│  ├─ run.py                    # install-level: the runner (§6)
│  ├─ manifest.json             # install-level: every path the system created (§11)
│  ├─ HANDOFF.md                # install-level: written last — what was built + how to run (§8)
│  ├─ wakeup.html               # goal-level: morning digest, regenerated each iteration (§7)
│  ├─ ROADMAP.md                # goal-level: the current goal's plan, organised by track (§3, §4)
│  └─ bus/                      # goal-level: agent coordination (§5)
│     ├─ state.json             # multi-track state — per-track cursors, scopes, branches
│     ├─ inbox/                 # append-only agent-to-agent messages
│     ├─ log/                   # one file per session: prompt + control block
│     ├─ decisions/             # one file per principal-agent call (§5)
│     └─ intent-corrections.md  # human corrections fed back to the principal (§5)
├─ .claude/
│  ├─ commands/overnight.md     # the ONE command that loads system context (§1)
│  └─ agents/overnight-*.md     # the agent roles (§4) — prefixed, tracked
└─ <the human's real work>      # on a dedicated branch per track, plus an integration branch
```

`CLAUDE.md` is **not** in this list — the system never touches it (see rule 3).
Note the split: **install-level** files persist across goals (the runner, the
command, the agents, orientation, manifest); **goal-level** files (ROADMAP,
bus, wakeup.html) belong to the current goal and are archived + reset when a
goal completes (§11).

Paths in this file are written relative to the project root. Everything the
system owns sits under `.overnight/` (so `ROADMAP.md`, `run.py`, `bus/`,
`wakeup.html` mean `.overnight/ROADMAP.md`, etc.) except the command and agent
files, which Claude Code requires under `.claude/`. The human runs the loop
with `python .overnight/run.py`; reads progress each morning at
`.overnight/wakeup.html`.

Integration rules — these are what keep it clean (do all of them):

1. **Keep the apparatus separable from the human's work.** The default is to
   gitignore it (`.overnight/`, the `overnight-` files under `.claude/`) so
   the project's commits only ever contain real work — but this is a **setup
   decision to raise with the human** (§9), not an assumption. Gitignoring
   keeps git clean but the working state lives only on this machine;
   committing the apparatus survives across machines and versions the run
   history at the cost of some git noise. Ask, then do what they choose.
   Either way, the work the agents produce is committed on dedicated branches
   (one per track, plus an integration branch) — that's the deliverable,
   never gitignored.
2. **Everything the system writes inside the project is namespaced**
   (`.overnight/` or an `overnight-` prefix) and listed in
   `.overnight/manifest.json`.
3. **Never touch `CLAUDE.md`.** Orientation does not live there — it lives in
   `.overnight/orientation.md` and is loaded on demand: interactively by the
   `/overnight` command, and for headless runs by the runner injecting it
   into each session's prompt (§1). This keeps the human's own `CLAUDE.md`
   (if any) entirely theirs. The system stays dormant until summoned.
4. **Archives live outside the project** (§11), so they never enter its git.

Within the working area, every agent may **read** freely. **Write scopes are
partitioned by track** when there is more than one track (§3, §4) — agents
are not sandboxed from each other's existence, but their writes are bounded
so parallel tracks don't race on the same files. The discipline at the
project boundary is unchanged: the system's own apparatus stays namespaced
and tracked (and gitignored unless the human chose to commit it), so when
the goal ends the project is exactly as it was, plus the work, minus the
scaffolding.

---

## 1. How the human communicates with it

There is no persistent "it" holding the conversation in its head — every
session is amnesiac, and `CLAUDE.md` is left untouched, so the system stays
**dormant until summoned**. State lives in files; a session becomes aware of
the system only when it loads them. Three channels — two conversational, one
a read-only artifact:

**To run it (headless):** `python .overnight/run.py` — the autonomous
overnight loop. The human doesn't chat with this; they start it and walk
away. The runner orients each `claude -p` session itself, by injecting
`.overnight/orientation.md` and the current state into the session's prompt
— it does not rely on `CLAUDE.md`. To **stop it cleanly** while it's
running: `touch .overnight/STOP` (from any terminal) or Ctrl-C in the
runner's terminal — both finish the in-flight iteration and exit. To
**check state** after an unexpected exit (laptop died, terminal closed,
power blip): `python .overnight/run.py --inspect` runs a read-only
diagnostic and tells the human what's done, what's partial, and what
the recovery plan is — see §6.

**To read what happened (asynchronous, in the morning):** open
`.overnight/wakeup.html` in a browser. It's a self-contained read-only
digest the runner regenerates every iteration that closes work — per-track
status, steps closed last night, principal-agent decisions, blockers,
cost-per-step (§7). There is no approve button; the human reads, doesn't
intervene mid-run.

**To talk to it (interactive):** open `claude` in the repo and run the one
command — **`/overnight`**. That command's whole job is to load
`.overnight/orientation.md` and `bus/state.json` so the session enters
"system context": now it knows what the system is, whether a goal is
active, and where it stands. From that point on, **the human just talks** —
there is no command zoo to memorise. The session reads intent from plain
language:

- "how's it going?" / "what's left?" → summarise progress, cost, remaining
  steps (and which tracks are active).
- "bias toward more detail" / "redo the X step" → record the adjustment so
  the next run picks it up (or do it now if small).
- "the principal made the wrong call on Y" → append to
  `bus/intent-corrections.md` so the principal reads it next invocation
  (§5).
- "split content from infra into separate tracks" → tell the planner so it
  can restructure the roadmap.
- "what state are we in?" / "diagnose" → run the inspect flow (§6) and
  report back: what's done, what's partial, what's orphaned, recovery plan.
- "stop after this iteration" → create `.overnight/STOP` so the running
  loop exits cleanly at the next iteration boundary.
- "I think we're done with this" → confirm, then archive the goal (§11).

So `/overnight` is the single doorway into the conversational system;
`wakeup.html` is the read-only morning artifact. If `/overnight` loads and
finds **no active goal** (a previous goal was completed and archived), the
session says so and waits — offering to look back at past goals or to set
up a new one (§11). Tell the human only this: run `/overnight` to talk to
the system, open `wakeup.html` to read what happened.

**Fallback if the command isn't there.** The slash command is just
convenience — all it does is read `.overnight/orientation.md`. If it was
never created, got removed, or isn't available in their editor, the human
can summon the system with a plain prompt instead:

> "Read `.overnight/orientation.md` and `.overnight/bus/state.json`, then
> act as the overnight system — tell me where the current goal stands, or
> that there's no active goal."

Put this exact fallback line in `orientation.md` and in HANDOFF.md, so it's
recorded somewhere the human can always find it. The command and the
prompt do the same thing; neither is load-bearing on its own.

> Quick start for the human: "Read START_HERE.md and let's set this up."
> I'll explain what this is, we'll scope the goal together, and I'll build
> it. After that: run it with `python .overnight/run.py`; read each
> morning at `.overnight/wakeup.html`; talk to it any time with
> `/overnight` then plain conversation.

---

## 2. Your job, in order

0. **Get your bearings.** If the directory isn't empty, survey it first —
   read the README, manifests, structure, recent git history — and form an
   honest picture of the existing project before proposing anything. **If
   an `.overnight/` install already exists** (from previous goals), do not
   clobber it: load its state, recognise this as a returning install, and
   jump to the §11 re-entry behaviour — resume an active goal, or if none
   is active, ask whether to review past goals or set up a new one. Check
   the archive (§11) too: a similar past goal is reference material.

1. **Welcome and orient the human.** Assume they're new to this — they
   likely grabbed this file to try it out and have no idea what it sets
   up. Before any questions, explain in plain language, in your own
   words: what this is (a system that will work on one goal
   autonomously, overnight, on one or more parallel tracks), what the
   two of you are about to do together (scope the goal, then you
   assemble the machine), roughly how it'll work afterward (it runs
   while they're away; they read `wakeup.html` in the morning; they
   talk to it or run it; they can stop and finish it any time), and set
   honest expectations (it's powerful but the first run wants
   supervision; it uses their Claude plan within caps; **it will hold
   the line on quality over speed**). Keep it short and human, check
   they're comfortable, invite questions — then ease into the
   conversation. Don't interrogate a stranger; bring them in.

2. **Understand the goal (§9)** — through conversation, not a form.
   Don't write anything until you genuinely understand the goal and
   have played your understanding back to the human.

3. **Assemble the apparatus** (§0): the install-level pieces (the
   `/overnight` command, `orientation.md`, the runner, the agents) and
   the goal-level pieces (ROADMAP organised by track, the bus including
   the decisions log and the corrections feedback file, `wakeup.html`),
   contained and tracked in `manifest.json`.

4. **Walk the human through it** and revise on their feedback,
   conversationally.

5. **Write `.overnight/HANDOFF.md`** — what was built, the run command,
   the `wakeup.html` path, the `/overnight` command, and how to finish
   (§8).

---

## 3. Principles the assembled system MUST embody

These are hard-won; encode them into `.overnight/orientation.md`,
`ROADMAP.md`, and the agent prompts so every future session inherits them.

- **Quality first, always.** This is the principle above all others. Do
  what is best for the project. Never mark a step `done` just to make
  progress look good. If a step needs another night to be done right,
  the planner says so, the runner halts cleanly, and the morning note
  explains. Budget and iteration caps stop the run; they never lower
  the bar.

- **No human-in-the-loop overnight.** The human reads in the morning
  (`wakeup.html`); nothing waits on their approval mid-run. Anything
  that would need them is recorded as a block in the wakeup digest and
  — if possible — another track keeps moving. Conversation with the
  human happens before the run starts and after it ends via
  `/overnight`, never during.

- **Intelligence lives in the agents, not the runner.** Keep `run.py`
  as dumb as the project allows. It runs a session, reads the decision
  the session handed back, and mechanically does that — call this
  agent, poll that source, loop, enforce caps, regenerate the digest.
  Judgment calls ("fix this now or make it a roadmap task?", "what do
  we work on next?", "should we fan out into a second track?", "which
  agent should handle this?") belong to agents and are expressed
  through the control protocol, never hardcoded as Python logic. When
  in doubt about where a decision goes: if it needs judgment, an agent
  makes it.

- **Contain the apparatus; never clobber the human's work.** In an
  existing project you are a guest. Follow the concrete layout and
  integration rules in §0 exactly: everything the system owns lives
  under `.overnight/` (plus `overnight-`-prefixed files under
  `.claude/`), every created path is recorded in `manifest.json`, and
  `CLAUDE.md` is never touched (orientation loads on demand from
  `.overnight/orientation.md`, §1). This is what lets the system
  uninstall cleanly later (§11) without disturbing anything that was
  here before, or the work it produced.

- **One task per session.** Each `claude -p` run does ONE roadmap step
  in a fresh context. Batching many steps into one session measurably
  lowers quality because attention spreads thin. Fresh context each
  time keeps it sharp; the state lives in files, not in conversation
  memory.

- **Roadmap is a floor, not a ceiling.** A session is allowed to: do
  more than one step if they're tightly coupled, split a step that
  turned out too big, add new steps it discovers are needed, reorder
  what's left, or **revisit and redo an earlier step** if reviewing
  later work exposed a problem. The roadmap is a living document the
  planner maintains, not a fixed track.

- **One goal can have many tracks; the planner schedules them.** The
  roadmap is organised into one or more *tracks* — workstreams that
  can advance independently (e.g. infra, content, design). Default is
  one track; the planner opens more only when **all three** hold:
  (a) the tracks operate on disjoint write paths in the filesystem,
  (b) running them concurrently doesn't risk integration bugs that
      wouldn't surface in either track alone, and
  (c) it actually improves quality or unblocks a goal that would
      otherwise stall — **never parallelize for speed alone.**
  The planner can also discover tracks mid-goal and spawn them then.
  It owns the roadmap *and* schedules what runs next at every
  iteration boundary; the runner asks "what's next?" and executes what
  comes back.

- **File scope per track.** When parallel tracks exist, the planner
  partitions the filesystem: each track has a declared write scope
  (paths it owns). Other tracks may read but not write into a track's
  scope. Anything that needs to cross scopes serializes through the
  planner. Each track lives on its own branch; the planner schedules
  merge points into an integration branch, and the reviewer runs an
  integration pass after each merge.

- **Deterministic verification gate.** A step is `done` only if the
  project's declared checks pass — build / tests / lint / typecheck /
  render, elicited in the interview (§9). The reviewer runs them
  after every builder `done`; failure routes the step back to the
  planner as `blocked`, never silently accepted. For pure-content
  projects, the gate is "the artifact builds without errors AND a
  fresh reviewer session finds no factual gaps against cited source
  material."

- **Each session leaves its track in a coherent state**
  (builds/lints/tests if applicable) so the next amnesiac session can
  pick up safely.

- **Stay grounded in the project's source of truth** — its spec,
  requirements, and any material the human provided. Don't fabricate.
  An admitted gap is better than a confident error; wrong output erodes
  trust faster than missing output does. Builder output cites where
  each substantive claim came from, so the reviewer can verify against
  source rather than vibes-check.

- **Separate doing from reviewing.** The session that produced
  something must not be the one that signs off on it; a fresh reviewer
  catches what the maker rationalised.

- **Stop safely, never spin.** On ambiguity, exhausted budget, repeated
  non-progress, or a step revisited more than `MAX_STEP_VISITS` (~3)
  times across sessions, halt and leave a clear message in
  `wakeup.html` rather than looping.

---

## 4. Agent roles to define (`.claude/agents/`)

Define these as Claude Code subagents. Tune the exact set with the human —
some projects want more, some fewer — but the spine is:

- **principal** — carries the human's intent and taste. Its system prompt
  is built from your conversation: what they're making, who it's for,
  their quality bar, their preferences and red lines. The runner consults
  this agent when a *project decision* comes up so choices match what the
  human would have said, without waking them. **Every invocation writes a
  structured entry to `bus/decisions/`** (question, decision, rationale,
  track, timestamp) — these surface in `wakeup.html` for the human to
  scan and flag wrong ones. **Before each call the principal reads
  `bus/intent-corrections.md`** so morning feedback shapes tonight's
  judgement; this is how the loop closes without ever waking the human.
  (Name the agent for the human's role if that fits better, e.g. "the
  founder", "the author", "the lead".)

- **builder** — does one roadmap step on one track: produces/edits the
  actual project files within that track's declared write scope, grounded
  in the source of truth and citing where claims came from, then leaves
  its track coherent. Cannot write outside its scope.

- **reviewer** — read-only on project files; **runs the project's
  declared checks** (build/test/lint/typecheck/render) as part of
  verification. Checks the last step against the project's requirements
  and source of truth, and for build/correctness problems; files results
  back through the bus so the planner can re-schedule (revisit, split,
  escalate). After a track merge, runs an integration pass to catch
  problems that wouldn't show in either track alone. Never edits project
  files; writes only inside `bus/`.

- **planner** — owns `ROADMAP.md` *and* schedules work. Adds, splits,
  reorders, reopens steps per §3 based on what the others report.
  Assigns each step to a track. Decides when to fan out into parallel
  tracks and when to join them. **Produces the per-iteration work order
  the runner executes**: for each track either `{track, step_id, prompt,
  scope}` (advance), `idle` (nothing to advance right now), or `done`.
  The runner asks the planner what's next at every iteration boundary
  and runs whatever it says.

The planner subsumes what some projects might split into a separate
"orchestrator" — having one agent own both the roadmap and the schedule
keeps decisions coherent. For very simple linear goals the planner
trivially schedules one track of one step at a time; the same machinery
scales up to many tracks.

**Tool permissions — encode these as Claude Code subagent restrictions,
not just intent.** The restriction is what makes the separation real;
without it "reviewer is read-only" is just a wish.

- builder: read everything; write only within its track's declared write
  scope; may run build/test commands to self-check before declaring
  `done`.
- reviewer: read everything; execute the project's declared checks;
  write only inside `bus/`.
- planner: read everything; write to `ROADMAP.md`, `bus/state.json`, and
  `bus/` messages.
- principal: read everything; write only inside `bus/decisions/`.

Scale the set to the project: a small job may live with builder +
reviewer + planner + principal; a large, high-quality one may add
domain-specific helpers. Decide with the human.

---

## 5. The message bus (`bus/`)

The agents are amnesiac across sessions, so they coordinate through files.
Keep it dead simple and human-readable.

- **`bus/state.json`** — the shared state the runner and planner read
  between sessions. Multi-track from day one (single-track goals just have
  one entry under `tracks`):
  ```json
  {
    "goal": "<short goal name>",
    "started_at": "<ISO timestamp>",
    "tracks": {
      "T1": {
        "name": "infra",
        "scope": ["app/server/**", "infra/**"],
        "branch": "overnight/T1-infra",
        "cursor": "step-3",
        "status": "active",
        "step_visits": {"step-3": 1},
        "blockers": []
      }
    },
    "integration_branch": "overnight/integration",
    "cumulative_cost_usd": 12.34,
    "cumulative_iterations": 47,
    "last_status": "<one-line summary>"
  }
  ```

- **`bus/inbox/`** — append-only messages between agents, one markdown
  file per message, `NNN-from-<agent>-to-<agent>.md`, with a short header
  (from, to, topic, track, timestamp) and a body.

- **`bus/log/`** — one file per session: the prompt it got, the control
  block it returned, which track it was on. Any iteration is replayable
  from here.

- **`bus/decisions/`** — one file per principal-agent call: the question
  asked, the decision made, the rationale, the track, the timestamp. The
  wakeup digest surfaces every entry from the last run so the human can
  scan in seconds and flag wrong ones.

- **`bus/intent-corrections.md`** — the human writes here (via the
  `/overnight` conversation, or by hand) when they disagree with a
  principal decision. The principal agent reads this file at the start
  of every call and lets it shape today's judgement. This is how morning
  feedback closes the loop into tonight's autonomy without ever requiring
  the human to be in the loop mid-run.

The runner tells each session in its prompt **where to look** — read
`bus/state.json` for your track's cursor and any unread inbox messages
addressed to you before you start. Agents leave replies the same way.
That's the whole protocol — no framework, just a convention every agent
prompt describes.

---

## 6. The runner (`run.py`) — keep it thin

`run.py` is a dumb dispatcher, not a brain (see §3). It drives the CLI the
human already pays for and does only mechanical work; every real decision
comes from an agent via the control block.

**Per-iteration protocol:**

1. **Consult the planner.** Invoke the planner agent with `bus/state.json`
   and the latest control blocks from each track. It returns a per-track
   work order: for each track either `{track, step_id, prompt, scope}`
   (a step to advance), `idle` (nothing to advance right now), or `done`.

2. **Spawn sessions.** For each track with an advance, spawn one
   `claude -p "<prompt>" --output-format json --permission-mode acceptEdits`
   session on that track's branch. When the planner returns work for N > 1
   tracks, spawn them concurrently (subprocess pool). Default is N = 1; the
   planner only goes higher when it judges parallel safe under §3.

3. **Collect control blocks.** Each session ends its reply with a fenced
   ` ```control ` block. Make it **action-oriented**, so the agent tells
   the runner what to do next rather than the runner deciding. For
   example:
   ```json
   {
     "track": "T1",
     "step_id": "step-3",
     "status": "done|blocked|partial|needs-decision|continue",
     "action": "next_step|delegate|poll|consult_principal|merge_track",
     "agent": "builder",
     "checks_passed": true,
     "summary": "...",
     "next_hint": "optional hint for the planner",
     "reason": "..."
   }
   ```
   Append each to `bus/log/`. Update `bus/state.json` (cursors, visit
   counters, blockers).

4. **Verification gate.** For each builder `done`, the runner invokes the
   **reviewer** to run the project's declared checks. If they fail, flip
   the step's status to `blocked` before the next iteration. A step is
   never `done` without a green gate.

5. **Project decisions.** For each `needs-decision`, the runner invokes
   the **principal** with the question; logs the answer to
   `bus/decisions/`; feeds the decision back into the next iteration's
   prompt.

6. **Track merges.** When the planner schedules a merge, the runner
   merges that track's branch into the integration branch and invokes
   the reviewer for an integration pass before continuing.

7. **Regenerate the wakeup digest** (§7) so a human opening it mid-night
   still sees current state.

8. **Re-consult the planner.** Loop.

**The runner does NOT decide the next prompt itself** — the planner does.
The runner is plumbing.

**Caps, always (`run.py` enforces):**
- `MAX_ITERATIONS` (global)
- `MAX_BUDGET_USD` (global; per-track distribution tracked and shown to
  the planner so it can rebalance)
- `MAX_STEP_VISITS` (~3 per step) — the loop-detection guard from §3;
  halt and note in `wakeup.html` if any single step is touched more than
  this many times across sessions
- Running cost printed each iteration, per-track distribution included

**Subscription billing guard.** Before starting, verify
`ANTHROPIC_API_KEY` is **not** set in the environment and `claude` is
logged in. Refuse to start otherwise.

### Stopping, diagnosing, recovering

The runner is the long-lived process the human starts and walks away
from. The principle: **the human should never be stuck wondering what
happened.** Three real situations get explicit handling.

**1. Graceful stop while it's running** (the human wakes up, the runner
is mid-loop, they need to leave). The runner polls a flag file between
iterations:

- `touch .overnight/STOP` from any terminal → finish the current
  iteration cleanly (let in-flight sessions complete and write their
  control blocks), regenerate `wakeup.html`, then exit. Remove the
  flag file on clean exit so the next start isn't blocked.
- **SIGINT (Ctrl-C) and SIGTERM** in the runner's terminal trigger the
  same graceful stop. Two SIGINTs within 5 seconds escalate to
  immediate kill — the human's escape hatch if graceful is hung.
- `touch .overnight/KILL` → immediate stop; terminate in-flight
  sessions; mark their steps `partial` so the next run can recover.
- The runner prints clearly what's happening: "received stop signal,
  finishing iteration N on tracks T1, T2, will exit cleanly..."
- The human can also tell `/overnight` "stop after this iteration" and
  the session creates `.overnight/STOP` on their behalf.

**2. Inspect mode — "I have no idea what state we're in"** (laptop died
overnight, terminal crashed, power blip, ambiguous wakeup). The human
needs to know what's definitely done and what may be half-finished
*before* committing to another run.

`python .overnight/run.py --inspect` runs a one-shot **read-only**
diagnostic. It does not spawn builder or planner sessions and does not
mutate the project. It:

- Reads `bus/state.json` and the last entries in `bus/log/`.
- Checks each track's git branch for uncommitted changes and for
  commits added since the last logged session (orphan work).
- Detects orphan sessions — control blocks expected but missing from
  `bus/log/`.
- Verifies the integration branch state.
- Invokes a fresh **reviewer** session in audit mode (no builds, no
  edits) that produces a structured diagnostic:
  - what is definitively `done` and check-passed,
  - what is `partial` (started but not check-passed or not finalized),
  - what is `orphaned` (mutation on disk with no corresponding control
    block),
  - what the recommended recovery action is per track (re-run step X,
    discard branch Y, hand back to the planner, etc.).
- Writes the result into `bus/diagnostic.md` and into a refreshed
  `wakeup.html` so the human can read it either way.

The same diagnostic is available conversationally: ask `/overnight`
"what state are we in?" or "diagnose" and the session runs the
inspection and writes the same files. Two paths, one result.

**3. Crash recovery on the next normal run.** Every
`python .overnight/run.py` invocation begins with the same orphan-detection
logic — silently, automatically. If a previous run died mid-iteration
without writing its control block, the next invocation marks the
affected step `partial` and lets the planner decide whether to retry,
split, or shelve. Never silently re-run a step that may have partially
mutated the repo. The runner also detects when the previous shutdown
wasn't clean (no `STOP` flag removal recorded) and prints a one-line
warning so the human knows. `--inspect` is for when the human wants
visibility *before* committing to another run; the silent path handles
the common case where they just want to keep going.

**4. Rate-limit handling.** The Claude plan has a rolling 5-hour usage
window and a weekly cap (and, from 15 June 2026, a separate monthly
Agent SDK credit pool — §10). If a `claude -p` call returns a
rate-limit error, the runner parses the reset time from the response
and decides:

- If the reset is within `RATE_LIMIT_WAIT_THRESHOLD_HOURS` (default 6,
  comfortably covering any 5-hour rolling window plus buffer), enter
  **wait mode**: pause spawning new sessions, let in-flight ones
  finish if they can, regenerate `wakeup.html` with a clear "waiting
  for rate-limit reset, resuming at HH:MM" banner, and sleep until
  reset plus a small buffer. Then resume normally — the next iteration
  picks up where things stopped, no work lost.
- If the reset is beyond the threshold (typically a weekly cap hit
  with reset days away), halt with a clear note in `wakeup.html`.
  Waiting days idle is wasteful; the human restarts `run.py` when the
  window opens.
- During wait mode the runner still polls `STOP`/`KILL` and honours
  SIGINT/SIGTERM — the human can abandon the wait any time.
- Every rate-limit event (hit and resumption) is logged to `bus/log/`
  with the reset timestamp so it's traceable.

This is not about dodging the plan — caps exist for a reason. It's that
"ran out an hour before reset" should not waste a whole night; the
runner can wait and continue cleanly.

**External inputs / live feedback (optional).** A project may have a
source of new work the system should watch — a database of user
comments, an issue tracker, an inbox. If so, the runner can periodically
check whether there's new content (a quick read, or a small `claude`
skill/command the project sets up with its own API keys), and when there
is, hand it to an agent to **triage**. The agent decides — not Python —
whether each item is a quick fix to do now or a new roadmap task for the
planner to schedule. Small things may get fixed on the spot; larger ones
become tracked steps. Python only ferries items to the agent and does
what the agent's control block says.

Worst case must be "it stopped and left a note in `wakeup.html`." Never
"it ran all night in circles." Never "it shipped something shallow."

---

## 7. The wakeup digest (`.overnight/wakeup.html`)

A self-contained HTML page the runner regenerates at every iteration that
closes work. The human opens it in a browser each morning. It is
**read-only** — there is no approve button, no control surface. The
human's responses go through the `/overnight` conversation or by writing
to `bus/intent-corrections.md`, not here.

**Requirements:**

- **Self-contained.** Inline CSS only; no external scripts or CDN
  dependencies. Must open offline. Don't reach for a JS framework —
  plain HTML + a little inline SVG for the cost chart is enough.
- **Always at the same path** (`.overnight/wakeup.html`) so the human's
  bookmark and muscle memory work.
- **Scannable in 60 seconds.** A human knows the state of their project
  from this single file, without ever reading raw logs or running the
  CLI.

**Sections, in order:**

1. **Header.** Goal name; date range covered by this digest; total cost
   and duration; caps remaining (iterations, USD); overall status.
2. **Per-track status cards.** One card per active or recently-completed
   track: name, scope, % complete (steps done / total), current status
   (active / blocked / done), current blocker if any.
3. **Steps closed this run, grouped by track.** Each row: step ID,
   one-line summary, link to its diff (a local `bus/log/...` file),
   link to reviewer notes, check-pass status (green/red).
4. **Principal-agent decisions this run.** Every entry from
   `bus/decisions/` added since the last digest: question asked,
   decision made, rationale, track, timestamp. The user scans in
   seconds; flagging a wrong decision means writing into
   `bus/intent-corrections.md` (usually via `/overnight`).
5. **Open review items / blocks / stalls.** Anything that didn't close
   cleanly, with a one-line "what the planner intends to try next."
6. **Cost-per-step bars.** Simple inline-SVG bar chart. Surfaces runaway
   steps before they eat tomorrow night.

The digest is the human's morning interface to the system. Treat it as
a first-class artifact: if it's ugly or unscannable, the human will
give up on reading it and the loop closes badly.

---

## 8. HANDOFF.md (write this last, to `.overnight/HANDOFF.md`)

- One paragraph: what got assembled, what the goal is, which tracks were
  identified, what deterministic checks were declared.
- A checklist for the human: skim `.overnight/ROADMAP.md` (and its
  per-track organisation), the agent prompts, the declared check
  commands.
- **Run it:** `python .overnight/run.py` — the overnight loop.
- **Stop it cleanly:** `touch .overnight/STOP` from any terminal, or
  Ctrl-C in the runner's terminal, or tell `/overnight` "stop after
  this iteration." All three finish the in-flight iteration, write the
  morning digest, then exit. (`touch .overnight/KILL` or two Ctrl-Cs
  within 5 seconds force-stop if graceful is hung — partial steps will
  recover on the next run.)
- **Diagnose state if unsure:** `python .overnight/run.py --inspect`
  for a read-only diagnostic (what's done, what's partial, recovery
  plan) — same thing conversationally with `/overnight` saying "what
  state are we in?" Useful when the laptop died overnight, the
  terminal crashed, or any other "I have no idea where things stand."
- **Read it:** `.overnight/wakeup.html` — open each morning; what to
  look for; how to flag wrong principal decisions (write to
  `bus/intent-corrections.md`, usually via `/overnight`).
- **Talk to it:** open `claude` here and run **`/overnight`** to enter
  system context, then just talk — ask how it's going, request changes,
  flag corrections, stop the run, diagnose state, or say you're done.
  No other commands to remember.
- **If `/overnight` isn't available**, include the plain fallback prompt
  verbatim:
  > "Read `.overnight/orientation.md` and `.overnight/bus/state.json`,
  > then act as the overnight system — tell me where the current goal
  > stands, or that there's no active goal."
- **Finishing:** tell it (in `/overnight` context) you're done with the
  goal; it archives and resets for a possible next goal. Mention that
  fully removing the system from the project is a separate step it can
  do on request (§11).
- A note that they can keep discussing to refine anything before the
  first run.

---

## 9. Understand the project — through conversation, not a checklist

By now you've welcomed and oriented the human (§2 step 1), so they know
what they're getting into. This is the next part of that same
conversation, not a separate questionnaire. Assume they may still be
fuzzy on the system — explain *why* you're asking something when it
isn't obvious, and offer sensible defaults they can simply accept rather
than forcing every decision onto a newcomer.

Do **not** fire a fixed list of questions and tick boxes. Have a real
conversation. Your goal is genuine understanding of what the human wants
and how they'd judge it, and *you* decide when you have it: ask
follow-ups, dig into vague answers, surface tradeoffs, propose options
and react to their taste, let them lead and follow the threads that
matter to them. A short, unusual project needs different questions than
a large one — adapt.

You are ready to move on only when you could honestly answer, in the
human's own voice, things like:

- What are we building or doing, and what do "done" and "good" look like
  to them?
- **What is the bounded goal, and how will we know it's complete?**
  (This scopes the roadmap and tells the system when to stop and
  archive.)
- If this lives in an existing project: what's already there, what may
  the system touch, and what is off-limits?
- What is the source of truth the work must stay grounded in, and where
  is it?
- **What deterministic checks define "done" for a step?** Build, test,
  lint, typecheck, link-check, the static-site renderer — whatever
  proves correctness without a human reading the result. For pure-content
  projects: the render/lint command plus a fresh reviewer pass against
  cited source. This becomes the verification gate (§3, §6); without
  declared checks the gate degenerates to "the reviewer thinks it's
  fine," which is exactly the silent-failure mode the gate exists to
  prevent.
- **Are there natural tracks the goal splits into** (e.g. infra /
  content / design), and do they want parallel advancement where safe?
  Parallel burns through the 5-hour plan window faster — N concurrent
  sessions ≈ N× the rate. Speed-vs-thrift is a real trade. The planner
  can discover tracks mid-goal too; this just sets the starting hint.
- Which decisions should the principal agent make on their behalf, and
  which must always come back to them? (Every principal call is logged
  to `bus/decisions/` and surfaced in `wakeup.html` either way — see
  §5, §7.)
- Where does the output live — stack, repo, fresh or existing? How
  should per-track branches and the integration branch be named?
- How autonomous do they want it — full runs within caps (default), or
  is there a specific event they'd want paged for rather than logged?
  (Default per §3 is no human-in-the-loop mid-run.)
- How should the apparatus relate to git — gitignored (default; clean
  history, state stays on this machine) or committed (survives across
  machines, versions the run history, some git noise)? And where should
  the archive live (default `~/.overnight/archive/<project>/<goal>/`, a
  synced folder for cross-machine, or a separate git repo if they want
  it versioned)?

Treat that as the understanding you need, not a script to read aloud —
and expect to need more for any project that doesn't fit the mould. When
you believe you understand, play back a one-paragraph summary and let
them correct it before you build anything.

---

## 10. Hard constraints (bake these in; they protect the human)

- **Run on their subscription, not API billing.** The CLI uses their
  Claude plan as long as they're logged in (`claude login`) and
  `ANTHROPIC_API_KEY` is NOT set in the shell. Put this check in
  `run.py` and in HANDOFF.md.
- **Mind the limits.** Today this draws from interactive plan limits
  (the rolling five-hour / weekly windows); from 15 June 2026 it draws
  from the separate monthly Agent SDK credit instead. Parallel tracks
  multiply window burn proportionally — N concurrent sessions ≈ N× the
  rate. The runner detects rate-limit errors, parses the reset time
  from them, and **waits through rolling 5-hour resets** before
  resuming (§6) — better than wasting a night because the limit was
  hit an hour before reset. It halts with a note only when the wait
  would be excessive (typically a weekly cap with reset days away).
  The planner accounts for window burn when scheduling. "Make the most
  of the plan," not blow past it.
- **One git branch per track, plus an integration branch.** Auto-accept
  edits can't clobber anything outside the overnight branches. The
  human reviews the integration branch before merging into their own
  main.
- **Respect third-party rights.** If the project uses material, assets,
  data, or code the human doesn't own, make sure its licence or
  permission covers the intended use — especially if the output will be
  sold or published. Flag any uncertainty in HANDOFF rather than
  quietly assuming it's fine.
- **For others, not just them:** if the human ever points this at
  running on behalf of paying customers, that crosses into territory
  their plan may not cover — surface it, don't silently do it.

---

## 11. Lifecycle: run, finish a goal, start the next, or uninstall

The install persists across goals; each individual goal has a beginning
and an end.

**Re-running a goal.** `python .overnight/run.py` again resumes the
current goal — the per-track cursors and state persist in the bus, so a
second night picks up where the first stopped. Between runs the human
adjusts by editing `ROADMAP.md`, dropping notes in `bus/inbox/`, writing
to `bus/intent-corrections.md`, or just talking in `/overnight` context
("last night was too terse — bias toward more detail"). Re-runs absorb
those adjustments. The human reads `wakeup.html` each morning before
trusting the next run.

**Finishing a goal.** When the human says they're done — in
`/overnight` context, just "I think we're done with this" — confirm,
then:

1. **Archive a retrospective.** Write to the archive: what the goal
   was, the roadmap as it actually ended up (with track structure), key
   decisions, what worked, and — if they tell you — what didn't.
   Include the final agent prompts. This is the goal's memory.
2. **Leave the work.** The output (the integration branch / merged
   changes) stays — that's the deliverable.
3. **Reset the goal-level files** (`ROADMAP.md`, `bus/`, `wakeup.html`)
   so the slate is clean, but **keep the install** (the `/overnight`
   command, runner, agents, orientation). The system is now idle, ready
   for a next goal.

**Re-entry (the next time).** Two equal paths, both fine:
- The human re-gives the kickoff prompt → you read START_HERE, find the
  existing `.overnight/` install (§2 step 0), recognise it as a
  returning install — this is normal, not a conflict — and proceed.
- The human just runs `/overnight` → it loads context, sees no active
  goal, says so, and waits. Offer to review past goals (from the
  archive) or scope a new one. If they pick a new goal, run the same
  scoping conversation (§9) and assemble fresh goal-level files,
  reusing the archive as reference (below).

**Reuse across goals.** When the human says "remember that goal we did?
let's do something similar, but this time…", read the relevant archived
retrospective first and treat it as reference — repeat what worked,
avoid what they flagged. Surfacing "last time you said X didn't work,
so I'll do Y instead" is exactly the behaviour to aim for. The archive
is how the system improves instead of starting from zero.

**Uninstalling entirely** (separate from finishing a goal — only when
the human asks to remove the system from this project): use
`manifest.json` to delete exactly the paths the system created
(`.overnight/`, the `overnight-*` files under `.claude/`) and drop any
`.gitignore` lines it added. `CLAUDE.md` needs no cleanup — the system
never touched it. Archives outside the repo remain (they're the
memory); mention them so the human can delete them too if they want.
The project is left exactly as it was, plus whatever work was produced.

**The archive — concrete location.** Default **outside** the repo so it
never enters any project's git and is reusable across projects:
`~/.overnight/archive/<project-name>/<goal-name>/`, holding
`retrospective.md`, the final `roadmap.md`, and the final agent
prompts. Confirm the path at setup — a Dropbox / iCloud / OneDrive
folder or a dedicated git repo are fine alternatives if they want sync
across machines or versioning. One folder per goal.

---

*Begin by welcoming the human as a newcomer (§2 step 1): explain plainly
what this is and how you'll work together, set honest expectations, then
ease into the conversation in §9. Build the machine, not the project.
Hold the line on quality.*
