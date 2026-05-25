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

When you finish this bootstrap conversation, the apparatus exists as a **specific,
self-contained structure** — not scattered files. This exact layout is the system;
follow it so it stays walled off from the human's project and removes cleanly.

```
<project>/
├─ .overnight/                  # everything the system owns (gitignored by default — §8)
│  ├─ orientation.md            # install-level: the briefing a session loads to enter context (§1)
│  ├─ run.py                    # install-level: the runner (§6)
│  ├─ manifest.json             # install-level: every path the system created (§10)
│  ├─ HANDOFF.md                # install-level: written last — what was built + how to run (§7)
│  ├─ ROADMAP.md                # arc-level: the current arc's plan (floor, not ceiling — §3)
│  └─ bus/                      # arc-level: agent coordination (§5)
│     ├─ state.json
│     ├─ inbox/
│     └─ log/
├─ .claude/
│  ├─ commands/overnight.md     # the ONE command that loads system context (§1)
│  └─ agents/overnight-*.md     # the agent roles (§4) — prefixed, tracked
└─ <the human's real work>      # on a dedicated branch
```

`CLAUDE.md` is **not** in this list — the system never touches it (see rule 3).
Note the split: **install-level** files persist across arcs (the runner, the
command, the agents, orientation); **arc-level** files (ROADMAP, bus) belong to
the current arc and are archived + reset when an arc completes (§10).

Paths in this file are written relative to the project root. Everything the
system owns sits under `.overnight/` (so `ROADMAP.md`, `run.py`, `bus/` mean
`.overnight/ROADMAP.md`, etc.) except the command and agent files, which Claude
Code requires under `.claude/`. The human runs the loop with
`python .overnight/run.py`.

Integration rules — these are what keep it clean (do all of them):

1. **Keep the apparatus separable from the human's work.** The default is to
   gitignore it (`.overnight/`, the `overnight-` files under `.claude/`) so the
   project's commits only ever contain real work — but this is a **setup decision
   to raise with the human** (§8), not an assumption. Gitignoring keeps git clean
   but the working state lives only on this machine; committing the apparatus
   survives across machines and versions the run history at the cost of some git
   noise. Ask, then do what they choose. Either way, the work the agents produce
   is committed on a dedicated branch — that's the deliverable, never gitignored.
2. **Everything the system writes inside the project is namespaced** (`.overnight/`
   or an `overnight-` prefix) and listed in `.overnight/manifest.json`.
3. **Never touch `CLAUDE.md`.** Orientation does not live there — it lives in
   `.overnight/orientation.md` and is loaded on demand: interactively by the
   `/overnight` command, and for headless runs by the runner injecting it into each
   session's prompt (§1). This keeps the human's own `CLAUDE.md` (if any) entirely
   theirs. The system stays dormant until summoned.
4. **Archives live outside the project** (§10), so they never enter its git.

Within the working area, every agent may read and edit freely — they are not
sandboxed from each other. The discipline is only at the boundary: the system's
own apparatus stays namespaced and tracked (and gitignored unless the human chose
to commit it), so when the arc ends the project is exactly as it was, plus the
work, minus the scaffolding.

---

## 1. How the human communicates with it

There is no persistent "it" holding the conversation in its head — every session
is amnesiac, and `CLAUDE.md` is left untouched, so the system stays **dormant
until summoned**. State lives in files; a session becomes aware of the system only
when it loads them. Two channels:

**To run it (headless):** `python .overnight/run.py` — the autonomous overnight
loop. The human doesn't chat with this; they start it and walk away. The runner
orients each `claude -p` session itself, by injecting `.overnight/orientation.md`
and the current state into the session's prompt — it does not rely on `CLAUDE.md`.

**To talk to it (interactive):** open `claude` in the repo and run the one
command — **`/overnight`**. That command's whole job is to load
`.overnight/orientation.md` and `bus/state.json` so the session enters "system
context": now it knows what the system is, whether an arc is active, and where it
stands. From that point on, **the human just talks** — there is no command zoo to
memorise. The session reads intent from plain language:

- "how's it going?" / "what's left?" → summarise progress, cost, remaining steps.
- "bias toward more detail" / "redo the X step" → record the adjustment so the
  next run picks it up (or do it now if small).
- "I think we're done with this" → confirm, then archive the arc (§10).

