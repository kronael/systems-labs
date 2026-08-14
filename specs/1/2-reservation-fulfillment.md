---
status: draft
---

# Reservation fulfillment service

## Brief

Design and build a reservation system that accepts requests, enforces legal
state changes, performs asynchronous fulfillment work, and recovers accepted
work after application or database restarts.

A reservation claims one resource for a half-open time interval, and two live
reservations for the same resource must never overlap.

PostgreSQL is the required system of record. The design must use PL/pgSQL for a
meaningful transactional rule and must incorporate `LISTEN`/`NOTIFY`, while
stating precisely what guarantees each mechanism does and does not provide.
The data layout, the shape and placement of the transactional rule, the path
from accepted work to its fulfillment, and the process decomposition are the
learner's decisions.

## Prepared scaffold

The supplied Compose stack starts PostgreSQL, OpenTelemetry collection, the
fault controller, and an optional second application replica. It includes
migration wiring, generated clients, a million-row deterministic dataset,
concurrency workloads, notification inspection, and the black-box grader.

The stack also includes the fulfillment provider: a prepared endpoint outside
the learner's processes that accepts a fulfillment request for a reservation
identity, records it in an inspectable log, and acknowledges it. That log is
the observable boundary of the fulfillment effect — the grader reads it, and
an effect applied twice is two recorded requests for one reservation. How and
when the design produces that request are the learner's decisions.

The learner owns the database design, application topology, worker behavior,
and application Compose layer. Standard Make targets start the environment,
load data, run feature tests, inject disconnects and restarts, and capture query
plans and evidence.

## Requirements

Clients can create a reservation with an idempotency key, request legal state
changes, and query current and terminal status. Reusing a key with the same
request has one effect; reusing it with different data fails visibly.

No two live reservations for one resource may overlap in time. This holds under
concurrent conflicting requests, not merely in a quiet system, and it holds
through every supported write path. It is a constraint of the product rather
than this lab's study: the study is the delivery and recovery of accepted
asynchronous work — what the required notification mechanism does and does not
guarantee, and how accepted work survives without it.

Invalid state transitions must be impossible through every supported write
path. The fulfillment effect is the request to the prepared fulfillment
provider, recorded there against the reservation's identity. Every
acknowledged reservation requires that effect exactly once; a reservation
cancelled before its effect was applied requires none. Every acknowledged
reservation that requires fulfillment remains discoverable and eventually
reaches a terminal state after worker replacement,
listener disconnect, or process restart. Multiple workers may operate at once
without applying one fulfillment effect twice.

Database errors must map to clear client outcomes. Only errors classified as
transient may retry, and retry must have a stated transaction boundary.

The scale target is a sustained 500 reservation requests per second with
fulfillment keeping pace, 200 concurrent clients mixing creations, state
changes, and status queries, a contention profile in which one resource in a
hundred receives a fifth of all reservation attempts with overlapping
intervals, and the prepared million-row dataset behind every
required query. Fulfillment latency and recovery time are measured against the
learner's declared service level, not a fixed number.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- which invariants belong in PostgreSQL and which belong in application code —
  the non-overlap constraint among them — and what the rejected placements
  cost;
- why the selected PL/pgSQL boundary is transactional and testable;
- what role notification plays and how work remains recoverable without it;
- how multiple workers operate at once without one fulfillment effect being
  applied twice, and how work stranded by a replaced worker still completes;
- how idempotency identity relates to request payload and fulfillment effect;
- which indexes serve each public access pattern and worker operation;
- which failures retry and why the transaction boundary is safe.

One further question presupposes part of a design and is solution-bearing; it
publishes to `HINTS.md`, never to the task:

- how startup, reconnect, and the initial database view avoid a missed-change
  window.

The document must compare at least two viable worker designs without turning a
known library name into the argument.

## Adversarial evaluation

The grader disconnects every listener across a committed reservation, runs
competing workers, repeats client requests, kills a worker at several effect
boundaries, holds a listener transaction open, restarts PostgreSQL, and queries
the million-row dataset. Its workload throughout includes many clients
contending on one resource with deliberately overlapping intervals, so the
schedule that falsifies the notification and recovery design also exercises
the non-overlap constraint; at the end the final reservation set is read and
no two live reservations for any resource may overlap.

The grader observes only HTTP, SQL-visible state, notifications, process
lifecycle, query plans, metrics, and the submitted evidence. It does not
require a particular schema or coordination primitive.

## Acceptance evidence

All acknowledged reservations remain recoverable and reach the correct
terminal state. Invalid transitions fail at the authoritative boundary, and no
two live reservations overlap after any schedule.
Duplicate requests and competing workers produce one logical effect. Failure
is returned to the client or exposed in queryable status rather than only
logged.

The report includes exact request and effect histories, notification timing,
reconnect behavior, worker contention, transaction retries, pool wait time,
and `EXPLAIN (ANALYZE, BUFFERS)` for every required large-data query. It names
the limit of the chosen notification and recovery design.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **RabbitMQ** makes delivery a durable, acknowledged broker product — and pays
  for it with a second system whose acknowledgement can never commit in the
  same transaction as the row it describes.
- **Temporal** owns retry, state transitions, and worker recovery as platform
  features, which moves the recovery rule out of the learner's design and into
  a workflow engine's event history.
- **Kafka** retains what a notification forgets, so a disconnected consumer
  replays from its recorded position — at the cost of operating a log and
  reasoning about its own progress semantics.

Read their documentation on delivery guarantees, acknowledgement, and
recovery. The lab does not run them.

## Scope

The expected focused time is ten to fourteen hours. PostgreSQL, data
generation, telemetry, and faults are prepared. Kafka, generic workflow engines,
replication, cloud databases, and external network calls inside database
transactions are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  PostgreSQL, evidence, and retry contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  the serverless recast of this product and what it removes.
- [`NOTIFY`](https://www.postgresql.org/docs/current/sql-notify.html) —
  notifications reach only sessions that already executed `LISTEN`, delivery
  happens at commit, and a full async queue makes the commit itself fail.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [`LISTEN`](https://www.postgresql.org/docs/current/sql-listen.html) — a
  registration ends with its session, and a new listener receives events
  committed after an instant during its registering transaction's commit step
  — slightly later than any database state that transaction could have
  observed in queries. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
