---
status: draft
---

# Serverless reservation fulfillment

## Brief

Design and build the reservation system from
[`../1/2-reservation-fulfillment.md`](../1/2-reservation-fulfillment.md) again,
on an execution environment that runs one request per instance, executes
nothing between events, and freezes when its runtime and every extension have
completed with no events pending. Clients create reservations with an
idempotency key, request legal state changes, and query status. A reservation claims one resource for a half-open time interval, and
two live reservations for the same resource must never overlap.

The required environment is the function execution model with a partitioned
key-value store as the system of record. The store's data layout, the
enforcement of the non-overlap rule, the path from an accepted reservation to
its fulfillment, and the treatment of repeated requests are the learner's
decisions.

This lab is taken after the local one. The product is identical on purpose —
the same public behaviour and the same invariants. What changes is the
execution model together with the store it is normally paired with, because
holding the local relational store fixed would produce a shape no practitioner
deploys. The pairing is honest rather than controlled; it swaps one deployable
shape for another and lets the same product expose the difference.

## Prepared scaffold

The supplied environment starts the local function runtime, the key-value store,
the fault controller, and a client generator that mixes creations, state
changes, and status queries with a declared contention profile and a published
retry horizon — the longest delay after which a client retries a request. It
also starts
the same fulfillment provider the local lab prepared: an endpoint outside the
learner's handlers that accepts a fulfillment request for a reservation
identity, records it in an inspectable log, and acknowledges it. That log is
the observable boundary of the fulfillment effect — checks read it, and
an effect applied twice is two recorded requests for one reservation. Beyond
client traffic, the runner supports the platform's documented invocation
sources, declared in the lab config; whether the design uses any of them is
open.

The fault controller freezes the environment at the freeze barrier and thaws
it on the next invocation, destroys an environment between invocations, holds
concurrent conflicting requests at one resource, and delays a store response
past a handler's remaining time. The course supplies the client protocol,
and the fault schedules behind `make fault`. The learner owns the handlers and
their tests. No cloud account is required.

## Requirements

No two live reservations for one resource may overlap in time. This holds under
concurrent conflicting requests at the declared contention profile. This
invariant is the lab's study, and the environment here will not hold it for
the design. This lab opens the phase, so it also carries the phase's lifecycle
contract: the freeze at the barrier where the runtime and every extension have
completed with no events pending, environment reuse and the process memory it
exposes, and the invocation ceiling that terminates work rather than
completing it. The design must state what each of the three does to the
invariant, because the later labs in this phase treat that contract as an
environment fact rather than restating it.

Reusing an idempotency key with the same request has one effect; reusing it with
different data fails visibly. The local pairing holds this guarantee without a
time bound; here it is openly weakened to a declared duration, because state
that outlives an invocation must live in the store, where retention is
metered — permanence stops being the free side effect it was beside the local
system of record, and the recast makes the duration a declared, priced design
decision. The weakened guarantee has a floor: the declared duration must
cover at least the published client retry horizon, so a vacuous declaration
fails on the workload itself. The submission states the declared duration and
what a client that retries beyond it observes, and the evidence must show
both.

The fulfillment effect is the request to the prepared fulfillment provider,
recorded there against the reservation's identity. Every acknowledged
reservation requires that effect exactly once; a reservation cancelled before
its effect was applied requires none. Every acknowledged reservation that
requires fulfillment reaches a terminal state, and no fulfillment effect is
applied twice. Work started before a
response is returned may never resume in the same environment; the design states
where unfinished work lives and what causes it to be picked up. Fulfillment
without a worker is a constraint the freeze schedule exercises, not a second
study.

Invalid state transitions must be impossible through every supported write path.
Configuration comes from the standard TOML contract.

The scale target is 500 reservation attempts per second offered against a
concurrency ceiling of 32 environments fixed in the lab configuration, a
contention profile in which one resource in a hundred receives a fifth of all
attempts with overlapping intervals, and a prepared million-item dataset behind
every required query. An environment count is not by itself a rate ceiling:
the rate the ceiling admits is the environment count divided by the invocation
duration. The prepared workload therefore calibrates the mean invocation
duration during its warm-up and sets the offered rate above the admitted rate
that measurement implies, so throttling is reachable in a test run. These
numbers size the problem; they are not pass thresholds. Rejection and latency are judged against the configured
ceiling and the learner's stated service level.

