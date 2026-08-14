---
status: reference
---

# Phase 2 — the same problems, without a process

This file is author-facing. Like everything under `specs/`, it is not part of
the learner tree; any learner-facing orientation is derived from it later.

## What this phase is about

Phase 1 built five systems on software the learner operates. This phase takes
four of those products and rebuilds them on an execution model that runs one
request per instance and freezes when its runtime and every extension have
completed with no events pending. Nothing runs between events. Process memory
survives inside a reused environment and is visible to whoever arrives next,
but no request is guaranteed to land in a reused environment, so memory can
neither be relied on nor treated as private.

The product is held constant on purpose — the same public behaviour, the same
invariants. What changes is the execution model together with the stores and
delivery it is normally paired with. Holding the phase 1 store fixed is a
supported shape, but it moves the lesson to connection management under an
environment count nobody controls, which is a different lab. The contrast is honest rather
than controlled: it does not isolate one variable, it swaps one deployable
shape for another and lets the same product expose the difference. What breaks
names the part of the phase 1 design that was load-bearing.

One lab comes first and has no phase 1 partner: metered billing introduces the
execution model on an unfamiliar product, so the recasts that follow vary one
thing rather than two.

`1/3` has no counterpart here. Replaying a retained log through metered
invocations teaches the same lesson at higher cost, and a recast must reach a
failure the original could not. The reasoning is in the
[serverless contrast track](../0/6-serverless-contrast-track.md).

## The labs

- [Metered billing API](1-metered-billing-api.md) — subscriptions, metered
  usage, and period-close invoicing, where the environment freezes between
  invocations.
- [Reliable record import](2-reliable-record-import.md) — the import product
  from `1/4`, on a hosted queue with a poller the learner does not write.
- [Serverless reservation fulfillment](3-serverless-reservation-fulfillment.md)
  — the reservation product from `1/2`, with a partitioned key-value store as
  the system of record.
- [Serverless auditable transfer](4-serverless-auditable-transfer.md) — the
  transfer product from `1/5`, carrying the commit gap onto an environment
  where nothing runs between events.
- [Serverless quote aggregation](5-serverless-quote-aggregation.md) — the quote
  product from `1/1`, where admission is a platform ceiling rather than the
  design's own decision.

## The technologies

**AWS Lambda** is an execution model rather than a server: code runs in an
environment created on demand, one request at a time, and that environment is
frozen when the runtime and every extension have completed with no events
pending — a handler can return before that point — and thawed for a later
request. Every lab in this phase requires it, exercised through a local
runtime emulator, because the lifecycle is the subject. The phase fixes the
platform's standard execution mode: one invocation in flight per environment
and the fifteen-minute invocation ceiling. The same lifecycle documentation
describes other modes that admit concurrent invocations in one environment and
runs measured in months; those modes are excluded, because every premise in
this phase holds only for the standard mode. Documentation:
<https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html>.

**DynamoDB** is a partitioned key-value and document store built to be accessed
by key or index; it also offers table and index scans, but the access patterns
this phase's products serve are not meant to be served by them. This phase uses
DynamoDB Local. The phase chooses it because it
is the store this execution model is normally paired with. Documentation:
<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html>.

**ElasticMQ** exposes an SQS-compatible REST interface locally, carrying the
leased delivery model — a visibility window, redelivery, at-least-once — with no
account. Project: <https://github.com/softwaremill/elasticmq>. The semantics it
implements are Amazon SQS's:
<https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html>.

**The fault controller** must reproduce the freeze locally, at the platform's
own barrier — the runtime and every extension complete, no events pending —
or none of these labs can be falsified without a cloud account. That is a
scaffold requirement rather than a technology, and it is recorded as an
open question in [the shared scaffold](../0/5-shared-scaffold.md).

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names what a practitioner would have
reached for instead — a relational store behind the same functions, a workflow
service, a container platform, an API gateway cache, a change-data stream — and
the one thing each does differently at that lab's boundary. They appear as
reading, never as dependencies.

The most important neighbour is phase 1 itself. Each recast requires a direct
comparison with the local design — in its `ARCHITECTURE.md` or its evidence
report — including one property the local version had that the serverless one
cannot recover. The metered billing lab, which has no phase 1 partner, compares
alternative designs instead.

## Prerequisites

This phase is not usable à la carte. Every recast requires its phase 1
partner's retained artifacts, not merely the memory of having done it: the
partner's `ARCHITECTURE.md`, for the required direct comparison, and its
evidence report, for the required contrast of observed behaviour. The quote
recast further compares against the local lab's recorded baseline on the same
host, so that baseline must exist on the machine where the phase 2 evidence is
produced. A learner who skipped the partner lab, ran it on another machine, or
discarded its evidence cannot produce this phase's acceptance evidence. Only
the metered billing lab, which has no partner, carries no such prerequisite.

## Cloud

Every required gate runs locally on the runtime emulator. The optional smoke run
confirms that the emulated freeze and concurrency behaviour match the service,
and stays inside the guardrails in [cloud access](../../docs/cloud-access.md).
