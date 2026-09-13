> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [`../../docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  the serverless recast of this product and what it removes.
- [Duffel API reference — Offers](https://duffel.com/docs/api/offers) — the
  reported behaviour the product's expiry rests on: "An offer is only
  available to create an order for a limited time by the traveller before it
  expires, typically within 30 minutes", and past its stated expiry an offer
  "can no longer be used to create an order".
- [wrk2](https://github.com/giltene/wrk2) — a closed-loop generator sends its
  next request only after the previous response arrives, so it coordinates
  with the server and stops measuring during exactly the slow periods it
  exists to find. Solution-bearing: this belongs in hints/, never in
  README.md.
- Implementation pointers do not exist while the spec is `draft`.
