# Bugs

## S31 — `7/4` fits none of the four shapes its track admits (2026-09-12, proposed)

`docs/blockchain-track.md` admits four shapes and states that every candidate
is one of them: validator enhancement, chain data processing, end-to-end
program development, and permissionless deployment and delivery. Its
origination table classifies `7/4` as end-to-end program development, while
`specs/7/4-reliable-transaction-dispatcher.md` puts authoring an on-chain
program outside the problem. The lab is a client that dispatches transactions
and reports outcomes, which is none of the four.

The same track states that a phase-7 speed is expressed against the node's own
rate rather than against a wall-clock target, because the local validator fixes
throughput. `specs/7/2-settlement-program-and-client.md` states 200
settlements per second, a wall-clock rate.

### Proposal, needs sign-off

Either the track admits a fifth shape for a chain client that authors no
program, or `7/4` is reclassified and its Scope changed to match. Restating
`7/2`'s speed against the validator's own rate changes what the lab measures.
Both are catalog decisions.

- **Severity:** medium
- **Scope:** phase 7 taxonomy and scale contract
- **Affected:** `docs/blockchain-track.md`,
  `specs/7/4-reliable-transaction-dispatcher.md`,
  `specs/7/2-settlement-program-and-client.md`
- **Source:** `docs/blockchain-track.md:11-22,44,164-167`;
  `specs/7/4-reliable-transaction-dispatcher.md:172-174`;
  `specs/7/2-settlement-program-and-client.md:86-87`
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S30 — `8/3`'s freshness bound can be declared loose enough to remove the scarcity (2026-09-12, partial)

`8/3` claims a corpus its budget cannot cover, so revisiting one page is always
the choice not to visit another. A long enough run with a loose enough
freshness bound satisfied every stated number and removed that shortfall.

2026-09-12: the coverage half is closed. The spec now states that the evidence
window is shorter than one uninterrupted pass at the ceiling, which costs
200,000 / 20 = 10,000 seconds, so no run length satisfies the requirements and
covers the corpus.

What remains is the freshness bound. It is learner-declared, and a loose enough
declaration still removes the revisit pressure that makes the budget contended.

### Proposal, needs sign-off

Bound the admissible freshness declaration so the revisits it forces, plus the
`robots.txt` refetches and the post-error retries, provably exceed the
allowance. That is a curriculum number, so nothing moves until the user
decides.

- **Severity:** medium
- **Scope:** phase 8, acceptance evidence
- **Affected:** `specs/8/3-web-crawl-and-index.md`
- **Source:** CTO audit 2026-09-12; `specs/8/3-web-crawl-and-index.md:112-122,222-235`
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S29 — scripted service times do not determine the replayer latency oracle (2026-09-12, proposed)

The replayer spec derives client latency from scripted target service times.
That script does not determine send delay, transport delay, target queueing,
response arrival, or the outcome of a timeout race. Its exact-percentile check
can therefore disagree with a correct report. Define the independent observation
points, comparable time base, and timeout classification that compute each
event's expected latency; use the service script only for target behavior.

- **Severity:** high
- **Scope:** replayer acceptance evidence
- **Affected:** `specs/6/5-rate-accurate-replayer.md`, `specs/0/5-shared-scaffold.md`
- **Source:** `specs/6/5-rate-accurate-replayer.md:47–56,110–125`; `specs/0/5-shared-scaffold.md:62–69`
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S28 — the market-history hot-key gate lacks a local capacity model (2026-09-12, proposed)

The market-history lab claims its hot-symbol workload falsifies a key design,
but requires DynamoDB Local and declares no capacity-injection contract. AWS
documents that Local ignores provisioned throughput and has no table
partitioning. More generated trades cannot reproduce that hosted failure.
Specify a deterministic capacity model with observable throttling evidence,
or narrow the required lesson to behavior the local dependency models.

- **Severity:** high
- **Scope:** market-history failure gate
- **Affected:** `specs/4/1-market-history-api.md`
- **Source:** `specs/4/1-market-history-api.md:14–24,48–53,73–80,121–125`; `specs/01-systems-labs.md:567–572`; [AWS local usage notes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.UsageNotes.html), opened 2026-09-12
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S27 — exact crash barriers lack an execution hold contract (2026-09-12, proposed)

The shared controller reads barriers from history, while the transfer lab
requires crashes immediately before and after local publication and consumer
progress. An observed event does not hold execution: the process can cross the
next boundary before the controller stops it. Learners own the schema and
topology, so a named progress point is not a universal dependency hook either.
Specify how adapters hold and release each required boundary and how an
unreached or unapplied fault fails the run; prove the contract on the template.

- **Severity:** high
- **Scope:** fault controller and lab observation adapters
- **Affected:** `specs/0/5-shared-scaffold.md`, `specs/1/5-auditable-transfer-service.md`
- **Source:** `specs/0/5-shared-scaffold.md:103–113`; `specs/1/5-auditable-transfer-service.md:27–30,75–85`
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S26 — the phase-4 corpus is not shared and the phase-2 pair drifted (2026-08-29, proposed)

