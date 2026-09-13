> Spoilers. Open only when stuck.

# Reliable transaction dispatcher — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule fires at named barriers, never on a timer and never at
random:

- it holds the submission of a named Solana payment at the RPC boundary until
  the blockhash it carries has expired, then releases it, so the dispatcher
  faces a transaction that is in flight, unlandable, and unacknowledged;
- it kills the dispatcher between the submission of a named payment and the
  recording of that payment's outcome, then restarts it against the same
  nodes;
- it raises the Ethereum node's minimum inclusion price after a named
  payment's transaction enters the pool, leaving that transaction priced
  below inclusion and every later transaction from its account queued behind
  it; after the dispatcher reports recovery, the schedule lowers the price
  again, so a transaction that was merely forgotten rather than displaced
  becomes minable;
- it restarts each node with its ledger intact at the submission of a named
  payment, while that payment and others are in flight.

Evaluation observes chain state through each node's own RPC, the dispatcher's
public outcome surface, process lifecycle, and exact per-payment histories:
for every identity, every transaction that attempted it and every transaction
that landed. It does not require a named recovery pattern, and it keeps
watching after the report — an outcome that flips after it was reported is a
failure regardless of counts.
