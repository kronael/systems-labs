# Bugs

## S16 — S3's resolution is superseded: no lab has a worked solution (2026-08-23, fixed)

`S3` moved each lab's `golden/` and `rotten/` out of the lab directory to a
repository-root `reference/` tree excluded by the publish step. That was
deterrence, and it left the underlying problem: a worked systems design answers
every `Architecture questions` bullet at once.

The owner's position closes it properly — labs have no worked solution at all.
The reasoning is that a challenge has one correct answer, so a golden reference
is well defined, while a systems lab admits many correct designs, so no
implementation is canonical and "rotten" is one of countless ways to be wrong.
`golden/` was never load-bearing here as it is in `challenges/`, where
`make bench` runs it as the oracle for expected output; this grader asserts
invariants over observed histories and needs no oracle.

Two things replace it. The cited source documenting the real reported behaviour
is the lab's ground truth. `grader/histories/` holds synthetic accept and reject
fixtures per checker, written by hand against the history model and never
recorded from a fault run, which prove the grader works without any design
existing. `template/` keeps a working implementation because it is the
scaffold's regression test and not a lab.

- **Severity:** high
- **Scope:** repository contract, shared scaffold, per-lab authoring
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`,
  `CLAUDE.md`
- **Source:** owner decision, 2026-08-23
- **Status:** fixed
- **Fix:** `reference/` removed from the repository contract, planned
  boundaries, licence, teaching, and fault contracts; `grader/histories/` added
  to the per-lab shape; `0/5` template section rewritten; `CLAUDE.md` golden
  rule replaced, 2026-08-23

## S15 — nothing enforces the earned-dependency rule (2026-08-23, open)

`01-systems-labs.md` states that if the naive small tool would pass the same
gates at the same scale, the lab has not earned its dependency and the scale
target is too low. Nothing checks it. `make teaching-lint` catches solution
leaks in a README; CI proves the golden reference passes every gate and the
rotten reference fails exactly one contract. Neither establishes that a
single-process, single-store design would fail this lab's scale target, so the
rule is an author's promise rather than a gate.

`challenges/` solved the same problem executably: `rotten/` must pass the small
suite and time out on every generated large case, and `make sys-rotten` enforces
that contract at the root. That route is closed here — `S16` removed worked
implementations from labs, so there is no simple design to run and time.

What remains is an authoring check rather than a gate: each lab spec states,
in `Scope`, what a single-process single-store design would fail at this scale
target, and a reviewer confirms the claim is specific enough to be wrong. If
no such sentence can be written, the dependency is unearned. Whether that is
enough, or whether the rule should be dropped rather than left unenforceable,
is the open question.

- **Severity:** medium
- **Scope:** verification contract, shared scaffold, per-lab authoring
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`,
  `CLAUDE.md`
- **Source:** review of the gates against the `challenges/` model, 2026-08-23
- **Status:** open
- **Fix:**

## S14 — phase 3–8 briefs still carry disclaimed-mechanism enumerations (2026-08-14, open)

The S12 rule — a brief states the categories of decision the learner owns and
names no candidate mechanism even to exclude it — was applied to the ten
phase 1 and 2 briefs the fourth review covered. Every phase 3–8 lab brief
still ends with a "does not prescribe" enumeration of candidate mechanisms —
3/1, 3/2, 4/1, 4/2, 4/3, 4/5, 6/1–6/5, 7/1–7/4, 7/6, and 8/1–8/5 — which the
amended brief contract and teaching lint now forbid. Same reasoning, outside
the reviewed scope, so the drift is recorded here rather than changed,
matching S11's treatment of the hour budgets.

- **Severity:** medium
- **Scope:** phases 3–8, brief contract
- **Affected:** all lab specs under `specs/3/`, `specs/4/`, `specs/6/`,
  `specs/7/`, `specs/8/`
- **Source:** fourth external review's S12 applied 2026-08-14, same reasoning
  extended
- **Status:** open
- **Fix:**

## ✅ FIXED 2026-08-14 — S12 — learner-facing briefs name solution methods the teaching contract forbids (2026-08-14, fixed)

