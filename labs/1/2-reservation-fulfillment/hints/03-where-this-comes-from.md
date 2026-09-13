> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  PostgreSQL, evidence, and retry contracts.
- [`docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [`docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
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
  this belongs in `hints/`, never in `README.md`.
- [JetStream reliable delivery](https://www.synadia.com/blog/jetstream-reliable-delivery-dlq-replay)
  — what this broker does when a consumer keeps failing a message, and what it
  leaves to the application. Solution-bearing: this belongs in `hints/`,
  never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
