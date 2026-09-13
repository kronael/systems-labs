> Spoilers. Open only when stuck.

# Internet route observatory — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule drops a named peer session after a named update is
acknowledged and restores it later, withdraws a prefix at one collector while
a second collector still announces it, replays a convergence burst at a named
prefix whose transient paths must not surface as routing changes, sends
malformed paths, kills processing at a named Kafka offset mid-stream, and
disconnects the optional recorder. Every fault fires at a named barrier, never
on a timer and never at random.

Checks observe source and API contracts, Kafka-visible identities, queryable
state, scope statements, traces, freshness, lag, resource bounds, and exact
accepted histories. They do not require a particular database schema or
streaming framework.
