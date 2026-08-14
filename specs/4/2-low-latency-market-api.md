---
status: draft
---

# Low-latency market API

## Brief

Design and build a low-latency market API that serves recent trades and candles
with DynamoDB-compatible storage and Valkey available. The service must
preserve durable market history and explain its freshness when Valkey is cold,
full, slow, restarted, or unavailable.

Valkey and DynamoDB Local are required dependencies. The prompt does not
prescribe cache keys, read/write policy, TTLs, invalidation, fill coordination,
locking, stale-data policy, replication, or application-process count.

## Prepared scaffold

The supplied Compose stack starts Valkey, DynamoDB Local, OpenTelemetry
collection, two optional application slots, and the fault controller. It
includes a deterministic market-history loader, hot-key and working-set
generators, memory-pressure controls, cache inspection, latency measurement,
and the black-box grader.

The learner owns the API, cache policy, application topology, and application
Compose layer. Standard Make targets load durable market data, run cold and warm
queries, drive concurrent replicas, change Valkey memory and eviction policy,
restart dependencies, and capture evidence.

## Requirements

Recent-trade and candle responses must match durable market history under the
declared freshness contract. Every response includes an `as_of` value and
enough status to distinguish fresh, accepted-stale, cache bypass, and failure.
If neither dependency can satisfy the contract, the caller receives a typed
non-2xx response.

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
- how a failed fill owner or expired coordination state recovers;
- how timeouts and resource limits prevent the cache from worsening an outage;
- which Valkey durability, eviction, and availability properties the design
  relies on and which it explicitly does not rely on.

At least two cache policies must be compared with the supplied access pattern.

## Adversarial evaluation

The grader aligns expirations, evicts a popular entry before its TTL, drives a
same-key burst through two replicas, kills one instance during a fill, slows
DynamoDB, makes Valkey time out, restarts Valkey empty, and changes the source
generation.

Evaluation observes public responses, freshness metadata, dependency requests,
Valkey state and memory, traces, metrics, and exact comparison with the
durable dataset. It does not require a named cache-aside, lock, or stale-
while-revalidate pattern.

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

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **Memcached** is a cache and nothing more — multithreaded, no persistence,
  no rich values — and manages memory in per-size slab classes, so eviction
  pressure lands within an item's size class rather than across the whole
  keyspace.
- **groupcache** and similar in-process caches keep hot entries inside each
  application replica, removing the network hop and the shared store; every
  replica then holds a private view, and coherence between replicas becomes
  the design problem.
- **Amazon ElastiCache** runs the same engine as a managed service with
  failover, so a cache loss arrives as an empty replacement node on the
  provider's schedule; it is excluded from this course by cost policy.

Read their documentation on eviction, expiry, and failover. The lab does not
run them.

## Scope

The expected focused time is four to six hours. Valkey, DynamoDB Local, market
data, two-replica topology, telemetry, workload, and faults are prepared.
Valkey Cluster, Sentinel, a distributed write lock, CDN, durable queue, and
ElastiCache are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, Valkey,
  evidence, and failure contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [Key eviction](https://valkey.io/topics/lru-cache/) — at `maxmemory` Valkey
  evicts by the configured policy, and its LRU and LFU are approximations
  that sample a handful of keys per decision rather than track exact recency.
- [`EXPIRE`](https://valkey.io/commands/expire/) — expired keys are reclaimed
  on access plus a background sampling cycle that tolerates a fraction of
  expired keys lingering in memory, so a TTL is not an exact deadline.
- Implementation pointers do not exist while the spec is `draft`.
