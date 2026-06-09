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
| 1 | Office DVR — instant replay & timeline scrubber | ✨ | Scrub the whole floor backward like a DVR; see what every agent did at any second. |
| 2 | Team Templates ("Org Charts") | 🪑 | Save a whole roster + repos + prompts and re-spawn the team in one click. |
| 3 | Drag-to-Assign on the floor | 🪑 | Drag an issue, a task card, or a file onto an avatar to give it the work. |
| 4 | The Office Cast — persona presets | 🪑✨ | One-click personalities (Dwight the bug-hunter, Oscar the reviewer) that are really tuned prompt+model+guardrail bundles. |
| 5 | Daily Standup & End-of-Day digest | 🪑✨ | GOD writes a plain-English "here's what the office did" summary on a schedule. |
| 6 | Plain-language guardrails | 🪑 | Type "don't spend more than $5 today, never force-push" and it compiles to real breaker rules. |
| 7 | Conflict Huddle — collision detection | 🪑✨ | When two agents touch the same files, they're pulled into a huddle to reconcile before they clobber each other. |
| 8 | Pocket Office — remote approvals & dispatch | 🪑 | Approve escalations and dispatch work from your phone via the ingress we already run. |
| 9 | Voice dispatch — "Hey Michael…" | 🪑✨ | Hands-free, talk to the GOD orchestrator and watch the floor react. |
| 10 | Office Cam — shareable replay export | ✨ | Export a sped-up clip of your agents working — a built-in viral loop. |

---

## 1. Office DVR — instant replay & timeline scrubber ✨

**What it is.** A scrubber bar under the floor that lets you drag time backward
and watch the office re-animate: avatars walk back to old stations, envelopes
un-fly, tool bubbles reappear. Click any moment to see the exact roster state,
who held which task, and what the blackboard said. "Jump to" markers for
escalations, breaker trips, and task completions.

**Why clients love it (🪑).** The single hardest question with autonomous agents
is *"wait, what just happened?"* Today you reconstruct it from logs. The DVR
turns that into a drag of the mouse. When an agent does something surprising at
3am, you scrub to it in seconds instead of grepping transcripts.

**The edge (✨).** Every competitor shows you a log. Nobody lets you *re-watch the
office.* The floor is our unfair advantage — this is the feature that makes the
visualization indispensable instead of decorative.

**How it fits.** We already have the **append-only event log** (`hive.ts`) — the
DVR is a deterministic replay of that log into the existing Pixi scene's avatar
state machine. No new data plane; we're rendering history we already record. The
scrubber reads the same events the live floor consumes; "playback mode" just
swaps the event source from live → seeked.

**Effort / risk.** Medium. Main work is a clean separation between "live event
feed" and "seekable event source," plus snapshotting roster/board state at
intervals so seeking is O(1) instead of replaying from zero.

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

## 5. Daily Standup & End-of-Day digest 🪑✨

**What it is.** On a schedule (reusing the **Schedules** tab), GOD composes a
plain-English briefing: *what each agent shipped, what's blocked, what it spent,
what's waiting on you.* Delivered as a desktop notification, written to the
blackboard, and optionally pushed to Slack via the ingress we already run.

**Why clients love it (🪑).** The whole pitch is "agents work while you sleep."
The missing half is *waking up to a readable summary instead of a wall of
terminals.* A standup is exactly the artifact a manager wants — and it's the
office metaphor's most natural ritual.

**The edge (✨).** Competitors give you logs and dashboards; we give you a
manager's morning briefing authored by the orchestrator that actually ran the
work. It also closes the loop on cost: the digest cites the **cost ledger**, so
spend shows up in the same place as progress.

**How it fits.** Scheduled mission → GOD reads the **event log + task board + cost
ledger** → emits a summary to notifications + blackboard + Slack. Every input
already exists; this is a prompt + a delivery fan-out.

**Effort / risk.** Low-medium. Mostly prompt design and a delivery adapter we
mostly have.

---

## 6. Plain-language guardrails 🪑

**What it is.** A natural-language rules box: *"Don't spend more than $5 today.
Never force-push to main. Ask me before touching the billing repo. Stop any
agent that loops more than 3 times."* It compiles to concrete **circuit-breaker**
and **HITL-gate** rules, shown back to you as a confirmable list before they arm.

**Why clients love it (🪑).** The breaker and budgets are powerful but live behind
settings. Letting people state intent in English — and *showing the compiled
rules* — makes safety approachable for non-experts and fast for experts. It lowers
the trust barrier to actually leaving agents unattended.

**The edge.** Turns our safety stack (steer → constrain → stop ladder, budgets,
HITL gate) into something a first-time user can configure in one sentence.

**How it fits.** We already have the **cost/runaway circuit breaker**, **per-agent
budgets**, and the **HITL gate** driven through hook returns. This is a small
NL → rules compiler in front of existing knobs, with a human-readable preview so
nothing arms silently.

**Effort / risk.** Medium. Keep it safe-by-construction: the compiler only emits
from a fixed rule vocabulary, and every rule is shown for confirmation — no
free-form code execution.

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

