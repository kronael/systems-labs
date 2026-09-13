---
status: draft
---

# Settlement program and client

## Brief

Design and build a settlement system that lives on a Solana chain: it
registers participants, holds their balances in on-chain accounts, executes
multi-party settlements in which debits and credits net to zero, and answers
queries about any participant's balance and any settlement's status and
history. The system is an on-chain program plus the client that drives it end
to end — creating accounts, funding them, executing settlements, and reading
the result back. Settlement nets because gross flows dwarf net obligations —
the clearing corporation for the US equity market estimates that netting
reduces the value of settlement obligations by approximately 98%, hundreds of
trillions of dollars of trades collapsing into a few trillion that actually
settle — which is why a settlement here nets across thousands of participants
at once.

A settlement either takes full effect or has no observable effect, and it
takes effect exactly once no matter how often it is submitted or how large it
is. The runtime bounds the program before its logic does: a transaction has a
compute unit budget, a byte-size ceiling, a cap on how much an account may
grow per call, and a minimum balance an account must hold to stay alive. Each bound, reached, demands a design change rather than
a parameter change.

The assignment is the whole system: the program's account model, the
instruction set, the client's submission and recovery behavior, the query
path, and end-to-end tests. The account layout, the instruction
decomposition, how a settlement that spans several transactions is split and
tracked, the addressing scheme, and the client's submission and retry design
are the learner's decisions.

## Prepared scaffold

The supplied Compose profile extends the standard scaffold with a local
`solana-test-validator` whose ledger persists across restarts. The course
supplies the settlement generator, which emits seeded participant
registrations and settlement batches whose sizes are chosen to cross each
runtime limit; and the fault controller with declarative scenario files.

The runtime limits are environment facts, current as of 2026-08-14: a
transaction may consume at most 1,400,000 compute units, and an instruction
receives 200,000 by default unless the transaction requests otherwise; a
serialized transaction is at most 1,232 bytes, and every account it
references costs 32 bytes of that; a program may grow an account by at most
10,240 bytes in one call, and an account's data may never exceed 10 MiB; an
account stays on chain only while it holds the rent-exempt minimum for its
size, roughly `(bytes + 128) × 3,480 lamports × 2 years`.

The learner owns the program, in Rust, and the client, in Go.
Everything runs against the local validator with airdropped lamports. No
mainnet funds and no cloud account are involved. Public RPC endpoints are
opt-in, bounded, cached, and never on a request path; no gate requires them.

## Requirements

A settlement is a set of debits and credits over named participants. The
program verifies on chain that the set nets to zero before any balance
moves; the client must never be the only place that check happens. An
admitted settlement spans anywhere from two participants to the largest size
in the scale target, and the same contract holds across that whole range even
though no single transaction can carry the large ones.

Every settlement has a stable identity. Submitting the same settlement twice,
or resuming after a crash, applies it exactly once. For every identity the
system reports one of three states — not applied, in progress, applied — and
that report is truthful against chain state: a settlement reported applied
has moved every balance, one reported not applied has moved none, and one in
progress must never be mistaken for either by a concurrent reader.

Accounts the system creates stay alive. Every account is funded to the
rent-exempt minimum for its current size, including at every intermediate
size while state grows. A funding shortfall surfaces as an error to the
caller before durable state is put at risk, never as an account that silently
ceases to exist.

Lamports locked as rent stay proportional to state actually retained. The
learner declares an overhead factor — locked rent over the minimum for the
bytes in use — and the evidence reports the observed factor. Pre-paying for
storage the workload has not yet needed fails this requirement on cost, not
on style.

The scale target is 10,000 registered participants, 50,000 settlements
offered open-loop at 0.5 × r / m settlements per second with 64 in flight,
100 balance and status queries per second served concurrently, at least 8 MiB
of on-chain account data retained by the end of the run, and a largest single
settlement netting across 2,000 participants. A separate calibration on the
same host measures r, the confirmed transactions per second the node sustains,
and m, the transactions per settlement, for the same seeded workload and
confirmation level; that calibration holds fixed for the evidence run. These
numbers size the problem so that the compute unit budget, the message size
ceiling, and the per-call growth cap are all reached; they are not pass
thresholds. Thresholds stay relative, calibrated, structural, or
learner-declared, per the verification contract.

The evidence must show compute units consumed per transaction against
participants per settlement, transactions issued per settlement against its
size, bytes of account growth per step, and lamports spent on fees and locked
as rent. This lab requires those measurements because the limits are the
lesson; how they are collected is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what one transaction can atomically claim under the compute and size
  ceilings, and what the system's unit of atomicity is when a settlement
  exceeds it;
- how a settlement that spans many transactions presents all-or-nothing
  effect to a concurrent reader, and what state encodes "in progress";
