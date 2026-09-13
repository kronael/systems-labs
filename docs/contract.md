---
status: draft
---

# Systems Labs

## Decision

The repository name is **Systems Labs** and the directory name is
`systems-labs`.

"Assignments" describes work handed in to a teacher. "Labs" describes the
actual learning loop here: run a system, put it under load, break part of it,
recover it, and explain the evidence. The name also leaves room for independent
practice outside a course.

The core curriculum contains seventeen original, end-to-end labs. Ten came from
a scored selection over twenty expanded candidates; two were added afterwards
where the catalog had a gap; three more arrived with the local–serverless split
as serverless recasts of phase 1 products; and two more close the dimensions a
phase 1 orthogonality review found missing — a record shape that changes under
live traffic, and conflicting work the store declines to complete. The
selection record preserves the rejected candidates and reasons, so breadth is
cut by evidence rather than taste. Three separate catalogs — low-level,
blockchain, and retrieval — are recorded under `docs/` and enter only after
the core is `accepted`.

The repository is GPL-3.0, matching `challenges/`. External sources supply
concepts and tool documentation. They do not supply copied assignments,
solutions, fixtures, or checks.

## Contracts at a glance

Each section below fixes one thing. This list is the index, not a substitute.

**Rationale** — **Decision**: seventeen core labs, three further catalogs under `0/`, GPL-3.0, sources cited and never copied. **Problem**: bounded exercises cannot teach boundary failure. **Critique**: the first draft rewarded tool exposure; the fix gave each technology room for its failure model.

**Pedagogy** — **Learning model**: predict, build, test, break, inspect, correct, prove, across three gates whose sum is less than the lab's budget, because the redesign between them is the teaching. **Lab brief contract**: one system, no mechanism named even to disclaim it, difficulty from the quirk, a scale target that is not a pass threshold, named neighbouring systems. **Teaching contract**: separate files hold the task, the hints, the answer key, and the author reasoning, so a learner can avoid the answer without effort; an assistant must be asked twice.

**Harness** — **Standard laboratory scaffold**: one dependency-only Compose profile, runnable before learner code exists; specialized labs extend it. **Fault injection contract**: one shared controller, faults at named barriers only, every scenario declaring what must survive.

**Catalog** — **Selected labs**: seventeen labs, four phases, catalog in `index.md`. **Local and serverless phases**: phase 2 rebuilds phase 1 products on an execution model the learner cannot operate. **Scope boundaries**: no consensus implementation, no console navigation, no parity claim; Lambda only where the brief fixes that shape.

**Technology** — **Technology spine**: a fixed small set, the heavier tool per category, driven into the regime where its quirk fires or the dependency is unearned. **Zero-cost and AWS contract**: every required gate runs locally with no account.

**Proof** — **Adversarial contract**: the learner predicts before the run; verification detects no preferred pattern. **Data contract**: one adapter for real and generated input, decimals stay decimal, schedules name exact records. **Repository contract**: who owns which file, and what `README.md` may never say. **Verification contract**: one Make vocabulary, no hard-coded performance number. **Verification**: no grader binary and no skill — each lab carries `EVALUATION.md`, and only the fault controller and generator stay compiled.

**Provenance** — **Source placement**, **Licence and corresponding source**, **Research ledger**, **Per-lab source map**, **Sources deliberately excluded**: who is credited, what may be copied, and where a recipient obtains the complete source.

**Gate** — **Planned repository boundaries**, **Code pointers**, and the **Approval boundary** that blocks implementation while this spec is `draft`.

## Problem

`challenges/` teaches bounded algorithmic and concurrent problems well, but a
production system fails at its boundaries: the database commits while the
broker publish fails, a consumer sees the same record twice, a replica answers
with stale data, a cold function misses its latency budget, or an autoscaler
makes a backlog worse.

These lessons need persistent processes, real dependencies, failure injection,
load, deployment, and evidence. A correct submission is a useful, running
vertical slice with proved invariants and measured behavior, not a function
that returns one value or a patch to an instructor-owned application.

## Critique of the initial design

The initial draft had the right failure-oriented philosophy but four structural
weaknesses:

- Starting from a complete weak system made the exercise too similar to
  `challenges/`: a learner could repair one decision without learning how the
  ingestion, state, serving, telemetry, and deployment boundaries fit together.
- Sixteen mandatory labs in 70 to 100 hours rewarded tool exposure. Kafka,
  Flink, Kubernetes, Lambda, CRDTs, and OpenTofu each have failure semantics
  that deserve more than a brief tour.
- Generated city rides and optional Wikidata proved mechanics but did not make
  the result intrinsically interesting or useful after grading.
- The technology set covered one retained log but omitted two important
  contrasts: a leased work queue and a transactional but ephemeral database
  notification. It also omitted access-pattern-first and memory-bound NoSQL.

The corrected design cut the catalog to ten labs at that time — it has since
grown to seventeen, as the Decision section records — and gave each retained
technology enough space for its native failure model. Every lab produces a
small complete product and discovers one system's non-obvious contract by
forcing the case where the obvious mental model fails.

## Learning model

Every lab follows the same loop:

1. **Predict** the behavior of the named technology at a boundary case.
2. **Build** a thin but complete path from source through state to a useful
   API, file, alert, or materialized view.
3. **Test** that path across the public process boundary with real dependencies.
4. **Break** one dependency at a named, deterministic boundary.
5. **Inspect** native evidence: SQL state, offsets, queue leases, partition
   keys, traces, histograms, plans, or infrastructure state.
6. **Correct** the design without hiding the technology behind a replacement.
7. **Prove** the invariant after recovery and explain the platform's limit.

Each lab has three depth gates:

- **Product gate** — a 60-to-120-minute end-to-end slice accepts input,
  persists or transforms it, exposes a useful result, and survives restart.
- **Failure gate** — a two-to-four-hour deterministic scenario falsifies the
  naive design and requires the learner to make the stated invariant hold.
- **Evidence gate** — a one-to-two-hour load or deployment run records the
  operational tradeoff and one limitation that remains.

Passing means all three gates pass. The prepared material contains protocols,
dependency bootstrapping, data schemas, generators, and `EVALUATION.md`.
It does not contain an application topology or the path that connects input to
output. The
learner owns the architecture, service boundaries, state design, event
handling, telemetry, and end-to-end tests.

A lab stays completable in its stated hours by obeying a hard scope budget: one
primary architecture problem, at most four public operations, a fixed set of
external dependencies, one main failure schedule, and one small evidence
report. The learner chooses the application process count and internal
boundaries. A second broker, database, UI framework, or cloud service is
admitted only when the comparison is the problem itself.

The labs are conceptually cumulative but mechanically independent. Each
directory contains the public contracts established by its prerequisites and
accepts a fresh implementation. Every lab ends as a complete deployable
product rather than a migration fragment.

The three gates time the passes: the product slice, the falsification run, and
the evidence run. A lab's budget is larger than their sum, because the pedagogy
is the redesign between them — the naive design is falsified and rebuilt,
sometimes more than once — and that loop is where most of the hours go.
Individual labs run six to twenty-five focused hours, so the seventeen core
labs run to roughly 200 to 300 focused hours: the budgets the catalog selects
sum to 203 at the low end and 294 at the high one. The first four establish runtime,
delivery, and change semantics; the fifth makes those semantics span two
systems that cannot commit together; the sixth turns inward, to work the store
declines to complete; the five serverless labs rebuild three of those products
on an execution model the learner cannot operate; the rest integrate them into
real-data, NoSQL, analytical, and portable products.