## 8. Pocket Office — remote approvals & dispatch 🪑

**What it is.** Act on the office from your phone: get a push when GOD escalates,
**approve/deny from the notification**, read the standup, and fire a quick
dispatch ("have someone look at the failing deploy") — all without opening the
desktop app.

**Why clients love it (🪑).** Agents run for hours; you don't sit at the machine
the whole time. The bottleneck for true autonomy is *you being reachable for the
one approval that matters.* Remote approvals remove the leash without removing
control.

**The edge.** "Agents that work while you're away" only pays off if you can
unblock them from away. This makes the unattended pitch real.

**How it fits.** We already expose local endpoints through **tunnelmole** for
Slack/webhook ingress, and we already have an **approvals queue** + **desktop
notifications**. Pocket Office is: escalation → push/Slack with action buttons →
the button hits the existing approve/deny endpoint over the tunnel. Slack
interactive messages are the fastest v1 (no app-store build needed).

**Effort / risk.** Medium. v1 = Slack action buttons (cheap, leverages ingress);
native push is a later, larger step.

---

## 9. Voice dispatch — "Hey Michael…" 🪑✨

**What it is.** Push-to-talk (or a wake phrase) to the GOD orchestrator: *"Michael,
have someone fix the failing CI on the api repo and tell me when it's green."*
Transcribed locally, handed to GOD, and you watch the floor react. GOD can talk
back through the standup/notification channel.

**Why clients love it (🪑).** When your hands are on the keyboard writing code,
the fastest way to delegate is to *say it.* Talking to a manager rather than
filling in dispatch forms is the most natural interface the office metaphor
implies.

**The edge (✨).** A voice-driven *multi-agent office* doesn't exist anywhere. It's
the most demo-able, "whoa" feature on the list and it's a clean fit: we already
have a single natural-language entry point — GOD — so voice is just a new input
modality into a route that exists.

**How it fits.** Local/STT transcription → text → existing **GOD dispatch** path.
No new orchestration; GOD already accepts natural language and routes. Optional
TTS on the digest for spoken replies.

**Effort / risk.** Medium. STT integration + a press-to-talk affordance. Keep it
optional and local-first for privacy.

---

## 10. Office Cam — shareable replay export ✨

**What it is.** "Export clip" turns a slice of the **Office DVR** (#1) into a
sped-up, shareable video — your agents shipping a feature in 20 seconds, envelopes
flying, the breaker catching a runaway — watermarked and ready for social.

**Why clients love it.** People *want* to show off their swarm. Giving them a
one-click, genuinely cool artifact scratches that itch — and it's how the product
spreads. The floor is already beautiful; let people post it.

**The edge (✨).** A built-in viral loop. Every shared Office Cam clip is an ad for
the product, and the visual floor is the only harness that has something worth
filming. It also doubles as a *changelog artifact* — drop a clip in a PR to show
what the agents did.

**How it fits.** Builds directly on the DVR (#1): render the seeked event range to
an offscreen Pixi canvas at high speed, capture frames, encode to mp4/webm. We
already produce marketing footage of the floor — this productizes that pipeline
for users.

**Effort / risk.** Medium, and *gated on #1* — ship the DVR first, then this is a
mostly-mechanical capture/encode pass.

---

## How they stack up

```
            Nobody-else-has-this  ▲
                                  │   9 Voice          1 Office DVR
                                  │        ✨               ✨
                                  │   4 Cast      7 Huddle    10 Office Cam
                                  │     🪑✨        🪑✨            ✨
                                  │           5 Standup
                                  │              🪑✨
                                  │   6 Guardrails
                                  │       🪑      3 Drag-assign
                                  │                  🪑     8 Pocket Office
                                  │                            🪑
                                  │              2 Org Charts
                                  │                  🪑
                                  └──────────────────────────────────►
                                          Daily ease-of-use value
```

**If we ship in waves:**

1. **Quick wins (low effort, high daily value):** 2 Org Charts → 4 Office Cast →
   3 Drag-to-Assign → 5 Standup digest. Mostly config and renderer glue on top of
   plumbing we already have; visibly better within a release.
2. **The signature bet:** 1 Office DVR → 10 Office Cam. This is the pair that makes
   the floor *the reason* to use Munder Difflin instead of a CLI. Do them together.
3. **Trust & reach:** 6 Plain-language guardrails → 8 Pocket Office → 7 Conflict
   Huddle. These are what let people actually leave the office running unattended.
4. **The flex:** 9 Voice dispatch. Smaller surface than it sounds (one NL entry
   point already exists), outsized demo value.

---

### Also considered (next round)

- **Performance Reviews** — per-agent scorecards (success rate, $/task, tokens,
  rework) rendered as an on-theme "annual review." Great retro material; folds
  naturally into the cost ledger + event log.
- **Dry-run / "What-if" approvals** — agents plan and show a diff + cost estimate
  before executing, gated by the HITL queue. Pairs with #6.
- **Watercooler** — agents proactively post reusable learnings to the shared
  MemPalace so the team gets smarter over time, not just per-session.