Resolved across all ten phase 1 and 2 briefs and both governing documents.
Each brief's "does not prescribe" enumeration is replaced with a statement of
the decision categories that lab leaves to the learner — concurrency and
admission for `1/1`, data layout and the fulfillment path for `1/2`, event
layout and rebuild for `1/3`, and so on — naming no candidate mechanism. The
brief contract in `01-systems-labs.md` now says plainly that a disclaimed
mechanism is still a named mechanism and that a brief states categories only.
`CLAUDE.md`'s spec-conventions rule reversed from requiring the enumeration
to forbidding it. `make teaching-lint` (in `01` and `0/5`) now fails on a
mechanism name in any learner-facing artifact whether it is prescribed,
disclaimed, or merely mentioned, against one curriculum-wide mechanism
vocabulary. The phase READMEs carry no such enumerations and are marked
author-facing; `0/6` keeps its mechanism names as deliberate author-facing
pairing analysis and now opens by saying so. The same pattern in phases 3–8
is outside the reviewed scope and recorded as S14.

The original entry is kept below.

The governing spec says a lab brief never names the mechanism that solves it,
and the repository contract says learner-facing task text may not name,
describe, compare, or rule out a solution method. The lab `Brief` sections
systematically do exactly that in their "does not prescribe" clauses: worker
pools, semaphores, schemas, partition keys, offset policies, relays, cache
policies, retry rules, and similar implementation shapes. `CLAUDE.md` also
requires these enumerations, so the authoring rule contradicts the governing
teaching contract rather than merely violating it in one lab. If `Brief`
publishes into `README.md`, the task leaks the design vocabulary; if that
sentence is stripped, the publication mapping is unspecified.

- **Severity:** high
- **Scope:** teaching contract, phases 1 and 2
- **Affected:** `CLAUDE.md`, `specs/01-systems-labs.md`, all ten lab specs in
  `specs/1/` and `specs/2/`
- **Source:** `specs/01-systems-labs.md:127`,
  `specs/01-systems-labs.md:584`, `CLAUDE.md:189`, and each lab's `Brief`
- **Status:** fixed
- **Fix:** in-tree edits to all ten briefs under `specs/1/` and `specs/2/`,
  `specs/01-systems-labs.md` (brief contract, teaching-lint), `CLAUDE.md`
  (spec conventions, verification contract), `specs/0/5-shared-scaffold.md`
  (teaching lint), and `specs/0/6-serverless-contrast-track.md`
  (author-facing marker), 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S13 — GPL distribution contract withholds the controller's corresponding source (2026-08-14, fixed)

Resolved by specifying the source route rather than reopening the learner
tree. `01-systems-labs.md` gained a "Licence and corresponding source"
section: the source repository — recipe sources, `reference/`, and `specs/`
included — is public under GPL-3.0 in its entirety, so the publish step's
exclusions decide which tree a file lands in, never whether it is published;
the learner distribution carries the GPLv3 text in a root `LICENSE` and, in
the same place, names the source repository as the no-charge
equivalent-access route to the complete corresponding source of the compiled
fault controller. The fault-injection and teaching contracts and `0/5`'s
`shared/faults/` now state excluded-is-not-withheld and add fetching the
source repository as a third deliberate act beside disassembly and mid-run
reads, replacing "the readable-source path is closed". The planned
repository boundaries gained `systems-labs/LICENSE`. The root `LICENSE` file
itself is still an artifact to create, alongside the promised root `README`,
`NOTICE`, and `THIRD_PARTY.md`.

The original entry is kept below.

The repository declares itself GPL-3.0, but the planned learner distribution
conveys a compiled Go fault controller while excluding the seeded recipe
sources compiled into that binary and says the readable-source path is closed.
GPLv3 section 6 requires a conveyed object-code work to be accompanied by, or
offered equivalent access to, its machine-readable Corresponding Source; the
recipe inputs are part of the preferred form needed to generate and modify the
controller. The current tree also declares GPL-3.0 without carrying a root
copy of the license. The default learner tree may omit recipes for accidental-
disclosure deterrence, but recipients must have a clear GPL-compliant route to
the complete source, including those recipes.

- **Severity:** high
- **Scope:** licensing, learner publish step, fault controller
- **Affected:** `specs/01-systems-labs.md`,
  `specs/0/5-shared-scaffold.md`, planned root licensing files
