# Specification index

## Core catalog

The seventeen core labs, in course order. Each row names the lab's system, the
architecture pressure it applies, and the environment prepared for it.

| # | Spec | System to design | Architecture pressure | Prepared environment |
|---|------|------------------|-----------------------|----------------------|
| 01 | [Resilient quote service](1/1-resilient-quote-service.md) | Fare-search aggregation API over quotes that expire | Overload, partial provider failure, cancellation, shutdown | Provider simulators, telemetry, load generator |
| 02 | [Reservation fulfillment service](1/2-reservation-fulfillment.md) | Stay reservation API with leased asynchronous fulfillment | A lease that is not a lock, a database-enforced invariant, worker recovery | PostgreSQL, NATS JetStream, fault controller, large dataset |
| 03 | [Order activity dashboard](1/3-order-activity-dashboard.md) | Replayable order and customer views | Ordering, progress versus effect, rebuild, skew | Kafka, PostgreSQL, generated producers |
| 04 | [Uninterrupted catalog service](1/4-uninterrupted-catalog-service.md) | Catalog API that serves through changes to its record shape | A shape change against live traffic, two shapes coexisting, an interrupted change resumed | PostgreSQL, declared shape changes, request recorder, 25-million-item catalog |
| 05 | [Auditable transfer service](1/5-auditable-transfer-service.md) | Money ledger with a downstream audit product | Cross-system commit gaps, contention, replay | PostgreSQL, Kafka, history checker |
| 06 | [Shared budget service](1/6-shared-budget-service.md) | Shared spending budgets under concurrent claims | An invariant no single write can be judged against, work the store declines to complete, a retry boundary the application owns | PostgreSQL, overlap-profiled claim workload, fault controller |
| 07 | [Serverless reservation fulfillment](2/3-serverless-reservation-fulfillment.md) | The same reservation product on functions | The phase's lifecycle contract, an invariant no store enforces, fulfillment without a worker | Lambda runtime emulator, DynamoDB Local |
| 08 | [Serverless auditable transfer](2/4-serverless-auditable-transfer.md) | The same transfer product on functions | A commit gap with nothing running between events, a freeze that strands the publish | Lambda runtime emulator, ElasticMQ, DynamoDB Local |
| 09 | [Serverless quote aggregation](2/5-serverless-quote-aggregation.md) | The same quote product on functions | An expiry the design does not own crossing a freeze it does not control, fan-out that scales with environment count | Lambda runtime emulator, provider simulators, external store |
| 10 | [Metered billing API](2/1-metered-billing-api.md) | Subscriptions, metered usage, period-close invoicing | Work that outlives a response, a close larger than one invocation | Lambda runtime emulator, ElasticMQ, DynamoDB Local |
| 11 | [Reliable record import, serverless](2/2-reliable-record-import.md) | Interval meter readings imported on functions | A lease timer the learner does not own, partial batch outcomes | Lambda runtime emulator, ElasticMQ, DynamoDB Local |
| 12 | [Internet route observatory](3/1-internet-route-observatory.md) | Observed routes, churn, and collector health | Observation scope, vantage-point disagreement, convergence versus change | Kafka, RIPE-compatible input, store profiles |
| 13 | [Recoverable route analytics](3/2-recoverable-route-analytics.md) | Stateful routing views with two recovery paths | Checkpoints, external effects, reconstruction, schema change | Kafka, Flink, PostgreSQL, checkpoint storage |
| 14 | [Market history API](4/1-market-history-api.md) | Recent trades and candle queries | Access patterns, hot symbols, pagination, retention | DynamoDB Local, generated and Kraken data |
| 15 | [Low-latency market API](4/2-low-latency-market-api.md) | Freshness-aware accelerated market queries | Cache authority, eviction, stampede, degradation | Valkey, DynamoDB Local, two-replica load |
| 16 | [Exact trade analytics](4/3-exact-trade-analytics.md) | Exact aggregates over full trade history | Background merges, asynchronous mutation, insert frequency, freshness | ClickHouse, trade generator, cached recordings |
| 17 | [Portable market ingestion](4/5-portable-market-ingestion.md) | One domain contract across two compute environments | Lifecycle, delivery, rollout, drift, secrets, cost | Compose, `kind`, Lambda runner, OpenTofu |

