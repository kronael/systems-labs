---
status: draft
---

# Recoverable route analytics

## Brief

Design and build the analytics service behind a route-churn report — the
numbers the Internet routing community publishes about churn each year: how
often each prefix updates, how long a prefix keeps updating once a change
begins, and which prefixes and origins produce the most updates. The service
maintains these views continuously from a stream of route observations and
can recover through two independent paths: restart from saved processing
state and reconstruction from the retained event history. Both paths must
produce the same externally visible report.

Kafka is the required source, Flink is the required stateful processing engine,
and PostgreSQL is the required query store. The internal state model, the
data layout, the checkpoint and reconstruction path, and the process
decomposition between the two recovery routes are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts Kafka, a local Flink cluster, PostgreSQL,
OpenTelemetry collection, checkpoint storage, and the fault controller. It
includes generated and cached route observations — already decoded, each
carrying a prefix, an origin autonomous system, and the provider's event
time — compatible schema versions, checkpoint inspection, retention fixtures,
exact crash barriers, a query API shell, and a lab configuration fixing the
report window and the quiet interval. The generated workload reproduces the
concentration real churn reports measure: a small share of prefixes and
origins carries most of the update volume.

The learner owns the Flink job, query-serving integration, database design,
recovery controls, and application Compose layer. Standard Make targets run a
normal stream, restore saved state, launch an empty-state reconstruction,
inject worker and sink failures, and compare results by exact identity.

## Requirements

The service publishes the three views a route-churn report rests on, computed
in event time over the report window the lab configuration fixes:

- **Per-prefix update rate** — how many updates each tracked prefix received
  in each window.
- **Time to stability** — the event-time span of each change episode: an
  episode opens when an update disturbs a quiet prefix and closes when no
  further update arrives for the configured quiet interval, and the report
  lists each episode with its duration and the period's average.
- **Noisiest prefixes and origins** — the prefixes and the origin autonomous
  systems ranked by update volume, each with its share of the total.

Beside the report, the service exposes the current route observed for each
prefix, processing lag, saved-state age, and reconstruction status.

A stability duration is a statement about event time: a record that arrives
late or out of order changes the measured answer itself, not merely when the
answer appears. The rule for late and reordered records is the learner's to
declare, must be documented, and must hold identically on both recovery paths.

A saved-state restart resumes accepted work. A full reconstruction starts
without operator or query state and reaches the same accepted result: the
same episodes with the same durations, the same counts, the same rankings,
the same current routes.

Queries remain available during reconstruction and never mix incompatible old
and new state. A reconstructed result becomes active only after validation.
Insufficient Kafka history, incompatible state, or an unsupported event schema
must fail visibly before a partial result is exposed.

Repeated processing around recovery may occur, but it cannot corrupt externally
visible state. One additive event-schema change must work across old and new
records under a documented compatibility rule.

The scale target is a sustained 20,000 observations per second over one
million tracked prefixes, 200 million retained events available for
reconstruction, saved processing state in the tens of gigabytes, and 50
concurrent queries served throughout recovery. These numbers size the problem;
they are not pass thresholds. Restore and reconstruction durations are
measured against the learner's declared bounds, not a fixed number.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- which state belongs to Flink, Kafka, PostgreSQL, and the query boundary;
- what a checkpoint covers and which effects remain outside that boundary;
- how external effects behave when failure occurs around checkpoint completion;
- how restore and full reconstruction can be compared by exact identity
  across the report's views;
- how queries avoid partial or mixed reconstruction state;
- how event time, watermarks, and late records behave after recovery, and
  what a late record does to a stability duration the report has already
  published;
- how schema and saved-state compatibility are decided;
- how retention limits whether reconstruction is possible.

At least two sink or activation designs must be compared. Naming an
"exactly-once" mode does not answer the boundary questions.

## Adversarial evaluation

The failure schedule kills a task around a PostgreSQL effect and checkpoint,
removes the newest saved state, restarts workers during rebalance, introduces
old and new event versions, delays events across watermarks at a prefix whose
stability duration the report has already published, and supplies a history
with an insufficient retention prefix.

