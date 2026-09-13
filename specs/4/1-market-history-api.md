---
status: draft
---

# Market history API — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule sends a concentrated symbol burst, overlaps provider
cursors, repeats identities, kills ingestion immediately before a named
trade's durable write and again immediately after it, forces multi-page
queries, reads a named trade through a secondary access path immediately after
its write, retains physically expired items, and introduces malformed
precision data.

The concentrated burst falsifies a key design only because the controller
supplies the ceiling the local store does not: downloadable DynamoDB ignores
provisioned throughput settings, bounds read and write speed only by the host
machine, and partitions no table, so no volume of generated trades makes it
throttle. The transport layer is the layer this lab's required gate depends
on. In front of the store it reads each request's table and index keys,
operation type, and item size, holds the request before dispatch, applies the
scenario's declared partition mapping and per-partition capacity budget,
forwards what that budget admits, and refuses the rest with the store's own
throttling outcome, applying none of what it refused. The same layer supplies
the propagation delay the local store also omits: it holds a named trade's
secondary-index read behind the write it follows, for the interval the
scenario declares. Injection ends at its
declared release boundary, and a run in which it never activates fails rather
than passes.

That gate proves that the submitted key design holds its invariants and its
declared service level inside a declared capacity model. It does not prove
how the hosted service's own partition adaptation would treat that design;
`EVALUATION.md` states that boundary, and the optional hosted smoke run is
what compares the modelled limit against the real one.

Checks observe public APIs, DynamoDB requests and items, source histories,
telemetry, key distribution, request counts, the requests the transport layer
admitted and refused, and exact candle reconciliation.
They do not require a particular single-table or multi-table pattern.
