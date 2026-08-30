---
status: draft
---

# Reservation fulfillment service

## Brief

Design and build a reservation system for stays: a stay holds one room for a
range of nights. The system accepts reservation requests, enforces legal state
changes, performs the asynchronous work that fulfills each accepted stay, and
recovers accepted work after application, broker, or database restarts.

A reservation claims one room for a half-open range of days: the arrival day
is part of the stay, the departure day is not, because departure morning frees
the room for that day's arrival. A room holds one stay per night, so two live
reservations for the same room conflict exactly when they share a night, and
back-to-back stays — one ending the day the other begins — share no night and
do not conflict. Both halves of that rule must hold while many clients request
overlapping ranges for the same room at once.

The required environment is PostgreSQL as the system of record and a self-run
stream broker with per-message acknowledgement carrying the fulfillment work.
The data layout, the placement of the reservation rules, how fulfillment work
is consumed, how repeated and failed deliveries are handled, and the process
decomposition are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts PostgreSQL, the broker, OpenTelemetry
collection, the fault controller, and an optional second application replica.
It includes migration wiring, generated clients, a million-row deterministic
dataset, and concurrency workloads.

The stack also includes the fulfillment provider, standing in for the desk
that makes a room ready for its stay: a prepared endpoint outside the
learner's processes that accepts a fulfillment request for a reservation
identity, records it in an inspectable log, and acknowledges it. Each recorded
request is an order the desk carries out, so the log is the observable
boundary of the fulfillment effect — an effect applied twice is two recorded
orders for one stay, the same room readied twice. How and when the design
produces that request are the learner's decisions.

The learner owns the database design, application topology, worker behaviour,
and application Compose layer. Standard Make targets start the environment,
load data, run feature tests, inject disconnects and restarts, and capture
query plans and evidence.

## Requirements

Clients can create a reservation with an idempotency key, request legal state
changes, and query current and terminal status. Reusing a key with the same
request has one effect — a retried request is the same stay, never a second
one; reusing it with different data fails visibly.

No two live reservations for one room may share a night; stays that meet only
at the boundary day do not conflict. This holds under concurrent conflicting
requests, not merely in a quiet system, and it holds through every supported
write path. Invalid state transitions must be impossible through every
supported write path.

The fulfillment effect is the request to the prepared fulfillment provider,
recorded there against the reservation's identity: the order that readies the
room for the stay. Every acknowledged reservation requires that effect exactly
once. Applied twice, the desk does one stay's work twice; never applied, the
reservation holds its nights while the room is never made ready — a stay kept
in the record and missed in the world. A reservation cancelled before its
effect was applied requires none: no order was placed, so there is nothing to
undo.

Fulfillment work is delivered at least once. Any unit of work may be delivered
again, at any point, and the design must state what makes a repeated delivery
safe. Several workers may consume at once. The consumer's in-flight ceiling is
a declared number, and the submission explains what that number does to
fulfillment latency, to redelivery, and to contention in the store.

Work that exhausts its delivery attempts does not disappear and does not retry
forever. It remains inspectable, and an operator can reintroduce it after a
fix; the submission states where such work lives and how it returns to the
fulfillment path. Transient and terminal failures must be distinguished, and
the submission states how the code decides.

Every acknowledged reservation that requires fulfillment remains discoverable
and eventually reaches a terminal state after worker replacement, broker
restart, or process restart. Database errors must map to clear client outcomes.
Only errors classified as transient may retry, and retry must have a stated
transaction boundary.

The scale target is a sustained 500 reservation requests per second with
fulfillment keeping pace, 200 concurrent clients mixing creations, state
changes, and status queries, a contention profile in which one room in a
hundred receives a fifth of all reservation attempts with overlapping night
ranges, 256 fulfillment items in flight at once, 2 percent of fulfillment
deliveries repeated, and the prepared million-row dataset behind every required
query. These numbers size the problem; they are not pass thresholds.
Fulfillment latency and recovery time are measured against the learner's
declared service level.

The evidence must state how many fulfillment deliveries were repeated, how many
of those produced a second effect, the distribution of delivery attempts, and
the count of reservations in each terminal state. How that measurement is
produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- which invariants belong in PostgreSQL and which belong in application code —
  among them the rule that a shared night is a conflict and a shared boundary
  day is not — and what the rejected placements cost;
- what an acknowledgement asserts about a unit of fulfillment work, and when it
  is honest to send one;
- how the in-flight ceiling was chosen, and what it costs on each side;
- how several workers consume at once without one fulfillment effect being
  applied twice, and how work stranded by a replaced worker still completes;
- how idempotency identity relates to request payload and fulfillment effect;
- what happens to work that exhausts its delivery attempts, and how an operator
  returns it to the fulfillment path;
- which failures are transient and which are terminal, and how the code decides
  without guessing;
