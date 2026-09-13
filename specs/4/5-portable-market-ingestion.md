---
status: draft
---

# Portable market ingestion — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule sends identical event histories through both
environments, kills a Kubernetes worker immediately after a named record's
durable effect, sends SIGTERM once a named record has been accepted, deploys a
version that fails readiness, rolls back with backlog present, expires the SQS
lease of a named record, fails one named batch item, repeats the invocation
carrying a named record, changes a managed Kubernetes field outside OpenTofu,
and scans plan and state for supplied secret sentinels.

Checks observe public input and query contracts, transport-visible
histories, DynamoDB results, Kubernetes rollout state, Lambda-shaped
responses, OpenTofu plans and state, traces, resource use, and cost
evidence. No fixed application process count or adapter pattern is
required.
