---
status: reference
---

# Serverless contrast track

This file is author-facing. Like everything under `specs/`, it is not part of
the learner tree, and its pairing analysis is solution-bearing on purpose:
naming what each phase 1 answer rested on is the analysis. Nothing in it may
be quoted into a learner-facing artifact.

## Decision

Phase 1 runs software the learner operates. Phase 2 runs the same problems on
an execution model the learner cannot operate. The pairing is the lesson: a
design that was correct in phase 1 becomes unavailable in phase 2, and the
learner discovers which part of it was load-bearing only because the platform
removed it.

This is the one place in the curriculum where reusing a product is deliberate.
Everywhere else a repeated problem is forbidden, because a port inherits the
original's checks, failure schedule, and answer. A port across *languages*
inherits the answer and adds a stricter compiler. A port across *execution
models* confiscates the answer. What is held constant is the product — the same
public behaviour and the same invariants. What changes is the execution model
together with the stores and delivery it is normally paired with. Holding the
phase 1 store fixed is deployable — functions with a relational store are a
supported and documented shape — but it moves the lesson to connection
management under an environment count the learner does not control, which is a
different lab from the one each pairing is for.
The contrast is honest rather than controlled: it does not isolate one
variable, it swaps one deployable shape for another and lets the same product
expose the difference.

## What the platform takes away

Every phase 2 lab loses the same four things, and each phase 1 answer rests on
at least one of them:

- **The long-lived process.** A worker pool, a background thread, or an
  in-process scheduler may survive inside a reused environment, but none of
  them runs while the environment is frozen and none is guaranteed to exist
  when the next request arrives. Work between events has nowhere it is
  guaranteed to run.
- **The held connection.** A connection can outlive one request in a reused
  environment, but it is amortized across only that environment's requests.
  Each environment opens its own, and a burst opens many at once.
- **Process memory as state.** Memory survives inside a reused environment and
  is visible to whoever arrives next, but nothing guarantees any given request
  lands in a reused environment, so an in-process cache, an accumulated
  counter, or coordination through shared variables can neither be relied on
  nor treated as private.
- **Control of concurrency.** Admission is a platform setting rather than an
  application decision. Above the ceiling the platform throttles; it does not
  queue on the learner's terms.

And it adds one thing that has no phase 1 equivalent: the environment
**freezes once the runtime and every extension have completed with no events
pending** — a handler can return before that point — and thaws under a later
caller, so unfinished work resumes in a stranger's invocation or never runs
at all.

## The pairings

| Local | Serverless | What the platform removes | What the learner must build instead |
|-------|-----------|---------------------------|-------------------------------------|
| [1/1 quote service](../specs/1/1-resilient-quote-service.md) | [2/5 quote aggregation](../specs/2/5-serverless-quote-aggregation.md) | The long-lived process, its shared connections, and admission control the design owns | External shared state, a quote expiry that runs on across a freeze the design does not control, and provider fan-out that scales with environment count rather than with a pool the service sizes |
| [1/2 reservation fulfillment](../specs/1/2-reservation-fulfillment.md) | [2/3 serverless reservation](../specs/2/3-serverless-reservation-fulfillment.md) | The commit-time listener, the worker pool, and a store able to enforce non-overlap itself | A hand-built exclusion rule over a store that cannot express one, and an event-driven fulfillment path |
| [1/5 auditable transfer](../specs/1/5-auditable-transfer-service.md) | [2/4 serverless transfer](../specs/2/4-serverless-auditable-transfer.md) | Any process that outlives a request to carry committed state downstream | A commit gap closed by events, across a freeze that can strand the publish |

[2/1 metered billing](../specs/2/1-metered-billing-api.md) has no phase 1 partner by
design. It lands once the execution model is familiar, because a product the
learner has never built and an execution model they have never used are two
variables at once. The phase opens on a recast instead, which varies the
execution model alone and carries the lifecycle contract the later labs treat
as an environment fact.

[2/2 reliable record import](../specs/2/2-reliable-record-import.md) lost its partner
when phase 1 merged the standalone import lab into
[1/2](../specs/1/2-reservation-fulfillment.md). No phase 1 lab now holds that product.
What survives is the model contrast, which was always the point of the pairing:
`1/2` runs a leased delivery the learner operates, and `2/2` receives the same
model as a hosted contract with the poller and the batch outcome supplied. The
product is no longer held constant there, so `2/2` is a model contrast rather
than a recast, and the table above is the set of true recasts.

