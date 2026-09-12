# Bugs

## S32 — the emulator is the wrong place to look for the limit (2026-09-12, proposed)

`S1`, `S27`, `S28` and `S29` are one defect in four places, and all four are now
settled as NOT REPRODUCIBLE against primary sources. Each lab's lesson lives in
a limit — a freeze, a capacity ceiling, an exact crash boundary, a latency
distribution — and each required gate asks a local emulator to produce it. An
emulator reproduces the API and omits the limit. AWS's own emulator implements
freeze as a metrics stub, and AWS's own page says the local store ignores
provisioned throughput, has no table partitioning, and throws no transaction
conflict.

The reframe: **the emulator owes the API, and the controller owes the limit.**
That is the split three public projects already build on, and each one answers
a different layer of what `specs/0/5-shared-scaffold.md` needs.

- **Transport.** Shopify's Toxiproxy is "a TCP proxy to simulate network and
  system conditions for chaos and resiliency testing"; its toxics "manipulate
  the pipe between the client and upstream", are added and removed over an HTTP
  API, and it is built "to work in testing, CI and development environments,
  supporting deterministic tampering with connections". Its toxic types include
  bandwidth, latency, timeout, reset_peer and limit_data.
  <https://github.com/Shopify/toxiproxy>, read 2026-09-12.
- **Verification.** Jepsen's Maelstrom is "a workbench for learning distributed
  systems by writing your own". It injects partitions, node kills and process
  pauses, and its checkers read operation histories to find safety violations
  up to strict serializability, generating a minimal example of each anomaly.
  It is the working proof of the stance this repository already states — assert
  invariants over observed histories, never compare against a reference run —
  and it is a teaching tool, which makes it the closest neighbour this
  curriculum has. <https://github.com/jepsen-io/maelstrom>, read 2026-09-12.
- **Clock.** TigerBeetle's VOPR, after FoundationDB's simulator, goes further:
  "In the simulator, all non-deterministic parts of the system are stubbed out.
  This includes the clock, network, and disk operations", and a run is
  "deterministic based on a seed number and the Git commit", so a failure
  replays exactly.
  <https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md>,
  read 2026-09-12.

### Proposal, needs sign-off

Give the shared controller three layers, and let each finding name the layer it
needs rather than asking its dependency for a behaviour the dependency does not
have.

1. **Transport** — a proxy in front of every dependency, so a capacity ceiling,
   a refusal, or a latency schedule is applied where the emulator declines to.
   Closes `S28`, and supplies the stall `S29` needs.
2. **Process** — suspend and resume at a named barrier rather than kill, so a
   freeze is a freeze and not a destroyed environment, and so the hold happens
   before the boundary is crossed rather than after it is observed. Closes `S1`
   and `S27`.
3. **Clock** — a time source the run controls and a seed that replays it, so an
   event's expected latency is computed rather than measured. Closes `S29`.

This is one decision, and it decides four findings. It is also the decision the
repository already half-states: its project memory records that an emulator
omits the limits, so a lab whose lesson lives in a limit needs the controller
to supply it. What is missing is the contract, not the belief.

- **Severity:** high — it gates every required gate in phases 2, 4 and 6
- **Scope:** shared scaffold, fault controller
- **Affected:** `specs/0/5-shared-scaffold.md`, and through it `S1`, `S27`,
  `S28`, `S29`
- **Source:** the four settled findings below; Toxiproxy, Maelstrom and the
  TigerBeetle VOPR, each read 2026-09-12
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S31 — the phase-7 taxonomy has no shape for a client that authors no program (2026-09-12, proposed)

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

The taxonomy is the incomplete half, not the lab: dispatching transactions and
establishing their outcome is real phase-7 work that authors no program. Admit
a fifth shape and keep `7/4`'s scope. The track's opening sentence becomes:

> Phase 7 covers Solana and Ethereum through five shapes: validator
> enhancement, chain data processing, end-to-end program development,
> permissionless deployment and delivery, and chain transaction clients that
> submit transactions and establish outcomes without authoring a program.

`7/4`'s rows then read "Chain transaction client".

For `7/2`, replace "offered open-loop at 200 settlements per second with 64 in
flight" with a rate the node fixes: 0.5 x r / m settlements per second with 64
in flight, where a separate calibration on the same host measures r confirmed
transactions per second and m transactions per settlement for the same seeded
workload and confirmation level, and the calibration holds fixed for the
evidence run. The learner then reports completed settlements against offered
load, utilisation as s x m / r, and transaction amplification by settlement
size. It costs one calibration run.

Both are catalog decisions, so nothing moves until the user decides.

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

2026-09-12, arithmetic: my own wording above is too strong, and a bound that
simply exceeds the allowance would make the lab's promise infeasible rather
than scarce. The rewrite rate is 200,000 x 0.06 / 3,600 = 3 1/3 pages per
second against a ceiling of 20 x 1 = 20 requests per second, so revisiting
every rewritten page still leaves 16 2/3 requests per second. Contention is not
forced by the stated numbers, and no single duration follows from them.

### Proposal, needs sign-off

Make the declaration itself carry the constraint, rather than picking a number.
The learner declares a freshness bound F inside the evidence window. A prepared
overlap window requires K updates to already-served pages, R `robots.txt`
requests, and E post-error retries before that bound expires, each positive.
Let U count the eligible unfetched pages competing for the same allowance, and
A(F) the host's available request slots under the declaration. The declaration
is admissible when

    K + R + E <= A(F) < K + R + E + U

so the promised work fits and the corpus does not. At a constant grant g that
reads (K+R+E)/g <= F < (K+R+E+U)/g; under the fault windows the recorded
allowance supplies A(F). The evidence then names the pages that lost service.

