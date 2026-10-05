> Spoilers. Open only when stuck.

# Exact trade analytics — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule replays a trade stream containing exact duplicates,
out-of-order arrivals, corrections, and withdrawals at named identities, then
queries during ingest, immediately after a burst, and after the store has been
idle. It drives insert frequency into the regime where the store rejects work,
restarts the ingestion service at a named trade inside a batch, and restarts
the store once a named correction has been accepted.

It withdraws a named trade after an aggregate has already served it, and
queries at each of those points afterwards, reading the store through every
path that can still return a hidden row so the service's completion statement
can be compared with what the store itself still holds.

No answer after a withdrawal's acknowledgement includes the trade, in any
window. The statement of when the trade left the store is no earlier than the
first point at which no path can still return it; a statement issued earlier
fails, and a statement never issued fails. Across the store restart the
service still tells hidden from gone.

Checks do not inspect private functions or require a named table engine. They
compare every aggregate against an independently computed exact answer
derived from the accepted input.