## Lab brief contract

Each lab is phrased as "design and build a system that does X under these
requirements." The prompt fixes the useful product, public behavior,
technology environment, resource limits, failures, and evidence. It does not
give the service decomposition, database schema, event keys, transaction
boundaries, retry algorithm, cache policy, recovery mechanism, or deployment
topology.

Each brief describes exactly one system. It never splits the product into
named services, and it never names the mechanism that solves it: a brief that
says "outbox", "worker pool", or "write-through cache" has already answered its
own question. Naming a mechanism in order to disclaim it is the same defect
one level subtler — a disclaimed mechanism is still a named mechanism, and a
brief that says it does not prescribe a worker pool has told the learner which
candidates are in play before the learner has thought. A brief instead states
the categories of decision the learner owns — the internal concurrency model,
the data layout, the delivery and recovery path, the process decomposition,
the coordination between components — and names no candidate under any of
them, not even to exclude it. Where a lab needs a second dependency, that
dependency belongs to the fixed environment, not to a second product the
learner must build.

A good task carries the lab; the grading apparatus does not. The measure of a
brief is whether it sends the learner to the primary documentation, to a
post-mortem, to their own experiment on the running system — and whether the
first design they commit to teaches them something when it fails. A lab that
needs elaborate checking machinery to be interesting has the wrong task.
Effort belongs in the task and its quirk.

Every lab is hard, and the difficulty comes from the quirks of the system under
study. The hard part is the boundary case where the obvious mental model is
wrong: a notification that never replays, a lease that is not a deadline, an
index that is not yet consistent. Difficulty must never come from input
formats, parsing chores, obscure APIs, or volume of boilerplate. A lab that is
merely laborious has failed this standard, and so has a lab whose product path
is obvious once the environment is running.

Every lab states a **scale target**: a speed, a load, and an amount. The speed
is a rate the system must sustain, the load is the concurrency or fan-in it
must accept, and the amount is the data volume it must hold or process. These
three numbers fix the size of the problem, so a design that works only at toy
size fails on its own terms rather than on a reviewer's taste.

