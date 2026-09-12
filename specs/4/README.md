---
status: reference
---

# Phase 4 — NoSQL, analytics, and platform portability

This file is author-facing. Like everything under `specs/`, it is not part of
the learner tree; any learner-facing orientation is derived from it later.

## What this phase is about

Phase 4 puts pressure on stores and platforms whose contracts differ from a
relational database's in ways that only show under load: a partitioned store
whose access patterns must be designed before the data exists, a cache whose
eviction and expiry are approximations, and a column store whose
reconciliation is background work on its own schedule. Each lab is sized so that a design
which ignores the store's own contract fails on its own terms. The closing
lab widens the pressure to two platforms at once: one domain contract across
a long-lived Kubernetes deployment and a Lambda-shaped batch deployment, with
infrastructure held as code accounting for every resource, drift, and secret;
the judgment under test is where portability genuinely ends.

## The labs

- [Market history API](1-market-history-api.md) — a market-data service that
  ingests trades and serves recent trades, candles, lookup by trade identity,
  and ingestion health.
- [Low-latency market API](2-low-latency-market-api.md) — a market API that
  serves recent trades and candles fast and explains its freshness when the
  cache is cold, full, slow, or gone.
- [Exact trade analytics](3-exact-trade-analytics.md) — an analytics service
  that answers exact aggregate questions over the full trade history while new
  trades arrive continuously.
- [Portable market ingestion](5-portable-market-ingestion.md) — a
  market-record normalization system that runs in both prepared environments
  and exposes the same accepted-record and query contract from each.

## The technologies

**Docker Compose** runs every prepared environment in this phase: a
dependency-only profile with the store, telemetry, generators, and the fault
controller, runnable before any learner code exists. The learner adds an
application layer to it. Documentation:
[docs.docker.com/compose](https://docs.docker.com/compose/).

**Amazon DynamoDB** is a partitioned key-value and document store: tables,
conditional writes, secondary indexes, TTL, and paginated queries. The phase
chooses it because it inverts relational habit — the access patterns come
first and the data design serves them, with each partition holding a fixed
throughput budget. The labs run
[DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html),
the offline implementation of the same API; a bounded hosted run is optional.
Documentation:
[DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html).

**Valkey** is an in-memory data store used here as the cache in front of
durable history. The phase chooses it because its memory limit is a real
boundary with observable consequences: eviction policies are sampled
approximations and expiry is not an exact deadline, so a design must state
what it relies on. Documentation:
[valkey.io/topics](https://valkey.io/topics/).

**ClickHouse** is an analytical column store built for aggregates over
billions of rows. The phase chooses it because its correctness contract is
unusual and fires under load: sparse primary keys do not enforce uniqueness,
merges and mutations are background work, and small frequent inserts can
outrun the store's housekeeping. The learner meets ClickHouse here and
nowhere else in the core catalog. Documentation:
[clickhouse.com/docs](https://clickhouse.com/docs).

**AWS Lambda** is the portable ingestion lab's second compute model, exercised
through a local Lambda-compatible runner: isolated environments, one invocation
at a time, a freeze after every response, recycling, a concurrency cap, and an
invocation ceiling. Here it is one of two targets a single domain contract must
run on; [phase 2](../2/README.md) is where that lifecycle is the subject rather
than a portability constraint. Documentation:
[AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html).

**Kubernetes** is the portable ingestion lab's long-lived compute model:
declarative workloads, probes, rollouts, and graceful termination, in which
replacement, rebalance, SIGTERM, and a faulty rollout are ordinary events a
design must survive. The learner meets it through [kind](https://kind.sigs.k8s.io/),
a real control plane in local Docker containers, so the mandatory path needs
no cloud account. Documentation: [kubernetes.io/docs](https://kubernetes.io/docs/home/).

**Apache Kafka** carries delivery on the portable ingestion lab's long-lived
path: partitions, consumer groups, offsets, and rebalances. The phase chooses
it because its acknowledgement and recovery model differs from a queue's in
exactly the ways a portable design must not paper over. Documentation:
[kafka.apache.org/documentation](https://kafka.apache.org/documentation/).

**ElasticMQ** is a local message queue with an SQS-compatible interface. The
phase chooses it because it reproduces the queue semantics the portable
ingestion lab depends on — visibility timeouts, at-least-once
redelivery, batched delivery — with no cloud account. Documentation:
[github.com/softwaremill/elasticmq](https://github.com/softwaremill/elasticmq);
the semantics it models are specified in the
[Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html).

**CloudEvents** is the portable ingestion lab's required input envelope, a
small specification describing event data the same way across HTTP, Kafka,
queues, and functions — a portable contract needs an envelope that no
transport owns. Documentation: [cloudevents.io](https://cloudevents.io/).

**OpenTelemetry** supplies the shared trace and metric format in the labs
where the measurement is the evidence — the market history and low-latency
labs must separate waiting from executing — and the semantic telemetry
fields the portable ingestion lab must emit across containers and functions.
Documentation: [opentelemetry.io/docs](https://opentelemetry.io/docs/).

**OpenTofu** is the portable ingestion lab's required infrastructure tool:
Terraform-compatible HCL, providers, plans, and state. State is the lesson —
every submitted resource is owned, an out-of-band change must surface in the
next plan, and secrets must never appear in state or plan text. Elsewhere it
is only the optional bounded sandbox for the hosted smoke resources the cost
contract allows. Documentation: [opentofu.org/docs](https://opentofu.org/docs/).

**Kraken recent trades** is the opt-in real data source. CI and verification
use generated seeded input; a real recording is bounded, checksummed, and cached,
and changes provenance rather than the pass criteria. Documentation:
[Kraken API — recent trades](https://docs.kraken.com/api/docs/rest-api/get-recent-trades/).

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names the technologies a
practitioner would have reached for instead — Apache Cassandra, MongoDB,
PostgreSQL, Memcached, groupcache, Amazon ElastiCache, Apache Druid,
Elasticsearch or OpenSearch, Knative, Temporal and Pulumi — and the one thing
each does differently at that lab's boundary. They appear as reading
rather than as dependencies: the labs never run them, but a learner who knows
what the alternatives guarantee can defend the choice the lab forced.

## Cloud

No cloud account is needed: every required gate runs locally — DynamoDB
Local, Valkey, ClickHouse, Kafka, and ElasticMQ in Compose, plus `kind`, the
Lambda-compatible runner, and the local OpenTofu sandboxes. An account is
optional and buys two things: the market history lab's bounded hosted
DynamoDB on-demand smoke run with short-retention logs, and the portable
ingestion lab's bounded real deployment through the OpenTofu account shell.
Both stay within the permitted services — Lambda, SQS, DynamoDB on-demand,
short-retention logs — and the by-the-hour exclusions listed in
[cloud access](../../docs/cloud-access.md).
