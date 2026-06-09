# Feature Ideas — 10 things to bring to the table

> Ten proposals for Munder Difflin, weighted toward two goals: **make the floor
> effortless for the operator**, and **ship at least a few things no other
> multi-agent harness has.** Each idea is grounded in the architecture we already
> ship (the append-only event log, the hive, the GOD orchestrator, the Pixi.js
> floor, the cost ledger, the circuit breaker, OTel, and Slack/webhook ingress)
> so none of it is hand-wavy — it's the next layer on what's already here.
>
> Legend: **🪑 Ease-of-use** · **✨ Nobody-else-has-this** · **🪑✨ both**
>
> _Drafted 2026-06-09. This is a proposal doc, not a commitment — see
> [`CONTRIBUTING.md`](./CONTRIBUTING.md) before picking one up._

---

## The shortlist

| # | Feature | Bucket | One-liner |
|---|---|---|---|
| 1 | Ask-the-Office | 🪑✨ | One plain-English search bar over every agent's memory, the event log, and the cost ledger — with clickable provenance. |
| 2 | Team Templates ("Org Charts") | 🪑 | Save a whole roster + repos + prompts and re-spawn the team in one click. |
| 3 | Drag-to-Assign on the floor | 🪑 | Drag an issue, a task card, or a file onto an avatar to give it the work. |
| 4 | The Office Cast — persona presets | 🪑✨ | One-click personalities (Dwight the bug-hunter, Oscar the reviewer) that are really tuned prompt+model+guardrail bundles. |
| 5 | Hiring Fair — auto-staffing | 🪑✨ | Describe the job in one line; GOD proposes who to hire, with what models and budgets, and you approve. |
| 6 | Memory Inspector | 🪑✨ | Browse, pin, correct, and forget what an agent knows — teach it instead of hoping. |
| 7 | Conflict Huddle — collision detection | 🪑✨ | When two agents touch the same files, they're pulled into a huddle to reconcile before they clobber each other. |
| 8 | Spend Forecast | 🪑 | See the estimated bill before you commit, then watch a live burn-down flag drift early. |
| 9 | Autopilot Dial | 🪑 | One Observe / Assist / Autopilot control for how much the office decides on its own. |
| 10 | Suggestion Box | ✨ | Idle agents file vetted, opt-in improvements you triage with a tap — nothing acts unprompted. |

---

## 1. Ask-the-Office — one search bar over everything the office knows 🪑✨

**What it is.** A global command-palette **Ask** box that answers plain-English
questions across every agent's long-term memory, the append-only event log, the
blackboard, and the cost ledger at once: *"Who touched the auth module yesterday
and why?"* · *"What did Dwight learn about the flaky tests?"* · *"Where did we
spend the most this week?"* Answers come back with **clickable provenance** —
which agent, which event, which file — so you can jump straight to the source.

**Why clients love it (🪑).** The knowledge is already captured, but it's scattered
across five panels. One bar to interrogate the whole office turns "go dig through
tabs" into "ask a question." It's the shortest path from *something happened* to
*here's exactly what, and why.*

**The edge (✨).** Per-store memory search exists; nobody offers a single
natural-language interface that *fuses* memory + event log + cost across the whole
roster and answers with citations you can click. The hive is the corpus — this
makes it queryable like a teammate you can just ask.

**How it fits.** We already ship the semantic **MemPalace** + text search, the
**append-only event log**, the **blackboard**, and the **cost ledger**.
Ask-the-Office is a retrieval layer that fans a query across those sources and has
GOD compose a cited answer. It degrades to plain text search when the semantic
index isn't installed — the same graceful-degradation contract we already honor.

**Effort / risk.** Medium. Mostly retrieval plumbing + answer synthesis; the data
and the search primitives already exist.

---

## 2. Team Templates — "Org Charts" 🪑

**What it is.** Save the current floor — every agent's command, working dir,
repo, model, token budget, git-isolation setting, and opening prompt — as a named
**Org Chart**. Spawn it later in one click. Ship a few starters: *Bug Triage
Squad*, *Docs Team*, *Release Train*, *Research Pod*.

