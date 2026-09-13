> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold, queue,
  Lambda, cost, and evidence contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [`../../docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  the pairing this lab belongs to and what the platform supplies instead.
- [`../1/2-reservation-fulfillment.md`](../../../../labs/1/2-reservation-fulfillment/README.md) —
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
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
