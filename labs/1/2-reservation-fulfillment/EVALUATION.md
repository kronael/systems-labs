> Spoilers. Open only when stuck.

# Reservation fulfillment service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule freezes a worker past the acknowledgement window while a
fulfillment effect is in flight, kills one of several workers mid-effect,
delays the store until the window expires, restarts the broker with work
unacknowledged, repeats the declared share of deliveries, injects work that
fails permanently at known reservation identities, repeats client requests, and
restarts PostgreSQL. Its workload throughout includes many clients contending
on one room with deliberately overlapping night ranges, back-to-back stays
meeting at a boundary day among them, so the schedule that falsifies the
delivery design also exercises both halves of the conflict rule. At the end
the final reservation set is read and no two live reservations for any room
may share a night.

Checks observe only HTTP, SQL-visible state, the provider's request log, broker
state, process lifecycle, query plans, metrics, and the submitted evidence.
They do not require a particular schema, worker structure, or coordination
primitive.

## What a weak design gets wrong

A simple design fails this
lab at its scale target: a single process consuming work one unit at a time
cannot hold 500 reservations per second with 256 units in flight, and a design
that treats delivery as exclusive produces a second fulfillment effect the
moment a worker runs long.