Checks observe APIs, Kafka positions, Flink checkpoints and metrics,
PostgreSQL state, attempt histories, activation behavior, and reconstruction
checksums. They do not require a named connector or sink pattern.

## Acceptance evidence

Normal processing, saved-state restore, and full reconstruction agree on the
report by exact identity: the same change episodes with the same durations,
the same per-window update counts per prefix, the same rankings, and the
same current route per prefix — never totals alone. Query availability obeys
the submitted contract. Repeated attempts do not corrupt results. Missing
history or incompatible state fails loudly before activation.

The report includes checkpoint and failure timelines, Kafka offsets, state
size, watermark progress, sink attempts, restore duration, reconstruction
duration, earliest retained position, activation history, and the exact
comparison between recovery paths.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Kafka Streams** — [documentation](https://kafka.apache.org/documentation/streams/).
  Keeps its processing state in changelog topics on the broker it already
  reads from, so restoring saved state and reprocessing history are one
  mechanism rather than two paths to reconcile.
- **Spark Structured Streaming** — [documentation](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html).
  Binds a query to its checkpoint location and permits only limited query
  changes across restarts, so an upgrade is a rebuild by default rather than
  a compatibility decision.
- **Materialize** — [documentation](https://materialize.com/docs/). Keeps
  the derived views inside the same system that computes them, so there is
  no external query store to keep consistent across recovery — and no
  independent second recovery path to compare against.

The lab does not run them.

## Scope

The expected focused time is fifteen to twenty hours. Kafka, Flink,
PostgreSQL, input, telemetry, checkpoint storage, and faults are prepared,
but reconciling two independent recovery paths to the same exact report is
not, and a first design usually passes restore before it is falsified on
reconstruction, or the reverse, forcing at least one rebuild of the
activation boundary. The learner does not become a routing analyst: the
views are named and their meanings fixed above, and the report window and
quiet interval come from the lab configuration. Protocol attribute parsing,
path analysis, comparing vantage points, judging whether an observed change
is genuine, custom connectors, hosted stream services, data lakes,
multi-cluster Kafka, and RPKI logic are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, Flink,
  Kafka, evidence, and failure contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- Quirk origination: the Flink documentation on [checkpoints versus
  savepoints](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/checkpoints_vs_savepoints/),
  [fault-tolerance guarantees of sources and
  sinks](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/guarantees/),
  and [window
  lateness](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/operators/windows/).
  The falsified belief is that a checkpoint is a backup and an exactly-once
  switch makes results exactly-once everywhere. The pages fix that checkpoints
  are Flink-owned recovery state, dropped by default when a job terminates;
  that the exactly-once guarantee covers state inside Flink, while what an
  external system observes across recovery depends on that system's own
  coordination with the checkpoint; and that an element arriving after the
  watermark has passed its window is dropped by default. Solution-bearing:
  this belongs in HINTS.md, never in README.md.
- Domain grounding, judged neutral — it reports the numbers a real churn
  report publishes and touches no recovery design: [Geoff Huston, *BGP
  updates in 2025*, APNIC Blog, 9 January
  2026](https://blog.apnic.net/2026/01/09/bgp-updates-in-2025/), measured
  from a single vantage point (AS131072). Most update messages "come from a
  pool of between 30,000 to 80,000 prefixes" of the roughly 1.2 million
  advertised; the "daily average time for an unstable prefix to reach
  stability is now between 20 and 45 seconds"; "Less than 5% of the unstable
  prefixes caused half of all BGP updates during December 2025", and "Fifty
  origin Autonomous System Numbers (ASNs) accounted for one-third of all BGP
  IPv4 updates in this period". These measurements are the lab's three views
  with their published values. The article's explanation of the convergence
  timescale is the observation lab's hint territory and never publishes into
  this lab's learner-facing text; the figures ground the product.
- Implementation pointers do not exist while the spec is `draft`.
