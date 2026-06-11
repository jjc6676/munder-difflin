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
| 1 | The Interview — best-of-N tryouts | ✨ | Give the same task to 3 candidate agents in parallel worktrees, compare side-by-side, hire the winner. |
| 2 | Team Templates ("Org Charts") | 🪑 | Save a whole roster + repos + prompts and re-spawn the team in one click. |
| 3 | Drag-to-Assign on the floor | 🪑 | Drag an issue, a task card, or a file onto an avatar to give it the work. |
| 4 | The Office Cast — persona presets | 🪑✨ | One-click personalities (Dwight the bug-hunter, Oscar the reviewer) that are really tuned prompt+model+guardrail bundles. |
| 5 | Assembly Line — review-gated pipelines | 🪑✨ | Chain agents into write → review → document lines where work physically can't skip a sign-off. |
| 6 | Done Means Done — proof-gated tasks | 🪑 | A task card can't move to *done* until its proof command passes; every done card ships with a receipt. |
| 7 | Conflict Huddle — collision detection | 🪑✨ | When two agents touch the same files, they're pulled into a huddle to reconcile before they clobber each other. |
| 8 | The Vault — secrets desk | 🪑 | Agents request credentials, you approve, secrets are injected scoped and redacted from every log. |
| 9 | Clock Out — pause the whole office | 🪑 | One click safely freezes every agent at a clean boundary; one click resumes with each agent re-briefed. |
| 10 | Branch Offices — federated hives | ✨ | Link two offices — your desktop and a teammate's — so their agents exchange mail like one company. |

---

## 1. The Interview — best-of-N tryouts ✨

**What it is.** Right-click a task → **"Interview candidates."** The harness spawns
2–4 agents — different personas, models, or prompts — each in its **own isolated
worktree**, all working the *same* task in parallel. When they finish, you get a
side-by-side comparison: the diffs, whether each one's tests pass, what each cost,
and how long each took — plus a short GOD-written verdict. Click **Hire** on the
winner (its branch is kept and its approach noted to memory); the rest are archived.

**Why clients love it.** "Which model/prompt should I use for this?" is unanswerable
in the abstract — so let them *compete on the actual task*. For anything important,
best-of-3 beats best-guess, and the side-by-side makes the choice obvious instead
of theoretical.

**The edge (✨).** Other tools run agents in parallel; nobody runs them in parallel
**on the same task as a structured competition** with a hire/archive decision at the
end. It's also the most honest eval loop there is: your repo, your task, real diffs —
not a benchmark.

