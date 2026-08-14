---
status: draft
---

# Finality-aware transfer index

## Brief

Design and build a system that indexes token transfers from the laboratory's
Ethereum node and answers balance and history queries, with an explicit
finality label on every answer.

A receipt is not a fact. The node delivers transfer logs as blocks arrive,
and when the head reorganizes, a log it has already delivered comes back
marked `removed: true` while a competing branch's logs take its place. Any
block behind the head can stop being part of the chain; only a finalized
block carries the protocol's guarantee that it will not, and the protocol
names the ladder itself: the block parameter admits `latest`, `safe`, and
`finalized`. On the public network the finalized boundary trails the head by
roughly two epochs — about thirteen minutes — so a product that answers only
from finalized history is always minutes stale, and a product that answers
only from the head reports transfers that later never happened. The system
therefore serves two views of every question: a fast view that follows the
head and may change, and a settled view that contains finalized history and
never changes. The fast view must never contaminate the settled one.

The assignment is the whole service: ingestion from the node, state, the
query API, recovery, and end-to-end tests. The prompt does not prescribe a
storage layout, a table split, a reorg-handling rule, a checkpoint strategy,
or a query rewrite.

## Prepared scaffold

The supplied Compose stack starts a local Ethereum development node with
token contracts already deployed, the transfer workload driver, the fault
controller, and the black-box grader. The node's control surface can mint
blocks on command, fork the head at a named height, and advance or halt
finality, so every schedule lands on the same boundary in every run. The
workload driver records each transfer it submits, and the grader computes
the exact surviving balances and histories from the node's own chain.

The learner owns the indexing service, its state, the query API, and their
Compose layer. Standard Make targets start the environment, replay
schedules, run faults and load, and collect evidence. CI never contacts a
public endpoint.

## Requirements

The product answers three questions: the balance of a holder for a token,
the transfer history of a holder, and the position of the index — the head,
safe, and finalized blocks the node reports and the block each view has
reached. Every balance and history answer names its view and the block it is
grounded on.

A settled answer is permanent. The same question with the same ground
returns the same balances and the same history for the rest of the run,
across restarts and reorganizations, and no identity from an abandoned
branch ever appears in it. The fast view follows the head: after a
reorganization it reflects the surviving branch, and a withdrawn transfer
disappears from it with an outcome that is observable rather than silent. A
transfer observed more than once contributes once.

The two views degrade honestly. When finality stops advancing, the settled
view ages and its answers say so; fast data is never promoted to keep the
settled view fresh. When the index is behind the node, answers state their
ground rather than blocking or guessing. A restart mid-range loses no
accepted transfer, double-counts nothing, and yields a settled history
identical to an uninterrupted run.

The scale target is one million transfers among 50,000 holders across 20
token contracts over 250,000 blocks, indexing that keeps pace with the
node's own block production — through a cold start over the full range and
at the live head — and 200 concurrent clients querying both views while
indexing continues and while a reorganization is being absorbed. These
numbers size the problem; they are not pass thresholds. Gates stay relative
to the node's rate, structural, or declared by the learner.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what one indexed transfer is, and which of the chain's identities — block
  hash, block number, transaction hash, log index — name it durably;
- what the index believes about a block the node has delivered, and at which
  moment that belief is allowed to change;
- which signal advances the settled boundary, why it can be trusted, and
  what happens while it stalls;
- what a reader of the fast view can observe during a reorganization, which
  intermediate states are visible, and which are promised never to be;
- how the settled view is kept provably free of the fast view's mistakes,
  and what that proof costs on the query path as history grows;
- how the index resumes after a restart when the last blocks it wrote about
  may since have been withdrawn;
- what trace a withdrawn transfer leaves, and what that trace claims to a
  reader who saw the transfer before the fork;
- what the settled view's promise actually rests on, and how the laboratory
  node's finality differs from the public network's.

The architecture must compare at least two designs for the boundary between
the views and state the failure modes and one residual limitation of the
chosen design.

## Adversarial evaluation

