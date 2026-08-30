---
status: draft
---

# Market history API

## Brief

Design and build a market-data service that ingests trade records and serves
recent trades, fixed-interval candles, lookup by provider trade identity, and
ingestion health. The product handles a few quiet symbols and a highly active
symbol without losing records, corrupting aggregates, or hiding stale results.

DynamoDB is the required data model and API. The mandatory system uses
DynamoDB Local; a bounded hosted smoke run is optional. The data layout, the
aggregate representation, the write path, and the process decomposition
between ingestion and serving are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts DynamoDB Local, OpenTelemetry collection, and
the fault controller. It includes a deterministic million-trade generator, an
opt-in bounded Kraken recent-trades recorder, provenance and replay tools,
and DynamoDB inspection, hot-symbol and cursor-overlap scenarios.

The learner owns ingestion, serving, data design, and the application Compose
layer. Standard Make targets start the database, generate or replay input,
exercise every query, interrupt ingestion, inspect access behavior, and collect
evidence. The optional OpenTofu sandbox creates only the bounded hosted table
and smoke inputs allowed by the cost contract.

## Requirements

The system accepts versioned trades with stable provider identity, symbol,
side, decimal price and quantity, provider time, observation time, and source
provenance. Binary floating-point conversion may not change financial values.
Malformed or unsupported input must have a visible outcome.

Clients can query recent trades for a symbol and bounded time range, candles
for a symbol and interval, one trade by provider identity, and ingestion
freshness. Results remain complete across DynamoDB pagination. Duplicate or
overlapping source pages produce one logical trade and correct candles.

The design has a stated logical retention policy. Physical TTL timing cannot
change query correctness. Every access pattern has a bounded request shape;
table scans are not accepted as an architecture for public queries.

The scale target is 100 million retained trades across 20 symbols, a
sustained ingest of 4,000 trades per second with seven in ten writes landing
on the hottest symbol, and 200 concurrent clients whose range queries span
multiple result pages. These numbers size the problem, not the pass bar:
latency and request cost are measured against the learner's declared service
level, not a fixed number.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- the exact access patterns and how the data model serves each one;
- how popular symbols distribute traffic without unbounded query fan-out;
- how trade identity, event time, sort order, and duplicate ingestion relate;
- how candles remain correct across retry, overlap, and concurrent ingestion;
- where strong or eventual consistency is acceptable and visible to callers;
- how pagination participates in correctness rather than only performance;
- how logical retention differs from asynchronous physical deletion;
- which hosted DynamoDB behavior DynamoDB Local cannot validate.

At least two candidate key designs must be evaluated against the supplied hot-
symbol distribution and public queries.

## Adversarial evaluation

The failure schedule sends a concentrated symbol burst, overlaps provider
cursors, repeats identities, kills ingestion around a durable write, forces
multi-page queries, reads through a secondary access path immediately after
a write, retains physically expired items, and introduces malformed
precision data.

Checks observe public APIs, DynamoDB requests and items, source histories,
telemetry, key distribution, request counts, and exact candle reconciliation.
They do not require a particular single-table or multi-table pattern.

## Acceptance evidence

All accepted trades are queryable under the stated consistency contract.
Duplicates have one logical effect, candles reconcile with exact source
identities, pagination is complete, logically expired data obeys the retention
contract, and hot-symbol work stays within the submitted distribution and fan-
out bounds.

The report includes the access-pattern map, candidate-design comparison, key
distribution, requests and pages per operation, conflicts, item sizes,
consistency observations, logical-expiry behavior, ingestion recovery, and a
hosted-capacity estimate clearly separated from local evidence.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Apache Cassandra** — [documentation](https://cassandra.apache.org/doc/latest/)
  and **ScyllaDB** — [documentation](https://docs.scylladb.com/). Partition
  by key in the same way but let a partition grow without bound and degrade
  instead of throttling, which turns the hot-key problem from a provider
  quota into an operator's tail latency.
- **MongoDB** — [documentation](https://www.mongodb.com/docs/manual/). Can
  index any field after the fact, so access patterns need not be fixed
  before the data exists — and the same skew returns later as the choice of
  shard key.
- **PostgreSQL** — [documentation](https://www.postgresql.org/docs/current/).
  Would answer every query in this lab from one node with ordinary indexes
  and no key design at all, and pays with a vertical ceiling in place of a
  partition limit.

The lab does not run them.

## Scope

The expected focused time is ten to fourteen hours. DynamoDB Local, data
generation, Kraken recording, telemetry, faults, and inspection are prepared,
but the access-pattern map and key design are not, and a key design that
serves the public queries cleanly is routinely the one the hot-symbol share
falsifies, forcing a redesign before it holds at the ingest target. Order
placement, accounts, streaming WebSockets, DAX, global tables, PartiQL, and
production AWS are outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  DynamoDB, real-data, cost, and evidence contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [Partition key design](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)
  — each DynamoDB partition serves a fixed per-second budget of read and
  write units, so one hot key throttles while the table sits far below its
  capacity.
- [Paginating query results](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)
  — a `Query` returns at most 1 MB per call, and only the absence of
  `LastEvaluatedKey` proves a result set is complete.
- Implementation pointers do not exist while the spec is `draft`.
