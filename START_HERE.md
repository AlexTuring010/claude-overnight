# START HERE — bootstrap for a self-assembling overnight builder

You (Claude Code) are reading START_HERE.md, dropped into a repository. The
directory may be **empty** (a fresh project) or an **existing project** you
should work inside. This file is a **seed**, not a spec. Across **three short
sessions** separated by `/clear` (§2 explains why), the work is to understand
the situation, agree on a bounded goal, then assemble a small autonomous
system that pursues that goal step by step on one or more parallel tracks,
overnight, on its own — and that cleans up after itself once the goal is met.
**Your job in this first session** is just the front of that: scope the goal
through conversation and write `.overnight/INTENT.md`. Sessions two and three
build the apparatus in clean context — don't try to do their work here.

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
runner staying dumb, the verification gate (deterministic where possible,
honestly-soft where not — §3), the read-only morning digest — is in service
of this.

---

## 0. The end state we are assembling

When the bootstrap finishes (across Phases A → B → C in §2), the apparatus
exists as a **specific, self-contained structure** — not scattered files.
This exact layout is the system; follow it so it stays walled off from the
human's project and removes cleanly.

```
<project>/
├─ .overnight/                  # everything the system owns (gitignored by default — §9)
│  ├─ orientation.md            # install-level: the briefing a session loads to enter context (§1)
│  ├─ run.py                    # install-level: the runner (§6)
│  ├─ manifest.json             # install-level: every path the system created (§11)
│  ├─ HANDOFF.md                # install-level: written last — what was built + how to run (§8)
│  ├─ NEXT.md                   # bootstrap-transient: kickoff line for the next /clear handoff; deleted after Phase C (§2)
│  ├─ INTENT.md                 # goal-level: captured human intent for this goal (§2 Phase A)
│  ├─ BUILD_PLAN.md             # goal-level: system-construction roadmap (§2 Phase B)
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
                                # — when >1 track runs concurrently, each branch is checked
                                # out into its own git worktree under `.overnight/worktrees/`
                                # (see §3, §6 step 2)
```

`CLAUDE.md` is **not** in this list — the system never touches it (see rule 3).
Note the split: **install-level** files persist across goals (the runner, the
command, the agents, orientation, manifest); **goal-level** files (INTENT,
BUILD_PLAN, ROADMAP, bus, wakeup.html) belong to the current goal and are
archived + reset when a goal completes (§11); **bootstrap-transient** files
(`NEXT.md`) bridge the `/clear` handoffs during setup and are deleted when
Phase C finishes (§2).

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
> I'll explain what this is, we'll scope the goal together across **2-3
> short sessions** (a `/clear` between each, for the same reason the
> overnight runtime uses fresh context per step — I'll explain when
> we get there), then the system gets built. After that: run it with
> `python .overnight/run.py`; read each morning at
> `.overnight/wakeup.html`; talk to it any time with `/overnight` then
> plain conversation.

---

## 2. Your job, in order — three phases, two `/clear` boundaries

Setup happens across **three sessions, not one**. This is deliberate:
the overnight runtime enforces fresh context per step (§3) because
polluted context produces measurably worse work. The same is true of
the setup that *builds* the runtime. A single ever-filling session
would assemble the agent prompts, orientation, and roadmap with the
whole noisy conversation loaded — and every overnight night thereafter
would inherit that drift. Bootstrap is upstream of every night; its
quality multiplies.

So split: long conversation in Phase A (fine, that's where nuance
lives), then `/clear`, focused planning in Phase B, then `/clear`,
then the build in Phase C using fresh subagents per artifact. Two
`/clear` boundaries, three phases. **Explain this to the human upfront
in Phase A** — same principle that makes overnight work, applied to
setup — so the `/clear` when it comes is expected, not jarring.

Each `/clear` boundary has a kickoff line for the next session.
Because `/clear` erases exactly the message that contains it, the
handing-off session also **writes the kickoff to
`.overnight/NEXT.md`** before signing off, and tells the human
where to find it. State lives in files, not in conversation
memory — that includes the single most load-bearing instruction at
every transition. A newcomer who clears before copying, or who
closes the terminal between sessions, recovers by opening
`NEXT.md`.

