# Finality-aware transfer index

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
query API, recovery, and end-to-end tests. The storage layout, how the fast
and settled views are separated and reconciled, the reorg-handling rule, the
checkpoint and recovery strategy, and the query path are the learner's
decisions.

## What you are given

The supplied Compose stack starts a local Ethereum development node with
token contracts already deployed, the transfer workload driver, and the
fault controller. The node's control surface can mint blocks on command,
fork the head at a named height, and advance or halt finality, so every
schedule lands on the same boundary in every run. The workload driver
records each transfer it submits, and verification computes the exact
surviving balances and histories from the node's own chain.

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`HINTS.md`, because saying it here would point straight at the answer.

- **The Graph** — [documentation](https://thegraph.com/docs/en/)
- **TrueBlocks** — [documentation](https://trueblocks.io/docs/)
- **Etherscan** — [documentation](https://docs.etherscan.io/)

Stuck? See `HINTS.md`.
