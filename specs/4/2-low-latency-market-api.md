---
status: draft
---

# Low-latency market API

## Brief

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

## Prepared scaffold

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

## Architecture questions

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

`HINTS.md`-bound, because it presupposes a mechanism: which of the store's
durability, eviction, and availability properties the design relies on.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule aligns the expirations of named keys, evicts a named
popular entry before its expiry, drives a burst on one named key through two
replicas, kills one instance at the moment a named key's fill is granted,
slows DynamoDB at a named request, makes Valkey time out at a named request,
restarts Valkey empty once a named key has been served, and changes the source
generation at a named record.

Checks observe public responses, freshness metadata, dependency requests,
Valkey state and memory, traces, metrics, and exact comparison with the
durable dataset. They do not require a named cache-aside, lock, or
stale-while-revalidate pattern.

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

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Memcached** — [documentation](https://memcached.org/). A cache and
  nothing more — multithreaded, no persistence, no rich values — and
  manages memory in per-size slab classes, so eviction pressure lands
  within an item's size class rather than across the whole keyspace.
- **groupcache** — [documentation](https://github.com/golang/groupcache).
  Keeps hot entries inside each application replica, removing the network
  hop and the shared store; every replica then holds a private view, and
  coherence between replicas becomes the design problem.
- **Amazon ElastiCache** — [documentation](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html).
  Runs the same engine as a managed service with failover, so a cache loss
  arrives as an empty replacement node on the provider's schedule; it is
  excluded from this course by cost policy.

The lab does not run them.

## Scope

The expected focused time is eight to twelve hours. Valkey, DynamoDB Local,
market data, two-replica topology, telemetry, workload, and faults are
prepared, but the cache policy and cross-replica fill coordination are not,
and a first policy that passes warm and cold traffic is routinely the one
the same-key burst across two replicas falsifies. Valkey Cluster, Sentinel, a
distributed write lock, CDN, durable queue, and ElastiCache are outside the
problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, Valkey,
  evidence, and failure contracts.
- [`../../docs/lab-selection.md`](../../docs/lab-selection.md) — selection rationale.
- [Key eviction](https://valkey.io/topics/lru-cache/) — at `maxmemory` Valkey
  evicts by the configured policy, and its LRU and LFU are approximations
  that sample a handful of keys per decision rather than track exact recency.
  Solution-bearing: this belongs in HINTS.md, never in README.md.
- [`EXPIRE`](https://valkey.io/commands/expire/) — expired keys are reclaimed
  on access plus a background sampling cycle that tolerates a fraction of
  expired keys lingering in memory, so a TTL is not an exact deadline.
  Solution-bearing: this belongs in HINTS.md, never in README.md.
- [Get OHLC data](https://docs.kraken.com/api/docs/rest-api/get-ohlc-data) —
  "The last entry in the OHLC array is for the current, not-yet-committed
  timeframe": the venue's own contract makes the newest candle provisional,
  so an answer's age states how settled it is.
- Implementation pointers do not exist while the spec is `draft`.