The import pairing also revisits the selection record's cut of the NATS
candidate. That cut reasoned that a third messaging product adds vocabulary
while SQS carries the leased model; the split reassigned the model instead —
phase 1 now studies it as software the learner runs, phase 2 as a hosted
contract.

## The pairing that was rejected

`1/3 order activity dashboard` has no phase 2 counterpart. Its lesson is that a
view can be rebuilt by replaying a retained log, and the serverless version
neither sharpens nor complicates that: it replays the same events through
metered invocations, so the exercise gets more expensive without getting more
instructive. The managed broker this would need is also excluded by the cost
policy. A recast must falsify something the original could not reach; this one
only relocates the original.

## Environment and load

Phase 1 load is offered rate against a process the learner sizes. Phase 2 load
is offered rate against a ceiling the learner declares to the platform, so the
same product needs a different number to be interesting.

| Lab | Environment | Scale target shape |
|-----|-------------|--------------------|
| 2/1 | Runtime emulator, ElasticMQ, DynamoDB Local | Metering events per period, invocation count at period close, freeze at the platform's barrier |
| 2/2 | Runtime emulator, ElasticMQ, DynamoDB Local | Records imported, in-flight batches, duplicate ratio, poison ratio |
| 2/3 | Runtime emulator, DynamoDB Local | Reservation attempts per second, contention on one resource, concurrency ceiling below offered rate |
| 2/4 | Runtime emulator, ElasticMQ, DynamoDB Local | Accepted transfers per second, duplicate resubmission ratio, account skew |
| 2/5 | Runtime emulator, provider simulators, external store | Offered rate several times the declared concurrency ceiling, provider latency schedule, cold-start share of the tail |

The ceiling is the phase's characteristic number. In phase 1 a burst above
capacity produces queueing the learner controls; in phase 2 it produces
throttling the learner only chooses how to absorb.

## Do the products still make sense?

A recast is only legitimate if the serverless version is a shape a practitioner
would actually deploy. Each was checked against that:

- **Billing and metering** — event-driven metering with a period-close job is a
  standard serverless product, and the freeze is the reason period close is
  hard.
- **Record import** — a queue-driven import behind functions is the canonical
  serverless shape, and the original lab was already written this way.
- **Reservation** — reservations behind functions with a key-value store are
  common, and the loss of a database-enforced invariant is the honest cost of
  that choice rather than an artificial handicap.
- **Money transfer** — payment endpoints behind functions are ordinary, and
  losing anything that can run between the two effects is the honest cost of
  the execution model rather than an artificial handicap.
- **Quote aggregation** — a read API fanning out to providers is the most
  common serverless workload there is, and the fan-out per invocation instead
  of per process is the real trade.

None of the five is a product invented to justify a constraint.

## Which model is useful where

The contrast is worth teaching because the answer is genuinely conditional, and
a learner who has built both can state the condition.

The self-run model wins where work must live between events: a consumer holding
a position in a log, a listener on a notification channel, a lease held in
process, a cache warmed once and used a million times. It also wins where the
invariant is relational — the phase 1 reservation lab can enforce non-overlap in
the storage engine, which nothing in the serverless pairing can reproduce
without serializing writes by hand. And it wins on steady, predictable load,
where a sized process is cheaper per request than a metered invocation.

The serverless model wins where demand is spiky or unattended, where the
per-request cost of idle capacity dominates, and where the operational surface
is the real expense — no patching, no capacity planning, no failover to design.
It also forces a discipline that survives the move back: state in the store
rather than in the process, idempotency at every entry point, and work that is
resumable by a stranger.

The distinction that matters is not cost. It is whether the design needs
something to be true *between* events. If it does, the serverless version pays
for that in machinery the platform used to provide. If it does not, the
long-lived process is capacity held for nothing.

## Governing references

- [`../01-systems-labs.md`](../specs/01-systems-labs.md) — the contracts both phases
  inherit, and the execution-model carve-out to the no-ports rule.
- [`1-lab-selection.md`](lab-selection.md) — the original scored selection.
- [`5-shared-scaffold.md`](../specs/0/5-shared-scaffold.md) — the fault controller that
  must reproduce the freeze locally, without which phase 2 cannot be graded.
