---
status: draft
---

# Reliable record import

## Brief

Design and build a batch import system for interval meter readings. A utility
collects a consumption reading from every meter in its fleet on a fixed
interval, and accepted readings feed billing, so a reading that arrives twice,
arrives late, or fails validation has a consequence someone pays for. A
reading is a meter identity, an interval timestamp, and a value; there is no
interchange format to parse.

Submitted readings enter a managed-queue-shaped environment, valid readings
become queryable, temporary failures retry, readings that fail validation
terminate in an inspectable state, and operators can explain the fate of every
identity.

The required environment uses the Amazon SQS API, the AWS Lambda batch event
and response shape, and the DynamoDB API. The mandatory run is local. How
delivery is received, how much work runs at once, how long claimed work stays
claimed, the data layout behind duplicate handling and terminal fates, and
the handler decomposition are the learner's decisions.

The lab stands alone; no other lab is a prerequisite. The subject is the
delivery environment — a lease the learner configures but does not run, and a
poller the learner does not write — and it arrives together with the store
that environment is normally paired with.

## Prepared scaffold

The supplied Compose stack starts ElasticMQ, DynamoDB Local, the
Lambda-compatible invocation harness, OpenTelemetry collection, and the fault
controller. It includes a seeded meter fleet with generated valid and poison
readings, a query client, queue and table inspection, and deterministic timing
barriers.

The learner owns the importer, query boundary, application topology, and
application Compose layer. The standard Makefile runs Lambda-envelope
conformance, expires leases, injects item and batch failures, and collects
delivery histories. An optional prepared OpenTofu sandbox supports a bounded
AWS smoke run.

## Requirements

A reading is identified by its meter and the interval it covers; a second
arrival of the same identity is a duplicate, not a second reading. Valid
readings become queryable with their normalized value and processing status,
and that queryable result is what billing reads: a reading must never count
twice, and a reading that failed validation must never count at all — it is
held for review instead. Repeated delivery produces one logical accepted
result. Permanent validation failures are visible to users and operators.
Temporary dependency failures remain eligible for later work.

A reading that terminates does not disappear: it remains inspectable, and an
operator can correct it and reintroduce it; the submission states where such a
reading lives and how it returns to the import path.

One failed item in a mixed batch must not make successful peers disappear or
become unknowable. A killed invocation must leave unfinished readings
recoverable, and so must a reading the environment reclaims and hands to
another consumer because its work was not resolved in time. How long a reading
has waited, how many times it has been delivered, each batch's item outcomes,
terminal failures, and result freshness must be observable.

The record-level contract runs through the supplied Lambda batch envelope, with
no long-lived process anywhere. Delivery-specific behavior may not leak into
the normalized result contract.

The fleet is 21,000 meters on a fifteen-minute interval — 96 readings per
meter per day, 2,016,000 readings a day. The scale target is an import of one
collection day, just over two million records: a sustained 1,000 records per
second through the queue and 256 messages in flight across concurrent batches,
with 2 percent of readings delivered more than once — collection batches
re-sent — and one poison reading in ten thousand, about two hundred a day.
Import latency is measured against the learner's declared service level, not a
fixed number.

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

The failure schedule expires a lease while work is active, kills an invocation around
its durable effect, repeats source identities, fails one item inside a mixed batch,
injects temporary DynamoDB errors, submits a persistent poison record, and
reintroduces a terminal record after its fix.

Evaluation observes the SQS API, Lambda-shaped responses, DynamoDB-visible
results, public query behavior, process lifecycle, traces, metrics, and exact
delivery histories. It does not require a named idempotency pattern.

## Acceptance evidence

Every submitted identity has an explainable outcome. Accepted readings are
queryable, duplicates cannot corrupt results, successful batch peers remain
successful, temporary failures retry, poison work reaches a terminal
inspectable boundary, a corrected and reintroduced reading completes, and a
killed invocation loses no unfinished work.

The report records send, receive, lease, attempt, result, acknowledgement,
retry, and terminal identities with queue age, batch size, concurrency, and
duration. It contrasts the delivery semantics observed in the local run —
a lease the environment expires and a batch outcome the platform envelope
carries — with the hosted service's documented contract, and states which of
the documented properties the local substitutes could not prove. Any claim
about the hosted service itself comes only from the optional smoke run and
must be labelled as such.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

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

## Scope

The expected focused time is seven to ten hours. Queue, database, invocation
harness, telemetry, workloads, and faults are prepared. SNS, EventBridge, Step
Functions, API Gateway, and production AWS are outside the required problem.
The optional smoke run follows the course cost ceiling.

No other lab is a prerequisite, and every artifact the acceptance evidence
needs is produced within this lab. Estimating a value for a missing or failed
reading is excluded: the practice that holds failed readings for review also
fills gaps with estimates, and estimation is a second problem with its own
rules and its own failure model. The import ends where a reading is accepted,
retried, or held — it never invents a value.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, queue,
  Lambda, cost, and evidence contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  the pairing this lab belongs to and what the platform supplies instead.
- [`../1/2-reservation-fulfillment.md`](../1/2-reservation-fulfillment.md) —
  the self-run form of the leased-delivery model this lab receives as a hosted
  contract. The products differ; the delivery model is the contrast.
- [California VEE rules](https://www.sdge.com/sites/default/files/documents/VEE.pdf)
  — *Standards for Validating, Editing, and Estimating Monthly and Interval
  Data*: a reading that fails a required validation check does not flow onward
  as-is; it becomes "data that failed at least one of the required validation
  checks but was determined to represent actual usage" only through reread and
  manual inspection, or is replaced under the estimation rules this lab
  excludes. Grounds the requirement that a failed reading is held inspectable
  and returns only after review. The CPUC EV submetering protocol
  ([R.18-12-006, Attachment A](https://docs.cpuc.ca.gov/PublishedDocs/Published/G000/M496/K420/496420292.PDF))
  states the billing consequence: "If the file fails any of the above checks,
  the file will be rejected and not used for billing purposes."
- [Visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
  — delivery is at-least-once even inside the visibility window, extension
  stops at a hard twelve-hour limit measured from first receipt, and a
  standard queue caps in-flight messages and then returns `OverLimit`.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
