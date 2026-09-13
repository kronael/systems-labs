> Spoilers. Open only when stuck.

# Auditable transfer service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule kills processes immediately before and immediately after
a named transfer's database commit, its broker acknowledgement, its downstream
effect, and its consumer progress. It
repeats the client request of a named transfer, runs concurrent transfers
against hot accounts, delays Kafka at a named transfer, restarts PostgreSQL at
a named transfer, and rebuilds the audit product.

Each of those barriers is a point the controller holds, never one it notices
once it has passed. The lab declares an adapter for every barrier it names,
mapping it to an effect visible at a boundary the environment fixes — the
store's commit, the broker's acknowledgement, the downstream effect, and the
consumer's recorded progress — and the adapter holds every path able to cross
that boundary until the fault has applied and been confirmed. An effect that is
atomic takes a fault immediately before it or immediately after it and admits
no point inside it. A barrier that the run never maps, never reaches, or
crosses before its fault applies fails the run. The mechanism is the
[fault injection contract](../01-systems-labs.md#fault-injection-contract)'s,
and this lab's required gate depends on its process layer.

Checks observe public APIs, SQL and Kafka state, exact histories,
telemetry, process lifecycle, and resource bounds. They do not require a named
outbox, transaction coordinator, relay topology, or consumer framework.
