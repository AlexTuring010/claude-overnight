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
│  ├─ ROADMAP.md                # the arc's plan (floor, not ceiling — §3)
│  ├─ run.py                    # the runner (§6)
│  ├─ orientation.md            # the briefing CLAUDE.md points sessions to (§1)
│  ├─ manifest.json             # every path the system created (§10 teardown)
│  └─ bus/                      # agent coordination (§5)
│     ├─ state.json
│     ├─ inbox/
│     └─ log/
├─ .claude/
│  ├─ agents/overnight-*.md     # the agent roles (§4) — prefixed, tracked
│  └─ commands/overnight-*.md   # the human's slash commands (§1) — prefixed, tracked
├─ CLAUDE.md                    # gets a small marked block, or created if absent
├─ HANDOFF.md                   # written last (§7)
└─ <the human's real work>      # committed on a dedicated branch
```

Paths in this file are written relative to the project root. Everything the
system owns sits under `.overnight/` (so `ROADMAP.md`, `run.py`, `bus/` mean
`.overnight/ROADMAP.md`, etc.) except the agent and command files, which Claude
Code requires under `.claude/`. The human runs the loop with
`python .overnight/run.py`.

Integration rules — these are what keep it clean (do all of them):

1. **Keep the apparatus separable from the human's work.** The default is to
   gitignore it (`.overnight/`, the `overnight-` files under `.claude/`,
   `HANDOFF.md`) so the project's commits only ever contain real work — but this
   is a **setup decision to raise with the human** (§8), not an assumption.
   Gitignoring keeps git clean but the working state lives only on this machine;
   committing the apparatus survives across machines and versions the run history
   at the cost of some git noise. Ask, then do what they choose. Either way, the
   work the agents produce is committed on a dedicated branch — that's the
   deliverable, never gitignored.
2. **Everything the system writes inside the project is namespaced** (`.overnight/`
   or an `overnight-` prefix) and listed in `.overnight/manifest.json`.
3. **`CLAUDE.md` is the one shared file you may touch** (sessions auto-read it).
   If it doesn't exist, create it (handled per the git choice above) containing
   only the orientation pointer to `.overnight/orientation.md`. If it already
   exists, append exactly one block delimited by `<!-- overnight:start -->` and
   `<!-- overnight:end -->` holding only that pointer — so teardown can remove
   precisely that block and leave the human's content untouched.
4. **Archives live outside the project** (§10), so they never enter its git.

Within the working area, every agent may read and edit freely — they are not
sandboxed from each other. The discipline is only at the boundary: the system's
own apparatus stays namespaced and tracked (and gitignored unless the human chose
to commit it), so when the arc ends the project is exactly as it was, plus the
work, minus the scaffolding.

---

## 1. How the human communicates with it (two channels)

There is no persistent "it" holding the conversation in its head — every session
is amnesiac. Communication works because **state lives in files and every session
reads them on startup** (via `CLAUDE.md`). There are two distinct channels; make
both clear to the human:

- **To run it (headless):** `python .overnight/run.py` — the autonomous overnight
  loop. The human doesn't chat with this; they start it and walk away.
- **To talk to it (interactive):** open `claude` in the repo and just talk. Because
  `CLAUDE.md` orients every session, a fresh interactive session already knows the
  arc, the roadmap, and the current state before the human says anything. So
  "did we cover X?", "bias toward more detail", or "we're done" all just work —
  the session reads the files and acts.

Give the human a small set of **slash commands** (`.claude/commands/overnight-*.md`)
as the unambiguous control surface, e.g.:

- `/overnight-status` — read the roadmap + state and summarise progress, cost, and
  what's left.
- `/overnight-adjust` — capture a change the human wants (record it to the bus /
  roadmap) so the next run picks it up.
- `/overnight-wrap-up` — the human is done: run teardown — archive the
  retrospective, then remove the apparatus (§10).

Tell the human these exist and that plain talking works too — the commands are
just the reliable version. Adapt the names to taste.

> Quick start for the human: "Read START_HERE.md and let's set this up." Talk
> through the arc, look over what I produce, refine by talking. When HANDOFF.md
> appears you're ready. Run with `python .overnight/run.py`; talk to it any time
> by opening `claude` here; finish with `/overnight-wrap-up`.

---

## 2. Your job, in order

0. **Get your bearings.** If the directory isn't empty, survey it first — read
   the README, manifests, structure, recent git history — and form an honest
   picture of the existing project before proposing anything. Also check the
   archive (§10): if a similar arc was done before, it's reference material.
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
3. **Assemble the apparatus** (§0), contained and tracked in `manifest.json`.
4. **Walk the human through it** and revise on their feedback, conversationally.
5. **Write HANDOFF.md** with the run command and a checklist.

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
  project you are a guest. Follow the concrete layout and integration rules in §0
  exactly: everything the system owns lives in a gitignored `.overnight/` (plus
  `overnight-`-prefixed, gitignored files under `.claude/`), every created path is
  recorded in `manifest.json`, and `CLAUDE.md` is touched only via one delimited
  block. This is what lets the system clear itself later (§10) without disturbing
  anything that was here before, or the work it produced.
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
- **The two channels, spelled out:** run it with `python .overnight/run.py`; talk
  to it any time by opening `claude` in this repo (it orients itself from `CLAUDE.md`).
- **The commands:** `/overnight-status`, `/overnight-adjust`, `/overnight-wrap-up`
  — what each does, and that plain talking works too.
- How to stop a run, and how to finish the arc (`/overnight-wrap-up`).
- A note that they can keep discussing with you to refine anything before running.

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

## 10. Lifecycle: run nightly, then archive and clear

This apparatus is meant to live only as long as the arc.

**Re-running.** Running `run.py` again resumes the arc — the roadmap cursor and
state persist in the bus, so a second night picks up where the first stopped.
Between runs the human can adjust: edit `ROADMAP.md`, drop notes in `bus/inbox/`,
or just talk to you ("last night's output was too terse — bias toward more
detail"). Re-runs absorb those adjustments. The human reviews each night's branch
diffs before trusting the next run.

**Teardown — when the human says the arc is done** (via `/overnight-wrap-up`, or
just telling an interactive session "we're finished, clear yourself" — it knows
what they mean because `CLAUDE.md` oriented it on startup), do this in order:

1. **Archive a retrospective.** Write to the archive: what the arc was, the
   roadmap as it actually ended up (not just as planned), key decisions, what
   worked, and — if the human tells you — what didn't. Include the final agent
   prompts. This is the arc's memory.
2. **Leave the work.** The actual output (the branch / merged changes) stays —
   that's the deliverable. Teardown removes scaffolding, not results.
3. **Remove the apparatus** using `manifest.json`: delete exactly the paths the
   system created (`.overnight/`, the `overnight-*` files under `.claude/`), remove
   the delimited block from `CLAUDE.md` (or the whole file if the system created
   it), and drop any `.gitignore` lines it added. Delete the working branch only
   if the human has merged it. The project is left exactly as it was, plus the
   work, minus every trace of the scaffolding.

**The archive — concrete location.** Default to **outside** the repo so it never
enters any project's git and is reusable across projects:
`~/.overnight/archive/<project-name>/<arc-name>/`, holding `retrospective.md`,
the final `roadmap.md`, and the final agent prompts. Confirm the path with the
human at setup (a dedicated git repo is a fine alternative if they want it backed
up and versioned). One folder per arc.

**Reuse across arcs.** When the human later says "remember that arc we did? let's
do something similar, but this time…", read the relevant archived retrospective
first and treat it as reference — repeat what worked, avoid what they flagged as
not working — then set up the new arc informed by it. The archive is how the
system improves over time instead of starting from zero each time. Surfacing
"last time you said X didn't work, so I'll do Y instead" is exactly the behaviour
to aim for.

---

*Begin by welcoming the human as a newcomer (§2 step 1): explain plainly what this
is and how you'll work together, set honest expectations, then ease into the
conversation in §8. Build the machine, not the project.*