- **Source:** `specs/01-systems-labs.md:26`,
  `specs/01-systems-labs.md:198`, `specs/0/5-shared-scaffold.md:105`, GNU
  GPLv3 sections 1 and 6
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` (licence and
  corresponding source, teaching contract, fault injection contract, planned
  repository boundaries) and `specs/0/5-shared-scaffold.md`
  (`shared/faults/`), 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S7 — `2/1` states an invariant its own schedule no longer falsifies (2026-08-14, fixed)

Resolved by the first option: the redelivery scenario returned to `2/1`'s
adversarial schedule as an environment fact rather than a subject. The grader
now redelivers the queue message carrying usage event 771003 after its work is
acknowledged, with the schedule stating in place that the queue's delivery
contract is another lab's subject and the redelivery is an environment fact
the invariant must survive. The Requirements' no-doubled-charge clause is
falsified again without making the delivery contract this lab's study.

Narrowing `2/1` to one primary model removed its redelivery fault scenario,
because the queue's delivery contract is `2/2`'s subject rather than this
lab's. Its Requirements still say a redelivered message may not double a
charge, so the lab now asserts a property nothing in its own adversarial
schedule tests. Either the scenario returns as an environment fact rather than
a subject, or the invariant moves to where it is falsified.

- **Severity:** medium
- **Scope:** phase 2
- **Affected:** `specs/2/1-metered-billing-api.md`
- **Source:** external review follow-up, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edit to `specs/2/1-metered-billing-api.md` Adversarial
  evaluation, 2026-08-14 (no git in this repository)

## S8 — the concurrency-ceiling standard is applied inconsistently (2026-08-14, open)

`2/3` and `2/4` now fix a concurrency ceiling in lab configuration, 32 and 64
environments, so the offered rate provably exceeds what the platform admits.
`2/5` keeps a learner-declared ceiling with only a relational sizing rule. The
argument for the exception is that `2/5`'s brief makes the declared ceiling a
design surface, and the admitted-fraction evidence requirement guards the
gaming risk. That may be right, but one phase should not carry two standards
without saying which applies when.

- **Severity:** low
- **Scope:** phase 2, scale-target contract
- **Affected:** `specs/2/5-serverless-quote-aggregation.md`, `specs/01-systems-labs.md`
- **Source:** external review follow-up, 2026-08-14
- **Status:** open
- **Fix:**

## S9 — cut `2/5` or fold its unique content elsewhere (2026-08-14, proposed)

Both external reviewers ranked `2/5` last and proposed cutting it. Admission
control — the local quote lab's central subject — is confiscated by the
platform, so the learner's buildable artifact reduces to one handler plus a
declared ceiling. Its distinctive lessons — throttling above the ceiling,
cold-start cost, state outside the process — are already exercised by `2/1`,
`2/3`, and `2/4`. Its one unique item is fan-out amplification per instance:
provider-side load multiplied by environment count rather than shared through
a process-wide pool, which fits elsewhere as a required evidence number.

### Proposal, needs sign-off

Cut `2/5`, or fold the fan-out-amplification measurement into another phase 2
lab as a required evidence number, and record the cut in the serverless
contrast track. Cutting a lab is a curriculum decision; nothing moves until
the user decides.

- **Severity:** medium
- **Scope:** phase 2, curriculum structure
- **Affected:** `specs/2/5-serverless-quote-aggregation.md`,
  `specs/2/README.md`, `specs/0/6-serverless-contrast-track.md`,
  `specs/01-systems-labs.md`
- **Source:** two external reviews, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S10 — `2/1` opens the phase with a two-variable jump (2026-08-14, proposed)

`2/1` is the phase's largest lab — its honest budget is fifteen to twenty-five
hours — and it introduces the execution model and an unfamiliar product at
once. That is exactly the two-variable jump the phase's own ordering rationale
("the recasts that follow vary one thing rather than two") exists to avoid.
Both external reviewers flagged the opening slot.

### Proposal, needs sign-off

Move `2/1` out of the opening slot: open the phase with a recast whose product
the learner already owns, and let the metered billing lab land once the
execution model is familiar. Reordering labs is a curriculum decision; nothing
moves until the user decides.

- **Severity:** medium
- **Scope:** phase 2, lab ordering
- **Affected:** `specs/2/README.md`, `specs/2/1-metered-billing-api.md`,
  `specs/0/6-serverless-contrast-track.md`
- **Source:** two external reviews, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S11 — phase 3–8 time budgets still price the redesign loop at zero (2026-08-14, open)

The phase 1 and 2 Scope budgets were restated honestly on 2026-08-14: the old
figures summed the three gates and priced the falsify-and-rebuild loop —
the pedagogy itself — at nothing, and were short by roughly a factor of two.
Every phase 3–8 lab still carries a gate-sum figure (four to twelve declared
hours), and `specs/0/1-lab-selection.md` still states "fits into three to
eight focused hours" as a selection criterion. The same reasoning applies. The
master's corrected total (150 to 250 focused hours for the sixteen core labs)
anticipates the correction; restating the phase 3–8 figures was outside the
reviewed scope, so the drift is recorded here rather than changed.

- **Severity:** low
- **Scope:** phases 3–8, selection record
- **Affected:** `specs/3/`, `specs/4/`, `specs/6/`, `specs/7/`, `specs/8/`,
  `specs/0/1-lab-selection.md`
- **Source:** external review of phases 1–2 applied 2026-08-14, same reasoning
  extended
- **Status:** open
- **Fix:**

## ✅ FIXED 2026-08-14 — S3 — the worked solution ships inside the lab directory (2026-08-14, fixed)

Resolved as proposed. `golden/` and `rotten/` moved out of the per-lab tree to
a repository-root `reference/<lab>/` pair, excluded from the learner
distribution by an explicit publish step — the same step that excludes
`specs/`. The waiver sentence is replaced with the `challenges/` position: the
worked solution is never shown to the solver. The proposed per-lab shape
dropped both directories, and `0/5`'s `template/` now proves the passing and
the failing reference against `reference/template/`, so CI stops enforcing
the leaky layout while still checking both.

The original entry is kept below.

`specs/01-systems-labs.md` places `golden/` at lab top level, beside the
learner's `app/`, and says in as many words that public availability does not
reduce the learner's required authorship. For an algorithmic exercise that
position is survivable — the golden answer is fifteen lines. For a systems lab
it is not: a golden submission is a complete worked design, which answers every
`Architecture questions` bullet at once. `rotten/` compounds it, because
diffing the two yields the fix and names which contract is the trap.

`specs/0/5-shared-scaffold.md` locks the shape into CI by requiring `template/`
to ship both.

The sibling `challenges/` repository is stricter than this on a far smaller
risk: its own guide says the golden solution is never shown to the solver.

### Proposal, needs sign-off

Move `golden/` and `rotten/` out of the lab directory to a repository-root
`reference/` tree, omitted from the learner distribution by an explicit publish
step. Replace the waiver sentence with the `challenges/` position. Update the
proposed per-lab shape and the `template/` requirement so CI stops enforcing
the leaky layout.

- **Severity:** high
- **Scope:** repository contract, shared scaffold
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` (repository contract,
  per-lab shape, planned boundaries) and `specs/0/5-shared-scaffold.md`
  (`template/`), 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S4 — the failure schedule ships as readable files (2026-08-14, fixed)

