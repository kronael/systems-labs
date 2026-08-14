---
status: draft
---

# Recoverable route analytics

## Brief

Design and build a continuously updated routing analytics service that can
recover through two independent paths: restart from saved processing state and
reconstruction from the retained event history. Both paths must produce the
same externally visible route and churn results.

Kafka is the required source, Flink is the required stateful processing engine,
and PostgreSQL is the required query store. The prompt does not prescribe
operator graph, keying, checkpoint settings, sink coordination, table design,
generation model, schema migration, or activation protocol.

## Prepared scaffold

The supplied Compose stack starts Kafka, a local Flink cluster, PostgreSQL,
OpenTelemetry collection, checkpoint storage, and the fault controller. It
includes generated and cached route observations, compatible schema versions,
checkpoint inspection, retention fixtures, exact crash barriers, a query API
shell, and the black-box grader.

The learner owns the Flink job, query-serving integration, database design,
recovery controls, and application Compose layer. Standard Make targets run a
normal stream, restore saved state, launch an empty-state reconstruction,
inject worker and sink failures, and compare results by exact identity.

## Requirements

The service exposes current observed routes, event-time churn, collector
freshness, processing lag, saved-state age, and reconstruction status. A saved-
state restart resumes accepted work. A full reconstruction starts without
operator or query state and reaches the same accepted result.

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
- how restore and full reconstruction can be compared by exact identity;
- how queries avoid partial or mixed reconstruction state;
- how event time, watermarks, and late records behave after recovery;
- how schema and saved-state compatibility are decided;
- how retention limits whether reconstruction is possible.

At least two sink or activation designs must be compared. Naming an
"exactly-once" mode does not answer the boundary questions.

## Adversarial evaluation

The grader kills a task around a PostgreSQL effect and checkpoint, removes the
newest saved state, restarts workers during rebalance, introduces old and new
event versions, delays events across watermarks, and supplies a history with an
insufficient retention prefix.

Evaluation observes APIs, Kafka positions, Flink checkpoints and metrics,
PostgreSQL state, attempt histories, activation behavior, and reconstruction
checksums. The grader does not require a named connector or sink pattern.

## Acceptance evidence

Normal processing, saved-state restore, and full reconstruction agree on exact
route and window identities. Query availability obeys the submitted contract.
Repeated attempts do not corrupt results. Missing history or incompatible
state fails loudly before activation.

The report includes checkpoint and failure timelines, Kafka offsets, state
size, watermark progress, sink attempts, restore duration, reconstruction
duration, earliest retained position, activation history, and the exact
comparison between recovery paths.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **Kafka Streams** keeps its processing state in changelog topics on the
  broker it already reads from, so restoring saved state and reprocessing
  history are one mechanism rather than two paths to reconcile.
- **Spark Structured Streaming** binds a query to its checkpoint location and
  permits only limited query changes across restarts, so an upgrade is a
  rebuild by default rather than a compatibility decision.
- **Materialize** keeps the derived views inside the same system that computes
  them, so there is no external query store to keep consistent across
  recovery — and no independent second recovery path to compare against.

Read their documentation on state stores, checkpoint compatibility, and view
maintenance. The lab does not run them.

## Scope

The expected focused time is six to eight hours. Kafka, Flink, PostgreSQL,
input, telemetry, checkpoint storage, and faults are prepared. Custom
connectors, hosted stream services, data lakes, multi-cluster Kafka, and RPKI
logic are outside the problem.

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
  watermark has passed its window is dropped by default.
- Implementation pointers do not exist while the spec is `draft`.