Every fault fires at a named barrier through the node's own control surface,
never on a timer and never at random. The grader forks the head at a named
block and mints a competing branch one block longer, so a named transfer the
index has already served returns marked `removed: true` and a competing
transfer takes its place. It repeats the fork at depth 32, past the safe
head and short of the finalized boundary. It delivers one named transfer,
waits until the fast view serves it, withdraws it, and asserts that no
settled answer ever contained it. It kills the indexer at a named block
mid-range and restarts it. It halts finality while the head keeps growing,
then queries the settled view. Balance and history queries run against both
views in every window: before the fork, inside the contested range, and
after finality passes it.

Evaluation compares every answer with balances and histories computed
independently from the node's surviving chain. Counts alone prove nothing: a
settled history must contain exactly the surviving identities and no
identity from an abandoned branch. No storage layout, framework, or client
library is required.

## Acceptance evidence

Balances and histories at both views match the independently computed answer
at every query point. A settled answer never changes across the run, and the
final settled history diffs clean against the logs of the finalized chain —
free of orphaned rows by comparison, not by assertion. The fast view
converges to the surviving branch after every fork. The restart run yields a
settled history identical to the uninterrupted run.

The evidence report records the index's position against the node's head,
safe, and finalized blocks over time; every reorganization with its depth,
the identities withdrawn, and the time the fast view took to converge; query
latency by view against the learner's declared service level; and the
divergence between fast and settled answers across the run. How the
measurements are produced is the learner's choice; no telemetry stack is
required. The report names what the settled view's permanence still depends
on.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **The Graph** moves the reorganization into the framework: `graph-node`
  reverts a subgraph's entities automatically inside a configured reorg
  threshold that defaults to 250 blocks, and a deeper fork can leave the
  store inconsistent — the query answer carries no finality label.
- **TrueBlocks** refuses the fast view: its Unchained Index answers from
  blocks roughly 28 behind the head and calls anything younger unripe, so it
  never withdraws an answer and never gives a fresh one.
- **Etherscan** moves the whole boundary to a third party: a hosted index
  answers at `latest` or a recent block number with no finality label, and
  its reorg handling is invisible behind the service.

Read their documentation on reorg thresholds, index ripeness, and hosted
answers. The lab does not run them.

## Scope and data

The expected focused time is five to seven hours. The node, token contracts,
workload driver, fault schedules, and grader are prepared; the learner
builds the indexing service and the query API. Every required gate runs
against the local node with no mainnet funds and no cloud account. A bounded
recording from a public RPC endpoint is opt-in, checksummed, cached under
the shared sources directory, and never fetched on a request path; CI uses
the local node only. Writing token contracts, wallet and key management,
non-fungible tokens, price data, and consensus-layer verification are
outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  fault-injection, data, evidence, and verification contracts.
- [`../0/3-blockchain-track.md`](../0/3-blockchain-track.md) — the track
  record where this candidate and its origination are filed.
- Quirk origination:
  [Ethereum JSON-RPC API](https://ethereum.org/en/developers/docs/apis/json-rpc/)
  — fixes the log object's `removed` field, "`true` when the log was
  removed, due to a chain reorganization", and the block parameter's `safe`
  and `finalized` tags naming the latest safe head and the latest finalized
  block.
- [Execution API log schema](https://github.com/ethereum/execution-apis/blob/main/src/schemas/receipt.yaml)
  — the canonical specification carries `removed` on every log, so a
  withdrawn log is protocol surface, not a provider extension.
- [MetaMask `eth_getLogs` reference](https://docs.metamask.io/services/reference/ethereum/json-rpc-methods/eth_getlogs/)
  — the track record's cited page; a provider surface documenting the same
  `removed` semantics.
- [Proof-of-stake finality](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/)
  — the first block of each epoch is a checkpoint; a checkpoint that
  attracts two-thirds of staked ETH is justified, the justified checkpoint
  before it becomes finalized, and finality arrives roughly two epochs of 32
  twelve-second slots — about thirteen minutes — behind the head. Reverting
  a finalized block costs an attacker at least one-third of all staked ETH.
- Implementation pointers do not exist while the spec is `draft`.
