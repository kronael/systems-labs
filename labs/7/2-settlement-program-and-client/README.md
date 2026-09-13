# Settlement program and client

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Ethereum (an EVM contract)** — [documentation](https://ethereum.org/en/developers/docs/smart-contracts/)
- **Stellar Soroban** — [documentation](https://developers.stellar.org/docs/build/smart-contracts/overview)
- **PostgreSQL** — [documentation](https://www.postgresql.org/docs/current/)

Stuck? See `hints/`.