### Phase A — Intent (this session, conversational)

0. **Get your bearings.** If the directory isn't empty, survey it first
   — read the README, manifests, structure, recent git history — and
   form an honest picture of the existing project before proposing
   anything. **If an `.overnight/` install already exists** (from
   previous goals), do not clobber it: load its state, recognise this
   as a returning install, and jump to the §11 re-entry behaviour —
   resume an active goal, or if none is active, ask whether to review
   past goals or set up a new one. Check the archive (§11) too: a
   similar past goal is reference material. **If `.overnight/NEXT.md`
   exists**, a previous bootstrap was interrupted between phases —
   read it, point the human at the right next session and the line to
   paste, and stop. Don't try to resume Phase B or Phase C work in
   this Phase A session; the whole point of the `/clear` is that the
   resume happens in a fresh one.

1. **Welcome and orient the human.** Assume they're new to this — they
   likely grabbed this file to try it out and have no idea what it
   sets up. Before any questions, explain in plain language, in your
   own words: what this is (a system that will work on one goal
   autonomously, overnight, on one or more parallel tracks), what the
   two of you are about to do together (**scope the goal in this
   session, then `/clear` and build the system across two more short
   sessions — say this now, briefly explain why: same fresh-context
   principle the overnight runtime uses**), roughly how it'll work
   afterward (it runs while they're away; they read `wakeup.html` in
   the morning; they talk to it or run it; they can stop and finish
   it any time), and set honest expectations (it's powerful but the
   first run wants supervision; it uses their Claude plan within
   caps; **it will hold the line on quality over speed**). Keep it
   short and human, check they're comfortable, invite questions —
   then ease into the conversation. Don't interrogate a stranger;
   bring them in.

2. **Understand the goal (§9)** — through conversation, not a form.
   Don't write anything until you genuinely understand the goal and
   have played your understanding back to the human.