**How it fits.** Per-agent **git worktrees** already guarantee candidates can't
collide. The **per-agent model selector**, **cost telemetry**, and **archival**
already exist — the comparison view is assembled from the diff (git tab plumbing),
the proof run (#6), and the cost ledger. GOD writes the verdict the same way it
adjudicates today.

**Effort / risk.** Medium. The spawn-N-and-compare orchestration is new; everything
it composes already ships. Cost multiplies by N — surface that clearly up front and
default candidates to cheaper models.

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

## 5. Assembly Line — review-gated pipelines 🪑✨

**What it is.** Wire personas into a **production line**: Dwight implements →
Oscar reviews → Pam documents → ship. Each station is a real gate — when Dwight
finishes, his diff is automatically mailed to Oscar; the task card physically
cannot advance until Oscar **approves** (his approval is a structured hive
message, not vibes). A rejected handoff bounces back with Oscar's notes attached.
On the floor you *see* the line: the envelope moves desk → desk → desk, and the
card walks the Kanban columns in lockstep.

**Why clients love it (🪑).** Today, multi-step quality control means manually
re-dispatching between agents and hoping nothing skips a step. A line you define
once — and then just feed tasks into — is how people actually want to run repeated
work (features, bug fixes, release notes). Set up the line, then it's drag-and-drop
(#3) into the *front* of it.

**The edge (✨).** Everyone has parallel agents. **Sequenced agents with enforced
sign-off gates between them** — where the harness, not the prompt, enforces the
sequence — is real separation. It converts "a bunch of agents" into an actual
process, which is what teams buy.

**How it fits.** The Kanban already has **dependencies**; the hive already
routes structured messages between agents; the **HITL gate** already blocks on
hook returns. A line is a task-template whose stages auto-create dependent cards
assigned to the right personas (#4), with the router carrying the diff at each
handoff. The floor animation reuses the existing envelope flight.

**Effort / risk.** Medium. The state machine (advance / reject / bounce-with-notes)
needs care, but every primitive it composes is shipping today.

---

## 6. Done Means Done — proof-gated tasks 🪑

**What it is.** Every task card gets an optional **proof** — a command that must
exit green (`npm run typecheck`, `npm test`, a build) before the card is *allowed*
to move to **done**. The **harness** runs the proof in the agent's worktree; the
agent's claim of "finished!" doesn't count. Failed proof bounces the card back to
*doing* with the failing output attached. Every card that does land in done carries
a **receipt**: the diff summary, the proof output, tokens spent, dollars, and
wall-clock — assembled automatically.

**Why clients love it (🪑).** The #1 trust-killer with agents is being told a task
is done when it isn't. Making *done* a verified state — enforced by the harness,
not the model's self-report — is the single biggest upgrade to "can I actually walk
away?" And the receipt answers the follow-up question ("what did that cost me?")
before it's asked.

**The edge.** Competing tools treat the agent's final message as the result. We
treat it as a *claim* that gets checked. "Done means done" is a one-sentence pitch
that every burned agent user immediately understands.

**How it fits.** The Kanban tracks status; **worktrees** give a clean place to run
the proof; the PTY layer can already spawn commands; the **cost ledger + event
log** supply the receipt's numbers; the **CI watcher** pattern extends naturally
to local proof runs. Bounce-backs reuse the existing work-order handoff.

**Effort / risk.** Low-medium. Run-command-and-gate is straightforward; the main
design choice is sensible default proofs per repo (e.g. detect `npm run typecheck`).

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

## 8. The Vault — secrets desk 🪑

**What it is.** A locked drawer for credentials. You register secrets (API keys,
tokens) once, in the main process. When an agent needs one, it *asks* — a request
lands in the approvals queue ("Dwight wants `STRIPE_TEST_KEY` for repo X"). On
approval, the secret is injected into **that agent's** environment, scoped to that
session — and the live terminal stream + logs **redact the value** everywhere it
appears. Per-persona policy too: *Creed never gets secrets* (see #4).

**Why clients love it (🪑).** Today the honest answer to "how do I give an agent an
API key?" is *paste it somewhere and hope* — it ends up in shell history, transcripts,
memory files. A request → approve → scoped-inject → redact flow removes the most
uncomfortable moment in agent adoption. It's the feature security-conscious teams
ask about before anything else.

**The edge.** Local-first harnesses ignore this entirely. A built-in, visible,
auditable secrets path ("every grant is in the event log") is a serious-tool
signal that pairs with the playful floor instead of fighting it.

**How it fits.** The main process already owns the PTYs (env injection at spawn
or via session restart), the **approvals queue** already handles grant/deny, the
**SQLite durable store** can hold the encrypted vault (OS-keychain wrapped), and
redaction is a filter on the existing `pty:data:<id>` stream + transcript writes.

**Effort / risk.** Medium. Redaction is best-effort by nature (an agent could
transform a secret before printing it) — document that honestly; scoping +
auditing is the real win.

---

## 9. Clock Out — pause the whole office 🪑

**What it is.** One button: **Clock Out**. Every agent is brought to a clean
stop at its next safe boundary (hook return / turn end, not `SIGKILL` mid-write),
in-flight mail stays queued, the roster and task states snapshot, and the floor
dims to "after hours." Later — same machine, after a reboot, on battery in a café —
hit **Clock In**: the team respawns (existing restore path) and each agent gets an
automatic re-brief ("here's the task you were on, here's the last thing you did")
built from its memory and the event log.

**Why clients love it (🪑).** Real life interrupts: laptop lid closes, meeting
starts, battery dies. Today that means killing sessions and reconstructing context
by hand. A safe pause that *keeps the office's place* removes the anxiety tax on
running a big roster — you can walk away mid-mission without losing the mission.

**The edge.** "Suspend the whole multi-agent operation and resume it cold" is an
operational feature competitors don't have because they don't persist enough to
attempt it. We already persist most of it — this finishes the thought.

**How it fits.** **Graceful stop** via hook returns exists; **session IDs, window
state, and the cost ledger already persist** in the durable store; **Restore team**
already respawns a roster. Clock Out chains them: graceful-stop-all → snapshot →
(later) restore-all + a GOD-composed re-brief mailed to each agent from the
event log. The "after hours" floor state is a small scene treatment.

**Effort / risk.** Low-medium. Highest-leverage glue feature on the list — almost
every component exists; the new work is the orchestrated sequence + re-brief.

---

## 10. Branch Offices — federated hives ✨

**What it is.** Link two offices into one company. Your desktop's hive and a
teammate's (or your laptop's) exchange mail: agents address `oscar@scranton` or
`dwight@stamford`, envelopes fly to the door and "arrive" on the other floor, and
each office's GOD coordinates with the other on shared tasks. Cross-office
escalations land in *both* approval queues; the shared blackboard syncs.

**Why clients love it.** The moment two people on a team both run Munder Difflin,
the obvious question is "can my agents talk to yours?" Today the answer is no.
Branch Offices turns a single-player tool into a team tool — *my* docs agent can
ask *your* backend agent a question overnight — without anyone standing up a server.

**The edge (✨).** Multi-human, multi-agent, peer-to-peer, **no cloud middleman**.
Every competitor doing "team agents" routes through their SaaS. Our hive is already
**a local git repo of plain files with a single-committer router** — which means
federation can literally be a git remote: offices push/pull hive state, and the
router treats remote agents as one more inbox to deliver to. The architecture was
accidentally built for this.

**How it fits.** The hive's git-backed, file-based design is the whole trick:
add a remote, namespace agents by office, and extend the router's delivery table
with remote inboxes. The **tunnelmole ingress** we already run covers the
NAT-traversal story for live nudges between syncs. Single-committer-per-office
preserves the no-`index.lock` guarantee.

**Effort / risk.** High — this is the moonshot. Conflict semantics, identity, and
trust between offices need real design. But it's the feature that changes what the
product *is*, and the on-disk architecture gives us a head start nobody else has.

---

## How they stack up

```
            Nobody-else-has-this  ▲
                                  │   10 Branch Offices    1 The Interview
                                  │          ✨                  ✨
                                  │   5 Assembly Line   7 Huddle
                                  │        🪑✨            🪑✨
                                  │   4 Cast
                                  │     🪑✨
                                  │                  6 Done Means Done
                                  │   8 Vault              🪑
                                  │     🪑       3 Drag-assign
                                  │                  🪑     9 Clock Out
                                  │                            🪑
                                  │              2 Org Charts
                                  │                  🪑
                                  └──────────────────────────────────►
                                          Daily ease-of-use value
```

**If we ship in waves:**

1. **Quick wins (low effort, high daily value):** 2 Org Charts → 4 Office Cast →
   6 Done Means Done → 3 Drag-to-Assign. Config, gating, and renderer glue over
   plumbing we already have — and "done means done" is the trust headline.
2. **Walk-away confidence:** 9 Clock Out → 8 The Vault → 7 Conflict Huddle. The
   set that lets people run a big roster unattended without anxiety.
3. **The signature bet:** 5 Assembly Line → 1 The Interview. Sequenced, gated,
   competitive agents — the floor stops being a metaphor and becomes a process.
4. **The moonshot:** 10 Branch Offices. The git-backed hive makes federation
   plausible for us and painful for everyone else — the feature that changes what
   the product is.

---

### Also considered (next round)

- **Undo / Time-machine** — one-click, git-backed revert of an agent's last change
  across its worktree when it goes wrong. Pairs with #7.
- **"Needs You" tray** — one keyboard-driven queue unifying everything blocked on
  the human: approvals, review gates (#5), secret requests (#8), huddles (#7).
- **Visitor badge** — a read-only live share link to the floor so a teammate or
  manager can watch a run without installing anything. A stepping stone to #10.