The scale target is not a pass threshold. Thresholds still follow the
[Verification contract](#verification-contract) and stay relative, calibrated,
structural, or learner-declared, because absolute latency numbers flake across
machines. The scale target says how much work the system faces; the gate says
how well it must behave while facing it.

Where a lab runs against an admission ceiling, one rule decides who sets it.
**The lab fixes the ceiling when the ceiling is an environment fact** it needs
in order to guarantee the regime — the offered rate must provably exceed what
is admitted, and a learner free to raise the ceiling could make the pressure
disappear. **The learner declares the ceiling when choosing it is the design
decision the lab is about**, and then the evidence must report the admitted
fraction, so a ceiling chosen to dodge the problem is visible rather than
rewarded. A phase may contain both; what it may not contain is two labs of the
same kind disagreeing.

Some labs additionally fix the **execution shape**. Where the environment makes
the constraint the lesson, a brief may require the product to run as
event-driven functions that keep state in an external store, with no long-lived
process to hold state between events. Process memory is not absent there — it
survives inside a reused environment and is visible to whoever arrives next —
but nothing guarantees a request reaches a reused environment, so it can be
neither relied on nor treated as private. That is an environment constraint like any
other dependency; it still does not give the learner the state model, the key
design, or the recovery rule.

The shape a brief fixes is the platform's standard execution mode: one
invocation in flight per environment, and an invocation ceiling measured in
minutes. The same vendor lifecycle documentation describes other modes that
admit concurrent invocations in one environment and runs measured in months;
those modes are excluded, because every premise the serverless labs rest on
holds only for the standard mode.

Every lab names its **neighbouring systems**: the two or three technologies a
practitioner would reasonably have reached for instead. The lab does not
compare them for the learner and does not run them.

The naming and the comparison separate. The task carries the names and the
documentation links, because a learner who never learns what the alternatives
are cannot defend the choice the lab forced on them, and a name on its own
solves nothing. The one thing each neighbour does differently at this lab's
boundary is the comparison, and it points straight at the quirk, so it belongs
to `hints/`.

This is also where excluded technologies belong. RabbitMQ, NATS, Cassandra, and
the rest stay out of the dependency set and appear here as reading. A named
neighbour costs the catalog nothing to maintain and gives the learner the
comparison the exclusion policy otherwise denies.

Every submission contains an `ARCHITECTURE.md` that explains system boundaries,
state ownership, invariants, failure model, capacity assumptions, alternatives,
selected tradeoffs, and known limits. Pattern names do not substitute for the
reasoning. `hints/` is the only learner-facing place for optional
solution-bearing guidance.

## Teaching contract

Five artifacts hold five different things, and the separation is what makes the
lab teachable:

- `README.md` — the task and the landscape, and that is all. What the product
  does, what must hold, what evidence must exist, and the names of the two or
  three technologies a practitioner would have reached for instead, with links
  to their documentation. Nothing about how, and nothing about where it breaks.
  Naming a neighbour orients; saying what it does differently at this lab's
  boundary points at the quirk, so that sentence stays in `hints/`.
- `hints/` — the design reading, opened deliberately, one file per hint,
  indexed by `hints/README.md`. What each neighbouring system does
  differently at this lab's boundary, rejected designs, and the
  solution-bearing citations.
- `EVALUATION.md` — what a strong solution looks like and how to check one,
  opened deliberately. It is the answer key: the properties a good design
  holds, the boundaries to observe, the independently computed results a check
  needs, and what separates a working demo from a reliable system. It is
  protected exactly as `hints/` is — by the learner choosing not to open it
  while they are still solving — and by nothing else.
- The fault schedules — the edge cases, as executable failure
  schedules. The learner meets them by running `make fault`, after committing
  to a design. A schedule is never a readable artifact at rest: its seeded
  recipe is compiled into the fault controller binary — Go, per the language
  policy — and a `make fault` run materializes the schedule, runs it, and
  removes it, so a readable form exists only while the run is in flight. What
  the learner receives is an executable that produces the schedule, never a
  source that describes it; the recipe sources stay outside the learner
  distribution with `labs/`, published in the source
  repository that the
  [licence contract](#licence-and-corresponding-source) requires the
  distribution to name. What this achieves is
  deterrence, not impossibility: the learner owns the machine and can read a
  materialized schedule mid-run, disassemble the controller, or fetch the
  recipe sources from the source repository, but each is a
  deliberate act. The standard is the same as `hints/` — recovering the
  answer takes a deliberate act, and nothing hands it over by accident.
- `labs/` — author-facing. The reasoning about what is optimal and why lives
  here, and this directory is not part of the learner tree.

A lab spec's `Architecture questions` section splits across the first two
artifacts. A question stated at the level of the property the design must
defend belongs to the task and publishes into `README.md`. A question that
presupposes a mechanism — where a thing lives, how a component establishes
something — is solution-bearing: it publishes into `hints/`, or it is
rewritten until it names only the property. Each lab spec marks which of its
questions are `hints/`-bound.

The hints are a directory, `hints/`, holding one file per hint, because a
learner who needs one answer should not have to read past four others to reach
it. `hints/README.md` lists what each file answers and nothing more, so the
list itself spoils nothing. Every hint file, and that index, opens with the
exact line `> Spoilers. Open only when stuck.` — the spoiler warning,
including its leading `>`. Every lab `README.md` ends with the exact line
``Stuck? See `hints/`.`` and says nothing else about hints anywhere. That
wording is fixed, identical in every lab, never paraphrased.

**A failure mode is a hint.** The lab `README.md` states the product, the
requirements, the limits, and the evidence, and never says how a design fails,
which boundary it fails at, or what a client sees when it does. Those belong in
`hints/`. Predicting the failure is the learner's first task, so a requirement
written as "the design states what a client sees when X" is the task, and a
sentence that says what goes wrong at X is the answer.

An assistant working alongside a learner in a lab directory acts as a teacher
rather than an answer key. It does not state the approach, name the technique,
or reproduce a hint from `hints/` until the learner has explicitly asked for
one twice. Two separate, explicit requests; frustration and a vague statement
of being stuck are not requests. Before that point it asks guiding questions,
names a concept worth reviewing, or explains why a proposed approach fails —
and it never writes the solving code.

## Standard laboratory scaffold

Every lab starts as a runnable environment before learner code exists. The
standard scaffold contains:

- a dependency-only Docker Compose profile with fixed service names, networks,
  health checks, volumes, and telemetry wiring;
- a learner-owned `compose.yml` for any application topology;
- a Makefile, TOML configuration, generated contracts, data generators,
  source replay, deterministic fault controller, and evidence collector;
- the command vocabulary fixed by the
  [Verification contract](#verification-contract), which is the only list of
  Make targets in this specification.

Specialized labs extend the same harness rather than replacing it. The Flink
lab adds a prepared local Flink cluster and checkpoint storage. The serverless
labs add a Lambda-compatible runner, DynamoDB Local, and — where the brief
fixes a queue — ElasticMQ. The
platform lab adds a prepared `kind` cluster and OpenTofu sandbox. These layers
provide external systems and empty workload slots; they do not choose the
learner's application architecture.

## Fault injection contract

One fault controller is shared by every lab. Labs differ only in their fault
schedules, which are declarative: each names a trigger, a target, and an
effect. Each lab's seeded schedule recipes are compiled into the controller
binary rather than shipped as readable files beside it: the learner
distribution carries an executable that produces a schedule, not a source
that describes one. The recipe sources live outside that distribution,
excluded by the same publish step that excludes `labs/`,
and remain published in the source repository, which the
[licence contract](#licence-and-corresponding-source) requires the
distribution to name as the route to the controller's corresponding source.
`make fault` — unchanged for the learner — materializes the schedule it runs
and removes it when the run ends, and a frozen aggregate digest over every
schedule the controller materializes is checked in CI, so a recipe cannot
drift without the change being declared. The controller is Go, per the
language policy, and it drives the mechanisms below through the container
runtime and the dependencies' own control surfaces.

Faults fire at **named barriers**, never on a timer and never at random. A
scenario says "when record 4711 is acknowledged, freeze the broker", so the
failure lands at the same boundary on every run and verification can assert an
exact history. Random chaos proves nothing twice.

**A barrier holds execution; it is not a thing the controller notices.** An
effect observed after the fact has already happened, and the next effect may
happen during the gap, so a controller that watches a history and then acts
cannot land a fault at an exact boundary. Each barrier is therefore declared
with an adapter that maps it to an observable effect at a public boundary and
holds every path able to cross that boundary until the fault is applied and
confirmed, releasing only afterwards. An atomic effect admits no interior
barrier: a fault lands immediately before it or immediately after it, never
"around" it. A run fails, rather than passes, when a declared barrier is never
mapped, is never reached, is crossed before its fault applied, or when the
fault never activated.

**The controller supplies the limit the environment omits.** An emulator
reproduces a hosted API and leaves out the limits that make the API
interesting, so a lab whose lesson lives in a limit cannot get that limit from
the emulator and must not pretend otherwise. The controller supplies it at
whichever of three layers the limit belongs to:

- **transport** — a proxy between the application and the dependency applies a
  declared capacity budget, a refusal, or a latency schedule, and refuses
  excess with the dependency's own failure shape, applying none of what it
  refused;
- **process** — suspend and resume at a barrier, which models an environment
  that is reused, as distinct from `SIGKILL`, which models one that was
  destroyed;
- **clock** — a time source the run controls, seeded so a run replays, so that
  an expected time is computed rather than measured.

Each lab names the layer its required gate depends on, and says in
`EVALUATION.md` that the gate proves a declared model rather than the hosted
service's own behaviour. Where an account exists, `make smoke` is what compares
the modelled limit against the real one.

The mechanisms, from most deterministic to most realistic:

- **Process and container lifecycle** — `SIGTERM` for graceful shutdown,
  `SIGKILL` for crash, and freeze or thaw to model a stalled node that keeps
  its state and its TCP connections. A freeze is the cheapest way to produce
  the case an operator never expects: a node that is neither alive nor gone.
- **Network** — per-container `netem` for delay, jitter, loss, reorder, and
  bandwidth limits; packet filtering between named container pairs for
  partitions, including the asymmetric and one-way partitions that break
  membership assumptions; and a fault proxy in front of a dependency for
  connection resets, half-open connections, and slow reads that no packet rule
  reproduces cleanly.
- **The dependency's own control surface** — stop one broker to force leader
  election and observe the truncation that follows; add or remove a consumer to
  force a rebalance; shrink retention to make a replay boundary real; pause
  replica replay to create replica lag on demand; terminate a database backend
  mid-transaction. These are the most realistic faults available, because the
  system does its own recovery rather than a harness pretending on its behalf.
- **Clock** — per-container clock skew to separate event time from processing
  time, expire a lease early, and make a timeout fire while work is still in
  flight.
- **Storage** — a fault block device for frozen writes, dropped unflushed data,
  a single `EIO` from one flush, and truncation at an exact offset. This layer
  needs a privileged container and is the one mechanism whose feasibility in
  the target environment is unproven; the low-level track depends on it.

Every scenario declares what must remain true after recovery. A fault that the
system survives without a stated invariant to check is entertainment, not a
gate.

## Selected labs

The core curriculum is seventeen labs across four phases: six local, five
serverless, two on real Internet streaming, and four on NoSQL, analytics, and
portability. Phase 5 no longer exists and its lab became `4/5`; the gap stays
open on purpose.

[`index.md`](index.md#core-catalog) is the single catalog. It names every lab
with its system, architecture pressure, and prepared environment, together
with the selection record that produced it. That mapping is written there and
nowhere else, because a second copy drifts.

## Local and serverless phases

Phase 1 runs software the learner operates. Phase 2 runs three of the same
products on an execution model the learner cannot operate, plus two products
that stand alone. The pairing is the subject: a phase 1 design becomes
unavailable in phase 2, and what breaks names the part that was load-bearing.

This is the single exception to the no-ports rule. A port across *languages* is
forbidden, because it inherits the original's checks, failure schedule, and
answer while adding only a stricter compiler. A port across *execution models*
is the opposite — it confiscates the answer. The product is held constant —
the same public behaviour and the same invariants — while the execution model
changes together with the stores and delivery it is normally paired with,
because holding the phase 1 store fixed moves the lesson to connection
management under an environment count nobody controls, which is a different
lab. The contrast is honest rather than controlled: it swaps one deployable
shape for another and lets the same product expose the difference. The exception extends no
further: no lab may be repeated in another language, and no phase 2 lab may
reuse a phase 1 lab's checks unchanged.

`1/3` has no serverless counterpart, and the reason is recorded in the
[serverless contrast track](../docs/serverless-contrast-track.md). A recast must
falsify something the original could not reach. Replaying a retained log
through metered invocations is the same lesson at higher cost.

## Scope boundaries

- The repository teaches system behavior, not cloud-console navigation.
- Consensus implementation is not a lab. Learners use databases and brokers and
  reason about their guarantees. This avoids turning the course back into an
  algorithm set.
- Kubernetes and AWS Lambda are the two production-shaped targets. Docker
  Compose and `kind` provide the mandatory, free local path.
- Real AWS runs are opt-in `make smoke` checks with an explicit TOML config, a
  maximum resource lifetime, and a documented cost ceiling. No normal test
  creates cloud resources, and the course never promises that an account is
  free of charge.
- Multi-cloud service parity is not claimed. Portability means a stable event
  and application contract plus honest platform adapters.
- Default workloads are generated locally from frozen seeds. Optional external
  data is recorded once by an explicit command, checksummed, cached under
  `${PREFIX:-/srv}/data/systems-labs/sources/`, and never fetched per request.
- Live-source adapters use bounded duration, identify themselves where the
  provider requires it, honor rate limits, record provenance, and surface
  disconnection or truncation. CI and verification use generated or cached input.
- Each lab supplies one supported starter, and the brief names it. Go is that
  starter wherever the environment does not dictate another: the kernel surface
  of phase 6 dictates Rust and C, an on-chain program dictates the chain's own
  language, and a browser bundle dictates TypeScript. A starter is a skeleton
  and a build file, never a second set of checks. Other languages remain
  possible for a learner and unsupported by the course. A menu of starters is a
  decision the brief owes the learner, not an option it hands them.

The Lambda environment is **required**, not offered, in the labs whose brief
fixes that execution shape. There the product must run as event-driven
functions that keep their state in an external store, against the local
Lambda-compatible runner, and a container long-running process is not an
accepted substitute. That constraint is the lesson: cold start, invocation
timeout, batch delivery, and external state stop being tradeoffs the learner
can design away.

Elsewhere Lambda is absent. Long-lived source and worker roles stay in the
container scaffold, because a platform that does not fit a long-lived Kafka
consumer or a database listener teaches nothing by being forced onto one.
Deciding which labs carry the constraint is a curriculum decision recorded in
each brief, not a learner choice.

## Technology spine

The course fixes a small set of environments while leaving their application
architecture open:

- **PostgreSQL** supplies schema and query design, MVCC and isolation, locks,
  transactions, constraints, PL/pgSQL functions, triggers, `LISTEN`/`NOTIFY`,
  query plans, concurrency, backup, and restore.
- **Apache Kafka** supplies partitions, consumer groups, offsets, rebalances,
  retention, replay, compaction, transactions, and observable lag.
- **NATS JetStream** supplies per-message acknowledged delivery with timed
  redelivery, bounded attempts, and stream retention — the leased-queue model
  as software the learner runs, orthogonal to Kafka's retained log.
- **ElasticMQ and optional Amazon SQS** supply visibility, redelivery, delay,
  batching, FIFO options, dead-letter configuration, and Lambda-shaped event
  delivery. This is the same leased model in its hosted shape, which is
  phase 2's subject.
- **DynamoDB Local and optional DynamoDB** supply partitioned key-value and
  document storage, conditional operations, secondary indexes, pagination,
  TTL, and batch APIs.
- **Valkey** supplies expiry, eviction, persistence options, memory limits,
  data structures, pipelining, and behavior under multiple clients.
- **OpenTelemetry and HDR histograms** supply a shared evidence format across
  processes, queues, databases, containers, and functions without fixing the
  service or trace topology.
- **Docker Compose, Kubernetes, and OpenTofu** supply prepared dependencies,
  orchestration, and infrastructure state. Terraform-compatible HCL is the
  portability boundary; Terraform is not a second curriculum.

Flink appears only in recoverable route analytics because event-time state and
checkpoint recovery are its subject. SQL and PL/pgSQL stay in the database
boundary, and Java is confined to the Flink job.

Course-owned code follows one rule: the language is chosen by what the code
must guarantee, not by taste.

- **Python** is the tooling language. Fixture generation, TOML handling,
  evidence reports, provenance manifests, and repository automation are Python, distributed as `uv`-managed projects and PEP 723
  single-file scripts.
- **Go** owns everything that must keep time under load: the open-loop workload
  generator, the fault controller, and the deterministic provider simulators.
  A generator that slows down with the system under test
  destroys the measurement these labs exist to teach, so this boundary is not
  negotiable.
- **Go** is the supported learner starter, except where the environment
  dictates another language.

Language breadth is incidental to system semantics.

Where a category admits one representative, the course picks the heavier one:
the system with the richer failure model, the more surprising operational
contract, and the larger set of things that can go wrong. A smaller tool would
often carry the product, and that is exactly why it is the wrong teaching
choice — a system that cannot fail interestingly cannot be studied. Kafka over
a simple queue, Flink over a hand-rolled stateful consumer, an access-pattern
store over a file: in each case the product is smaller than the tool.

That choice is only legitimate when the lab earns it. A lab that runs an
oversized system at a size the system does not notice teaches configuration and
nothing else, which is the "merely laborious" failure this specification
already rejects. So every lab that reaches for a heavyweight system must drive
it into the regime where its quirk actually fires, and must show that firing in
the evidence: a rebalance observed, a checkpoint restored, an eviction under
memory pressure, a partition running hot, a lease expiring mid-work. The
[scale target](#lab-brief-contract) exists to force that regime. If the naive
small tool would pass the same gates at the same scale, the lab has not earned
its dependency and the scale target is too low.

The task must also be deep enough to use the system to its limit rather than to
its tutorial. A lab that touches only a database's `SELECT` and `INSERT`, or a
broker's produce and consume, has not studied it. Each lab therefore names the
advanced surface its difficulty forces into play — consumer groups and
compaction and retention, materialized views and time-to-live and forced
merges, analyzers and custom scoring and deep pagination, conditional writes
and secondary indexes and pagination cursors — and the requirements are set so
that surface cannot be avoided. Beginner usage passing the gates means the task
is too shallow, not that the design was clever.

A lab may span several technologies when the interaction between them is the
lesson. Retrieval labs pair a search engine with a durable store; the spatial
lab pairs a spatial database with tile serving. The limit is one primary
falsified model per lab, not one dependency: a lab with three systems and three
separate surprises is three labs badly merged.

**ClickHouse** supplies the analytical column store: sparse primary keys that
do not enforce uniqueness, background merges on an unpredictable schedule,
asynchronous mutations, and a hard failure when small inserts outrun merging.
It entered the catalog because its correctness surprise fires under load rather
than under a contrived case.

**OpenSearch** supplies the retrieval surface for the search, retrieval, and
spatial track: analyzers, custom scoring, aggregations, geo queries, and the
gap between a fast candidate set and a correct answer. It is absent from the
core catalog and required in phase 8.

RabbitMQ, Pulsar, Cassandra/ScyllaDB, MongoDB, Temporal,
service meshes, Helm, Pulumi, and CDK are intentionally absent from the
dependency set. Each is useful, but adding a second representative of the same
category gives less learning return than falsifying the guarantees of the
chosen representative. The one category carried twice is the leased queue —
self-run JetStream in phase 1, the hosted SQS shape in phase 2 — and that
duplication is the subject of the
[local and serverless split](#local-and-serverless-phases), not a second tour
of the model. The others appear instead as
[neighbouring systems](#lab-brief-contract) in the labs whose boundary they
would change.

## Zero-cost and AWS contract

Every required gate runs locally without an AWS account. Kafka, PostgreSQL,
Valkey, ElasticMQ, DynamoDB Local, the Lambda Runtime Interface Emulator, and
`kind` provide the executable semantics that can be reproduced for free. Local
emulators are tested only for the behavior they actually model; they do not
stand in for hosted partition capacity, IAM, availability zones, throttling, or
control-plane failure.

As researched on 2026-08-13, AWS advertises one million Lambda requests and
400,000 GB-seconds per month in its free tier, and one million SQS requests per
month. New accounts can choose a free plan that lasts until six months or credit
depletion. These offers do not make a lab intrinsically free: eligibility,
regions, logs, data transfer, DynamoDB use, and pricing can change.

The optional AWS topology therefore uses Lambda, SQS, DynamoDB on-demand, and
short-retention logs only. It excludes MSK, EKS, NAT gateways, RDS,
ElastiCache, provisioned concurrency, and other continuously billed resources.
The smoke config states a maximum invocation count, dataset size, resource
lifetime, region, and dollar ceiling. OpenTofu applies common ownership and
expiry tags and rejects the excluded resource classes. A generated estimate is
evidence, not a guarantee of the provider bill.

## Adversarial contract

Every brief names an observable boundary but does not state what architecture
survives it. Before running a scenario, the learner predicts the behavior of
their own design and identifies the state that would prove the prediction.

Scenarios include a listener disconnect across a database commit, process
death around Kafka progress and an external effect, an acknowledgement window
expiring mid-work, a mixed-success batch, an execution environment frozen —
once the runtime and every extension have completed with no events pending —
with work outstanding, a hot DynamoDB key and paginated read,
Valkey eviction and
concurrent misses, Flink recovery beside an external sink, Kubernetes
termination during work, and OpenTofu drift or a secret sentinel in state.

Evidence contains the prediction, observed native and application state,
explanation, any architecture revision, and the remaining limitation.
Verification accepts any design that satisfies the public contract; it does not
detect a preferred pattern name or private code shape.

## Data contract

Real data enters through the same source adapter as generated data. Route labs
accept the RIPE RIS Live envelope; market labs accept
the Kraken recent-trades envelope. Each adapter produces a versioned internal
record containing source identity, provider event time, local observation time,
provider cursor or sequence when present, raw-record checksum, and provenance
manifest reference. Decimal prices and quantities remain decimal rather than
passing through binary floating point.

The generated source implements the same adapter contract and deliberately
models the ugly cases that a small live sample might not contain. Its TOML
parameters include seed, rate, burst shape, key skew, duplicate ratio,
late-arrival distribution, malformed-record ratio, disconnect boundary, and
cursor overlap. Every failure schedule names exact record identities, so
verification proves histories rather than aggregate counts.

Recording and replay are separate commands. Recording is bounded by duration
and byte count and writes an immutable checksummed object. Replay can preserve
original timing, scale time, or drive an open-loop rate. Normalization retains
the raw checksum, rejected records have an observable outcome chosen by the
architecture, and an upstream schema mismatch fails the ingestion command
rather than silently dropping fields.

The products expose useful results even with generated input: quotes,
reservations, order activity, imports, transfer audit, current routes, churn,
recent trades, and candles. A real recording changes provenance and realism,
not the product contract or pass criteria.

## Repository contract

The repository keeps the verification discipline of `challenges/` without
keeping its unit-sized submission model:

- `README.md` states the task, invariants, limits, interfaces, scale target,
  and acceptance evidence without naming the solution. It carries nothing
  else. In particular the spec's `Adversarial evaluation` section never
  propagates into it: the failure schedule and the edge cases are the lab's
  subject, and a learner who reads them in the task statement has been handed
  the design. `README.md` says which invariants must hold and what evidence
  must exist, never which states can break them. Beyond that, it must not
  name, describe, compare, or rule out any solution method. The ban imports
  the `challenges/` categories verbatim: techniques, data structures, memory
  orderings, recurrences, invariants, implementation shapes, failed
  strategies, naive or brute-force approaches, complexity analysis of
  candidate approaches, and phrases such as "the trick", "the trap", "the
  catch", or "the hard part". The banned invariants are a design's internal
  ones; the product invariants the task must hold are the task itself. It
  states the input limits without explaining how an approach behaves at those
  limits.
- `ARCHITECTURE.md` is learner-owned and records the chosen boundaries, state,
  invariants, failure model, capacity assumptions, and rejected alternatives.
- `hints/` contains architecture guidance, rejected designs, and
  solution-bearing citations, one file per hint, indexed by `hints/README.md`.
  It is opened by choice, never by default, and nothing in `README.md`
  summarizes it.
- `compose.yml` is learner-owned and declares the application processes and
  their connection to the prepared laboratory network.
- `app/` is learner-owned and contains a complete runnable product, including
  ingestion or request entrypoint, state, output boundary, telemetry, and
  graceful lifecycle.
- `tests/` is learner-owned and proves the happy path and recovery path across
  process boundaries. It does not merely call private functions.
- `starter/` contains generated clients, public contracts, dependency wiring,
  and deliberately empty application seams; it contains no end-to-end product
  path.
- **No lab has a worked solution.** No golden implementation, no rotten
  implementation, and no reference design exists for any lab, in the learner
  tree or outside it. A worked systems design answers every
  `Architecture questions` bullet at once, so authoring one creates a document
  whose only protection is that nobody reads it. Verification needs no oracle:
  it asserts invariants over observed histories rather than comparing output
  to a reference run.
- The fault schedules — duplicates, delayed acknowledgements, restarts,
  partitions, recovery — are not lab-directory files and not readable files
  anywhere in the learner distribution. Their seeded recipes are compiled
  into the shared fault controller under the
  [fault injection contract](#fault-injection-contract); `make fault`
  materializes the one it runs and removes it afterwards.
- `sources/` contains provider adapters and provenance manifests, never a live
  network call on an application request path.
- `workload/` generates schema-compatible large inputs from frozen seeds and
  replays bounded cached recordings without storing huge fixtures in Git.
- `EVALUATION.md` is the answer key: what a strong solution holds, how to check
  it, and the independently computed results a check needs. Opened by choice,
  like `hints/`. There is no grader binary. See
  [Verification](#verification).
- `infra/compose/dependencies.yml` starts only the fixed external systems,
  simulators, and telemetry services. Specialized labs add prepared Kubernetes
  and OpenTofu sandboxes under `infra/`; neither contains a solved application.
- `evidence/` is generated and ignored. It contains the run manifest, traces,
  latency histogram, throughput, resource peaks, and the observed history a
  check reads.

Proposed shape:

```text
NN-solution-neutral-name/
  README.md
  ARCHITECTURE.md
  hints/
  EVALUATION.md
  lab.toml
  compose.yml
  app/
  tests/
  starter/
  sources/
  workload/
  infra/
    compose/
      dependencies.yml
    kubernetes/
      sandbox/
    opentofu/
      sandbox/
```

The service boundary is language-neutral. The initial scaffolds use Go for
small services, SQL and PL/pgSQL for database work, and Java only
for the Flink job. Starter code handles protocol generation, dependency
startup, and fixture decoding. The learner still writes the complete path from
input to useful output; a submission cannot pass by editing one isolated
function.

Every lab's product contract contains:

- one learner-owned deployment definition and one command that starts the whole
  local topology;
- one bounded live or generated ingestion path, or one public request API;
- one explicit state model and recovery rule;
- one useful query, materialized view, file, or alert;
- health, graceful shutdown, and structured errors;
- one end-to-end feature test and one deterministic failure test;
- one load or capacity report based on generated data;
- one local deployment definition and an optional platform adapter only when
  the platform fits the workload.

Instrumentation is not a blanket requirement. A lab requires telemetry only
where the measurement *is* the evidence, as in the load, cache, and latency
labs: there the learner must separate waiting from executing, and no report is
possible without it. Elsewhere the lab states which numbers the evidence report
must contain and leaves the method open, because a mandatory tracing stack in
every lab is a tax that teaches nothing about the lab's own quirk. Observability
and alerting as a deployable product is a candidate for its own lab under the
infrastructure-as-code phase, not a tax spread across all of them.

Configuration follows the workspace convention: the first CLI argument is a
TOML lab config; the optional second argument is a secrets file. Environment
variables carry only runtime-injected secrets where a platform requires them.

## Verification contract

Every lab exposes the same root vocabulary:

- `make up` starts the prepared dependencies and the learner's application
  topology; `make down` stops them without deleting cached data. `make up`
  succeeding on an untouched checkout is the proof that the scaffold works.
- `make` formats, builds, lints, and runs the fast test.
- `make test` runs unit and contract tests in under five seconds.
- `make test-all` runs the local integration suite used by CI.
- `make fault` runs deterministic failure and recovery scenarios.
- `make bench` runs seeded load and writes the evidence report and the
  observed history that verification reads.
- `make teaching-lint` enforces the teaching contract mechanically: it fails
  if any lab `README.md` contains a scenario barrier name, a `Neighbouring
  systems` boundary-difference sentence, any citation marked solution-bearing, the name of
  a configuration parameter belonging to a dependency — a setting's name
  names a mechanism, and the task may state only the property the setting
  governs — or a mechanism name at all, whether prescribed, disclaimed, or
  merely mentioned. Ruling a candidate out names it as surely as ruling it
  in, so "no worker pool is required" fails the lint exactly as "use a
  worker pool" does. The same checks cover every course-authored
  learner-facing text except `hints/` and `EVALUATION.md`, which are
  solution-bearing by choice. The check is cheap by construction, because every `Code pointers`
  bullet is solution-bearing and none of them may appear, each lab's
  dependency set bounds the parameter vocabulary to scan for, and the
  mechanism vocabulary is one curriculum-wide list maintained with the lint.
  CI runs it.
- `make source` records a bounded real-data sample into the shared cache and
  writes its URL, retrieval time, cursor or time bounds, checksum, terms URL,
  and adapter version. It reuses an existing matching recording unless refresh
  is explicitly requested.
- `make smoke` runs the live-platform checks against a real account. It is the
  only target that deploys anything and the only one that needs an account;
  `make source` also reaches the network, to record a bounded sample into the
  shared cache. `make smoke` is the check that closes the emulator gap: the local gates run against an emulator that is more permissive
  than the service, so a design can pass every local gate and still violate a
  production limit.

- `make clean` removes named generated artifacts without touching cached source
  data.

`make source` and `make smoke` read a conventional TOML path — `source.toml`,
and `cloud.toml` with `keys.toml` alongside it — and both fail with that path
named when it is absent. The path is a default inside the Makefile, never an
argument the caller types; a target that needs a parameter to do its own job
has the wrong name.

Correctness gates use exact identities and histories, not counts alone.
Acknowledged records must appear exactly once where the contract says one
effect; permitted duplicate delivery remains visible in the history.

Performance gates avoid fixed numbers that flake across machines. A gate uses
one of these forms:

- a relative comparison against the learner's recorded baseline on the same
  host;
- a calibrated local capacity discovered during a warm-up phase;
- a structural bound such as queue depth, open connections, or peak resident
  memory;
- an explicit learner-owned SLO in the generated report.

Latency runs are pinned where possible, warmed, repeated, and recorded with an
HDR histogram. Open-loop generators preserve overload rather than slowing down
with the system. Fault injection uses named barriers or acknowledged offsets so
the failure lands at a reproducible boundary.

CI runs `make test-all` — which carries the fault-recipe digest check — and
`make teaching-lint` in containers. Hardware-sensitive benchmarks run as
release checks, not as arbitrary pull-request pass/fail timers. Cloud smoke
checks never run on untrusted pull requests.

## Verification

There is no grader binary and no grading framework, and there is no skill. Each
lab carries `EVALUATION.md`, and whoever is checking the work reads it — the
learner, a reviewer, or an agent the learner points at it.

`EVALUATION.md` is the answer key and is written as one. It states what a
strong solution holds and how to see that it holds: the observable boundaries,
the invariant each must satisfy, the record selectors a check ranges over, and
— where a lab needs one — the independently computed result a check compares
against, which no invariant shape can supply. It may say what a weak solution
typically gets wrong, because that is the reviewer's job and this file is not
the task.

**A skill was the wrong shape.** An agent loads a skill by default, so a
verification skill discloses its contents to anyone working in the directory,
without the learner ever choosing to see them. `EVALUATION.md` is a file, and
opening it is an act. That is the same standard `hints/` has always had, and
it is the only protection either one gets or needs: a learner who wants the
exercise does not open them.

This is honest about how these labs will be used. A learner working them has an
agent, will use it to build and to check, and is better served by an evaluation
standard they can read than by a binary they cannot.

No part of the judgement is compiled. Two parts of the run still are, because
neither can be described away: the fault controller, which must fire at exact
barriers, and the workload generator, which must hold an offered rate under
load. The run therefore stays reproducible even though the judgement over it
does not.

That is the cost, and it is real: two reviews can disagree where a compiled
checker could not. A lab whose correctness cannot be stated clearly enough in
`EVALUATION.md` for that judgement to be reliable has an unclear invariant,
which is a defect in the lab rather than a reason to build a framework.

## Source placement and attribution

The attribution model is stricter than a source pool:

1. Each lab `README.md` lists dataset provenance and the neighbour
   documentation links, and no quirk source at all. Every `Code pointers`
   citation is solution-bearing: the document that reports the behaviour a
   lab rests on states the behaviour, and stating it is the reading the
   learner is meant to do.
2. Each lab's `hints/README.md` lists the exact architecture and solution sources used by
   that lab, with title, author or project, URL, license, and whether the source
   is cited, adapted, or copied.
3. No source marked "citation only" contributes copied prose, code, fixtures,
   tests, or diagrams.

The repository copies one file, `LICENSE`, and the GPL-3.0 text carries its own
notice. The first commit that copies or embeds anything else adds a root
ledger in the same commit, recording path, upstream revision, copyright
notice, and licence. A dependency that is only run keeps its own licence and
its notice travels with distributed copies.

All lab prose, answer keys, fixtures, and workload generators are
original GPL-3.0 work. Apache-2.0, MIT, MIT-0,
BSD-2-Clause, CC0,
MPL-2.0, and PostgreSQL-licensed dependencies remain under their own licenses.
Their notices stay with distributed copies.

## Licence and corresponding source

The repository is GPL-3.0, and three rules discharge that licence.

**The licence text travels with every form of the work.** The source
repository carries the complete GPLv3 text in a root `LICENSE`, and the
publish step copies that file into the root of every learner distribution.

**The source repository is the complete corresponding source** for everything
the curriculum conveys — prose, answer keys, fixtures, workloads, the
fault controller and its seeded recipe sources, and `labs/` — and it
is public under GPL-3.0 in its entirety. The publish step's exclusions decide
which tree a file lands in, never whether it is published.

**The learner distribution names that repository.** It conveys object code —
the fault controller binary with each lab's recipes compiled in — so under
GPLv3 sections 1 and 6 its recipients must reach that binary's complete
corresponding source, the omitted recipes included. The distribution therefore
names the source repository at its root, beside the licence, as the no-charge
route, and that route stays equivalent in access to the distribution itself.

The teaching contract survives this because its standard is deterrence, not
impossibility. No failure schedule is readable inside the learner tree, and
fetching the source repository to read a recipe is a deliberate act, exactly
like opening `hints/`. A publish step that severed the source route would
not strengthen the teaching contract; it would violate the licence.

## Research ledger

The ledger records the sources consulted on 2026-08-13 and the exact reuse
policy for the repository.

| Source | What it contributes | License evidence | Repository use |
|--------|---------------------|------------------|----------------|
| `challenges/` | Runnable stubs, the problem versus hints split, the README ban list, deterministic seeded workloads, and ephemeral adversarial fixtures | GPL-3.0 and its source-project `NOTICE` | Adapt the repository pattern with attribution; do not copy individual tasks |
| [Cambridge Distributed Systems notes](https://www.cl.cam.ac.uk/teaching/2021/ConcDisSys/dist-sys-handout.pdf) | Failure models, clocks, replication, consistency, transactions, collaboration | Page 1 says [CC BY-SA](https://martin.kleppmann.com/2020/11/18/distributed-systems-and-elliptic-curves.html), but gives no version | Citation only; original explanations avoid an unclear ShareAlike boundary |
| [MIT 6.5840](https://pdos.csail.mit.edu/6.824/schedule.html) | Pedagogical progression from local service to replication and sharding | No reuse license found; the [collaboration policy](https://pdos.csail.mit.edu/6.824/labs/collab.html) restricts solution sharing | Sequence inspiration only; no assignment text, code, tests, or solution structure |
| [CMU BusTub](https://github.com/cmu-db/bustub) | Educational database decomposition and grader discipline | [MIT](https://github.com/cmu-db/bustub/blob/master/LICENSE); README asks users not to publish student solutions | Conceptual reference only; every database lab and its verification is original |
| [PostgreSQL concurrency control](https://www.postgresql.org/docs/current/mvcc.html) | Transactions, MVCC, isolation, locks, deadlocks, serialization failures | [PostgreSQL License](https://www.postgresql.org/about/licence/) | Tool documentation and citation; PostgreSQL runs as an unmodified dependency |
| [PostgreSQL PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql.html) and [`NOTIFY`](https://www.postgresql.org/docs/current/sql-notify.html) | Stored functions, trigger behavior, plan caching, commit-time notifications, payload and queue limits | [PostgreSQL License](https://www.postgresql.org/about/licence/) | Tool documentation and citation; lab code and explanations are original |
| [FoundationDB](https://github.com/apple/foundationdb) | Transaction conflict reasoning, distributed storage, deterministic failure testing | [Apache-2.0](https://github.com/apple/foundationdb/blob/main/LICENSE) | Conceptual reference; no simulator or implementation copied |
| [Apache Kafka](https://github.com/apache/kafka) | Logs, partitions, consumer groups, transactions, replay | [Apache-2.0](https://github.com/apache/kafka/blob/trunk/LICENSE) | Unmodified dependency plus documentation citations |
| [ElasticMQ](https://github.com/softwaremill/elasticmq) | Local SQS-compatible visibility, polling, delay, and duplicate-delivery behavior | [Apache-2.0](https://github.com/softwaremill/elasticmq/blob/master/LICENSE.txt) | Unmodified local dependency; AWS conformance is limited to the exercised contract |
| [NATS JetStream](https://github.com/nats-io/nats-server) | Self-run leased delivery: streams, consumers, acknowledgement, timed redelivery, bounded attempts | [Apache-2.0](https://github.com/nats-io/nats-server/blob/main/LICENSE) for the server; documentation cited without copying | Unmodified dependency plus documentation citations; added 2026-08-14 with the local rewrite of the import lab |
| [Apache Flink Training](https://github.com/apache/flink-training) | Stateful enrichment, windows, timers, exercise-and-test structure | [Apache-2.0](https://github.com/apache/flink-training/blob/master/LICENSE) | API and teaching reference; original domain, jobs, tests, and solutions |
| [Automerge](https://github.com/automerge/automerge) | CRDT-backed JSON state and sync behavior considered for the workspace candidate | [MIT](https://github.com/automerge/automerge/blob/main/LICENSE) | Candidate reference only; the workspace lab is not in the selected catalog |
| [Jepsen](https://github.com/jepsen-io/jepsen) | Histories, fault injection, and invariant checking | The module declares [EPL-1.0](https://github.com/jepsen-io/jepsen/blob/main/jepsen/project.clj); no root license file was found | Citation only; the repository states its invariants over observed histories in an original, narrow vocabulary |
| [CloudEvents specification](https://github.com/cloudevents/spec) | Portable event envelope across HTTP, Kafka, functions, and containers | [Apache-2.0](https://github.com/cloudevents/spec/blob/main/LICENSE) | Public specification and unmodified SDK dependency |
| [AWS Serverless Patterns](https://github.com/aws-samples/serverless-patterns) | Lambda event sources, queues, retries, DLQs, and infrastructure examples | [MIT-0](https://github.com/aws-samples/serverless-patterns/blob/main/LICENSE) | Pattern reference; infrastructure and examples are independently written |
| [AWS Lambda asynchronous error handling](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-error-handling.html) | Current retry and duplicate-delivery behavior | AWS site terms; no open reuse license claimed | Citation only; behavior is verified in optional live smoke tests |
| [AWS Lambda with SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html) | Polling, visibility, at-least-once delivery, batching, and partial batch failure | AWS site terms; no open reuse license claimed | Citation only; local contract tests and adapters are original |
| [DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html) | Offline DynamoDB API development and its boundary from the hosted service | AWS site terms; the download has its own license | Unmodified local dependency; no claim that it models hosted capacity or availability |
| [Valkey documentation](https://valkey.io/topics/) | Data structures, expiry, eviction, persistence, replication, and clustering | [BSD-3-Clause](https://github.com/valkey-io/valkey/blob/unstable/COPYING) for the server; documentation is CC BY-SA 4.0 | Unmodified dependency and citation; lab prose and workload are original |
| [AWS Free Tier](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html), [Lambda pricing](https://aws.amazon.com/lambda/pricing/), and [SQS pricing](https://aws.amazon.com/sqs/pricing/) | Current account plans, credits, monthly allowances, request and duration pricing | AWS site terms; no open reuse license claimed | Citation and cost-model inputs only; the local path remains mandatory because offers and prices change |
| [OpenTofu](https://github.com/opentofu/opentofu) | Providers, state, plans, modules, and Terraform-compatible HCL | [MPL-2.0](https://github.com/opentofu/opentofu/blob/main/LICENSE) | Default unmodified IaC executable; original modules remain GPL-3.0 |
| [Terraform](https://github.com/hashicorp/terraform) | Industry terminology and compatibility target | Terraform 1.6+ uses [BUSL-1.1](https://github.com/hashicorp/terraform/blob/main/LICENSE) | Optional executable only; no Terraform source or documentation is copied |
| [Kubernetes documentation](https://github.com/kubernetes/website) | Probes, rollouts, autoscaling, jobs, and portable deployment | [CC-BY-4.0](https://github.com/kubernetes/website/blob/main/LICENSE) | Citation only; manifests and explanations are original |
| [OpenTelemetry specification](https://github.com/open-telemetry/opentelemetry-specification) | Trace context, metrics, semantic conventions, latency attribution | [Apache-2.0](https://github.com/open-telemetry/opentelemetry-specification/blob/main/LICENSE) | Public specification and unmodified SDK dependencies |
| [HdrHistogram](https://github.com/HdrHistogram/HdrHistogram) | Wide-range latency histograms and coordinated-omission-aware measurement | Dual [CC0-1.0 or BSD-2-Clause](https://github.com/HdrHistogram/HdrHistogram/blob/master/LICENSE.txt) | Unmodified library or format; retain BSD notice when that option is used |
| [BenchBase](https://github.com/cmu-db/benchbase) | Variable-rate database workloads and latency/throughput reporting | [Apache-2.0](https://github.com/cmu-db/benchbase/blob/main/LICENSE) | Benchmark-design reference; generator and workload are original |
| [RIPE NCC RIS Live](https://ris-live.ripe.net/manual/) | Public WebSocket stream of BGP updates and a real source for route-state and event-time behavior | RIPE NCC service terms; no redistribution license claimed here | Citation plus opt-in bounded recording; no captured feed is committed or redistributed |
| [Kraken recent trades](https://docs.kraken.com/api/docs/rest-api/get-recent-trades/) | Public paginated trade ticks and a real source for cursor, rate-limit, precision, and duplicate behavior | Kraken service terms vary by region; no redistribution license claimed | Citation plus opt-in bounded recording for personal use; generated data is the distributable default |

## Per-lab source map

| Labs | Sources that belong in their HINTS or README |
|------|----------------------------------------------|
| 01 | OpenTelemetry, HdrHistogram |
| 02 | PostgreSQL concurrency, PL/pgSQL, NATS JetStream, BusTub, BenchBase |
| 03 | Kafka, PostgreSQL |
| 04 | PostgreSQL, BenchBase |
| 05 | PostgreSQL, Kafka, FoundationDB, Jepsen |
| 06 | PostgreSQL concurrency, FoundationDB, BenchBase |
| 07 | ElasticMQ, DynamoDB Local, AWS Lambda error handling, AWS pricing |
| 08 | CloudEvents, ElasticMQ, Lambda with SQS, DynamoDB Local, AWS Lambda error handling |
| 09 | DynamoDB Local |
| 10 | DynamoDB Local, ElasticMQ, Lambda with SQS |
| 11 | OpenTelemetry, HdrHistogram |
| 12 | RIPE RIS Live, Kafka, OpenTelemetry, HdrHistogram |
| 13 | RIPE RIS Live, Kafka, Flink Training, PostgreSQL |
| 14 | Kraken recent trades, DynamoDB Local, AWS pricing |
| 15 | DynamoDB Local, Valkey, HdrHistogram, OpenTelemetry |
| 16 | Kraken recent trades |
| 17 | CloudEvents, ElasticMQ, Lambda with SQS, DynamoDB Local, AWS pricing, OpenTofu, Terraform license, Kubernetes, OpenTelemetry |

Sources consulted after this ledger's date — the JetStream pages, the Lambda
lifecycle, concurrency, and quota pages, the DynamoDB transaction pages, the
ClickHouse documentation, and the PostgreSQL `ALTER TABLE`, explicit-locking,
and transaction-isolation pages — are carried in each lab's own
`Code pointers` and inherit the same citation-only policy.

## Sources deliberately excluded

- **DeathStarBench** is not a derivation source because its visible licensing
  signals conflict across repository metadata and prose. The curriculum does
  not need it.
- **MIT 6.5840 assignment material and solutions** remain outside the
  repository because no reuse license was found and the course restricts public
  solutions.
- **BusTub student projects and solutions** remain outside the repository even
  though the code license is MIT, respecting its academic-integrity request.
- **Terraform implementation and prose** remain outside the repository. The
  open default is OpenTofu; compatibility tests need only the public HCL and
  provider behavior.

## Planned repository boundaries

- `systems-labs/README.md` — course entry point, paths, prerequisites, and the
  seventeen-lab core catalog.
- `systems-labs/LICENSE` — the complete GPL-3.0 text, copied by the publish
  step into the root of every learner distribution together with the pointer
  to the source repository fixed by the
  [licence contract](#licence-and-corresponding-source).
- `systems-labs/Makefile` — stable root test, fault, benchmark, smoke, and clean
  contract.
- `systems-labs/shared/workload/` — seeded open-loop generators and cached data
  handling.
- `systems-labs/shared/telemetry/` — one trace and histogram contract across
  processes and platforms.
- `systems-labs/shared/faults/` — the fault controller with every lab's
  seeded schedule recipes compiled in, the recipe sources that the publish
  step excludes from the learner distribution, and the frozen aggregate
  digest over the schedules the controller materializes.
- `systems-labs/template/` — runnable dependency skeleton, public contracts,
  and empty application seams. Its trivial domain carries a working
  implementation, because the template is the scaffold's own regression test
  and is not a lab. The publish step excludes `labs/` from the learner
  distribution.

## Code pointers

- [`labs/README.md`](../labs/README.md) — selected specs and lifecycle status.
- [`../docs/lab-selection.md`](../docs/lab-selection.md) — candidate expansion,
  scores, cuts, and selected sequence.
- [`../docs/low-level-track.md`](../docs/low-level-track.md) — the Rust and C
  catalog for kernel-level quirks, recorded and not yet selected.
- [`../docs/blockchain-track.md`](../docs/blockchain-track.md) — the Solana and
  Ethereum catalog, recorded and not yet selected.
- [`../docs/search-and-retrieval-track.md`](../docs/search-and-retrieval-track.md)
  — the OpenSearch catalog for retrieval and spatial quirks, recorded and not
  yet selected.
- [`docs/shared-scaffold.md`](shared-scaffold.md) — the shared
  generator, fault controller, evidence writer, and the build order.
- [`../docs/serverless-contrast-track.md`](../docs/serverless-contrast-track.md)
  — the phase 1 and phase 2 pairing catalog and the rejected pairing.
- [`docs/cloud-access.md`](../docs/cloud-access.md) — how to obtain the
  optional cloud account, the verified free-tier allowances, and the
  zero-spend guardrail that comes before the first deployed function.
- The seventeen entries in the [core catalog](index.md#core-catalog) are the
  governing lab specs. Implementation pointers do not exist while they
  remain `draft`.

## Approval boundary

The draft fixes the name, curriculum, learning contract, repository shape, and
license policy. Creating `systems-labs/`, choosing exact dependency versions,
and authoring lab 01 are implementation work and remain blocked while this spec
is `draft`.
