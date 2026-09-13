# Order activity dashboard

Design and build an order activity system. Producers submit order events;
operators query current order status, per-customer totals, ingestion health,
and processing delay. The complete view must be rebuildable from retained
events while the existing view remains available.

Kafka is the required event transport and PostgreSQL is the query store. How
events are laid out on the transport, how consumption is organized, the query
store's data layout, the rebuild path, and the coordination between transport
and store are the learner's decisions.

## What you are given

The supplied Compose stack starts Kafka, PostgreSQL, OpenTelemetry collection,
and the fault controller. Generated producers emit deterministic valid,
duplicate, conflicting, hot-key, and schema-versioned activity. The scaffold
also contains client contracts, broker inspection, lag capture, scenario
barriers.

The learner owns every application process and the application Compose layer.
The standard Makefile starts dependencies, verifies connectivity, runs the
product, injects broker and process faults, launches a rebuild, and writes the
evidence bundle.

## Requirements

The system accepts created, paid, cancelled, and shipped events with stable
event and order identities. The query API returns current order status and
per-customer totals with visible freshness. Invalid or conflicting events must
remain inspectable.

The submission declares its ordering scope — which events the product promises
to observe in order, and within what key — and that declared scope must hold
under multiple partitions and producers. The declared scope is this lab's
study, and a vacuous declaration does not meet it: the scope must constrain
the keys the generated workload actually contends on — at minimum, the events
of one order relative to each other — and the evidence must show the scope
holding where it is hardest to hold: on the contended keys, at the skewed
customer, and across consumer replacement and rebuild.

Duplicate delivery must not corrupt the materialized result, consumer
replacement and rebalance must not lose acknowledged events, and work waiting
in the application must stay bounded under a skewed workload. These are
constraints the same schedule exercises, not separate studies.

The complete view must be rebuildable from retained events while the existing
view stays queryable and continues to answer correctly. A rebuild that cannot
be completed from retained history must fail rather than half-apply. The
rebuild is not a second study either: it is the check that the declared scope
and the materialized result were well-defined, because replaying retained
history must reproduce them.

The scale target is a sustained 5,000 events per second from sixteen
concurrent producers, a key skew that sends a third of all traffic to one
customer, and 20 million retained events that a rebuild must replay while the
live load continues. Freshness and rebuild duration are measured against the
learner's declared service level, not a fixed number.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- the declared ordering scope, why it is the right one for this product, why
  it is not vacuous against the workload, and how Kafka records preserve it;
- where the declared scope is hardest to hold — the contended keys, the skewed
  customer, replacement, rebuild — and how the design defends it there;
- how partitions affect both correctness and useful parallelism;
- how the supporting constraints hold across crash, shutdown, and rebalance
  boundaries: one logical effect per accepted event however often it is
  delivered, and no acknowledged event lost to a replacement instance;
- how invalid events remain visible without blocking unrelated work;
- how a rebuild reproduces the declared scope and the accepted state from
  retained history while the existing view stays available, and what
  retention, compaction, and schema evolution do to that guarantee.

The design must compare alternatives for at least the keying that carries the
declared scope.

## Acceptance evidence

Accepted activity produces the correct current and aggregate views. The
declared ordering scope is shown holding where the workload makes it hardest —
on the contended keys and at the skewed customer — not only in aggregate.
Repeated delivery has one logical effect while remaining visible in history.
Consumer replacement loses no accepted event. Full rebuild matches the active
accepted state by exact identities, not only counts.

The report records event identity, key, partition, offset, processing attempt,
durable effect, lag, assignment history, queue depth, and rebuild duration. It
states the guarantees at every Kafka-to-PostgreSQL boundary without using the
phrase "exactly once" as a substitute for evidence.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Apache Flink** — [documentation](https://nightlies.apache.org/flink/flink-docs-stable/)
- **Kafka Streams** — [documentation](https://kafka.apache.org/documentation/streams/)
- **RabbitMQ** — [documentation](https://www.rabbitmq.com/docs)

## What is outside the problem

The expected focused time is fifteen to twenty-five hours. Kafka, PostgreSQL,
producer workloads, telemetry, and faults are prepared. Flink, a schema-registry
service, multi-cluster replication, and a browser dashboard are outside the
problem.

Stuck? See `hints/`.