So `/overnight` is the single doorway into the system; conversation does
everything else. If `/overnight` loads and finds **no active arc** (a previous arc
was completed and archived), the session says so and waits — offering to look back
at past arcs or to set up a new one (§10). Tell the human only this: run
`/overnight` to talk to the system, then speak normally.

**Fallback if the command isn't there.** The slash command is just convenience —
all it does is read `.overnight/orientation.md`. If it was never created, got
removed, or isn't available in their editor, the human can summon the system with
a plain prompt instead:

> "Read `.overnight/orientation.md` and `.overnight/bus/state.json`, then act as
> the overnight system — tell me where the current arc stands, or that there's no
> active arc."

Put this exact fallback line in `orientation.md` and in HANDOFF.md, so it's
recorded somewhere the human can always find it. The command and the prompt do the
same thing; neither is load-bearing on its own.

> Quick start for the human: "Read START_HERE.md and let's set this up." I'll
> explain what this is, we'll scope the goal together, and I'll build it. After
> that: run it with `python .overnight/run.py`; talk to it any time with
> `/overnight` then plain conversation.

---

## 2. Your job, in order

0. **Get your bearings.** If the directory isn't empty, survey it first — read
   the README, manifests, structure, recent git history — and form an honest
   picture of the existing project before proposing anything. **If an `.overnight/`
   install already exists** (from previous arcs), do not clobber it: load its state,
   recognise this as a returning install, and jump to the §10 re-entry behaviour —
   resume an active arc, or if none is active, ask whether to review past arcs or
   set up a new one. Check the archive (§10) too: a similar past arc is reference
   material.
1. **Welcome and orient the human.** Assume they're new to this — they likely
   grabbed this file to try it out and have no idea what it sets up. Before any
   questions, explain in plain language, in your own words: what this is (a system
   that will work on one goal autonomously, overnight), what the two of you are
   about to do together (scope the goal, then you assemble the machine), roughly
   how it'll work afterward (it runs while they're away; they talk to it or run it;
   they can stop and finish it any time), and set honest expectations (it's
   powerful but the first run wants supervision; it uses their Claude plan within
   caps). Keep it short and human, check they're comfortable, invite questions —
   then ease into the conversation. Don't interrogate a stranger; bring them in.
2. **Understand the arc (§8)** — through conversation, not a form. Don't write
   anything until you genuinely understand the goal and have played your
   understanding back to the human.
3. **Assemble the apparatus** (§0): the install-level pieces (the `/overnight`
   command, `orientation.md`, the runner, the agents) and the arc-level pieces
   (ROADMAP, bus), contained and tracked in `manifest.json`.
4. **Walk the human through it** and revise on their feedback, conversationally.
5. **Write `.overnight/HANDOFF.md`** — what was built, the run command, the one
   `/overnight` command, and how to finish (§7).

---

## 3. Principles the assembled system MUST embody

These are hard-won; encode them into `.overnight/orientation.md`, `ROADMAP.md`,
and the agent prompts so every future session inherits them.

