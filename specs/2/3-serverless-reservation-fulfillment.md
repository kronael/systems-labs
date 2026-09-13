---
status: draft
---

# Serverless reservation fulfillment — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule drives many clients at one resource with deliberately
overlapping intervals, intervals that meet exactly at a boundary among them, so
the schedule exercises both halves of the rule: a shared interior conflicts and
a shared boundary does not. It then reads the final reservation set and asserts
that no two live reservations for that resource overlap. It freezes environments at the freeze
barrier with fulfillment outstanding, destroys environments between
invocations, repeats requests with the same and with altered payloads, repeats a
request long after its first acceptance, drives offered load past what the
configured ceiling admits, and delays a store response until a handler
runs out of time. At the contended resource the controller's transport layer
holds the concurrent attempts before dispatch and releases them together at a
named boundary, and it refuses the share the declared capacity budget excludes
with the store's own failure shape, applying none of what it refused. A run in
which that injection never activates, or never reaches its release boundary,
fails rather than passes.

Checks do not require a named item layout or serialization mechanism. They
observe HTTP, store-visible state, the request history the controller's
transport layer recorded at the store boundary, invocation counts, and the
submitted evidence.
