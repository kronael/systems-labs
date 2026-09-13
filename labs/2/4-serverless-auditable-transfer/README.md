# Serverless auditable transfer

Design and build the money-transfer system from
[`../1/5-auditable-transfer-service.md`](../../1/5-auditable-transfer-service/README.md)
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

## What you are given

The supplied environment starts the local function runtime, the key-value store,
the queue, the fault controller, and a client generator with a declared
duplicate-resubmission ratio and account skew.

The fault controller freezes the environment at the freeze barrier with audit
publication outstanding, destroys an environment between invocations, fails the
queue while the ledger stays healthy, fails the store while the queue stays
healthy, and replays a delivered audit message.

The local runner does not freeze by itself. The fault controller's process
layer supplies the freeze, at the barrier the platform states: the runtime and
every extension complete with no events pending, which a returned response
alone does not mark. The controller holds the environment at that barrier,
confirms the suspension, and thaws it when the next invocation is assigned.

The course supplies the client protocol, the history checker, and the fault
schedules behind `make fault`. The learner owns the handlers and their tests.
No cloud account is required.

## Requirements

An acknowledged transfer is durable. Its audit record eventually reaches the
downstream product, delivery is at least once, and the audit product must show
each acknowledged transfer exactly once however many times its record arrives.
No transfer may be applied twice under duplicate resubmission, and no balance
may go negative.

The product is held constant, so its invariants are `1/5`'s in full: total
value remains constant across accepted transfers, account histories and
balances agree, and one idempotency key and payload have one monetary effect
while conflicting reuse fails visibly. The execution model changes what has to
hold them, never which of them hold.

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

## What your ARCHITECTURE.md must explain

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

The document must compare the local design and this one directly, and name the
guarantee that became weaker.

## Acceptance evidence

Every acknowledged transfer is durable and appears exactly once in the audit
product. No balance goes negative under skew and duplicate resubmission. Audit
records survive a freeze at the publication boundary. Replayed audit messages
change nothing. An audit view rebuilt after the loss of its store reaches
exactly the acknowledged transfer set, from the durable state the submission
names.

The freeze in that schedule comes from the controller's process layer. The
required gate therefore proves a modelled freeze rather than the hosted
service's own behaviour, and `EVALUATION.md` states that. Where an account
exists, `make smoke` compares the modelled freeze against the service.

The admitted fraction of offered transfers is reported and defended against
the rate the configured ceiling admits — the admitted rate the workload
calibrated during its warm-up. A design that rejects nearly everything has
not met the scale target, and the evidence must make that visible.

The evidence report includes accepted transfers against the ceiling, audit
records delivered per accepted transfer, the delay distribution between ledger
durability and audit availability, the contention the controller's transport
layer recorded at the hot accounts — the concurrent attempts it held at one
account key and the outcome each attempt reached — and invocations per accepted
transfer. It states the residual window
in which a transfer is durable and its audit record is not yet available, and
what bounds that window.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Amazon DynamoDB Streams** — [documentation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- **AWS Step Functions** — [documentation](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- **PostgreSQL with a self-run Kafka** — [documentation](https://www.postgresql.org/docs/current/)

## What is outside the problem

The expected focused time is twelve to eighteen hours. The learner builds the
handlers and their tests. The runtime, store, queue, generator, and history
checker are prepared. Multi-currency support, authentication,
reversals, and multi-region replication are outside the problem.

The local transfer lab is a prerequisite, and its artifacts must be retained:
its `ARCHITECTURE.md` and its evidence report. The required comparison is
against that design and its record, not a memory of it; without those
artifacts the required evidence cannot be produced.

Stuck? See `hints/`.
