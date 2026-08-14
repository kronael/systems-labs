---
status: draft
---

# Reliable record import

## Brief

Design and build a batch import system for small structured records. Submitted
records enter a managed-queue-shaped environment, valid records become
queryable, temporary failures retry, permanently invalid records terminate in
an inspectable state, and operators can explain the fate of every identity.

The required environment uses the Amazon SQS API, the AWS Lambda batch event
and response shape, and the DynamoDB API. The mandatory run is local. How
delivery is received, how much work runs at once, how long claimed work stays
claimed, the data layout behind duplicate handling and terminal fates, and
the handler decomposition are the learner's decisions.

This lab is taken after the local import lab. The product is identical on
purpose — the same public behaviour and the same invariants. The subject is
the delivery environment — a lease the learner configures but does not run,
and a poller the learner does not write — and it arrives together with the
store that environment is normally paired with.

## Prepared scaffold

The supplied Compose stack starts ElasticMQ, DynamoDB Local, the
Lambda-compatible invocation harness, OpenTelemetry collection, and the fault
controller. It includes generated valid and poison records, a query client,
queue and table inspection, deterministic timing barriers, and the black-box
grader.

The learner owns the importer, query boundary, application topology, and
application Compose layer. The standard Makefile runs Lambda-envelope
conformance, expires leases, injects item and batch failures, and collects
delivery histories. An optional prepared OpenTofu sandbox supports a bounded
AWS smoke run.

## Requirements

Every source record has stable identity. Valid records become queryable with
their normalized value and processing status. Repeated delivery produces one
logical accepted result. Permanent validation failures are visible to users
and operators. Temporary dependency failures remain eligible for later work.

A record that terminates does not disappear: it remains inspectable, and an
operator can reintroduce it after a fix; the submission states where such a
record lives and how it returns to the import path.

One failed item in a mixed batch must not make successful peers disappear or
become unknowable. A killed invocation must leave unfinished records
recoverable, and so must a record the environment reclaims and hands to
another consumer because its work was not resolved in time. How long a record
has waited, how many times it has been delivered, each batch's item outcomes,
terminal failures, and result freshness must be observable.

The record-level contract runs through the supplied Lambda batch envelope, with
no long-lived process anywhere. Delivery-specific behavior may not leak into
the normalized result contract.

The scale target is an import of two million records, a sustained 1,000
records per second through the queue, and 256 messages in flight across
concurrent batches, with 2 percent of records delivered more than once and one
poison record in ten thousand. Import latency is measured against the
learner's declared service level, not a fixed number.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what constitutes acceptance, completion, retryable failure, and terminal
  failure;
- what an acknowledgement asserts about a record's fate, and when it is honest
  to send one;
- how the lease the design configures relates to work duration and worker
  death;
- how duplicate delivery is detected or tolerated at the effect boundary;
- how a mixed batch's item outcomes are reported so successful peers stay
  successful;
- how poison records stop consuming normal capacity, and how an operator
  returns a terminal record to the import path;
- how the amount of work in flight at once stays within downstream limits;
- which properties the local substitutes cannot prove about AWS.

The document must compare at least two acknowledgement or idempotency designs.

## Adversarial evaluation

The grader expires a lease while work is active, kills an invocation around
its durable effect, repeats source identities, fails one item inside a mixed batch,
injects temporary DynamoDB errors, submits a persistent poison record, and
reintroduces a terminal record after its fix.

Evaluation observes the SQS API, Lambda-shaped responses, DynamoDB-visible
results, public query behavior, process lifecycle, traces, metrics, and exact
delivery histories. It does not require a named idempotency pattern.

## Acceptance evidence

Every submitted identity has an explainable outcome. Accepted records are
queryable, duplicates cannot corrupt results, successful batch peers remain
successful, temporary failures retry, poison work reaches a terminal
inspectable boundary, a reintroduced record completes, and a killed
invocation loses no unfinished work.

The report records send, receive, lease, attempt, result, acknowledgement,
retry, and terminal identities with queue age, batch size, concurrency, and
duration. It contrasts the delivery semantics observed in the local run —
a lease the environment expires and a batch outcome the platform envelope
carries — with the self-run broker's from the local import lab. Any claim
about the hosted service itself comes only from the optional smoke run and
must be labelled as such.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **Kafka** replaces the per-message lease with a consumer-owned offset, so
  progress is a position the consumer moves rather than a clock the queue
  runs, and history survives acknowledgement.
- **RabbitMQ** redelivers when a channel closes rather than when a timer
  expires, which ties recovery to connection lifetime instead of a visibility
  window.
- **AWS Step Functions** moves retry, catch, and terminal-failure routing into
  a platform state machine, so the fate of an identity is orchestrated rather
  than designed.

Read their documentation on delivery, redelivery, and failure routing. The lab
does not run them.

## Scope and cost

The expected focused time is seven to ten hours. Queue, database, invocation
harness, telemetry, workloads, and faults are prepared. SNS, EventBridge, Step
Functions, API Gateway, and production AWS are outside the required problem.
The optional smoke run follows the course cost ceiling.

The local import lab is a prerequisite, and its artifacts must be retained:
its `ARCHITECTURE.md` and its evidence report with the observed delivery
histories. The acceptance evidence contrasts this run's delivery semantics
with that record, so without those artifacts the required evidence cannot be
produced.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, queue,
  Lambda, cost, and evidence contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  the pairing this lab belongs to and what the platform supplies instead.
- [`../1/4-reliable-record-import.md`](../1/4-reliable-record-import.md) — the
  local lab this one recasts.
- [Visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
  — delivery is at-least-once even inside the visibility window, extension
  stops at a hard twelve-hour limit measured from first receipt, and a
  standard queue caps in-flight messages and then returns `OverLimit`.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