**Why clients love it (🪑).** "Restore team" already rebuilds *last* session. Org
Charts let you keep *many* purpose-built teams and pick the right one for the job
— no re-adding agents by hand, no re-typing prompts. It's the difference between
a saved playlist and pressing shuffle.

**The edge.** Turns a one-off harness into a reusable workflow library. Templates
are also *shareable files* — a team can commit `org-charts/release-train.json` to
their repo so everyone spins up the identical office.

**How it fits.** Extends the existing config/persistence layer (SQLite durable
store + onboarding wizard). An Org Chart is just a serialized roster spec; the
add-agent flow already knows how to consume each field. Add a "Save as Org Chart"
button to the Floor tab and a picker on first run.

**Effort / risk.** Low. Mostly serialization + a small management UI. Lowest-risk,
highest-daily-value item on the list.

---

## 3. Drag-to-Assign on the floor 🪑

**What it is.** Direct manipulation: drag a GitHub issue card, a Kanban task, or
even a file from the file browser and **drop it onto an avatar** to assign it.
The envelope animation plays from your cursor to their desk. Drop onto Michael's
office to let GOD route it.

**Why clients love it (🪑).** Assignment today is a form. Dragging work onto a
character is faster, more legible, and frankly more fun — it matches the mental
model the floor already creates ("that's Pam, give it to Pam").

**The edge.** Makes the office metaphor *functional*, not cosmetic. The avatars
stop being a screensaver and become the control surface.

**How it fits.** We already ingest **GitHub issues** and have a **dependency-aware
Kanban**; assignment plumbing exists. This is a renderer-side drag layer over the
Pixi scene that resolves a drop target to an agent id and calls the existing
dispatch path. Drop-on-GOD reuses the escalation/routing route.

**Effort / risk.** Medium-low. Hit-testing avatars in Pixi + a drag ghost; the
backend action already exists.

---

## 4. The Office Cast — persona presets 🪑✨

**What it is.** One-click **personas** that are really curated bundles of *system
prompt + model + token budget + guardrail posture + default repo role*:
- **Dwight** — relentless bug-hunter; high autonomy, aggressive on tests.
- **Oscar** — meticulous code reviewer; read-mostly, never force-pushes.
- **Pam** — docs & release notes; cheap model, low budget.
- **Creed** — the chaos sandbox; strict breaker, isolated worktree, no secrets.

**Why clients love it (🪑).** New users don't know how to prompt an agent into a
*role*. Picking "a reviewer" or "a bug-hunter" from a roster is instantly legible
and gets a good result on the first try. It collapses prompt-engineering into a
character pick.

**The edge (✨).** This is the Dunder Mifflin theme doing real work. It's
on-brand, memorable, demoable, and — because each persona pins a model and a
budget — it's also a *cost* and *safety* feature in disguise. No other harness has
opinionated, named, theme-driven agent archetypes.

