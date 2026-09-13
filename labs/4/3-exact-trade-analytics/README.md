# Exact trade analytics

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **PostgreSQL** — [documentation](https://www.postgresql.org/docs/current/)
- **Apache Druid** — [documentation](https://druid.apache.org/docs/latest/design/)
- **Elasticsearch or OpenSearch** — [documentation](https://opensearch.org/docs/latest/)

## What is outside the problem

The expected focused time is fourteen to eighteen hours. The learner builds
the ingestion service, the schema, and the query API. ClickHouse, the
generator, the cached recordings, and the fault schedules are prepared, but
producing an exact answer against a store that reconciles in the background
is not, and a first design that trusts that reconciliation is falsified the
moment the arrival rate outruns it, forcing a rebuild of the write and query
path. Real Kraken recordings are opt-in, bounded, and
cached; CI uses generated input only. Dashboards, alerting, multi-node
replication, and cluster operations are outside the problem.

Stuck? See `hints/`.
