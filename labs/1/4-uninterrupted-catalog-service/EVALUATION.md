> Spoilers. Open only when stuck.

# Uninterrupted catalog service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule adopts each declared change while the full workload runs.
It starts one change at the moment a long-running listing request is open and
holds that request open across it; it kills the application process when a
named item identity has been published in the new shape and its neighbour has
not; it restarts PostgreSQL mid-adoption; it submits writes at named item
identities on both sides of that boundary while the change runs; it applies a
second change while one is in progress; it withdraws a change after it has
partly landed; and it repeats publications of already-acknowledged revisions
throughout.

At the end the full catalog is read. Every item is published in exactly one
shape, every acknowledged publication is present exactly once at its
acknowledged revision, and every value under the new shape matches the
declared change's stated rule — where the rule exempts an item, that means
the value's absence.

Checks observe only HTTP, SQL-visible state, the request recorder's history,
process lifecycle, query plans, metrics, and the submitted evidence. They do
not require a particular schema, adoption structure, or coordination
primitive.

## What a weak design gets wrong

A simple design fails this lab at its scale target: adopting a
declared change against 25 million items as one operation runs for minutes,
and every request arriving while it runs waits behind it, so the change a
small catalog absorbs invisibly becomes an outage the request recorder writes
down.
