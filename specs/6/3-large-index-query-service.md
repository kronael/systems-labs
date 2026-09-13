---
status: draft
---

# Large index query service — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule drives uniformly random point queries over the whole
keyspace until the touched data far exceeds resident memory and holds full
load through that transition. It starts range scans beside the point load,
raises the stream count to every available core, truncates a named retired
segment while queries against it are in flight, queries records appended
after startup at the declared visibility bound, and sends SIGTERM under full
load before restarting the service against a cold memory state. The harness
appends and seals segments throughout.

Checks do not inspect private functions or require a named access method.
They compare every answer against the generator's seeded truth and observe
per-class latency, resident memory over time, and the reported evidence.