3. **Write `.overnight/INTENT.md`** — the single artifact Phases B
   and C will read. Capture everything downstream needs in dense,
   reviewable form: goal; what "done" and "good" look like; source
   of truth; declared verification checks; builder self-check skills
   (§9); track structure hint; principal profile (taste, red lines,
   what they can decide on the human's behalf); git/archive choices;
   existing-project constraints. Play it back to the human; let them
   correct it. **Quality of INTENT.md matters more than quality of
   the conversation that produced it** — it's the bridge into clean
   context.

4. **Hand off to Phase B.** First, write the Phase B kickoff to
   `.overnight/NEXT.md` so it survives the `/clear`. The file's
   content is just the instruction the human will paste, with a
   short header so it reads clearly if opened directly:

   ```
   # Next: Phase B (build plan)

   `/clear` this session, then paste the line below as the first
   prompt in the fresh session:

   Read .overnight/INTENT.md and START_HERE.md (all of it — §2
   Phase B for your task, the rest for context to brief each
   artifact well), then produce .overnight/BUILD_PLAN.md and stop
   for review.
   ```

   Then tell the human:

   > "Intent captured in `.overnight/INTENT.md`. The kickoff for
   > the next session is also written to `.overnight/NEXT.md` — if
   > you `/clear` before copying the line below, or close the
   > terminal between now and the next session, just open that
   > file to recover it. Now `/clear` this session and paste this
   > in the fresh session — that one plans the build in clean
   > context, same principle that makes overnight work:
   >
   > `Read .overnight/INTENT.md and START_HERE.md (all of it — §2
   > Phase B for your task, the rest for context to brief each
   > artifact well), then produce .overnight/BUILD_PLAN.md and
   > stop for review.`"

   Stop here. Do not begin Phase B in this session.

### Phase B — Build plan (fresh session after `/clear`, short)

The human pastes the kickoff line above. You enter this phase with no
memory of Phase A — that's the point. Read `.overnight/INTENT.md` and
all of START_HERE.md (the `/clear` removed Phase A's conversation,
not your need for the spec — §2 Phase B is the task, but §1, §3, §4,
§5, §6 are what you need to brief each artifact well), then:

1. **Read INTENT.md carefully.** If anything is missing, contradictory,
   or unclear, write the gaps to `.overnight/INTENT_QUESTIONS.md` and
   stop — do not barrel through. The human will reload Phase A briefly
   to answer; better one extra session than a build founded on a
   guess.

2. **Produce `.overnight/BUILD_PLAN.md`** — the system-construction
   roadmap. One step per significant artifact: `orientation.md`, each
   agent prompt (principal, builder, reviewer, planner — and any
   domain-specific agents the goal needs), `run.py`, `ROADMAP.md` (the
   *goal's* roadmap, not this one), the `/overnight` command,
   `manifest.json`, `HANDOFF.md`. Each step has a short brief, the
   inputs to read (INTENT.md plus any already-built artifacts that
   this step depends on), the output path, and `status: pending`.
   Order steps so each can be built reading only earlier outputs.

3. **Play back the build plan.** Let the human reorder, drop, or add
   steps before Phase C builds anything.

4. **Hand off to Phase C.** First, overwrite `.overnight/NEXT.md`
   with the Phase C kickoff so the next session can recover it if
   the chat buffer is lost:

   ```
   # Next: Phase C (build)

   `/clear` this session, then paste the line below as the first
   prompt in the fresh session:

   Read .overnight/INTENT.md, .overnight/BUILD_PLAN.md, and all of
   START_HERE.md (§2 Phase C is your task; the rest informs
   spot-checks of subagent output), then build the system.
   ```

   Then tell the human:

   > "Build plan in `.overnight/BUILD_PLAN.md`. The kickoff for the
   > build session is in `.overnight/NEXT.md` if you lose the line
   > below. Now `/clear` and paste this — the build itself happens
   > in that session, using fresh subagents per artifact so each
   > piece gets clean context:
   >
   > `Read .overnight/INTENT.md, .overnight/BUILD_PLAN.md, and all
   > of START_HERE.md (§2 Phase C is your task; the rest informs
   > spot-checks of subagent output), then build the system.`"

   Stop here.

### Phase C — Build (fresh session after `/clear`)

The human pastes the kickoff line. You enter with no memory of Phase
A or B. Read INTENT, BUILD_PLAN, and this section, then:

1. **Take stock.** Check which BUILD_PLAN steps are already `done`
   (so re-entering this session after an interruption resumes
   correctly).

2. **For each pending step, dispatch a fresh subagent** (Task tool,
   `general-purpose` is fine) with a self-contained brief: "Read
   `.overnight/INTENT.md`, `.overnight/BUILD_PLAN.md`, and these
   already-produced artifacts: [paths]. Build only the step's named
   artifact at the declared output path. Return that path plus a
   one-paragraph summary of what you wrote and any assumptions you
   made." For artifacts that encode system principles or behaviour —
   `orientation.md` and the agent prompts in particular — also
   direct the subagent to read the relevant START_HERE.md sections
   (§3 for principles, §1 for the human-channel contract, §4 for
   agent roles, §5 for the bus protocol, §6 step 3 for the
   control-block schema). A brief that paraphrases these
   second-hand loses nuance; the subagent encodes from source. The
   parent (this session) orchestrates; the subagent authors each
   piece in a clean context. This is what avoids forcing the human
   into a `/clear` marathon — the freshness happens inside the
   Task call, not at the human's prompt.

3. **After each subagent returns**, mark the step `done` in
   BUILD_PLAN.md, record the path in `manifest.json`, and dispatch
   the next step. Spot-check the output before moving on — if a
   subagent produced something off-spec, fix it (or re-dispatch with
   a sharper brief) before the next step reads it.

4. **Walk the human through what got built** when all steps complete.
   Revise on their feedback — small tweaks in this Phase C session
   are fine; if they want to rethink intent, route them back to Phase
   A (`/clear` and re-scope).

5. **Smoke-test the apparatus before the first real run.** The
   entire system — runner, control-block parsing, rate-limit reactor,
   branch/worktree orchestration, wakeup generator, the
   `orientation.md` distillation itself — is freshly authored from
   prose every time, so the assembling session is the only thing
   that has read it end-to-end. Don't trust that overnight without
   proving it once. Run `python .overnight/run.py --smoke` (§6):
   the runner executes two no-op checks. The **pipeline check**
   dispatches a placeholder prompt that returns a well-formed
   control block with `status: done` and `checks_passed: true`,
   then verifies the block round-trips and parses, `bus/state.json`
   updates, a reviewer call fires, `wakeup.html` regenerates. The
   **orientation check** spawns a fresh session with only
   `orientation.md` loaded and asks it diagnostic questions about
   the §3 principles, bus layout, fallback prompt, and
   control-block schema — if the session can't answer from
   orientation alone, the distillation is thin. If either fails,
   the smoke run prints what broke and where; fix it in this
   session before writing HANDOFF.md. The failure modes here — a
   brittle rate-limit regex, a malformed control block, a path
   typo in the digest generator, an `orientation.md` missing the
   principles — are exactly the ones that surface at 3am if left
   for the first real run to discover.

6. **Write `.overnight/HANDOFF.md`** (§8) — what was built, the run
   command, `wakeup.html` path, `/overnight` command, how to finish.
   Then delete `.overnight/NEXT.md` — bootstrap is complete; no
   further `/clear` handoffs to bridge.

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
  planner maintains, not a fixed track. The planner also
  **annotates** steps with what later sessions will need to know:
  when an agent discovers something nonobvious that will matter for
  step M, the planner adds a one-line pointer under step M (e.g.
  *"Note from step-3: see `bus/log/...` — tests skip silently
  without `POSTGRES_TEST_URL`"*). Truly cross-cutting discoveries go
  into a *Standing notes* section at the top of `ROADMAP.md`;
  install-level ones (project conventions that outlive the goal) get
  folded into `orientation.md` on the next build so future goals
  inherit them. Pointers, not copies — the full reasoning stays in
  the source session log.

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

- **Parallel tracks live in git worktrees, not the main checkout.**
  A working tree can only have one branch checked out at a time, so
  two concurrent `claude -p` sessions on two branches in the same
  directory would collide. The runner creates one worktree per
  concurrently-active track (`git worktree add .overnight/worktrees/<track>
  <branch>`) and runs each session with that worktree as its cwd. A
  single-track goal needs no worktree — work happens in the main
  checkout. The integration branch gets its own worktree only when
  merges run concurrently with track work; otherwise it can be a
  checkout in the main directory between iterations. Worktrees are
  recorded in `manifest.json` and torn down when the goal is
  archived (§11). `.overnight/` itself is **not** duplicated per
  worktree — it lives in the main checkout, and the runner passes its
  absolute path into each session's prompt so every track reads/writes
  the same bus.

- **File scope per track.** When parallel tracks exist, the planner
  partitions the filesystem: each track has a declared write scope
  (paths it owns). Other tracks may read but not write into a track's
  scope. Anything that needs to cross scopes serializes through the
  planner. Each track lives on its own branch (in its own worktree
  when running concurrently — see above); the planner schedules
  merge points into an integration branch, and the reviewer runs an
  integration pass after each merge.

- **Verification gate (be honest about how hard it is).** A step is
  `done` only after the project's declared checks pass; the reviewer
  runs them after every builder `done`, and failure routes the step
  back to the planner as `blocked`, never silently accepted. The
  *strength* of the gate depends on what the project actually is, so
  don't oversell it:
  - **Code / build / library / data-pipeline goals — deterministic.**
    Build, tests, lint, typecheck, link-check, schema-validate. Pass
    or fail is a return code; the reviewer's role is to run the
    commands and route the result. This is the strong form.
  - **UI goals — deterministic, plus browser checks.** The above plus
    browser-level verification of declared acceptance criteria, via a
    project-declared browser skill (e.g. `dev-browser`, Playwright).
    "Works in the browser" is part of the gate, not a bonus. Still
    deterministic — assertions pass or fail.
  - **Pure-content goals (prose, notes, study sites) — soft.** The
    artifact must build/render without errors (deterministic part),
    AND a fresh reviewer session must find no factual gaps against
    cited source material (LLM judgment — *not* deterministic). Be
    plain about this with the human: the soft half can miss things a
    careful editor would catch; the safeguard is citation discipline
    in the builder (every substantive claim links back to source) so
    the reviewer can actually verify rather than vibes-check.
  Whichever form applies, the rule that "never silently `done` without
  a green gate" holds; what differs is how much trust the gate
  itself deserves.

- **Builder self-checks before declaring `done`.** The builder may
  (and for UI/integration work, should) invoke project-declared
  verification skills — test runners, browser drivers, link
  checkers — *itself* before handing off, so the reviewer's pass
  starts from already-checked work. This tightens the loop: the
  reviewer confirms rather than discovers, and ping-pong between
  builder and reviewer drops. The reviewer still runs the
  authoritative gate, but it should mostly agree.

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

These principles only persist if they're written somewhere future
sessions actually load. `orientation.md` is that file — the runner
injects it into every `claude -p` prompt and `/overnight` loads it
into every interactive session, so it is the sole connective tissue
between this assembling conversation and every night that follows.
Treat it as a content-spec'd artifact, not a free-form briefing.
**A Phase C build of `orientation.md` must contain, at minimum:**

1. **Identity** — one paragraph: what the system is, this goal's
   name, the track structure at build time, a pointer to the rest
   of the apparatus.
2. **The principles above** — verbatim or tightly condensed, all
   of them. Quality-first, no-HITL-overnight, intelligence-in-
   agents-not-runner, one-task-per-session, the honest
   verification-gate tiers, stop-safely. Drop any and future
   sessions inherit a diluted version.
3. **Bus layout** — what each file under `bus/` does, what
   `state.json` contains, how a session reads its inbox and
   writes its log entry.
4. **Control-block schema** (§6 step 3) — the exact contract
   every session must emit, so an agent dropped into the runner
   knows the shape to return.
5. **Per-role responsibilities and tool restrictions** (§4) — so
   a session loading orientation knows which agent it is, what it
   may read/write, and which self-check skills it can invoke.
6. **The fallback prompt** (verbatim from §1) — so the human can
   summon the system without `/overnight`.
7. **Pointers, not copies** — to `run.py`, the agent files,
   `manifest.json`, the archive location, and this goal's
   `INTENT.md` and `ROADMAP.md`. Long-form content stays in its
   source file; `orientation.md` is the index plus the
   load-bearing principles.

A fresh session must be able to read `orientation.md` in 2–3
minutes and then act correctly without ever opening START_HERE.md.
If that isn't true, `orientation.md` is incomplete — and the
smoke run (§6) is what proves it before the first real night.

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
  its track coherent. Cannot write outside its scope. **Invokes
  project-declared self-check skills** (test runners, browser drivers,
  link checkers — defined in §9) before declaring `done`, so the
  reviewer's gate confirms rather than discovers.

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
  and runs whatever it says. **Curates discoveries into the roadmap**:
  when a control block's `next_hint` surfaces something nonobvious
  that will matter for later steps, the planner annotates the
  relevant step (or the Standing notes, or promotes it to
  `orientation.md` on the next build) rather than letting the
  learning live only in the session log. Filters as it goes — not
  every hint is worth recording.

The planner subsumes what some projects might split into a separate
"orchestrator" — having one agent own both the roadmap and the schedule
keeps decisions coherent. For very simple linear goals the planner
trivially schedules one track of one step at a time; the same machinery
scales up to many tracks.

**Tool permissions — encode these as Claude Code subagent restrictions,
not just intent.** The restriction is what makes the separation real;
without it "reviewer is read-only" is just a wish.

- builder: read everything; write only within its track's declared write
  scope; may run project-declared self-check skills (build/test/browser/
  link-check — see §9) before declaring `done`.
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
        "worktree_path": ".overnight/worktrees/T1",
        "cursor": "step-3",
        "status": "active",
        "step_visits": {"step-3": 1},
        "blockers": []
      }
    },
    "integration_branch": "overnight/integration",
    "cumulative_iterations": 47,
    "cumulative_notional_usd": 12.34,
    "last_rate_limit": null,
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
   session with that track's worktree as the working directory.
   - **Single-track (N = 1):** no worktree — the session runs in the
     main checkout on the track's branch.
   - **Concurrent (N > 1):** before spawning, ensure each track has a
     worktree (`git worktree add .overnight/worktrees/<track> <branch>`,
     creating the branch if absent). Spawn sessions concurrently
     (subprocess pool), one per worktree, so two `claude -p` calls
     never share a working directory. This is what makes parallelism
     real — without separate worktrees, concurrent sessions on
     different branches either collide on the index or silently
     serialize.
   - Track-worktree mapping lives in `bus/state.json` (`worktree_path`
     per track when present) and is mirrored in `manifest.json` for
     cleanup. Default is N = 1; the planner only goes higher when it
     judges parallel safe under §3.

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
     "next_hint": "optional hint for the planner — also where the agent surfaces nonobvious discoveries that may matter for later steps (e.g. 'gotcha: tests skip silently without POSTGRES_TEST_URL — may matter for step-7') so the planner can annotate them in ROADMAP.md per §3",
     "decision_question": "required when status is needs-decision: the exact question to put to the principal, phrased so a yes/no or a short choice answers it",
     "reason": "..."
   }
   ```
   Append each to `bus/log/`. Update `bus/state.json` (cursors, visit
   counters, blockers).

   **Validation and repair.** Models don't always comply, and a single
   bad emission must not wedge a track. After extracting the block,
   parse it as JSON and validate against a strict schema (required
   fields, allowed `status`/`action` enums, `decision_question`
   required iff `status == needs-decision`).
   - **Missing block** (no fenced `control` section at all): treat as
     an orphan per §6 crash-recovery — mark the step `partial`, log
     the raw output, hand back to the planner.
   - **Malformed block** (block present but unparseable, or fails
     schema): make **one** repair attempt — re-prompt that same
     session with the raw block and a short error pointer ("your
     control block is missing field `track`; emit only a corrected
     ` ```control ``` block and nothing else"). If the retry also
     fails validation, stop retrying — log both attempts to
     `bus/log/`, mark the step `blocked` with reason
     `control_block_invalid`, and hand back to the planner. Two
     bad emissions on the same step is a stuck-track signal, not
     something to grind on.
   - Repair attempts count toward `MAX_STEP_VISITS` so a track that
     can't emit a valid block doesn't loop forever on retries.

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
- `MAX_ITERATIONS` (global) — the primary numeric cap.
- `MAX_STEP_VISITS` (~3 per step) — the loop-detection guard from §3;
  halt and note in `wakeup.html` if any single step is touched more than
  this many times across sessions.
