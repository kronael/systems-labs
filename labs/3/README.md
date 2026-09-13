---
status: reference
---

# Phase 3 — real Internet streaming

This page orients the phase: what its labs study, and why they sit together.
The labs themselves are the directories beside it.

## What this phase is about

This phase feeds the labs from the real Internet: BGP routing observations in
the RIPE RIS Live envelope, where a timestamp is the collector's receipt time
and ordering is guaranteed only within one peering session. The pressure is
what real streams do to state — observation scope, event time against
processing time, hot prefixes, bursts — and what recovery means for a
stateful pipeline that must come back without losing or corrupting what it
accepted.

## The labs

- [Internet route observatory](1-internet-route-observatory/README.md) — an
  operational view of Internet routing observations: current origins per
  prefix, recent changes, churn, collector health, and processing lag.
- [Recoverable route analytics](2-recoverable-route-analytics/README.md) — a
  continuously updated routing analytics service that recovers through two
  independent paths and must produce the same results from both.

## The technologies

**Docker Compose** runs multi-container applications from one declarative
file. Both labs start as dependency-only Compose stacks — including a local
Flink cluster and checkpoint storage in the analytics lab — healthy before
any learner code exists. Documentation: <https://docs.docker.com/compose/>.

**RIPE RIS Live** is the RIPE NCC's real-time stream of BGP messages from its
route collectors, delivered as JSON over a filterable WebSocket API. The
learner meets it as the data contract both labs accept. The phase chooses it
because it is a real Internet source whose documented properties — receipt-time
timestamps, per-session ordering — are the phase's subject, not an
inconvenience. CI never contacts RIPE: the deterministic generator proves
every requirement, and real recordings are opt-in, bounded, checksummed, and
cached. Manual: <https://ris-live.ripe.net/manual/>.

**Apache Kafka** is a distributed event log: partitioned topics, consumer
groups, offsets, retention, replay. The learner meets it as the required
transport in both labs. The phase chooses it because retention makes replay a
first-class operation — history survives consumption, so rebuilding state
from retained events is possible and the limits of that retention are real
and observable. Documentation: <https://kafka.apache.org/documentation/>.

**Apache Flink** is a distributed stateful stream-processing engine with
native event time, watermarks, windows, and checkpointed operator state. The
learner meets it in the analytics lab as the required processing engine, and
nowhere else in the curriculum, because event-time state and checkpoint
recovery are its subject. The phase chooses it as the heavyweight
representative of managed streaming state: what a checkpoint covers, and what
lies outside it, is the lab's question. Documentation:
<https://nightlies.apache.org/flink/flink-docs-stable/>.

**PostgreSQL** is a relational database with full transactions and rich
indexing. The learner meets it as the required query store in the analytics
lab and as one of the two prepared store profiles in the observatory lab. The
phase chooses it because query results must stay available and trustworthy
while the pipeline behind them fails and recovers, and an external SQL store
makes that state independently inspectable. Documentation:
<https://www.postgresql.org/docs/current/>.

**OpenTelemetry** is a vendor-neutral observability framework for traces,
metrics, and logs, wired into both prepared stacks. The phase chooses it
because freshness, per-partition lag, and recovery progress are required
evidence in both labs, and one telemetry contract covers whatever process
topology the learner builds. Documentation: <https://opentelemetry.io/docs/>.

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names the alternatives a
practitioner would weigh — RouteViews and CAIDA BGPStream on the data side,
Kafka Streams, Spark Structured Streaming, and Materialize on the processing
side — with the one thing each changes at that lab's boundary and a pointer
to its documentation. They appear as reading rather than as dependencies.

## Cloud

This phase needs no cloud account. Everything runs locally in Compose;
RIPE RIS Live is a public stream, and recording it is opt-in and needs no
account, as the per-phase table in
[cloud access](../cloud-access.md) records.