The evidence must report accepted and rejected attempts against the ceiling,
conflict rate at the contended resource, and fulfillment completion after
freezes. How that measurement is produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- which part of the local design became unavailable, and what replaced it;
- how the non-overlap invariant is established, and what that costs at the
  contended resource;
- what the idempotency guarantee promises, for how long, why that duration
  covers the published retry horizon, and what a client that retries beyond
  it observes;
- how a write rejected under contention is distinguished from one rejected by a
  rule, and which of the two may be retried;
- what the concurrency ceiling does to offered load above it, and why the chosen
  rejection behavior is the right one for this product;
- which invariant the local database enforced structurally, and what the
  submission now relies on instead.

The document must compare the local design and this one directly, and state one
property the local version had that this one cannot recover.

## Adversarial evaluation

The failure schedule drives many clients at one resource with deliberately overlapping
intervals, then reads the final reservation set and asserts that no two live
reservations for that resource overlap. It freezes environments at the freeze
barrier with fulfillment outstanding, destroys environments between
invocations, repeats requests with the same and with altered payloads, repeats a
request long after its first acceptance, drives offered load past what the
configured ceiling admits, and delays a store response until a handler
runs out of time.

Checks do not require a named item layout or serialization mechanism. They
observe HTTP, store-visible state, invocation counts, and the submitted
evidence.

## Acceptance evidence

No two live reservations overlap after any injected schedule. Every acknowledged
reservation reaches a terminal state, and no fulfillment effect is applied
twice. Repeated requests inside the declared idempotency window — which covers
the published client retry horizon — have one effect,
and behavior beyond that window matches what the submission declared. Load above
the ceiling degrades the way the design says it will.

The admitted fraction of offered attempts is reported and defended against the
rate the configured ceiling admits — the admitted rate the workload calibrated
during its warm-up. A design that rejects nearly everything has not met the
scale target, and the evidence must make that visible.

The evidence report includes accepted and rejected attempts against the ceiling,
conflict and cancellation rates at the contended resource, fulfillment
completion after each freeze, invocations per accepted reservation, and the
latency distribution separated into environments that were reused and
environments that were created. It names one guarantee the local version held
that this design does not.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Amazon Aurora Serverless v2** —
  [documentation](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html).
  A relational store behind the same functions keeps the exclusion constraint
  that made the local version's invariant structural, but reintroduces
  connection management from an environment count the learner does not
  control.
- **AWS Step Functions** —
  [documentation](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html).
  A workflow service moves the work that outlives a response into a managed
  state machine with its own durability and its own execution limits, so the
  freeze stops being the learner's problem and becomes a service contract to
  read.
- **AWS Fargate** —
  [documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html).
  A container platform keeps the long-lived process and the worker pool, and
  the local lab already showed what that buys. The comparison is what makes
  the cost of this execution model legible.

Read their documentation on transactions, durable execution, and lifecycle. The
lab does not run them.

## Scope

The expected focused time is ten to fifteen hours. The learner builds the
handlers and their tests. The runtime, store, generator, and fault schedules are
prepared. Multi-region behavior, authentication, capacity
provisioning, and a second product surface are outside the problem.

The local reservation lab is a prerequisite, and its artifacts must be
retained: its `ARCHITECTURE.md` and its evidence report. The required
comparison is against that design and its record, not a memory of it; without
those artifacts the required evidence cannot be produced.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, execution
  shape policy, cost, grading, and evidence contracts.
- [`../../docs/serverless-contrast-track.md`](../../docs/serverless-contrast-track.md) —
  why this pairing earns a lab and what the platform removes.
- [`../1/2-reservation-fulfillment.md`](../1/2-reservation-fulfillment.md) — the
  local lab this one recasts.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — the environment freezes when the runtime and each extension have completed
  and there are no pending events, and thaws on the next invocation, so
  unfinished work resumes under another caller or never.
- [Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
  — one in-flight request per environment, and throttling once concurrency is
  exhausted.
- [DynamoDB transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
  — the store's transaction and condition contract, and the limits that bound
  both. Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