- **Intelligence lives in the agents, not the runner.** Keep `run.py` as dumb as
  the project allows. It runs a session, reads the decision the session handed
  back, and mechanically does that — call this agent, poll that source, loop,
  enforce caps. Judgment calls ("fix this now or make it a roadmap task?", "what
  do we work on next?", "which agent should handle this?") belong to agents and
  are expressed through the control protocol, never hardcoded as Python logic.
  When in doubt about where a decision goes: if it needs judgment, an agent makes
  it.
- **Contain the apparatus; never clobber the human's work.** In an existing
  project you are a guest. Follow the concrete layout and integration rules in §0
  exactly: everything the system owns lives under `.overnight/` (plus
  `overnight-`-prefixed files under `.claude/`), every created path is recorded in
  `manifest.json`, and `CLAUDE.md` is never touched (orientation loads on demand
  from `.overnight/orientation.md`, §1). This is what lets the system uninstall
  cleanly later (§10) without disturbing anything that was here before, or the
  work it produced.
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

## 7. HANDOFF.md (write this last, to `.overnight/HANDOFF.md`)

- One paragraph: what got assembled and what the arc's goal is.
- A checklist for the human: skim `.overnight/ROADMAP.md` and the agent prompts.
- **Run it:** `python .overnight/run.py` — the overnight loop; how to stop it.
- **Talk to it:** open `claude` here and run **`/overnight`** to enter system
  context, then just talk — ask how it's going, request changes, or say you're
  done. No other commands to remember.
- **If `/overnight` isn't available**, include the plain fallback prompt verbatim:
  "Read `.overnight/orientation.md` and `.overnight/bus/state.json`, then act as
  the overnight system — tell me where the current arc stands, or that there's no
  active arc."
- **Finishing:** tell it (in `/overnight` context) you're done with the arc; it
  archives and resets for a possible next arc. Mention that fully removing the
  system from the project is a separate step it can do on request (§10).
- A note that they can keep discussing to refine anything before the first run.

---

## 8. Understand the project — through conversation, not a checklist

By now you've welcomed and oriented the human (§2 step 1), so they know what
they're getting into. This is the next part of that same conversation, not a
separate questionnaire. Assume they may still be fuzzy on the system — explain
*why* you're asking something when it isn't obvious, and offer sensible defaults
they can simply accept rather than forcing every decision onto a newcomer.

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
- How should the apparatus relate to git — gitignored (default; clean history,
  state stays on this machine) or committed (survives across machines, versions
  the run history, some git noise)? And where should the archive live (default
  `~/.overnight/archive/<project>/<arc>/`, or a path/repo they prefer)?

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

## 10. Lifecycle: run, finish an arc, start the next, or uninstall

The install persists across arcs; each individual arc has a beginning and an end.

**Re-running an arc.** `python .overnight/run.py` again resumes the current arc —
the roadmap cursor and state persist in the bus, so a second night picks up where
the first stopped. Between runs the human adjusts by editing `ROADMAP.md`, dropping
notes in `bus/inbox/`, or just talking in `/overnight` context ("last night was too
terse — bias toward more detail"). Re-runs absorb those adjustments. The human
reviews each night's branch diffs before trusting the next run.

**Finishing an arc.** When the human says they're done — in `/overnight` context,
just "I think we're done with this" — confirm, then:

1. **Archive a retrospective.** Write to the archive: what the arc was, the roadmap
   as it actually ended up, key decisions, what worked, and — if they tell you —
   what didn't. Include the final agent prompts. This is the arc's memory.
2. **Leave the work.** The output (the branch / merged changes) stays — that's the
   deliverable.
3. **Reset the arc-level files** (`ROADMAP.md`, `bus/`) so the slate is clean, but
   **keep the install** (the `/overnight` command, runner, agents, orientation).
   The system is now idle, ready for a next arc.

**Re-entry (the next time).** Two equal paths, both fine:
- The human re-gives the kickoff prompt → you read START_HERE, find the existing
  `.overnight/` install (§2 step 0), recognise it as a returning install — this is
  normal, not a conflict — and proceed.
- The human just runs `/overnight` → it loads context, sees no active arc, says so,
  and waits. Offer to review past arcs (from the archive) or scope a new one. If
  they pick a new arc, run the same scoping conversation (§8) and assemble fresh
  arc-level files, reusing the archive as reference (below).

**Reuse across arcs.** When the human says "remember that arc we did? let's do
something similar, but this time…", read the relevant archived retrospective first
and treat it as reference — repeat what worked, avoid what they flagged. Surfacing
"last time you said X didn't work, so I'll do Y instead" is exactly the behaviour
to aim for. The archive is how the system improves instead of starting from zero.

**Uninstalling entirely** (separate from finishing an arc — only when the human
asks to remove the system from this project): use `manifest.json` to delete exactly
the paths the system created (`.overnight/`, the `overnight-*` files under
`.claude/`) and drop any `.gitignore` lines it added. `CLAUDE.md` needs no cleanup —
the system never touched it. Archives outside the repo remain (they're the memory);
mention them so the human can delete them too if they want. The project is left
exactly as it was, plus whatever work was produced.

**The archive — concrete location.** Default **outside** the repo so it never
enters any project's git and is reusable across projects:
`~/.overnight/archive/<project-name>/<arc-name>/`, holding `retrospective.md`, the
final `roadmap.md`, and the final agent prompts. Confirm the path at setup (a
dedicated git repo is a fine alternative if they want it versioned). One folder
per arc.

---

*Begin by welcoming the human as a newcomer (§2 step 1): explain plainly what this
is and how you'll work together, set honest expectations, then ease into the
conversation in §8. Build the machine, not the project.*
