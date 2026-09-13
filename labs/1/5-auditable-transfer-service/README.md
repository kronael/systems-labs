# Auditable transfer service

Design and build a money-transfer system with a durable ledger and a separate
audit product. Clients create accounts, transfer integer minor currency units,
query balances, and inspect the downstream audit history. A successful API
response means the monetary effect is durable and its audit record cannot be
lost, even when the database and event system fail independently.

PostgreSQL is the required ledger store and Kafka is the required audit-event
transport. The ledger's data layout, the path an audit record travels from
monetary effect to audit product, the coordination between the two required
systems, and the downstream consumption are the learner's decisions.

## What you are given

The supplied Compose stack starts PostgreSQL, Kafka, OpenTelemetry collection,
and a deterministic fault controller. It includes generated accounts and
transfers, concurrent and duplicate workloads, Kafka and SQL inspection,
history checking, and exact crash barriers.

The learner owns all application processes, database schema, event schema, and
application Compose layer. Standard Make targets start dependencies, seed
accounts, run the public APIs, inject failures at database and broker
boundaries, replay events, and produce the evidence bundle.

## Requirements

Total value remains constant across accepted transfers. Account histories and
balances agree. One idempotency key and payload have one monetary effect;
conflicting reuse fails visibly. Invalid amounts, unknown accounts, and
transfers that violate the stated balance rule cannot commit.

Every acknowledged transfer eventually appears in the audit product. Repeated
event delivery may be observed but cannot create repeated logical audit
effects. Database or broker recovery must not require manual editing of money
or event state.

The audit API remains independently queryable and can reconstruct its accepted
state from the available event history. Backlog, oldest unpublished or
unprocessed work, transaction contention, and reconciliation failures must be
observable.

The scale target is 20 million retained transfers across 200,000 accounts, a
sustained 1,000 accepted transfers per second from 500 concurrent clients, a
duplicate resubmission ratio of 2 percent, and 1 percent of accounts receiving
half of all transfers. These numbers size the problem; they are not pass
thresholds. Throughput and latency are measured against the learner's recorded
baseline and declared service level, not a fixed number.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- the authoritative representation of money and how its invariant is checked;
- the isolation and concurrency model for transfers against the same account;
- the meaning and scope of an acknowledged transfer;
- how the design handles every ordering of PostgreSQL commit, Kafka publish,
  process death, and retry;
- how event and transfer identities support reconciliation;
- how multiple publishers or consumers coordinate safely;
- how the audit view recovers from repeated, delayed, or missing attempts;
- what guarantee is provided end to end and where it stops.

At least two cross-system publication designs must be compared. Pattern names
alone do not count as analysis.

## Acceptance evidence

After every scenario, value is conserved, balances match immutable history,
acknowledged transfers remain present, and each has one logical audit result.
Unacknowledged attempts obey the submitted contract. All duplicate attempts
and recovery actions remain visible.

The report correlates request, transfer, ledger, publication, Kafka,
consumption, and audit identities. It includes contention and retry counts,
backlog age, rebuild result, and a recomputed invariant that does not trust a
cached balance or row count.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **ActiveMQ** — [documentation](https://activemq.apache.org/components/classic/documentation)
- **Temporal** — [documentation](https://docs.temporal.io/)
- **CockroachDB** — [documentation](https://www.cockroachlabs.com/docs/stable/)

## What is outside the problem

The expected focused time is fifteen to twenty hours. PostgreSQL, Kafka,
workload, telemetry, faults, and history checking are prepared. The problem has one
currency and no payment provider, exchange rate, chargeback, replica, or
distributed transaction coordinator service.

Stuck? See `hints/`.
