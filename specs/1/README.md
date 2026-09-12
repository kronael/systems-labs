---
status: reference
---

# Phase 1 — runtime, delivery, and cross-system atomicity

This file is author-facing. Like everything under `specs/`, it is not part of
the learner tree; any learner-facing orientation is derived from it later.

## What this phase is about

The seven labs in this phase put a running process under pressure it cannot
design away: demand above capacity, a dependency that slows or dies, a restart
in the middle of accepted work, a record shape that changes while the system
is serving, and a store that hands work back instead of completing it.
Alongside that runtime pressure two of them introduce delivery models with
different promises — a leased queue that redelivers on a timer, and a retained
log that can be replayed — and each forces the learner to state precisely what
its model does and does not guarantee. One lab carries the pressure across a
system boundary: a successful API response must mean a monetary effect is
durable and its audit record cannot be lost, even though the database and the
event system fail independently. The last one carries it inward, to the case
where the store refuses: the invariant is an aggregate that no single write
can be judged against, conflicting work is declined rather than completed, and
what happens next belongs to the application. The seventh inverts the phase's
premise: every other lab asks a system to keep serving, and this one asks it
to stop. A transmitter left on the air is a continuing action, so an
authorization that outlives the evidence justifying it is a violation — while
a service that simply stays dark fails just as plainly. Its pause barrier is
the deliberate counterpart to `1/2`'s freeze past acknowledgement: there the
system must finish work it already accepted, here it must undo work already
in the world.

## The labs

- [Resilient quote service](1-resilient-quote-service.md) — a fare-search
  aggregation API that stays predictable under overload and partial provider
  failure, never returning a quote the provider no longer honours.
- [Reservation fulfillment](2-reservation-fulfillment.md) — a stay
  reservation API whose asynchronous fulfillment arrives on a lease,
  recovering accepted work after worker, broker, or database restarts.
- [Order activity dashboard](3-order-activity-dashboard.md) — replayable order
  and customer views that can be rebuilt while staying available.
- [Uninterrupted catalog service](4-uninterrupted-catalog-service.md) — a
  retail catalog API that keeps answering while a mandated price-display duty
  reshapes its records, with no maintenance window and no half-changed answer.
- [Auditable transfer service](5-auditable-transfer-service.md) — a
  money-transfer system with a durable ledger and a separately queryable audit
  product that survives independent failures of either side.
- [Shared budget service](6-shared-budget-service.md) — shared spending
  budgets that are never overspent under concurrent claims the store declines
  to complete, where every decision is final.
- [Spectrum compliance service](7-spectrum-compliance-service.md) — a fleet of
  radio transmitters kept legally on the air under a coordinator the service
  does not run, where an authorization must never outlive the evidence for it
  and the fleet must never stay dark once that evidence returns.

## The technologies

**Docker Compose** runs multi-container applications from one declarative
file. Every lab starts as a dependency-only Compose stack that is healthy
before any learner code exists, and the learner adds an application layer to
it; one local command brings up the whole laboratory. Documentation:
<https://docs.docker.com/compose/>.

**PostgreSQL** is a relational database with full transactions, constraints, a
procedural language (PL/pgSQL), and commit-time notifications
(`LISTEN`/`NOTIFY`). The learner meets it in the reservation lab as the
system of record, in the dashboard lab as the query store, in the catalog lab
as the store whose record shape changes under live traffic, in the transfer
lab as the required ledger store, and in the budget lab as the store that
declines conflicting work. The phase chooses it because one system holds a
real transaction boundary, a declared record shape, and an answer to
concurrent conflicting writes, and the exact guarantee of each is the study.
Documentation: <https://www.postgresql.org/docs/current/>.

**Apache Kafka** is a distributed event log: partitioned topics, consumer
groups, offsets, rebalances, retention. The learner meets it in the dashboard
lab as the required event transport and in the transfer lab as the required
audit-event transport, a durable, replayable system on the far side of the
commit gap whose guarantees end at its own boundary. The phase chooses it
because retention means history survives acknowledgement.
Documentation: <https://kafka.apache.org/43/>.

**NATS JetStream** is a persistence and delivery layer over NATS with
per-message acknowledgement, timed redelivery, and stream retention. The
learner meets it in the reservation lab as the queue environment. The phase
chooses it because it carries the leased delivery model — one of the phase's
two delivery promises — as software the learner runs and tunes.
Documentation:
<https://docs.nats.io/learn/jetstream/acknowledgment>.

**OpenTelemetry** is a vendor-neutral observability framework for traces,
metrics, and logs. Every scaffold in the phase ships its collection wired up.
The phase
chooses it because the quote lab's evidence must separate waiting to execute
from executing, and OpenTelemetry gives that measurement one format across
processes without fixing the service topology. Documentation:
<https://opentelemetry.io/docs/>.

**HdrHistogram** records value distributions across a wide dynamic range at
fixed precision. The quote lab's load evidence is reported as HDR histograms;
the other labs name the numbers their reports must contain and leave the
method open. The quote lab chooses it because overload produces latencies
spanning orders of magnitude, and this format keeps the tail visible instead
of averaging it away. Project:
<https://github.com/HdrHistogram/HdrHistogram>.

No infrastructure-as-code tool appears in this phase. Every dependency here is
software the learner starts locally, which is the property that defines the
phase.

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names the two or three technologies
a practitioner would have reached for instead — Envoy, HAProxy, resilience4j,
RabbitMQ, Temporal, Kafka Streams, Flink, Amazon SQS, ActiveMQ with XA
transactions, CockroachDB, MongoDB, MySQL with InnoDB, FoundationDB,
DynamoDB transactions, CME Globex, Kubernetes, recursive DNS resolvers — and the one thing each does differently at that
lab's boundary. They appear as reading with
documentation pointers, never as dependencies, so the exclusion costs the
learner the operation of a second system but not the comparison.

## Cloud

This phase needs no cloud account, and it has no optional cloud touchpoint
either. Every dependency is software the learner runs, which is what
distinguishes it from [phase 2](../2/README.md), where three of these same
products are rebuilt on an execution model the learner cannot operate.
