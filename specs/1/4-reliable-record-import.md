---
status: draft
---

# Reliable record import

## Brief

Design and build a batch import system for small structured records. Submitted
records enter a leased-delivery environment, valid records become queryable,
temporary failures retry, permanently invalid records terminate in an
inspectable state, and operators can explain the fate of every submitted
identity.

The required environment is a self-run stream broker with per-message
acknowledgement and a relational store. How delivery is consumed, how repeated
and failed deliveries are handled, the data layout behind each record's
queryable fate, and the process decomposition are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts the broker, the store, the record generator,
the fault controller, and an operator query surface. The generator emits a
declared share of duplicates and poison records, and the fault controller
freezes a consumer mid-record, kills one of several consumers, delays the store
past the acknowledgement window, and restarts the broker.

The course supplies the record schema, the generator, the fault schedules
behind `make fault`, and the black-box grader. The learner owns the import system and its tests. No cloud
account is required.

## Requirements

Every submitted identity reaches exactly one explainable fate: queryable,
retrying, or terminal. An operator can ask for any identity and receive its
current fate and how it got there.

A valid record becomes queryable exactly once, however many times it is
delivered. Delivery is at least once: any record may be delivered again, at
any point, and the design must state what makes a repeated delivery safe.

The consumer's in-flight ceiling is a declared number, and the submission
explains what that number does to throughput, to redelivery, and to the store
under the declared load.

A record that exhausts its delivery attempts does not disappear and does not
retry forever. It remains inspectable, and an operator can reintroduce it after
a fix; the submission states where such a record lives and how it returns to
the import path.

Transient and terminal failures must be distinguished, and the submission
states how the code decides. Configuration comes from the standard TOML
contract.

The scale target is an import of two million records, a sustained 1,000 records
per second, 256 records in flight at once, 2 percent of records delivered more
than once, and one poison record in ten thousand. These numbers size the
problem; they are not pass thresholds. Import latency is measured against the
learner's declared service level.

The evidence must state how many records were delivered more than once, how many
reached each terminal state, and the distribution of delivery attempts. How that
measurement is produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what an acknowledgement asserts about the record's fate, and when it is
  honest to send one;
- how the in-flight ceiling was chosen, and what it costs on each side;
- what happens to a record that exhausts its delivery attempts, and how an
  operator returns it to the import path;
- which failures are transient and which are terminal, and how the code decides
  without guessing;
- how a repeated delivery is made harmless at the effect boundary, and what
  identity that rests on;
- which query answers "what happened to identity X" and what it costs at the
  declared volume.

One further question presupposes part of a design and is solution-bearing; it
publishes to `HINTS.md`, never to the task:

- why a consumer that is merely slow receives a redelivery, and what makes the
  second delivery harmless.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

The grader freezes a consumer past the acknowledgement window while a record is
in flight, kills one of several consumers mid-record, delays the store until the
window expires, restarts the broker with records unacknowledged, submits the
declared duplicate share, and injects poison records at known identities. It
then queries the fate of every submitted identity.

The grader does not require a named consumer structure or terminal-state
topology. It observes the store, the operator query surface, broker state, and
the submitted evidence.

## Acceptance evidence

Every submitted identity has exactly one explainable fate. No valid record
becomes queryable twice under the declared duplicate share, a frozen consumer,
or a broker restart. Every poison record reaches an inspectable terminal state
and none retries forever. A reintroduced record completes.

The evidence report includes records per terminal state, the delivery-attempt
distribution, the count of records delivered more than once and how many of
those produced a second effect, import latency against the declared service
level, and the in-flight ceiling's measured effect on throughput. It names the
window in which a record is durable in the broker and not yet queryable.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about:

- **RabbitMQ** redelivers on consumer liveness rather than on a clock: an
  unacknowledged message returns when the channel or connection drops, so a
  merely slow consumer is not handed a duplicate, and its dead-letter exchange
  moves the message for you instead of signalling exhaustion.
- **Kafka** gives the consumer a position in a retained log rather than a
  per-message lease, so there is no redelivery timer at all, one bad record
  blocks its partition until the position moves past it, and useful parallelism
  is capped by partition count.
- **Amazon SQS** carries the same leased model as a hosted service, with the
  poller and the batch outcome contract supplied rather than written — which is
  the recast this curriculum takes in
  [`../2/2-reliable-record-import.md`](../2/2-reliable-record-import.md).

Read their documentation on acknowledgement, redelivery, and dead-lettering.
The lab does not run them.

## Scope

The expected focused time is ten to fourteen hours. The learner builds the
import system and its tests. The broker, store, generator, fault schedules, and grader
are prepared. Authentication, a submission UI, multi-tenant isolation, and
cross-datacenter replication are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  the serverless recast of this product and what it removes.
- [JetStream reliable delivery](https://www.synadia.com/blog/jetstream-reliable-delivery-dlq-replay)
  — what this broker does when a consumer keeps failing a message, and what it
  leaves to the application. Solution-bearing: this belongs in `HINTS.md`,
  never in `README.md`.
- [JetStream acknowledgement](https://docs.nats.io/learn/jetstream/acknowledgment)
  — the acknowledgement window is a timer, and a delivery not resolved before
  it expires is treated as a silent failure and redelivered. Solution-bearing:
  this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