Strengthened 2026-08-14 after the third external review, which judged the
first fix nominal: the seeds and generator parameters still shipped readable
under `shared/faults/`, so the schedule was reconstructable from source. The
recipes are now compiled into the Go fault controller binary, the recipe
sources are excluded from the learner distribution by the same publish step
that excludes `reference/` and `specs/`, and what the learner receives is an
executable that produces the schedule rather than a source describing it.
`make fault` is unchanged; the standard remains deterrence rather than
impossibility — disassembling the controller is a deliberate act — but the
readable-source path is closed. Stated in the teaching, fault injection, and
repository contracts of `01-systems-labs.md` and in `0/5`'s `shared/faults/`.

The first resolution is kept below.

Resolved as proposed. The fault schedules are seeded recipes held with the
shared fault controller outside the lab directory, under solution-neutral
names. `make fault` — exactly as available to the learner as before —
materializes the schedule it runs, runs it, and removes it, so the readable
form exists only while a run is in flight. A frozen aggregate digest over
every materialized schedule is checked under `make test-all`, adding no Make
target. Stated in the teaching, fault injection, and repository contracts of
`01-systems-labs.md` and in `0/5`'s `shared/faults/`.

The original entry is kept below.

`scenarios/` holds declarative fault schedules whose triggers are named
barriers, and a barrier name is an edge case stated in words. The master spec
concedes the problem rather than solving it: reading them early is possible,
nothing prevents it, and nothing advertises it. A learner who reads them before
designing has the failure schedule, which the teaching contract itself equates
to being handed the design. The files must be present, because `make fault` is
the learner's own command.

