# Reliable transaction dispatcher

Design and build a payment dispatch service that lands a queue of payments on
chain, each exactly once, and reports the outcome of every one. A payment names
a funding account, a recipient, and an amount on one of two chains: a local
Solana validator or a local Ethereum development node. The service accepts
payment orders, drives each to a terminal outcome, and answers for the fate of
every identity — landed, with the transaction that carried it, or failed, with
a reason that stays true after the report is written.

Both chains belong to the fixed environment and one service drives both. The
submission pipeline, the account and ordering strategy, the fee policy, the
abandonment rule, the crash-recovery design, and how a landed effect is
recognized are the learner's decisions.

## What you are given

The supplied Compose stack starts `solana-test-validator` and a local Ethereum
development node configured for block-interval mining and real fee ordering,
with the funding accounts pre-funded on both chains. It includes the seeded
payment-order generator, an RPC fault proxy in front of each node, chain
inspection tooling, and deterministic barriers named after payment
identities. Fault scenarios are declarative files driven by the course-wide
fault controller.

The learner owns the dispatcher, its public outcome surface, the application
Compose layer, and `ARCHITECTURE.md`. The starter is Go.
No cloud account and no mainnet funds are required.

## Requirements

Every payment has a stable identity. Every payment reaches a reported terminal
outcome, and a reported outcome is final: a payment reported failed must never
subsequently land, which requires the dispatcher to establish, before it says
so, that no earlier submission can still be included. Every reported outcome
names the assurance under which it was decided, and a payment may not be
reported landed while the chain could still drop the block that carried it.

Each queued payment lands exactly once on its designated chain, across
dispatcher crash and node restart. An independent observer reading each chain
must find, for every funding account, exactly the transactions the outcome
report attributes, with each queued payment attributed exactly once.

The environment fixes two sets of facts. On the Solana node a transaction is
valid only while the blockhash it carries stays within the node's most recent
151, about 60 to 90 seconds, and after expiry it will never execute. On the
Ethereum node an account executes one transaction per nonce in order, a
transaction priced below inclusion blocks every later transaction from its
account, replacing a pending transaction requires a fee premium, and a
transaction nobody watches any more can still be mined hours later.

A stalled payment must not stall unrelated payments on other funding accounts
or on the other chain. The report states the fee spent per payment, and total
spend stays within a learner-declared budget. One payment in a thousand is
unsatisfiable by construction and must terminate with a stated reason.

The scale target is 20,000 queued payments, 10,000 per chain, drawn against
eight distinct funding accounts on each chain with a declared skew toward one
hot account, both chains driven concurrently with at least 256 payments in
flight, and a drain rate of at least half the rate at which each node itself
confirms transactions on the same host. These numbers size the problem so a
design that serializes everything behind one account, or one chain, fails on
its own terms. They are not pass thresholds; thresholds stay relative,
calibrated, structural, or learner-declared.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what distinguishes a payment from the transactions that attempt it, and
  where that boundary lives;
- when resubmission is safe on each chain, and what evidence makes each
  answer true;
- how the dispatcher establishes that an earlier submission can never land,
  on each chain, and why the two proofs differ;
- what must be durable before a submission leaves the process, and what
  recovery reads first after a crash;
- how per-account transaction order on the Ethereum node constrains
  concurrency, and how throughput survives it across accounts;
- how a stranded account is detected and drained, and what draining costs;
- what the declared assurance of a reported outcome means, and what could
  reopen it;
- how fee spend per payment is bounded, and what happens when the bound and
  the backlog conflict;
- which properties the local nodes cannot prove about the public networks.

The document must compare at least two designs for the boundary between a
payment and the transactions that carry it.

## Acceptance evidence

Every queued payment identity appears exactly once in its chain's observed
history, including the payments named in the fault schedule. No reported
outcome changes after it is reported. The stranded Ethereum account drains
without manual intervention and without a second landing. Restarting the
dispatcher or a node loses no payment and duplicates none.

The report records, for every payment: chain, funding account, each submission
with its transaction identity, the landed transaction, the assurance under
which the outcome was decided, the fee paid, and the time from acceptance to
outcome. It records drain rate against each node's own confirmed rate,
in-flight depth over time, and the distribution of fee spent per payment. It
closes by contrasting, with the observed histories as evidence, what a
resubmission means on each chain and why the safe moment to give up differs.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **OpenZeppelin Relayer** — [documentation](https://docs.openzeppelin.com/relayer/)
- **Flashbots Protect** — [documentation](https://docs.flashbots.net/flashbots-protect/overview)
- **Temporal** — [documentation](https://docs.temporal.io/)

## What is outside the problem

The expected focused time is sixteen to twenty-two hours. Both nodes, funded
accounts, the order generator, the fault schedules, and chain inspection are
prepared. Payments use each chain's native transfer; authoring an on-chain
program, token standards, contract calls, MEV, multi-node clusters, and
cross-chain atomicity are outside the problem.

No step requires mainnet funds or a cloud account. Public RPC endpoints are
opt-in, bounded, cached under the shared source directory, and never on a
request path. CI and checks use the local nodes only.

Stuck? See `hints/`.