This costs the learner an explicit spending argument, and it costs the scaffold
author the prepared demand counts. It is a curriculum decision, so nothing
moves until the user decides.

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

### Settled 2026-09-12 — NOT REPRODUCIBLE from the script

The scripted service times cannot determine the answer, because the host adds
delay the script does not model. The Linux manual states it plainly:
"Furthermore, after the sleep completes, there may still be a delay before the
CPU becomes free to once again execute the calling thread", and that an
interval "will be rounded up to the next multiple" of the clock's granularity
(<https://man7.org/linux/man-pages/man2/nanosleep.2.html>, read 2026-09-12).

The contract this implies: observe the due time, the send time, and the arrival
of the response or the refusal independently, on one monotonic time base that
the run verifies is common; compute each event's latency as terminal time minus
due time, with the due-based deadline winning ties. The target's acceptance is
held after the named acknowledgement and released after the declared stall
while the replayer keeps running. Service times then describe the target only.

- **Severity:** high
- **Scope:** replayer acceptance evidence
- **Affected:** `specs/6/5-rate-accurate-replayer.md`, `specs/0/5-shared-scaffold.md`
- **Source:** `specs/6/5-rate-accurate-replayer.md:47–56,110–125`; `specs/0/5-shared-scaffold.md:62–69`
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S28 — the local store produces neither the capacity limit nor the conflict (2026-09-12, proposed)

The market-history lab claims its hot-symbol workload falsifies a key design,
but requires DynamoDB Local and declares no capacity-injection contract. AWS
documents that Local ignores provisioned throughput and has no table
partitioning. More generated trades cannot reproduce that hosted failure.
Specify a deterministic capacity model with observable throttling evidence,
or narrow the required lesson to behavior the local dependency models.

2026-09-12, re-verified against the vendor page: it is worse than throughput.
AWS states "Provisioned throughput settings are ignored in downloadable
DynamoDB", "the speed of read and write operations on table data is limited
only by the speed of your computer", "when you run DynamoDB locally, there is
no table partitioning", and "TransactionConflictExceptions aren't thrown by
downloadable DynamoDB for transactional APIs".

That last sentence pulls in a second lab. `2/3` requires the evidence report to
carry "conflict and cancellation rates at the contended resource"
(`specs/2/3-serverless-reservation-fulfillment.md:157-158`) and asks the
learner to distinguish a write rejected under contention from one rejected by a
rule (`:118-119`). A submission that establishes non-overlap with a
transactional write records a conflict rate of zero locally and a non-zero one
on the service, while a submission built on a conditional write sees its own
rejections. Two correct designs are then judged on different evidence, and the
required number measures the store rather than the design.

The contract this implies: observe the real table and index keys, the operation
type and the item size; hold each request before dispatch while applying a
declared partition mapping and a declared capacity budget; forward what the
budget admits and refuse the rest with the store's own throttling outcome,
applying none of the refused writes. Injection ends at its named release
boundary, and a run where it never activates fails. That proves a declared
capacity model, not the hosted service's partition adaptation, and the lab must
say so.

- **Severity:** high
- **Scope:** market-history failure gate
- **Affected:** `specs/4/1-market-history-api.md`,
  `specs/2/3-serverless-reservation-fulfillment.md`
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

### Settled 2026-09-12 — NOT REPRODUCIBLE under observation alone

The kernel states that the freezer is asynchronous: "Freezing of the cgroup may
take some time; when this action is completed, the 'frozen' value in the
cgroup.events control file will be updated to '1' and the corresponding
notification will be issued"
(<https://docs.kernel.org/admin-guide/cgroup-v2.html>, read 2026-09-12).

Reading a barrier from a history and then suspending therefore cannot establish
an exact boundary: execution advances during the gap. The contract this
implies: each named boundary maps to an observable effect through an adapter,
and the adapter holds every path that could cross that boundary before it is
crossed, releasing only after the crash and the prescribed recovery. A barrier
that is never mapped, never reached, crossed, or never applied fails the run.
An atomic effect gets no invented intermediate crash point.

- **Severity:** high
- **Scope:** fault controller and lab observation adapters
- **Affected:** `specs/0/5-shared-scaffold.md`, `specs/1/5-auditable-transfer-service.md`
- **Source:** `specs/0/5-shared-scaffold.md:103–113`; `specs/1/5-auditable-transfer-service.md:27–30,75–85`
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

### Settled 2026-09-12 — NOT REPRODUCIBLE

The emulator does not freeze. AWS's own Runtime Interface Emulator implements
`LocalSupervisor.Freeze` as a metrics stub, read at commit `c622c06`:

    func (s *LocalSupervisor) Freeze(ctx context.Context, req *model.FreezeRequest) (*model.FreezeResponse, error) {
        // We return mocked freeze/thaw cycle metrics to mimic usage metrics in standalone mode

It returns `CycleDeltaMetrics` and suspends nothing. Work outstanding when the
handler returns keeps running locally, so the belief this lab exists to falsify
is never falsified on the required gate. Source:
<https://raw.githubusercontent.com/aws/aws-lambda-runtime-interface-emulator/c622c06822a4e5390cc71f29df37594646b10700/internal/lambda/supervisor/local_supervisor.go>,
read 2026-09-12.

The contract this implies: observe the invocation-scoped `Next` from the
runtime and from every registered extension with no pending events — the
condition the vendor states — then freeze the whole environment before any
further work, confirm the suspension, and thaw only when the next invocation is
assigned, or destroy the environment instead. A run where completion or
suspension never arrives by the deadline fails rather than passes.

- **Severity:** high
- **Scope:** phase 2, fault controller
- **Affected:** all of `specs/2/`, `specs/0/5-shared-scaffold.md`
- **Source:** review of phase 1 and phase 4 dependencies, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**
