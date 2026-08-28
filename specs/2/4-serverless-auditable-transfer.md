---
status: draft
---

# Serverless auditable transfer

## Brief

Design and build the money-transfer system from
[`../1/5-auditable-transfer-service.md`](../1/5-auditable-transfer-service.md)
again, on an execution environment that executes nothing between events and
freezes when its runtime and every extension have completed with no events
pending. Clients create accounts, transfer integer minor currency units,
query balances, and inspect a downstream audit history. A successful API
response means the monetary effect is durable and its audit record cannot be
lost.

The required environment is the function execution model, a partitioned
key-value store as the ledger, and a queue for audit delivery. The ledger's
data layout, the path an audit record travels from monetary effect to audit
history, the recovery after an interruption, and the downstream consumption
are the learner's decisions.

This lab is taken after the local one. The product is identical on purpose.

## Prepared scaffold

The supplied environment starts the local function runtime, the key-value store,
the queue, the fault controller, and a client generator with a declared
duplicate-resubmission ratio and account skew.

The fault controller freezes the environment at the freeze barrier with audit
publication outstanding, destroys an environment between invocations, fails the
queue while the ledger stays healthy, fails the store while the queue stays
healthy, and replays a delivered audit message. The course supplies the client
protocol, the history checker, and the fault schedules behind `make fault`. The
learner owns the handlers and their tests. No cloud account is required.

## Requirements

An acknowledged transfer is durable. Its audit record eventually reaches the
downstream product, delivery is at least once, and the audit product must show
each acknowledged transfer exactly once however many times its record arrives.
No transfer may be applied twice under duplicate resubmission, and no balance
may go negative.

Nothing runs between events here. Work still outstanding when a response is
returned is not guaranteed to run, because the environment freezes once the
runtime and every extension have completed with no events pending; the
submission states what makes an acknowledged transfer's audit record survive
that freeze, however the design sequences the ledger write and the
publication.

The audit product must remain correct under every order and grouping in which
audit records arrive.

The local pairing additionally requires the audit product to reconstruct its
accepted state from retained event history. That invariant is openly weakened
here: the delivery environment retains nothing once a record is acknowledged,
so no replayable transport history exists, and the platform forces the
weakening rather than the design choosing it. The submission states what
replaces the invariant — from which durable state a lost audit view is
repopulated — and the evidence must show a reconstruction reaching exactly
the acknowledged transfer set, by identity.

Configuration comes from the standard TOML contract.

The scale target is 20 million retained transfers across 200,000 accounts,
1,000 transfers per second offered against a concurrency ceiling of 64
environments fixed in the lab configuration, a duplicate resubmission ratio of
2 percent, and 1 percent of accounts receiving half of all transfers. An
environment count is not by itself a rate ceiling: the rate the ceiling admits
is the environment count divided by the invocation duration. The prepared
workload therefore calibrates the mean invocation duration during its warm-up
and sets the offered rate above the admitted rate that measurement implies, so
throttling is reachable in a test run. These numbers size the problem; they
are not pass thresholds.

The evidence must report transfers accepted against the ceiling, audit records
delivered per transfer, and the delay between ledger durability and audit
availability. How that measurement is produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what a successful API response promises about the audit record, and what makes
  that promise true across a freeze;
- which work could run between events in the local design, and where that work
  lives now;
- how the hot-account skew interacts with contention at the store, and what a
  rejected write means for the client;
- how the audit product stays correct under every order and grouping in which
  audit records arrive;
- how duplicate delivery is absorbed without a process that remembers what it
  already saw;
- which failure of the queue is visible to the client and which is not, and why
  that division is correct for this product;
- what the residual window is between ledger durability and audit availability,
  and what bounds it.

One further question presupposes part of a design and is solution-bearing; it
publishes to `HINTS.md`, never to the task:

- why the audit consumer cannot assume it sees an atomic write as a unit, and
  what it does instead.

The document must compare the local design and this one directly, and name the
guarantee that became weaker.

## Adversarial evaluation

The failure schedule freezes the environment between the ledger write and the audit
publication, destroys environments between invocations, fails the queue while
the ledger stays healthy and the reverse, resubmits transfers with the same and
with altered payloads, replays delivered audit messages, and drives the hot
accounts past what the configured ceiling admits. It then replays the full client history
against balances and the audit product, and requires an audit view rebuilt
after the loss of its store to match the acknowledged transfer set by
identity.

Checks do not require a named publication mechanism. They observe HTTP,
store-visible state, queue traffic, invocation counts, and the submitted
evidence.

## Acceptance evidence

Every acknowledged transfer is durable and appears exactly once in the audit
product. No balance goes negative under skew and duplicate resubmission. Audit
records survive a freeze at the publication boundary. Replayed audit messages
change nothing. An audit view rebuilt after the loss of its store reaches
exactly the acknowledged transfer set, from the durable state the submission
names.

The admitted fraction of offered transfers is reported and defended against
the rate the configured ceiling admits — the admitted rate the workload
calibrated during its warm-up. A design that rejects nearly everything has
not met the scale target, and the evidence must make that visible.

The evidence report includes accepted transfers against the ceiling, audit
records delivered per accepted transfer, the delay distribution between ledger
durability and audit availability, conflict and cancellation rates at hot
accounts, and invocations per accepted transfer. It states the residual window
in which a transfer is durable and its audit record is not yet available, and
what bounds that window.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **A change-data stream from the store** turns publication into someone else's
  problem, at the cost of a delivery order that no longer matches the write's
  atomicity, which is the same trade this lab makes explicit.
- **A workflow service** makes the two-step effect a durable execution with its
  own retry and compensation semantics, moving the commit gap into a service
  contract rather than removing it.
- **A relational ledger with a self-run broker**, which is the local lab, and is
  the comparison that makes this design's residual window legible.

Read their documentation on change streams, durable execution, and delivery
guarantees. The lab does not run them.

## Scope

The expected focused time is twelve to eighteen hours. The learner builds the
handlers and their tests. The runtime, store, queue, generator, and history
checker are prepared. Multi-currency support, authentication,
reversals, and multi-region replication are outside the problem.

The local transfer lab is a prerequisite, and its artifacts must be retained:
its `ARCHITECTURE.md` and its evidence report. The required comparison is
against that design and its record, not a memory of it; without those
artifacts the required evidence cannot be produced.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, execution
  shape policy, cost, grading, and evidence contracts.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  why this pairing earns a lab and what the platform removes.
- [`../1/5-auditable-transfer-service.md`](../1/5-auditable-transfer-service.md)
  — the local lab this one recasts.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — the environment freezes when the runtime and each extension have completed
  and there are no pending events, so work still unfinished at that point may
  never run.
- [DynamoDB transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
  — an atomic write's changes propagate gradually to indexes and streams, so
  records from one transaction may appear at different times and interleave
  with records from others. Solution-bearing: this belongs in `HINTS.md`,
  never in `README.md`.
- [Queue event source mapping](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
  and [batch failure reporting](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)
  — batches arrive at least once, and the outcome of a batch containing a
  failed item is governed by a documented reporting contract.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
