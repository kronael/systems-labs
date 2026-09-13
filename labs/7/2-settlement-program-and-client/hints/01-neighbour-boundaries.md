> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Ethereum (EVM contract)** — prices compute instead of capping it per slot
  of work: the caller buys gas up to a block gas limit that moves by
  validator signalling, a transaction that runs out reverts every change but
  still pays for the work done, and contract storage persists with no
  minimum balance to maintain — so a bigger settlement is a more expensive
  transaction, not a redesign, until the block limit itself is the wall.
- **Stellar Soroban** — also enforces hard per-transaction resource limits on
  CPU instructions and ledger I/O, but its rent runs the other way at this
  boundary: a persistent entry whose TTL lapses is archived and restorable,
  not gone, so an underfunded account is a recoverable state instead of a
  disappearance.
- **PostgreSQL** — as the off-chain ledger a settlement service would
  ordinarily sit on, makes a settlement over any participant count one ACID
  transaction; the ceilings that shape this lab are operator-set timeouts
  and hardware there, not protocol constants that every node enforces
  identically.
