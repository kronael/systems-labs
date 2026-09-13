# Market history API

Design and build a market-data service that ingests trade records and serves
recent trades, fixed-interval candles, lookup by provider trade identity, and
ingestion health. The product handles a few quiet symbols and a highly active
symbol without losing records, corrupting aggregates, or hiding stale results.

DynamoDB is the required data model and API. The mandatory system uses
DynamoDB Local; a bounded hosted smoke run is optional. The data layout, the
aggregate representation, the write path, and the process decomposition
between ingestion and serving are the learner's decisions.

## What you are given

The supplied Compose stack starts DynamoDB Local, OpenTelemetry collection, and
the fault controller. It includes the shared deterministic trade generator, seeded to this lab's
scale target, an
opt-in bounded Kraken recent-trades recorder, provenance and replay tools,
and DynamoDB inspection, hot-symbol and cursor-overlap scenarios. The store
endpoint the stack publishes is served through the fault controller's
transport layer.

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

The store refuses work above a declared per-partition capacity budget, and it
refuses it unevenly, because the budget is per partition and the traffic is
not. The design states what an ingest refusal does to the trade that provoked
it and what a client sees while a partition is refusing. A read through a
secondary access path may not return the write it follows, and the design
states what a client sees in that window.

The design has a stated logical retention policy. Physical TTL timing cannot
change query correctness. Every access pattern has a bounded request shape, and the cost of a public
query must not grow with the retained history.

The scale target is 100 million retained trades across 20 symbols, a
sustained ingest of 4,000 trades per second with seven in ten writes landing
on the hottest symbol, and 200 concurrent clients whose range queries span
multiple result pages. These numbers size the problem, not the pass bar:
latency and request cost are measured against the learner's declared service
level, not a fixed number.

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Apache Cassandra** — [documentation](https://cassandra.apache.org/doc/latest/)
- **MongoDB** — [documentation](https://www.mongodb.com/docs/manual/)
- **PostgreSQL** — [documentation](https://www.postgresql.org/docs/current/)

## What is outside the problem

The expected focused time is ten to fourteen hours. DynamoDB Local, data
generation, Kraken recording, telemetry, faults, and inspection are prepared,
but the access-pattern map and key design are not, and a key design that
serves the public queries cleanly is routinely the one the hot-symbol share
falsifies, forcing a redesign before it holds at the ingest target. Order
placement, accounts, streaming WebSockets, DAX, global tables, PartiQL, and
production AWS are outside the problem.

Stuck? See `hints/`.