- how the zero-net check is enforced on chain when no single instruction can
  see every leg of the largest settlement;
- how a crashed client, a duplicate submission, and a resumed submission all
  converge on exactly-once effect;
- how state grows past the per-call cap while holding the rent-exempt
  minimum at every intermediate size, and who pays for each step;
- which runtime limit binds first at the scale target, which binds next, and
  what in the evidence shows each one firing;
- what the declared rent-overhead factor is, and what workload change would
  break it.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

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

## Acceptance evidence

Every participant's final balance equals the independently computed value.
Every admitted settlement appears in the history exactly once; no identity
appears twice and none is silently missing. At every query point during the
schedule, no settlement is observable half-applied: each identity's reported
state matches what the balances show. The underfunded-payer scenario ends
with an explicit error and no account below its rent-exempt minimum. The
crashed and restarted client leaves the interrupted settlement either fully
applied exactly once or fully unapplied, and reports which.

The evidence report includes compute units per transaction against
settlement size, transactions per settlement against settlement size, the
account-growth ledger with the rent balance at each step, completed
settlements against the offered load, utilisation as s × m / r for the
achieved settlement rate s, query latency during settlement against the
learner's declared level, total lamports spent on fees, and the observed
rent-overhead factor against the declared one. It names the settlement size
at which the design's transaction count changes shape, and the runtime limit
that would bind next beyond the scale target.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `hints/`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Ethereum (an EVM contract)** —
  [documentation](https://ethereum.org/en/developers/docs/smart-contracts/).
  Prices compute instead of capping it per slot of work: the caller buys gas
  up to a block gas limit that moves by validator signalling, a transaction
  that runs out reverts every change but still pays for the work done, and
  contract storage persists with no minimum balance to maintain — so a bigger
  settlement is a more expensive transaction, not a redesign, until the block
  limit itself is the wall.
- **Stellar Soroban** —
  [documentation](https://developers.stellar.org/docs/build/smart-contracts/overview).
  Also enforces hard per-transaction resource limits on CPU instructions and
  ledger I/O, but its rent runs the other way at this boundary: a persistent
  entry whose TTL lapses is archived and restorable, not gone, so an
  underfunded account is a recoverable state instead of a disappearance.
- **PostgreSQL** — [documentation](https://www.postgresql.org/docs/current/).
  As the off-chain ledger a settlement service would ordinarily sit on, makes
  a settlement over any participant count one ACID transaction; the ceilings
  that shape this lab are operator-set timeouts and hardware there, not
  protocol constants that every node enforces identically.

Read the Ethereum gas documentation, the Soroban state-archival
documentation, and the PostgreSQL transaction documentation. The lab does not
run them.

## Scope

The expected focused time is sixteen to twenty-two hours. The learner builds
the program and the client. The validator, generator, fault controller, and
scenario files are prepared. Token standards, program upgrades, priority fee
markets under real contention, multi-validator clusters, off-chain indexers,
and any mainnet or public-RPC dependency are outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `hints/` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../docs/blockchain-track.md) — blockchain
  track rationale and candidates.
- [NSCC rule filing SR-NSCC-2023-007 (Release No. 34-98213)](https://www.sec.gov/files/rules/sro/nscc/2023/34-98213.pdf)
  — why settlement nets at all: NSCC estimates that in 2022 "netting through
  NSCC's continuous net settlement ('CNS') accounting system reduced the value
  of CNS settlement obligations by approximately 98% or $510 trillion from
  $519 trillion to $9 trillion".
- [Compute budget](https://solana.com/docs/core/fees/compute-budget) — a
  transaction may consume at most 1,400,000 compute units, an instruction
  defaults to 200,000, and a transaction that would exceed a limit is not
  included in a block; the page also names the instruction that requests a
  different limit. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- [Program limitations](https://solana.com/docs/programs/limitations) — the
  runtime bounds a deployed program beyond compute: a 64-frame call stack,
  cross-program invocation depth of 4, and no access to most of `std`.
- [Transactions](https://solana.com/docs/core/transactions) — a serialized
  transaction is at most 1,232 bytes, the IPv6 minimum MTU of 1,280 minus 48
  bytes of headers, and every referenced account address costs 32 of them.
- [Accounts](https://solana.com/docs/core/accounts) — account data is capped
  at 10 MiB, and every account must hold a lamport balance proportional to
  its size — `(bytes + 128) × 3,480 lamports per byte-year × 2 years` — to
  remain on chain.
- [`MAX_PERMITTED_DATA_INCREASE`](https://docs.rs/solana-program-entrypoint/latest/solana_program_entrypoint/constant.MAX_PERMITTED_DATA_INCREASE.html)
  — the maximum number of bytes a program may add to an account during a
  single realloc is 10,240.
- Implementation pointers do not exist while the spec is `draft`.
