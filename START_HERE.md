# START HERE — bootstrap for a self-assembling overnight builder

You (Claude Code) are reading the only file in an otherwise empty repository.
This file is a **seed**, not a spec. Your job in this first conversation is to
talk with the human, then assemble a small autonomous system that will build
*their* project (e.g. a university exam-prep study site) phase by phase,
overnight, on its own. After you assemble it, the human runs one command and
walks away.

Do **not** start building their actual project. Build the *machine* that builds
it, and the plan that drives the machine.

---

## 0. The end state we are assembling

When you are done with this bootstrap conversation, the repo should contain:

1. `CLAUDE.md` — project conventions every future session reads automatically.
2. `ROADMAP.md` — a phased plan that is a **floor, not a ceiling** (see §3).
3. `.claude/agents/*.md` — the agent roles (see §4).
4. `bus/` — the file-based message board the agents coordinate through (§5).
5. `run.py` — the runner that drives `claude -p` sessions in a loop (§6).
6. `HANDOFF.md` — written last: what you built, what the human should check,
   and the single command to start the overnight run (§7).

Everything in the repo is fair game for every agent to read and edit. The
freedom is the point — agents are not sandboxed away from each other's work.

---

## 1. How the human uses this (tell them this up front)

> Open this repo in Claude Code and say: "Read START_HERE.md and let's set this
> up." Answer my questions, look over what I produce, and tweak by talking to me
> ("did we cover X?", "make the roadmap revisit the formula sheet at the end").
> You never have to edit the plan yourself unless you want to. When HANDOFF.md
> appears, you're ready to run it.

---

## 2. Your job, in order

1. **Interview first (§8).** Do not assume the project. Ask, then confirm a
   one-paragraph understanding back to the human before writing anything.
2. **Assemble the artifacts** in §0, grounded in their answers.
3. **Walk the human through it** and revise on their feedback, conversationally.
4. **Write HANDOFF.md** with the run command and a checklist.

---

## 3. Principles the assembled system MUST embody

These are hard-won; encode them into `CLAUDE.md`, `ROADMAP.md`, and the agent
prompts so every future session inherits them.

- **One task per session.** Each `claude -p` run does ONE roadmap step in a
  fresh context. Batching many steps into one session measurably lowers quality
  because attention spreads thin. Fresh context each time keeps it sharp; the
  state lives in files, not in conversation memory.
- **Roadmap is a floor, not a ceiling.** A session is allowed to: do more than
  one step if they're tightly coupled, split a step that turned out too big,
  add new steps it discovers are needed, reorder what's left, or **revisit and
  redo an earlier step** if reviewing later work exposed a problem. The roadmap
  is a living document the agents maintain, not a fixed track.
- **Each session leaves the repo in a coherent state** (builds/lints if
  possible) so the next amnesiac session can pick up safely.
- **Ground everything in source material.** Never invent course content. A
  confidently-wrong fact is worse than an admitted gap — this will be a paid
  product, so correctness beats coverage.
- **Separate building from reviewing.** The session that built a page must not
  be the one that signs off on it; a fresh reviewer catches what the builder
  rationalised.
- **Stop safely, never spin.** On ambiguity, exhausted budget, or repeated
  non-progress, halt and leave a clear message for the human rather than looping.

---

## 4. Agent roles to define (`.claude/agents/`)

Define these as Claude Code subagents. Tune the exact set with the human, but
the spine is:

- **programmer** — carries the human's intent and taste. Its system prompt is
  built from the interview: what they're making, who it's for, their quality
  bar, their preferences and red lines. The runner consults this agent when a
  *project decision* comes up ("should the formula sheet be its own page or
  inline?") so choices match what the human would have said, without waking them.
- **builder** — does one roadmap step: writes/edits the actual project files,
  grounded in source material, then leaves the repo coherent.
- **reviewer** — read-only. Checks the last step for correctness against source
  material and for build/render problems; files issues to the bus, never edits.
- **planner** — owns `ROADMAP.md`. Adds/splits/reorders/reopens steps per §3
  based on what the others report.

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

## 6. The runner (`run.py`)

Build a Python loop that drives the CLI the human already pays for. Requirements:

- Each iteration runs **one** `claude -p "<prompt>" --output-format json
  --permission-mode acceptEdits` session, in the repo, on a dedicated git branch.
- The prompt points the session at `bus/state.json` and its inbox, and tells it
  to do the next roadmap step per §3.
- Each session ends its reply with a fenced ` ```control ` block:
  `{"status": "continue|done|blocked", "next_prompt": "...", "summary": "...",
  "reason": "..."}`. The runner parses it, carries `next_prompt` to the next
  fresh session (the runner is the memory across the context boundary), and
  stops on `done`, `blocked`, a repeated/empty next_prompt (stall), or a cap.
- **Caps, always:** `MAX_ITERATIONS` and `MAX_BUDGET_USD`, with running cost
  printed each loop. Worst case must be "it stopped and left a note," never "it
  ran all night in circles."
- When a session's control block signals a *project decision* is needed, the
  runner invokes the **programmer** agent for an answer and feeds it back, rather
  than waking the human.

---

## 7. HANDOFF.md (write this last)

- One paragraph: what got assembled.
- A checklist for the human: skim `ROADMAP.md`, the agent prompts, `CLAUDE.md`.
- The single command to start the overnight run, and how to stop it.
- A note that they can keep discussing with you to refine any of it before running.

---

## 8. The interview (ask before building anything)

Ask these, then confirm understanding back before you write a line:

1. What exactly are we building? (e.g. a study site for which course/university;
   point me at the source material — slides, past papers — and where it lives.)
2. Who is it for and what does "good" look like to you? Your quality bar, your
   non-negotiables, your taste (tone, depth, visual style)?
3. What should the **programmer** agent decide on your behalf, and what must
   always come back to you?
4. Tech stack and repo for the output (e.g. an existing Next.js project to clone
   in, or start fresh)?
5. How hands-off do you want it — fully autonomous within caps, or pause at
   phase boundaries for your okay?

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
- **Source material is the lecturer's.** Fine for personal/educational use; if
  this becomes a paid product, prefer original explanations + the human's own
  past-paper analysis over reproducing lecture content, and flag that in HANDOFF.
- **For others, not just you:** if the human ever points this at running on
  behalf of paying customers, that crosses into territory their plan doesn't
  cover — surface it, don't silently do it.

---

*Begin by greeting the human and starting the interview in §8. Build the machine,
not the project.*