- **Window-usage is the cap that actually bites**, and it's enforced
  reactively, not predictively: the plan's rolling 5-hour and weekly
  windows are observable to us only as rate-limit errors from
  `claude -p`. §6.4 is how the runner respects them — wait through a
  5-hour reset, halt cleanly on a weekly cap. Don't try to model window
  burn in Python; let the API tell you.
- `MAX_NOTIONAL_USD` (optional, soft) — the API-equivalent dollar
  figure `claude -p` reports. On a Pro/Max plan this is **not money
  spent**; it's a useful proxy for "how much work happened." Setting it
  acts as a belt-and-braces halt ("stop after $X-equivalent of activity
  even if iterations remain"); leaving it unset is fine. Per-track
  distribution is tracked and shown to the planner regardless, so it
  can rebalance.
- Running notional cost and iteration count printed each iteration,
  per-track distribution included. Both are visibility numbers; the
  hard halts are `MAX_ITERATIONS`, `MAX_STEP_VISITS`, and the
  rate-limit reactor.

**Subscription billing guard.** Before starting, verify
`ANTHROPIC_API_KEY` is **not** set in the environment and `claude` is
logged in. Refuse to start otherwise.

**Preflight smoke mode (`--smoke`).** A one-shot self-test of the
apparatus, run automatically at the end of Phase C (§2) and available
on demand. Two complementary checks, both mechanical:

*Pipeline check* — dispatch one trivial placeholder prompt (e.g.
"reply with a control block declaring `status: done`,
`checks_passed: true`, and no file edits"); parse the returned
control block against the schema; update `bus/state.json` with
the (no-op) result; invoke a reviewer-mode call that returns
immediately; regenerate `wakeup.html`. If every stage round-trips
cleanly, the pipeline is green.

*Orientation check* — spawn one fresh `claude -p` session whose
only loaded context is `.overnight/orientation.md` (no INTENT, no
BUILD_PLAN, no START_HERE) and instruct it to answer four
diagnostic questions in its control-block `summary`: (a) name
three of the §3 principles, (b) the path where bus state lives,
(c) the fallback prompt verbatim, (d) the required fields of a
builder's control block. Parse the answers; if any is missing,
generic, or wrong, `orientation.md` is too thin to brief future
sessions — print which question failed and exit non-zero. This
is the test that proves the distillation worked; without it, an
under-specified orientation only surfaces as degraded overnight
quality weeks later.

If any stage of either check fails, print which stage and what
was wrong (e.g. "control block parse failed: missing `track`
field at offset 124"; "orientation check failed: session could
not produce the fallback prompt verbatim — `orientation.md` is
missing §1's exact line") and exit non-zero — leave the partial
state in place so the assembling session can diagnose. The smoke
run does **not** spawn builder work, does **not** create branches
or worktrees, and does **not** consume meaningful window budget.
It's there because the whole system was freshly authored from
prose and the only prior integration test is the human reading
the digest at 3am — too late to find a broken regex or a thin
orientation.

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

1. **Header.** Goal name; date range covered by this digest; duration
   and total **notional cost** (API-equivalent dollars `claude -p`
   reports — not money spent, on a plan); iterations used vs.
   `MAX_ITERATIONS`; current rate-limit window state (active / waiting
   for reset at HH:MM / weekly cap hit); overall status.
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
5. **Notes the planner added this run.** One-line entries for each
   roadmap annotation the planner wrote since the last digest — the
   discovery in a sentence, which step (or Standing notes section)
   it was filed under, link to the source `bus/log/...` for the full
   reasoning. Lets the human see what the system is figuring out
   without diffing `ROADMAP.md`.
6. **Open review items / blocks / stalls.** Anything that didn't close
   cleanly, with a one-line "what the planner intends to try next."
7. **Notional-cost-per-step bars.** Simple inline-SVG bar chart of the
   API-equivalent figure per step. Not money on a plan, but a useful
   relative signal — surfaces runaway steps before they eat tomorrow
   night's window allotment.

The digest is the human's morning interface to the system. Treat it as
a first-class artifact: if it's ugly or unscannable, the human will
give up on reading it and the loop closes badly.

---

## 8. HANDOFF.md (write this last, to `.overnight/HANDOFF.md`)

- One paragraph: what got assembled, what the goal is, which tracks
  were identified, what verification checks were declared — and
  whether the gate is fully deterministic (code/UI) or partly
  soft-reviewer-judged (pure content), per §3.
- A checklist for the human, **especially before the first run** (and
  worth re-skimming before any night that follows a significant
  re-scoping): *read* — don't just skim — each step on the first track
  in `.overnight/ROADMAP.md`; confirm the principal agent's profile
  matches your taste and red lines; read the declared check commands;
  glance at the runner caps (`MAX_ITERATIONS`, `MAX_STEP_VISITS`,
  optional `MAX_NOTIONAL_USD`). The first night's quality depends on
  these being right — five minutes here saves a night of wrong-shape
  work, and the agents will be running this same plan in clean
  contexts without you, so anything that's wrong in writing will be
  wrong in execution.
- **Smoke-test passed at assembly time** (§2 Phase C step 5). Re-run
  `python .overnight/run.py --smoke` after any change to the runner,
  the agent prompts, or `wakeup.html` generation — it's a cheap
  proof that the apparatus still round-trips before you trust it
  overnight.
- **Honest scope note on isolation.** The system runs `claude -p`
  with `--permission-mode acceptEdits`; builder agents can invoke
  declared self-check skills (test runners, build commands, browser
  drivers), which means arbitrary shell can run overnight. Git
  branch + worktree isolation contains file damage (throw the
  branch away), not command damage. Fine for personal, low-stakes
  work; if this project touches production credentials, paid
  external APIs, or shared infrastructure, run the loop inside a
  container or VM rather than relying on branch isolation alone.
  (§10)
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
- **What checks define "done" for a step, and how deterministic are
  they?** For code/UI work: build, test, lint, typecheck, link-check,
  browser assertions — return-code checks that prove correctness
  without a human reading the result. For pure-content work: the
  render/lint command (deterministic) plus a fresh reviewer pass
  against cited source (a soft, LLM-judged check — surface this to
  the human plainly, don't dress it up as deterministic). Either way,
  this becomes the verification gate (§3, §6). Without declared
  checks the gate degenerates to "the reviewer thinks it's fine,"
  which is exactly the silent-failure mode the gate exists to
  prevent — so push to declare *something*, even if part of it is
  soft.
- **What reusable skills should the *builder* invoke to self-check
  before declaring `done`?** This is separate from the reviewer's
  gate: the builder running its own checks first means the reviewer
  confirms rather than discovers, which tightens the loop and cuts
  ping-pong. For UI goals especially, this means wiring in a browser
  skill (Playwright, `dev-browser`) so "works in the browser" is part
  of the builder's promise, not only the reviewer's audit. For
  backend or library work it's typically the project's test runner
  plus typecheck. List the commands or skill names; they go into the
  builder agent's prompt and tool list (§4).
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
  from the separate monthly Agent SDK credit instead. **The dollar
  figure `claude -p` reports is notional** — what those tokens would
  cost via the API. On a plan it's a useful proxy for "how much work
  happened," not money spent; the real constraint is the window
  allotment. Parallel tracks multiply window burn proportionally — N
  concurrent sessions ≈ N× the rate. The runner can't predict window
  state from inside Python (the API doesn't expose remaining budget
  until it refuses); instead it reacts to rate-limit errors — parses
  the reset time, **waits through rolling 5-hour resets** before
  resuming (§6.4), and halts with a note only when the wait would be
  excessive (typically a weekly cap with reset days away). The planner
  factors observed window pressure into scheduling — backing off
  parallelism after a recent rate-limit hit, for instance — rather
  than pretending to know remaining capacity.
- **One git branch per track, plus an integration branch — and one
  worktree per concurrently-active track.** Auto-accept edits can't
  clobber anything outside the overnight branches. Concurrent tracks
  must run in separate git worktrees (`.overnight/worktrees/<track>/`)
  because a single working directory can only hold one branch at a
  time — see §3 and §6 step 2. The human reviews the integration
  branch before merging into their own main.
- **Blast radius is git-branch-level, not process-level — be honest
  about this.** `--permission-mode acceptEdits` plus declared
  self-check skills (test runners, build commands, browser drivers)
  means the agents can run arbitrary shell commands overnight,
  unsupervised. Branch and worktree isolation contains *file
  damage* (you can throw the branch away), not *command damage* (a
  test runner that drops a database, a build script that hits a
  paid API, a `rm` in a postinstall hook). For personal,
  low-stakes use this is fine; for anything touching production
  systems, shared infrastructure, or credentials, run inside a real
  sandbox (container, VM, or a dedicated machine) rather than
  trusting branch isolation alone. Surface this honestly in HANDOFF.md
  rather than implying the system is sandboxed when it isn't.
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

Either way, **the three-phase model from §2 applies to scoping the
new goal** — Phase A in this session for intent, `/clear`, Phase B
for the build plan, `/clear`, Phase C for the build. The install
pieces (runner, `/overnight` command, agents, orientation,
manifest) already exist and don't get rebuilt; BUILD_PLAN.md is
typically smaller this time, mostly the new goal-level files
(INTENT.md, ROADMAP.md, a refreshed principal profile if the domain
shifted, a refreshed HANDOFF.md). Skipping the `/clear` boundaries
to save a session would re-introduce exactly the upstream-context
drift §2 exists to prevent — don't.

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