**How it fits.** A persona is an Org Chart entry (see #2) with a canonical prompt
and a pinned model/budget. Pure config layered on the add-agent flow and the
per-agent model selector + budget we already ship. Ships as data, not code.

**Effort / risk.** Low. Risk is taste: keep personas useful first, funny second.

---

## 5. Hiring Fair — describe the job, GOD staffs the team 🪑✨

**What it is.** A one-line goal box: *"Ship a fix for the failing checkout flow
and write the release notes."* GOD proposes a **staffing plan** — which personas
(#4) to hire, which repos/worktrees, which models and budgets, in what order, with
dependencies — rendered as a reviewable **Org Chart** you tweak and approve. One
click spawns the team.

**Why clients love it (🪑).** The hardest part of any multi-agent tool is knowing
how to set it up for a given job. This inverts it: state the *outcome*, get a
sensible team. New users get a working office on their first try; experts skip the
manual roster-building entirely.

**The edge (✨).** Auto-staffing *from intent* doesn't exist in agent harnesses —
they make you assemble the team yourself. Sitting on top of Personas (#4) and Org
Charts (#2), it turns Munder Difflin into something that *forms its own org* around
the work in front of it.

**How it fits.** GOD already routes and assigns. Hiring Fair extends that to roster
*proposal*: GOD reads the goal + available personas + registered repos and emits an
Org Chart spec (#2's format), shown in the same reviewable picker. Spawning reuses
the existing add-agent + worktree path. Human approval stays mandatory before any
agent spins up.

**Effort / risk.** Medium. Prompt + a plan → Org-Chart serializer; everything
downstream already exists.

---

## 6. Memory Inspector — the agent's brain, and you can edit it 🪑✨

**What it is.** A per-agent **brain** panel: browse long-term memory as readable
cards, **pin** the things that must never be forgotten, **correct** a wrong belief
inline, and **forget** entries that are stale or mistaken. See what the
**MemoryReflector** condensed — and undo a bad condensation.

**Why clients love it (🪑).** Agents act on what they remember. When one is
confidently wrong (*"the deploy command is X"* — it isn't), today you can't easily
fix the belief and it keeps biting. Editable memory turns a frustrating recurring
mistake into a five-second correction.

**The edge (✨).** Memory is usually a black box. A first-class, human-editable
memory with pin / correct / forget — plus visibility into what the Reflector
condensed — is a real trust differentiator. It's the difference between *hoping*
the agent learns and *teaching* it.

**How it fits.** We already keep markdown-first per-agent long-term memory, a
searchable semantic palace, and the **MemoryReflector** that condenses over time.
The Inspector is a CRUD + pin/forget UI over that store, with edits flowing back
through the same write path the agents use. Pins are honored by the Reflector so
they survive condensation.

**Effort / risk.** Medium. UI + safe edit/forget operations; the store and search
already exist.

---

## 7. Conflict Huddle — collision detection 🪑✨

**What it is.** When two agents are editing the same files or branches, the
harness detects the overlap and stages a **huddle**: the avatars walk to a meeting
spot, their work pauses at a safe point, and GOD proposes who proceeds / who
rebases / who waits. You can rubber-stamp or override.

**Why clients love it (🪑).** The scariest failure mode of parallel agents is two
of them silently clobbering each other's work. Surfacing it *as it forms* — with a
clear, visual "they need to talk" moment — prevents the lost-work nightmare and
makes parallelism trustworthy.

**The edge (✨).** Git worktrees prevent *branch* collisions; nobody detects and
*mediates* logical conflicts at the task level with a human-legible reconciliation
step. The huddle animation makes an abstract race condition into something you can
literally see and resolve.

**How it fits.** We already provision **per-agent git worktrees** and watch git
state per agent. Add an overlap detector across active agents' touched paths;
on conflict, raise a structured escalation through the existing **approvals
queue** and play a huddle on the floor.

**Effort / risk.** Medium-high. Detection heuristics need tuning to avoid
false-positive nagging; start conservative (same file, both writing).

---

## 8. Spend Forecast — see the bill before you commit, watch it burn down 🪑

**What it is.** Before you dispatch a mission, GOD estimates **tokens, dollars,
and wall-clock** from the cost ledger's history of similar past work, and shows a
range. While it runs, a live **burn-down** projects final spend against the
estimate and flags drift early — long before the breaker has to trip.

**Why clients love it (🪑).** The number-one anxiety with autonomous agents is the
surprise bill. A believable up-front estimate and a live "on track / running hot"
projection replace dread with a number. People delegate bigger jobs when they can
see the cost coming.

**The edge.** The breaker stops runaways *reactively*; this is *predictive* — it
forecasts spend before and during, from your own historical ledger, not a static
price sheet. Cost foresight, not just cost limits.

**How it fits.** We already ship a durable **cost ledger**, real per-agent/session
token telemetry, and **OTel** per-model cost attribution. Spend Forecast reads the
ledger for comparable tasks to estimate, and the live OTel stream to project
burn-down. It feeds the same budgets / breaker we already have.

**Effort / risk.** Medium. The estimate improves with data; start with simple
per-task-type averages and honest confidence ranges.

---

## 9. Autopilot Dial — one control for how much the office decides on its own 🪑

**What it is.** A single, legible dial — **Observe / Assist / Autopilot** — per
agent or floor-wide:
- **Observe** — nothing acts without you.
- **Assist** — agents handle the routine, escalate anything with side effects.
- **Autopilot** — GOD resolves everything it safely can and only wakes you for the
  truly critical.

The current posture is always visible on the floor.

**Why clients love it (🪑).** Trust is earned gradually. A clear dial lets people
start cautious and turn autonomy up as they get comfortable — without hunting
through breaker settings and HITL toggles. It makes "how much leash" a one-gesture
decision.

**The edge.** GOD already escalates "only critical items," and the breaker has a
steer → constrain → stop ladder — but the *posture* is implicit and scattered.
Surfacing it as one named dial (and showing it on the floor) is the legible
front-end that makes the whole safety stack approachable.

**How it fits.** A preset over existing controls: the **HITL gate**, escalation
thresholds, and breaker aggressiveness. Each dial position maps to a known
configuration of knobs we already have — no new enforcement, just a unified
front-end and a floor indicator.

**Effort / risk.** Low-medium. Mostly mapping presets to existing settings + a
visible indicator.

---

## 10. Suggestion Box — idle agents that pitch in instead of sitting still ✨

**What it is.** When an agent finishes early or notices something off-task — flaky
tests, a stale TODO, a security smell, a doc that's drifted — it drops a
**low-priority suggestion** into a Suggestion Box queue (*"I noticed X; want me to
fix it?"*). You triage with a tap: **approve**, **dismiss**, or **snooze**.
Nothing acts without your nod.

**Why clients love it (🪑✨).** Agents currently go idle between assignments; this
turns slack time into a stream of vetted, opt-in improvements — the
proactive-but-safe behavior people wish their tools had. It surfaces work you
didn't know to ask for, without ever acting unprompted.

**The edge (✨).** Harnesses are reactive — they do what you dispatch. An office
that *proposes* its own work (gated by your approval) is a different posture
entirely, and it's pure upside: idle compute becomes a backlog of suggestions
instead of a wasted seat.

**How it fits.** Agents already write to outboxes and the blackboard, and
idle/inbox wakeups already exist. Suggestions are a new low-priority message class
routed to a Suggestion Box surface (a filtered view of the **approvals queue**),
so nothing executes until you promote one to a task — which then flows through the
normal dispatch + Kanban path.

**Effort / risk.** Medium. Needs guardrails so suggestions stay useful and rare
(rate-limit, dedupe), and they must never auto-execute.

---

## How they stack up

```
            Nobody-else-has-this  ▲
                                  │   1 Ask-the-Office     6 Memory Inspector
                                  │        🪑✨                  🪑✨
                                  │   10 Suggestion Box   5 Hiring Fair
                                  │          ✨               🪑✨
                                  │   4 Cast        7 Huddle
                                  │     🪑✨          🪑✨
                                  │
                                  │   9 Autopilot Dial
                                  │       🪑        3 Drag-assign
                                  │                   🪑     8 Spend Forecast
                                  │                              🪑
                                  │              2 Org Charts
                                  │                  🪑
                                  └──────────────────────────────────►
                                          Daily ease-of-use value
```

**If we ship in waves:**

1. **Quick wins (low effort, high daily value):** 2 Org Charts → 4 Office Cast →
   9 Autopilot Dial → 3 Drag-to-Assign. Config, presets, and renderer glue over
   plumbing we already have; visibly better within a release.
2. **The knowledge layer (the signature bet):** 1 Ask-the-Office → 6 Memory
   Inspector. Make the hive's accumulated knowledge *legible and correctable* —
   this is the pair that makes Munder Difflin the place you stay, not a CLI.
3. **Trust & cost:** 8 Spend Forecast → 7 Conflict Huddle. These are what let
   people delegate the bigger, scarier jobs unattended.
4. **The flex:** 5 Hiring Fair → 10 Suggestion Box. The office staffs itself for
   the work and pitches its own improvements — the "whoa" demos that sit cleanly
   on everything above.

---

### Also considered (next round)

- **Undo / Time-machine** — one-click, git-backed revert of an agent's last change
  across its worktree when it goes wrong. Pairs with #7.
- **Office Hours / quiet mode** — define when agents may work and spend (throttle
  overnight, "do not disturb" windows where escalations queue silently). Builds on
  the Schedules tab + breaker.
- **Take the wheel** — drop into any agent's session to drive manually, then hand
  back with GOD re-briefing the agent on what you changed. Builds on the command
  bar + mid-run steer.
