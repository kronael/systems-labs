> Spoilers. Open only when stuck.

# Portable market ingestion — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule sends identical event histories through both
environments, kills a Kubernetes worker immediately after a named record's
durable effect, sends SIGTERM once a named record has been accepted, deploys a
version that fails readiness, rolls back with backlog present, expires the SQS
lease of a named record, fails one named batch item, repeats the invocation
carrying a named record, suspends the environment on the process layer
immediately after a named record's result is written and before the batch
response returns, then thaws it for a later invocation, changes a managed Kubernetes field outside OpenTofu,
and scans plan and state for supplied secret sentinels.

Checks observe public input and query contracts, transport-visible
histories, DynamoDB results, Kubernetes rollout state, Lambda-shaped
responses, OpenTofu plans and state, traces, resource use, and cost
evidence. No fixed application process count or adapter pattern is
required.
