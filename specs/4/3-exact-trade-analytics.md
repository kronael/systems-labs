---
status: draft
---

# Exact trade analytics

## Brief

Design and build an analytics service over the full trade history that answers
aggregate questions — volume by symbol and interval, top movers, and the
largest trades in a window — while new trades arrive continuously and the same
trade can be delivered more than once.

Every answer must be exact. A duplicate delivery must not inflate a volume, and
a corrected trade must not leave its earlier version in the result. The service
states the age of the data behind each answer.

The assignment is the whole service: ingestion path, table design, write
policy, query path, correction handling, and end-to-end tests. The data
layout, the write path, and how exactness is produced against storage that
has not yet reconciled are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts ClickHouse, the trade generator, the source
replay for cached Kraken recordings, and the fault controller. The generator
emits duplicates, late arrivals, and corrections at declared ratios, and every
record carries a stable identity so checks can assert exact histories.

The learner owns the ingestion service, the schema, and the query API. No cloud
account is required.

## Requirements

The service answers aggregate queries over the full retained history and over
recent windows. Every answer is exact with respect to the accepted input: each
trade contributes once, and a correction supersedes the trade it corrects.

Answers carry a freshness statement. If the service cannot yet answer exactly
for the most recent interval, it says so rather than returning an approximate
number silently.

Ingestion continues while queries run. The service does not stop accepting
trades to make a query correct, and it does not let query cost grow without
bound as history accumulates.

The scale target is 2 billion stored trades across 500 symbols, a sustained
ingest of 200,000 trades per second, a duplicate ratio of 2 percent, a
correction ratio of 0.1 percent, and 50 concurrent analytical queries. Query
latency is measured against the learner's declared service level, not a fixed
number.

The evidence must show ingest rate, query latency by class, and the divergence
between a naive count and the exact answer over time. How that measurement is
produced is the learner's choice; no telemetry stack is required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what guarantees the chosen table design actually makes about duplicate rows,
  and at what moment that guarantee applies;
- how an exact answer is produced when the underlying storage has not yet
  reconciled the rows behind it;
- what the query path costs, and how that cost changes as history grows;
- how corrections are represented, and what a reader sees between the arrival
  of a correction and its effect;
- how the write path avoids the failure that small frequent writes cause in
  this class of store;
- what the freshness statement means operationally, and how it is derived;
- which answers would become approximate under sustained overload, and why
  that is acceptable or not.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

The failure schedule replays a trade stream containing exact duplicates,
out-of-order arrivals, and corrections at named identities, then queries
during ingest, immediately after a burst, and after the store has been idle.
It drives insert frequency into the regime where the store rejects work,
restarts the ingestion service mid-batch, and restarts the store itself.

Checks do not inspect private functions or require a named table engine. They
compare every aggregate against an independently computed exact answer
derived from the accepted input.

## Acceptance evidence

Aggregates match the independently computed answer at every query point,
including during ingest and immediately after a burst. Duplicates never inflate
a total. A correction is reflected once. The service survives a restart of both
itself and the store without double-counting the records in flight.

The evidence report includes sustained ingest rate, query latency by class
against the declared service level, storage growth against retained history,
the observed divergence between a naive read and the exact answer, and the
behavior at the insert frequency where the store begins to reject work. It
names the point at which exactness would have to be traded for latency.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **PostgreSQL** — [documentation](https://www.postgresql.org/docs/current/).
  Enforces uniqueness in the write path, so the duplicate problem never
  reaches the reader — and pays for it with write cost and a table size
  this workload would not tolerate.
- **Apache Druid** — [documentation](https://druid.apache.org/docs/latest/design/)
  and **Apache Pinot** — [documentation](https://docs.pinot.apache.org/).
  Target the same interactive analytical queries but organize ingestion
  around segments and real-time versus historical nodes, which moves the
  freshness question into the topology.
- **Elasticsearch or OpenSearch** — [documentation](https://opensearch.org/docs/latest/).
  Would make the top-N and recent-window queries easy and the exactness
  guarantee harder, because scoring and refresh intervals sit between a
  write and its visibility.

The lab does not run them.

## Scope

The expected focused time is fourteen to eighteen hours. The learner builds
the ingestion service, the schema, and the query API. ClickHouse, the
generator, the cached recordings, and the fault schedules are prepared, but
producing an exact answer against a store that reconciles in the background
is not, and a first design that trusts that reconciliation is falsified the
moment the arrival rate outruns it, forcing a rebuild of the write and query
path. Real Kraken recordings are opt-in, bounded, and
cached; CI uses generated input only. Dashboards, alerting, multi-node
replication, and cluster operations are outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale,
  where ClickHouse was recorded as the strongest first addition.
- [`ReplacingMergeTree`](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replacingmergetree)
  — deduplication happens only during a merge, merging runs in the background
  at an unknown time, and the engine offers eventual correctness rather than
  a guarantee that duplicates are absent.
- [Mutations](https://clickhouse.com/docs/en/sql-reference/statements/alter)
  — a mutation is an asynchronous background process, and the statement
  returns as soon as the entry is recorded rather than when the change
  applies.
- [MergeTree settings](https://clickhouse.com/docs/en/operations/settings/merge-tree-settings)
  — `parts_to_throw_insert` defaults to 3000, which is the ceiling small
  frequent inserts reach when they outrun merging.
- Implementation pointers do not exist while the spec is `draft`.