- which indexes serve each public access pattern and worker operation;
- which query answers "what happened to reservation X" and what it costs at the
  declared volume.

One further question presupposes part of a design and is solution-bearing; it
publishes to `HINTS.md`, never to the task:

- why a worker that is merely slow sees its work delivered again, and what
  makes the second delivery harmless when the first has already begun.

The document must compare at least two viable worker designs without turning a
known library name into the argument, and state one residual limitation.

## Adversarial evaluation

The failure schedule freezes a worker past the acknowledgement window while a
fulfillment effect is in flight, kills one of several workers mid-effect,
delays the store until the window expires, restarts the broker with work
unacknowledged, repeats the declared share of deliveries, injects work that
fails permanently at known reservation identities, repeats client requests, and
restarts PostgreSQL. Its workload throughout includes many clients contending
on one room with deliberately overlapping night ranges, back-to-back stays
meeting at a boundary day among them, so the schedule that falsifies the
delivery design also exercises both halves of the conflict rule. At the end
the final reservation set is read and no two live reservations for any room
may share a night.

Checks observe only HTTP, SQL-visible state, the provider's request log, broker
state, process lifecycle, query plans, metrics, and the submitted evidence.
They do not require a particular schema, worker structure, or coordination
primitive.

## Acceptance evidence

All acknowledged reservations remain recoverable and reach the correct terminal
state. Invalid transitions fail at the authoritative boundary, and no two live
reservations share a night after any schedule. No reservation receives a second
fulfillment effect under the declared repeat share, a frozen worker, or a
broker restart. Work that fails permanently reaches an inspectable terminal
state and none retries forever; a reintroduced unit completes. Failure is
returned to the client or exposed in queryable status rather than only logged.

The report includes exact request and effect histories, repeated deliveries and
how many produced a second effect, the delivery-attempt distribution, the
in-flight ceiling's measured effect on fulfillment latency, worker contention,
transaction retries, pool wait time, and `EXPLAIN (ANALYZE, BUFFERS)` for every
required large-data query. It names the window in which accepted work is
durable and not yet fulfilled, and the limit of the chosen recovery design.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and their
documentation links publish into `README.md`; the boundary difference stated
with each publishes into `HINTS.md`, because naming what a neighbour does
differently here points at this lab's quirk.

- **RabbitMQ** — [documentation](https://www.rabbitmq.com/docs/confirms).
  Redelivers on consumer liveness rather than on a clock: unacknowledged work
  returns when the channel or connection drops, so a merely slow worker is not
  handed a duplicate, and its dead-letter exchange moves the message for you
  instead of signalling exhaustion.
- **Kafka** — [documentation](https://kafka.apache.org/documentation/#design).
  Gives the consumer a position in a retained log rather than a per-unit lease,
  so there is no redelivery timer at all, one bad unit blocks its partition
  until the position moves past it, and useful parallelism is capped by
  partition count.
- **Temporal** — [documentation](https://docs.temporal.io/encyclopedia). Owns
  retry, state transitions, and worker recovery as platform features, which
  moves the recovery rule out of the learner's design and into a workflow
  engine's event history.

The lab does not run them.

## Scope

The expected focused time is fourteen to twenty hours. PostgreSQL, the broker,
data generation, telemetry, and faults are prepared. A simple design fails this
lab at its scale target: a single process consuming work one unit at a time
cannot hold 500 reservations per second with 256 units in flight, and a design
that treats delivery as exclusive produces a second fulfillment effect the
moment a worker runs long. Pricing, payment, and any record of who occupies a
room are outside the problem, as are generic workflow engines, replication,
cloud databases, and external network calls inside database transactions.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  PostgreSQL, evidence, and retry contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  the serverless recast of this product and what it removes.
- [RFC 5545, section 3.6.1](https://www.rfc-editor.org/rfc/rfc5545.html#section-3.6.1)
  — the calendar standard fixes the stay's interval convention in one sentence:
  'The "DTEND" property for a "VEVENT" calendar component specifies the
  non-inclusive end of the event.' Section 3.8.2.2 defines the property itself
  and adds only that its value 'MUST be later in time than the value of the
  "DTSTART" property'; the non-inclusive sentence is section 3.6.1's.
- [JetStream acknowledgement](https://docs.nats.io/learn/jetstream/acknowledgment)
  — the acknowledgement window is a timer, and a delivery not resolved before it
  expires is treated as a silent failure and redelivered. Solution-bearing:
  this belongs in `HINTS.md`, never in `README.md`.
- [JetStream reliable delivery](https://www.synadia.com/blog/jetstream-reliable-delivery-dlq-replay)
  — what this broker does when a consumer keeps failing a message, and what it
  leaves to the application. Solution-bearing: this belongs in `HINTS.md`,
  never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
