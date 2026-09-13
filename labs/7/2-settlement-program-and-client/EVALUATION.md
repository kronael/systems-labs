> Spoilers. Open only when stuck.

# Settlement program and client — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Faults fire at named barriers, never on a timer and never at random. The
schedule includes:

- the settlement whose leg count first drives a single-transaction execution
  past the per-transaction compute unit budget; verification asserts it still
  applies exactly once with exact balances;
- the settlement whose account list cannot fit one serialized transaction;
- the growth step that first requires more than 10,240 new bytes of account
  data;
- a payer that, at a named settlement, holds enough lamports for fees but
  not for the rent-exempt minimum of the growth about to happen;
- `SIGKILL` of the client after a named transaction of a multi-transaction
  settlement is confirmed and before the next is submitted, followed by a
  restart;
- duplicate submission of a named settlement, both while it is in progress
  and after it has applied;
- a validator restart at a named settlement barrier with the ledger
  preserved.

Verification replays the accepted settlement stream into an independent
balance computation and compares exact per-participant balances and the full
settlement history against chain state read over RPC. It does not inspect
program internals and does not require a named account layout.
