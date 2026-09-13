# Low-latency market API

Design and build a low-latency market API that serves recent trades and candles
with DynamoDB-compatible storage and Valkey available. A market answer has an
age by nature: the venue's own candle documentation states that the most recent
entry covers the current, not-yet-committed period, so the newest values are
not final and will change until the period closes. The service must preserve
durable market history and explain the freshness of every answer when Valkey
is cold, full, slow, restarted, or unavailable.

Valkey and DynamoDB Local are required dependencies. The cache policy, the
coordination between concurrent application replicas, and the recovery path
when Valkey cannot answer are the learner's decisions.

## What you are given

The supplied Compose stack starts Valkey, DynamoDB Local, OpenTelemetry
collection, two optional application slots, and the fault controller. It
includes a loader that reads the shared deterministic trade generator's
output, hot-key and working-set generators, memory-pressure controls, cache inspection, and latency measurement.

The learner owns the API, cache policy, application topology, and application
Compose layer. Standard Make targets load durable market data, run cold and warm
queries, drive concurrent replicas, change Valkey memory and eviction policy,
restart dependencies, and capture evidence.

## Requirements

Recent-trade and candle responses must match durable market history under the
declared freshness contract. The age of an answer is a property of the data,
not a service convention: the current candle period has not closed, so the
newest answer is the least settled one, while every committed period behind it
no longer changes. Every response includes an `as_of` value stating which
moment of the durable history the answer reflects, and enough status to
distinguish fresh, accepted-stale, cache bypass, and failure. A caller that
accepts a stale answer is not merely taking a cheaper one; it knowingly
chooses a slightly older, more settled view over the newest, least settled
one, and the contract must make that choice explicit. If neither dependency
can satisfy the contract, the caller receives a typed non-2xx response.

Valkey loss or eviction cannot require repair of durable market data. Cache
failure must not create unbounded waits or unbounded amplification against
DynamoDB. A same-key request burst from more than one application instance
must stay inside a submitted backend-load bound.

The scale target is 100 million durable trades across 50 symbols, a sustained
20,000 public requests per second across two application replicas with nine
in ten requests concentrated on the twenty most popular query shapes, and a
Valkey memory limit that holds roughly one third of the working set. These
numbers size the problem, not the pass bar: latency is measured against the
learner's declared service level, not a fixed number.

The system exposes cache hits, misses, fills, contention, eviction, memory,
backend queries, fallbacks, freshness, and end-to-end latency. A high hit rate
alone is not sufficient evidence.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- which store owns truth and what information a cached value carries;
- the freshness contract and the conditions for serving stale data or failing;
- how cache identity changes across query, source generation, and schema;
- how writes or new market data interact with previously cached results;
- how concurrent misses are controlled within and across application replicas;
- what a caller observes while a missing value is being produced, and how the
  service returns to answering for that value when the attempt does not finish;
- how timeouts and resource limits prevent the cache from worsening an outage;
- which Valkey durability, eviction, and availability properties the design
  relies on and which it explicitly does not rely on.

At least two cache policies must be compared with the supplied access pattern.

## Acceptance evidence

Warm, cold, evicted, and restarted cache states return results allowed by the
same public contract. Valkey loss never changes durable values. Backend
amplification, waiting work, and response latency stay within submitted bounds.
Unavailable fresh data becomes a visible client failure unless the declared
stale window permits the exact response.

The report includes hit and miss ratios, eviction count, memory and
fragmentation, backend queries per public request, concurrent-fill behavior,
freshness age, fallback count, and p50/p95/p99 for cold, warm, pressure, and
dependency-failure runs. It ties each result to the chosen policy.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Memcached** — [documentation](https://memcached.org/)
- **groupcache** — [documentation](https://github.com/golang/groupcache)
- **Amazon ElastiCache** — [documentation](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)

Stuck? See `hints/`.