`challenges/` already solved this exact problem and systems-labs has not
adopted the mechanism. Its adversarial fixtures are gitignored; the recipes
live at repository root under neutral names; `make bench` materializes one case
under `tmp/`, runs it, and unlinks it; and an aggregate digest stops the
recipes drifting.

### Proposal, needs sign-off

Adopt that mechanism for `scenarios/`: seeded recipes at repository root, one
schedule materialized per run and removed afterwards, with a digest check.
`make fault` stays exactly as available to the learner; the schedule stops
being a readable artifact.

- **Severity:** high
- **Scope:** teaching contract, fault controller
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` and
  `specs/0/5-shared-scaffold.md`, 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S5 — the task text states the quirk it is meant to hide (2026-08-14, fixed)

Resolved as proposed. The `challenges/` ban list is imported into the
repository contract's `README.md` bullet with its categories quoted verbatim,
including the clause that input limits are stated without explaining how an
approach behaves at those limits. The four announced quirks are stripped to
their properties: `1/2` lost the read-then-write sentence outright, `1/4`'s
timer-lease sentence became plain at-least-once delivery, `2/3`'s
primary-key-condition sentence is gone, and `2/4`'s atomicity-propagation
sentence became an order-and-grouping robustness requirement. The teaching
contract now states where `Architecture questions` publishes — property-level
questions into `README.md`, mechanism-presupposing ones into `HINTS.md` — and
all ten phase 1 and 2 labs were reviewed against the split: `HINTS.md`-bound
questions marked in `1/2`, `1/4`, `2/4`; property rewrites in `1/1`, `1/3`,
`2/1`, `2/3`; the questions in `1/5`, `2/2`, and `2/5` already name only
properties or the announced environment.

The original entry is kept below.

`Requirements` is README content by the repository contract, and in four labs it
announces the falsified belief outright: that change records from one atomic
write may interleave so the consumer must not assume atomicity; that conditions
apply to items named by primary key and transactions cannot be conditioned on a
query; that the acknowledgement window expires on a timer so a slow consumer is
handed a duplicate; and that a design which reads then writes must explain the
interleaving. The last one names the naive approach and its failure, which
`challenges/` bans explicitly.

`Architecture questions` compounds it. The section is learner-facing by
construction, and several of its bullets are leading questions that presuppose
the answer — how startup avoids a missed-change window, how workers establish
ownership, where an exhausted record lives, where the progress of a period
close lives. The teaching contract lists four artifacts and never says which
one holds this section.

### Proposal, needs sign-off

Import the `challenges/` ban list into the repository contract, including its
clause that limits are stated without explaining how an approach behaves at
them. Strip the quirk from the four `Requirements` sections. Split
`Architecture questions`: property-level questions stay with the task,
mechanism-naming ones move to `HINTS.md` or are rewritten to name only the
property.

- **Severity:** high
- **Scope:** teaching contract, phases 1 and 2
- **Affected:** `specs/1/2`, `specs/1/4`, `specs/2/3`, `specs/2/4`, `specs/01-systems-labs.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` and to `1/1`, `1/2`,
  `1/3`, `1/4`, `2/1`, `2/3`, `2/4`; the questions in `1/5`, `2/2`, and `2/5`
  were reviewed and already name only properties or the announced
  environment, 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S6 — nothing enforces the teaching contract (2026-08-14, fixed)

Resolved as proposed. `make teaching-lint` joined the verification contract
in `specs/01-systems-labs.md` and gained a section in
`specs/0/5-shared-scaffold.md`: it fails if any lab `README.md` contains a
scenario barrier name, a `Neighbouring systems` product name, or any citation
marked solution-bearing, and CI runs it. The check stays cheap because every
`Code pointers` bullet in phases 1 and 2 already carries an explicit neutral
or solution-bearing state.

The original entry is kept below.

No gitignore rule, CI check, naming lint, or publish step exists anywhere in the
specification. Every protection in the teaching contract is a sentence asking an
implementer to behave. `S3` is worse than unenforced — the waiver makes
non-enforcement the stated policy.

### Proposal, needs sign-off

Add one root target, `make teaching-lint`, run in CI: fail if any lab
`README.md` contains a scenario barrier name, a `Neighbouring systems` product
name, or any citation marked solution-bearing. It is cheap once every citation
carries an explicit neutral or solution-bearing marker, and it is the single
change that turns the contract from aspiration into a gate.

- **Severity:** medium
- **Scope:** verification contract
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` (verification
  contract) and `specs/0/5-shared-scaffold.md` (Teaching lint), 2026-08-14
  (no git in this repository)