The [selection record](../docs/lab-selection.md) expands all twenty candidates,
scores them, and preserves the ten cuts. ClickHouse was named there as the
strongest first addition and became entry 16. CRDTs remain the strongest
alternative conceptual branch, still out.

Phases 6, 7, and 8 are separate catalogs recorded under `0/`. They enter the
curriculum only after the core is `accepted`.

## All specifications

| Spec | Status | Summary |
|------|--------|---------|
| [01-systems-labs.md](01-systems-labs.md) | draft | Governing course spec: every cross-lab contract for scaffold, verification, faults, evidence, data, cost, and licensing |
| [../docs/lab-selection.md](../docs/lab-selection.md) | reference | Twenty expanded candidates, scoring model, ten retained labs, and ten explicit cuts |
| [../docs/low-level-track.md](../docs/low-level-track.md) | reference | Phase 6 in Rust and C: five specced kernel-quirk candidates with origination notes |
| [../docs/blockchain-track.md](../docs/blockchain-track.md) | reference | Phase 7 on Solana and Ethereum: validator, chain data, program, and permissionless delivery |
| [../docs/search-and-retrieval-track.md](../docs/search-and-retrieval-track.md) | reference | Phase 8 on OpenSearch: proteins, news, web crawl, spatial, and relevance evaluation |
| [0/5-shared-scaffold.md](0/5-shared-scaffold.md) | draft | The one generator, fault controller, evidence writer, and template that all 34 labs share, the verification vocabulary their evaluation keys are written in, and the build order |
| [../docs/serverless-contrast-track.md](../docs/serverless-contrast-track.md) | reference | Phase 2 paired against phase 1: what the execution model removes, which pairings earn a lab, and which model is useful where |
| [1/1-resilient-quote-service.md](1/1-resilient-quote-service.md) | draft | Fare-search aggregation under overload and partial provider failure, where a quote past its expiry is worse than none |
| [1/2-reservation-fulfillment.md](1/2-reservation-fulfillment.md) | draft | Stay reservation architecture whose asynchronous fulfillment arrives on a lease that expires while work is still in flight |
| [1/3-order-activity-dashboard.md](1/3-order-activity-dashboard.md) | draft | Queryable, replayable order activity architecture in a Kafka environment |
| [1/4-uninterrupted-catalog-service.md](1/4-uninterrupted-catalog-service.md) | draft | Retail catalog architecture that keeps answering while a mandated price-display change reshapes 25 million records, with no maintenance window |
| [1/5-auditable-transfer-service.md](1/5-auditable-transfer-service.md) | draft | Transfer and audit architecture spanning PostgreSQL and Kafka failure boundaries |
| [1/6-shared-budget-service.md](1/6-shared-budget-service.md) | draft | Shared budget architecture whose conflicting decisions the store declines to complete, leaving the retry boundary to the application |
| [1/7-spectrum-compliance-service.md](1/7-spectrum-compliance-service.md) | draft | Radio fleet kept legally on the air, where an authorization that outlives the evidence for it is a violation and staying dark is its own failure |
| [2/1-metered-billing-api.md](2/1-metered-billing-api.md) | draft | Subscriptions, metered usage, and period-close invoicing on an execution environment that freezes between invocations |
| [2/2-reliable-record-import.md](2/2-reliable-record-import.md) | draft | Interval meter readings imported on functions, in a leased-delivery environment the learner configures but does not run; one of the two phase 2 labs with no phase 1 partner |
| [2/3-serverless-reservation-fulfillment.md](2/3-serverless-reservation-fulfillment.md) | draft | The same reservation product on functions, with a partitioned key-value store as the system of record |
| [2/4-serverless-auditable-transfer.md](2/4-serverless-auditable-transfer.md) | draft | The same transfer product on functions, carrying the commit gap onto an environment where nothing runs between events |
| [2/5-serverless-quote-aggregation.md](2/5-serverless-quote-aggregation.md) | draft | The same fare-search product on functions, where admission is a platform ceiling rather than the design's own decision |
| [3/1-internet-route-observatory.md](3/1-internet-route-observatory.md) | draft | Route observations answered with the scope they rest on, where vantage points disagree and a burst is exploration |
| [3/2-recoverable-route-analytics.md](3/2-recoverable-route-analytics.md) | draft | Stateful route analytics with checkpoint and history-based recovery requirements |
| [4/1-market-history-api.md](4/1-market-history-api.md) | draft | Trade and candle query architecture over DynamoDB-compatible storage |
| [4/2-low-latency-market-api.md](4/2-low-latency-market-api.md) | draft | Freshness-aware market API with Valkey and DynamoDB as available components |
| [4/3-exact-trade-analytics.md](4/3-exact-trade-analytics.md) | draft | Exact aggregates over a growing trade history that duplicates and corrections must never distort |
| [4/5-portable-market-ingestion.md](4/5-portable-market-ingestion.md) | draft | One ingestion contract across local Kubernetes and Lambda-shaped environments |
| [6/1-bounded-memory-record-shipper.md](6/1-bounded-memory-record-shipper.md) | draft | Ordered record shipping to a stalled consumer inside a fixed memory ceiling |
| [6/2-crash-safe-record-store.md](6/2-crash-safe-record-store.md) | draft | Record store whose acknowledgement survives power loss and a one-shot flush error |
| [6/3-large-index-query-service.md](6/3-large-index-query-service.md) | draft | Query service over an index far larger than memory, where a pointer dereference is an I/O |
| [6/4-steady-state-request-service.md](6/4-steady-state-request-service.md) | draft | Request service whose resident memory must not drift upward across a long run |
| [6/5-rate-accurate-replayer.md](6/5-rate-accurate-replayer.md) | draft | Workload replayer that holds a declared rate and reports the latency the caller saw |
| [7/1-validator-state-stream.md](7/1-validator-state-stream.md) | draft | Account and slot updates streamed out of a validator that pays for the plugin's latency |
| [7/2-settlement-program-and-client.md](7/2-settlement-program-and-client.md) | draft | Settlement program and client bounded by the runtime rather than by their own logic |
| [7/3-finality-aware-transfer-index.md](7/3-finality-aware-transfer-index.md) | draft | Transfer index whose settled view stays free of logs the chain later withdrew |
| [7/4-reliable-transaction-dispatcher.md](7/4-reliable-transaction-dispatcher.md) | draft | Queued payments landed exactly once across two chains that disagree about retry |
| [7/6-permissionless-application-hosting.md](7/6-permissionless-application-hosting.md) | draft | An application delivered with no server, domain, or account its publisher operates |
| [7/7-multi-chain-deposit-service.md](7/7-multi-chain-deposit-service.md) | draft | Customer deposits held at addresses on two chains the service cannot sign for, where a movement never names the request that caused it |
| [8/1-protein-similarity-search.md](8/1-protein-similarity-search.md) | draft | Ranked sequence matches whose confidence belongs to the corpus, not to the match |
| [8/2-news-aggregation-service.md](8/2-news-aggregation-service.md) | draft | Feed items grouped under a declared policy, with a freshness the system can actually vouch for |
| [8/3-web-crawl-and-index.md](8/3-web-crawl-and-index.md) | draft | Polite bounded crawl whose real product is the revisit decision under a budget |
| [8/4-spatial-query-service.md](8/4-spatial-query-service.md) | draft | Exact area and proximity answers where the spatial index is only a filter |
| [8/5-relevance-evaluation-service.md](8/5-relevance-evaluation-service.md) | draft | A ranking plus the harness that proves a change to it survives unseen queries |