`S23` records the intent as one corpus and one generator, so the phase-4 stores
compare against identical data. They do not. `4/1` states 100 million trades
across 20 symbols, `4/2` states 100 million across 50, and `4/3` states 2
billion across 500, and `4/1`'s prepared generator is described as a
million-trade generator — a hundredth of its own lab's target. `4/2`'s scaffold
names a loader with no tie to the shared recording pipeline.

`2/4` claims the product is identical to `1/5` and then drops three of its
invariants: total value constant across accepted transfers, histories and
balances agreeing, and conflicting idempotency-key reuse failing visibly. It
flags a fourth as platform-forced and these three not at all. `2/3:14`
generalizes `1/2`'s stay-boundary rule to one resource for a half-open
interval and never restates the half that makes back-to-back stays legal.

### Proposal, needs sign-off

Both are design decisions, not typos — they change what a lab teaches. Either
the phase-4 labs share one corpus and one generator, which moves three scale
targets, or the comparison between stores is dropped as an aim. Either `2/4`
restores the three invariants, or it flags each as platform-forced and says
what replaces it.

- **Severity:** medium
- **Scope:** phase 4 corpus, phase 2 pairing
- **Affected:** `specs/4/1-market-history-api.md`,
  `specs/4/2-low-latency-market-api.md`,
  `specs/4/3-exact-trade-analytics.md`,
  `specs/2/3-serverless-reservation-fulfillment.md`,
  `specs/2/4-serverless-auditable-transfer.md`
- **Source:** three parallel spec hunts, 2026-08-29; re-verified 2026-09-12
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S1 — `2/1` may not be falsifiable on the local Lambda emulator (2026-08-14, proposed)

`2/1-metered-billing-api.md` teaches that the execution environment
freezes when the runtime and every extension have completed with no events
pending, so work started before that point resumes under a later caller or
never runs at all. Its required gates run on the local runtime emulator. It is unconfirmed whether that emulator reproduces freeze-at-response
or simply leaves the process running, in which case background work completes
locally and the lab's central belief is never falsified. A learner would pass
every local gate with a design that fails in production.

This is the same defect that disqualified the Firestore emulator from the
catalog: an emulator that is more permissive than the service it stands in for
turns a passing local gate into a false model. If it is confirmed, `2/1` needs
either a fault-controller mechanism that freezes the process at that barrier,
or the lab must move its required gate to the opt-in cloud smoke run, which
breaks the no-account rule for phase 2.

### Feasibility, measured 2026-08-14

The freeze mechanism works in this environment. A container running a loop that
appends a line every 200 ms was frozen with the cgroup freezer through
`docker pause`, held two seconds, and thawed: 8 lines before the freeze, 8
after it, 16 after the thaw. Background work stops and resumes, which is the
behaviour the phase 2 labs depend on. An earlier run of this probe reported
success while producing no output at all, and a second reported failure because
this host's logging driver cannot be read; only the file-observed run counts.

That settles the mechanism and leaves one question: whether the local runner
exposes the moment the runtime and every extension have completed with no
events pending, so the controller knows when to apply it. Freezing at the wrong
instant falsifies a behaviour the service does not have. The remaining work is
a signal, not a mechanism.

### Proposal, needs sign-off

Do not move the required gate to a real account. That breaks the no-account
rule for phase 4, puts credentials in CI where seeded local input is meant to
be the only input, and destroys deterministic falsification: the grader asserts
exact histories at named barriers, and no barrier can be imposed on AWS's own
scheduler.

Reproduce the freeze locally instead. Lambda's freeze is a process freeze, and
process lifecycle with freeze and thaw is already the fault controller's
cheapest mechanism layer. The controller suspends the process at the barrier
and thaws it when the next invocation arrives; killing it instead models an
environment that was destroyed rather than reused.

The barrier is **not** "the handler returned its response", which is what an
earlier draft of this proposal said. The vendor page is precise: Lambda freezes
"when the runtime and each extension have completed and there are no pending
events". A handler can return while an extension is still running, so a
controller built to the response barrier would freeze too early and falsify a
behaviour the service does not have. The barrier is runtime-and-extensions
complete with no pending events, and the same correction applies to every phase
2 spec that says the environment freezes when the response is sent. Background work stops mid-flight, timers return late, and the wall
clock jumps, which are the observable consequences the lab is about.

This makes the emulator's permissiveness irrelevant rather than routing around
it, and `make smoke` keeps its real job: confirming the emulated freeze matched
the service. It is a scaffold change, so it lands in
`specs/0/5-shared-scaffold.md` and needs sign-off before it ships.

Severity rose with the restructure. Four of the five phase 2 labs freeze at
the response barrier in their failure schedules, not just the billing lab; the
import lab (`2/2`) instead depends on the runner's lease and batch contract.
The open question is now also recorded in `specs/0/5-shared-scaffold.md`; the
mechanism proposal below still needs sign-off.

2026-08-14, third review follow-up: `2/1` no longer asserts that the runner
enforces the freeze barrier. Its scaffold section states the lifecycle as a
capability the environment must provide and points at the `0/5` open
question, so no lab claims a capability the scaffold has not established.
The mechanism proposal is unchanged and still needs sign-off.

- **Severity:** high
- **Scope:** phase 2, fault controller
- **Affected:** all of `specs/2/`, `specs/0/5-shared-scaffold.md`
- **Source:** review of phase 1 and phase 4 dependencies, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**