## ✅ FIXED 2026-08-14 — S2 — `1/4` is a cloud-native lab filed in a local open-source phase (2026-08-14, fixed)

Resolved by the phase 1 and phase 2 split rather than by either option below.
`1/4` moved to `2/2`, and phase 1 gained a new `1/4` on NATS JetStream, so the
local phase now holds no hosted-service emulator at all. The split itself is
recorded in `specs/0/6-serverless-contrast-track.md` and stated in
`specs/01-systems-labs.md`.

The original entry is kept below because its reasoning produced the split.



The catalog follows an unwritten split. Phases 1, 3, 6, 7, and 8 run the actual
software — PostgreSQL, Kafka, Flink, OpenSearch, PostGIS, a local validator, a
local Ethereum node, the kernel itself — so there is no emulator and no gap,
and a cloud version would be the same software under someone else's operation.
Phase 4 is the one phase where a hosted service's semantics are the lesson, so
it is the only phase with an emulator gap and the only one that needs an
account.

`1/4-reliable-record-import.md` violates that split. It requires the SQS API,
the Lambda batch event shape, and the DynamoDB API, which makes it three
emulators — two of them JVM — and phase 1's only cloud touchpoint.

### Proposal, needs sign-off

First, state the split in `01-systems-labs.md`, because it is a real
organising principle that nothing currently records.

Then fix `1/4`, one of two ways:

1. Move it to phase 4. Cleanest boundary, but phase 1 loses the leased-queue
   delivery model, and the contrast between an ephemeral notification, a
   retained log, and a lease is the phase's spine.
2. Reground it on NATS JetStream, which runs natively and keeps the lesson.
   Phase 1 keeps its three delivery models and drops two emulators; phase 4
   keeps the SQS and Lambda versions, where hosted semantics belong.

RabbitMQ is not a substitute, despite being the obvious candidate. SQS holds a
timer lease: the visibility timeout expires on the clock, so a merely slow
consumer is handed the same message again while it is still working. RabbitMQ
redelivers on consumer liveness — a channel drop or a nack — so slowness costs
nothing, and `consumer_timeout` kills the channel rather than expiring one
message's lease. That is a different failure model, and `1/4` would falsify a
different belief.

JetStream keeps it: `AckWait` is a real timer that triggers redelivery on
expiry, and `MaxDeliver` caps attempts. Dead-lettering is assembled rather than
configured — `MaxDeliver` stops redelivery but leaves the message in the
stream, and the queue is built by consuming the max-deliveries advisory
subject.

The result store does not need to be NoSQL. It only has to make valid records
queryable with a per-identity fate, which PostgreSQL already does and phase 1
already runs. NoSQL data-model lessons are contract-shaped and belong to
phase 4.

Recommendation: the second. It fixes the principle rather than relocating the
gap.

- **Severity:** medium
- **Scope:** phase 1, phase 4, curriculum structure
- **Affected:** `specs/1/4-reliable-record-import.md`, `specs/01-systems-labs.md`, `specs/1/README.md`
- **Source:** review of local versus hosted dependencies, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S1 — `4/4` may not be falsifiable on the local Lambda emulator (2026-08-14, proposed)

`4/4-subscription-billing-api.md` teaches that the execution environment
freezes at the response, so work started before returning resumes under a
later caller or never runs at all. Its required gates run on the local runtime
emulator. It is unconfirmed whether that emulator reproduces freeze-at-response
or simply leaves the process running, in which case background work completes
locally and the lab's central belief is never falsified. A learner would pass
every local gate with a design that fails in production.

This is the same defect that disqualified the Firestore emulator from the
catalog: an emulator that is more permissive than the service it stands in for
turns a passing local gate into a false model. If it is confirmed, `4/4` needs
either a fault-controller mechanism that freezes the process at the response
barrier, or the lab must move its required gate to the opt-in cloud smoke run,
which breaks the no-account rule for phase 4.

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
- **Status:** open
- **Fix:**
