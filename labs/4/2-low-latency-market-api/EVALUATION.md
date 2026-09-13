> Spoilers. Open only when stuck.

# Low-latency market API — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule aligns the expirations of named keys, evicts a named
popular entry before its expiry, drives a burst on one named key through two
replicas, kills one instance at the moment a named key's fill is granted,
slows DynamoDB at a named request, makes Valkey time out at a named request,
restarts Valkey empty once a named key has been served, and changes the source
generation at a named record.

Checks observe public responses, freshness metadata, dependency requests,
Valkey state and memory, traces, metrics, and exact comparison with the
durable dataset. They do not require a named cache-aside, lock, or
stale-while-revalidate pattern.
