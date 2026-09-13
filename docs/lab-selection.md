---
status: reference
---

# Lab Selection

## Decision

This record selected ten labs from twenty expanded candidates. It scores that
round and nothing else. The catalog now holds thirty-four labs, seventeen of
them core. The [core catalog](labs/README.md#core-catalog) table is
authoritative for what exists, and every lab admitted after this round is
recorded there.
Selection favors durable system judgment over product count: every survivor
produces a useful end-to-end system, exposes a false mental model through a
deterministic failure, and fits into six to twenty-five focused hours with supplied
dependency scaffolding.

The scoring dimensions are conceptual leverage, fit with the stated learning
goals, usefulness of the finished artifact, strength of the falsifiable quirk,
completion feasibility, and low maintenance or cloud burden. Each dimension is
scored from zero to five. A high maintenance score means the lab remains cheap
and stable to operate.

| Rank | Candidate | Leverage | Fit | Utility | Quirk | Feasible | Maintain | Total | Decision |
|------|-----------|----------|-----|---------|-------|----------|----------|-------|----------|
| 1 | Reservation fulfillment service | 5 | 5 | 5 | 5 | 4 | 5 | 29 | keep |
| 2 | Low-latency market API | 4 | 5 | 5 | 5 | 5 | 5 | 29 | keep |
| 3 | Resilient quote service | 5 | 4 | 4 | 5 | 5 | 5 | 28 | keep |
| 4 | Order activity dashboard | 5 | 5 | 5 | 5 | 4 | 4 | 28 | keep |
| 5 | Reliable record import | 5 | 5 | 5 | 5 | 4 | 4 | 28 | keep |
| 6 | Auditable transfer service | 5 | 5 | 5 | 5 | 4 | 4 | 28 | keep |
| 7 | Internet route observatory | 5 | 5 | 5 | 5 | 4 | 4 | 28 | keep |
| 8 | Market history API | 5 | 5 | 5 | 5 | 4 | 4 | 28 | keep |
| 9 | Recoverable route analytics | 5 | 5 | 5 | 5 | 3 | 3 | 26 | keep |
| 10 | Portable market ingestion | 5 | 5 | 5 | 5 | 3 | 3 | 26 | keep |
| 11 | ClickHouse market analytics | 4 | 3 | 5 | 4 | 4 | 3 | 23 | cut |
| 12 | CRDT workspace | 5 | 2 | 4 | 5 | 3 | 3 | 22 | cut |
| 13 | NATS request and stream service | 4 | 3 | 4 | 4 | 4 | 3 | 22 | cut |
| 14 | RabbitMQ routing service | 3 | 3 | 4 | 4 | 4 | 3 | 21 | cut |
| 15 | Temporal order workflow | 5 | 2 | 4 | 5 | 3 | 2 | 21 | cut |
| 16 | Multi-region failover | 5 | 4 | 5 | 5 | 1 | 1 | 21 | cut |
| 17 | eBPF network profiler | 5 | 2 | 4 | 5 | 2 | 2 | 20 | cut |
| 18 | Cassandra or Scylla time series | 4 | 3 | 4 | 4 | 2 | 2 | 19 | cut |
| 19 | OpenSearch route search | 3 | 2 | 5 | 4 | 3 | 1 | 18 | cut |
| 20 | Raft key-value store | 5 | 1 | 3 | 5 | 1 | 2 | 17 | cut |

## Expanded candidates

### 1. Resilient quote service — keep

Design a quote API that combines several provider results and stays useful
through bursts, slow providers, partial failure, cancellation, and shutdown.
The prepared providers and open-loop workload make overload observable without
dictating concurrency or admission architecture. It earns its place because
every later system needs defensible capacity and degradation decisions.

### 2. Reservation fulfillment service — keep

Design a reservation product whose accepted transitions are validated in
PostgreSQL and whose asynchronous work eventually reaches a visible terminal
state. The environment requires PL/pgSQL and `LISTEN`/`NOTIFY`; disconnects,
concurrent claims, and process death test the chosen state and recovery model.
It provides the requested PostgreSQL depth without prescribing a job pattern.

### 3. Order activity dashboard — keep

Design an order and customer activity product from Kafka input that remains
queryable during rebalances and can be rebuilt from retained history. Skew,
duplicates, gaps, schema changes, and process death around an external effect
test the learner's event identity, ordering, progress, and materialization
choices. The result is a useful view, not an isolated consumer exercise.

### 4. Reliable record import — keep

Design a batch import product using an SQS-compatible queue, a Lambda-shaped
runner, and DynamoDB-compatible storage. Mixed-validity batches, visibility
expiry, duplicate delivery, poison records, and constrained invocation time
test the chosen completion and retry semantics. The final system must make
accepted, rejected, delayed, and exhausted records observable to an operator.

### 5. Auditable transfer service — keep

Design a concurrent transfer product whose database results and downstream
audit product remain reconcilable across PostgreSQL and Kafka failures. Named
crashes occur at each cross-system boundary, while contention and replay test
the stated money and publication invariants. The prompt requires evidence of
the guarantees but does not name an outbox or any other solution pattern.

### 6. Internet route observatory — keep

Design an observatory for RIPE-compatible BGP updates that answers current
route, churn, and collector-health questions. Late, duplicated, withdrawn,
out-of-order, and highly skewed updates test the learner's definition of
observation, time, identity, and ownership. Generated input is deterministic;
an opt-in bounded recording makes the same product useful with real data.

### 7. Recoverable route analytics — keep

Design stateful route analytics that can recover through a prepared Flink
environment and also reconstruct required results from Kafka history. Crashes
around checkpoints and external effects, corrupted recovery material, and a
compatible schema change test convergence claims. This is the demanding capstone
for state, replay, evolution, and recovery evidence.

### 8. Market history API — keep

Design a DynamoDB-backed API over Kraken-compatible trades for recent history
and candle queries. Hot symbols, cursor overlap, duplicates, pagination,
concurrent writers, index visibility, and delayed expiry test the data model
against public access and retention requirements. DynamoDB Local supplies the
mandatory free path; hosted behavior remains an explicit optional boundary.

### 9. Low-latency market API — keep

Design a freshness-aware acceleration layer for the market API with Valkey and
DynamoDB available. Eviction, expiry alignment, a popular absent symbol,
concurrent misses, dependency slowdown, and replica disagreement test which
state the design can trust. It provides a second NoSQL model with concerns
orthogonal to the durable market store.

### 10. Portable market ingestion — keep

Design one market-ingestion contract for both a local Kubernetes environment
and a Lambda-shaped batch environment, with OpenTofu for optional cloud
resources. Shutdown, rollout failure, redelivery, drift, state inspection,
secret sentinels, and cost limits test the claimed portability boundary. The
learner decides the adapters, topology, and platform-specific state model.

### 11. ClickHouse market analytics — cut

The product stores trades in a columnar engine and serves wide time-range
aggregates. Parts, merges, primary-key ordering, materialized views, and late
mutation behavior provide strong quirks and a useful artifact. It loses to the
selected ten because the market store and Flink labs already teach data layout
and aggregation, while another stateful dependency increases image size and
maintenance. ClickHouse remains the first expansion after the core.

### 12. CRDT workspace — cut

The product synchronizes documents across disconnected clients and tests
convergence under reordered delivery. Causality, tombstones, compaction, and
revocation after offline edits expose distributed-state behavior clearly.
The Automerge adapter, client simulator, and meaningful authorization boundary
push the lab beyond the several-hour budget, and it does not strengthen the
requested PostgreSQL, Kafka, NoSQL, or Lambda spine.

### 13. NATS request and stream service — cut

The product combines request-reply with JetStream persistence and consumer
redelivery. The surprise is the boundary between ephemeral core NATS and
retained JetStream state. It is runnable and concise, but SQS already supplies
the orthogonal queue model and Kafka supplies replay. A third messaging product
adds vocabulary more than judgment.

### 14. RabbitMQ routing service — cut

The product routes work through exchanges, bindings, acknowledgements, and a
dead-letter exchange. Unacked redelivery, prefetch, and routing topology are
worth knowing. It ranks below SQS because the selected queue lab also connects
to Lambda and the low-cost AWS goal, while Kafka already supplies the managed
consumer-group contrast.

### 15. Temporal order workflow — cut

The product coordinates a durable order workflow with retries, timers, and
compensation. Replay-safe workflow code and activity idempotency are deep and
non-obvious. A server, SDK, UI, workflow process, workers, and versioning model
consume too much setup and maintenance for one several-hour lab. The outbox and
SQS labs cover the prerequisite failure semantics first.

### 16. Multi-region failover — cut

The product serves a replicated ledger through a regional outage and measures
RPO, RTO, and stale reads. The product would be useful, but honest
regional networking, managed databases, DNS behavior, and billing cannot be
reproduced faithfully with a cheap deterministic local topology. A toy model
would contradict the course's evidence standard.

### 17. eBPF network profiler — cut

The product attributes syscall and network latency to the services in another
lab. Kernel version, privileges, BTF availability, container nesting, and
platform-specific fallbacks expose real limits. Those same constraints make a
portable check fragile, while OpenTelemetry already supplies the mandatory
cross-platform evidence path. eBPF fits a Linux-specific follow-on course.

### 18. Cassandra or Scylla time series — cut

The product stores route or trade time series using partition keys, clustering
keys, compaction, tombstones, and tunable consistency. Those mechanics matter,
but a meaningful multi-node failure exercise requires more resources and time
than DynamoDB's access-pattern lab. Teaching both wide-column systems would
duplicate the key-design lesson before either is used deeply.

### 19. OpenSearch route search — cut

The product offers text and field search over route events. Refresh intervals,
segments, mappings, rejected indexing, and search consistency provide good
failure material. Search is not central to the requested systems spine, and a
memory-heavy cluster adds the highest routine maintenance burden among the
candidates. Prefix and origin queries remain exact operational views instead.

### 20. Raft key-value store — cut

The product implements leader election and replicated writes under partitions.
It offers excellent algorithmic depth but violates the repository boundary:
learners should reason about deployed system guarantees rather than turn the
course into another algorithm set. Building, testing, and debugging consensus
correctly also exceeds a several-hour product lab.

## Selected sequence

This round ordered its ten survivors in five groups:

1. Runtime and delivery semantics: resilient quotes, reservation fulfillment,
   Kafka-backed activity, and queue-backed record import.
2. Cross-system invariants: the auditable transfer service.
3. Real Internet streaming: route observation and recoverable analytics.
4. Serverless NoSQL: market history and low-latency market queries.
5. Platform judgment: portable market ingestion under Kubernetes,
   Lambda-shaped execution, and OpenTofu.

The grouping reuses event and evidence contracts without requiring copied
solutions, and each lab directory stays independently completable. The
curriculum's phase map is in [`labs/README.md`](../labs/README.md) and the
phase `README.md` files.

## Governing references

- [`docs/contract.md`](contract.md) — course-wide learning,
  evidence, source, cost, and repository contracts.
- [`labs/README.md`](../labs/README.md) — authoritative list and lifecycle status of the
  selected lab specs.
