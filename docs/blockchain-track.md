---
status: reference
---

# Blockchain track

## Decision

Phase 7 covers Solana and Ethereum through five shapes: validator enhancement,
chain data processing, end-to-end program development, permissionless
deployment and delivery, and chain transaction clients that submit transactions
and establish outcomes without authoring a program. It is a third catalog, and
every candidate is one of the five:

- **validator enhancement** — code that runs inside or beside a node and must
  survive the node's own constraints;
- **chain data processing** — indexing and reconciliation where the chain's
  finality model, not the data format, is the hard part;
- **end-to-end program development** — an on-chain program plus the client that
  must drive it correctly, which is hard on its own terms;
- **permissionless deployment and delivery** — an application published to users
  through content-addressed storage and on-chain state, with no server, domain,
  or account its publisher operates, where availability, currency, and
  immutability must each be established deliberately rather than assumed;
- **chain transaction client** — a client that submits transactions and
  establishes the outcome of each without authoring an on-chain program, where
  the chain's own rules of validity, ordering, and abandonment decide when a
  submission may be repeated and when an outcome may be declared final.

The environment is local: `solana-test-validator` and a local Ethereum
development node. Public RPC endpoints are opt-in, bounded, cached, and never
on a request path, exactly as the
[data contract](docs/contract.md#data-contract) requires. No lab requires
mainnet funds.

All six candidates have full specs. A seventh, cross-chain settlement audit,
was cut: its finality lesson largely repeated 7/3's. They are unscored, and they
enter the catalog only after the core catalog is `accepted`.

## Origination

Each quirk below is documented publicly. Every source is *cited*: no prose,
code, fixture, or test is copied, and each lab's failure schedule and checks
are original.

| # | Candidate | Shape | Quirk it falsifies | Origination |
|---|-----------|-------|--------------------|-------------|
| 7/1 | Validator state stream | Validator enhancement | A plugin can take its time, and every state it sees is real | [Agave Geyser plugin docs](https://docs.anza.xyz/validator/geyser); the validator pushes and the plugin reacts, so slow plugin work pushes back on the node. Reported edges: [slot streaming starts late](https://github.com/solana-labs/solana/issues/27842) and [no snapshot request from the plugin](https://github.com/solana-labs/solana/issues/31242) |
| 7/2 | Settlement program and client | End-to-end program | A program is limited only by its logic | [Compute budget](https://solana.com/docs/core/fees/compute-budget): 200k CU per instruction, 1.4M per transaction. [Program limitations](https://solana.com/docs/programs/limitations). Realloc capped at 10,240 bytes per call, 10 MB per account; rent exemption required |
| 7/3 | Finality-aware transfer index | Data processing | A receipt means it happened | [`eth_getLogs` `removed` flag](https://docs.metamask.io/services/reference/ethereum/json-rpc-methods/eth_getlogs/): orphaned logs are re-sent with `removed: true`. Proof-of-stake moves a block proposed → safe → finalized, roughly two epochs |
| 7/4 | Reliable transaction dispatcher | Chain transaction client | Send and wait works; retry is free | Solana: [blockhash expires after 151 blocks, about 60–90 seconds](https://solana.com/developers/guides/advanced/confirmation), and [durable nonces](https://solana.com/docs/core/transactions/durable-nonces) remove that window at the cost of an `AdvanceNonceAccount` first instruction. Ethereum: nonce gaps stall an account, replacement needs a fee bump |
| 7/6 | Permissionless application hosting | Permissionless delivery | Deployed means permanent, and permissionless means nobody can change it | [IPFS persistence](https://docs.ipfs.tech/concepts/persistence/): the network guarantees discoverability, not availability; unpinned data is garbage-collected, and pinning services pin for a fee. [IPNS](https://docs.ipfs.tech/concepts/ipns/): DHT records expire after 48 hours regardless of validity, and a node republishes only while it runs. [Deploying programs](https://solana.com/docs/programs/deploying): the upgrade authority can replace or close a program, `--final` removes it, and once immutable it can never be updated or closed |
| 7/7 | Multi-chain deposit service | End-to-end program | Money that moves says who moved it | [NEAR chain signatures](https://docs.near.org/chain-abstraction/chain-signatures): "a 'one way' solution to sign and execute outbound transactions happening on other blockchains", with a documented risk of a signature "replayed on a chain you did not intend to interact with". [FDIC, 89 FR 80135](https://www.govinfo.gov/content/pkg/FR-2024-10-02/html/2024-22565.htm): proposed 12 CFR 375.3 requires balances "at the beneficial ownership level", reconciliation "no less frequently than at the close of business daily", and record access "in the event of business interruption, insolvency, or bankruptcy of the third party" |

## Expanded candidates

### 7/1 Validator state stream — specced

Build a plugin that streams account and slot updates out of a local validator
into a store, plus a query API over the result, with a declared staleness
bound.

The plugin runs on the validator's path. Slow work there does not queue
harmlessly; it pushes back on the node. The plugin also sees updates for slots
that are later skipped, and it receives them before any commitment level
applies, so a naive writer publishes state that the chain never kept. There is
no way to ask the validator for a snapshot to start from, so the plugin must
define its own cold-start story.

Shape: validator enhancement. Languages: Rust.

Spec: [`../7/1-validator-state-stream.md`](../labs/7/1-validator-state-stream/README.md).

### 7/2 Settlement program and client — specced

Build an on-chain settlement program and the client that drives it end to end:
create accounts, fund them to rent exemption, execute settlements, and query
the result.

The hard part is that the program is bounded by the runtime rather than by its
own logic. A settlement over many participants exhausts the compute unit
budget; growing an account past 10,240 bytes in one call fails; an account that
falls below the rent-exempt minimum disappears; and a transaction that carries
enough accounts to be useful exceeds the message size limit. Each limit forces
a design change, not a parameter change.

Shape: end-to-end program development. Languages: Rust for the program, Go
for the client.

Spec: [`../7/2-settlement-program-and-client.md`](../labs/7/2-settlement-program-and-client/README.md).

### 7/3 Finality-aware transfer index — specced

Index token transfers from a local Ethereum node and serve balance and history
queries with an explicit finality label on every answer.

A receipt is not a fact. When the head reorganizes, previously delivered logs
come back with `removed: true` and a competing branch's logs arrive. An index
keyed on block number silently keeps the orphaned rows. The product must
therefore answer with two views — a fast unfinalized one and a settled
finalized one — and must never let the fast view contaminate the settled one.

Shape: data processing. Language: Go.

Spec: [`../7/3-finality-aware-transfer-index.md`](../labs/7/3-finality-aware-transfer-index/README.md).

### 7/4 Reliable transaction dispatcher — specced

Land a queue of payments on chain, each exactly once, and report the outcome of
every one.

On Solana the transaction expires with its blockhash after about a minute and a
half; a resubmission after expiry is a different transaction, and a naive retry
loop either gives up on a transaction that later lands or sends a second one
that also lands. On Ethereum a nonce gap stalls every later transaction from
the account, a replacement needs a fee bump, and a transaction the dispatcher
abandoned can still be mined. The two chains disagree about what a retry even
means, which is the contrast this lab exists to teach.

Shape: chain transaction client. Language: Go.

Spec: [`../7/4-reliable-transaction-dispatcher.md`](../labs/7/4-reliable-transaction-dispatcher/README.md).

### 7/6 Permissionless application hosting — specced

Publish and deliver an application — an on-chain program plus the interface
that drives it — to a user with no server, domain, or account the publisher
operates.

Every other lab in the catalog assumes a host the learner controls. This one
removes it, and the cost becomes visible. Content that nothing pins stops
resolving. A content address is the content, so currency lives in a separate
pointer that goes stale or expires. A program with a live upgrade authority can
be replaced under a user who already fetched the old interface, so immutability
is an act rather than a property, and it cannot be undone.

Shape: permissionless deployment and delivery. Languages: Rust, with a
TypeScript client.

Spec: [`../7/6-permissionless-application-hosting.md`](../labs/7/6-permissionless-application-hosting/README.md).

### 7/7 Multi-chain deposit service — specced

Hold customer deposits at addresses on two chains whose keys the service does
not have, and keep an account of who owns what.

The service composes a transaction and a signing network returns a signature
over it — outbound only, with no reply, ever. The addresses are ordinary
addresses, so a movement there carries nothing naming the request that caused
it, strangers can send to them, and a payload signed once can be included
twice. The service can watch the money and cannot ask anyone what it means.
Attribution becomes an argument it has to make and defend, and the
custodial-deposit rules make that answerable rather than philosophical: they
fix what a record must state, how often it must be reconciled, and that the
reconciliation must still run when the party in the middle has gone silent.

Scoped away from its two neighbours deliberately, because the first draft
collided with both. Reorganization is 7/3's subject — there a movement names
itself and the chain withdraws it; here a movement never names itself and the
chain withdraws nothing. Retry identity is 7/4's — that lab holds the keys and
can ask the chain what happened. This one has nobody to ask. Both exclusions
are written into 7/7's Scope, and 7/3 and 7/4 point back.

Shape: end-to-end program development. Language: Go.

Spec: [`../7/7-multi-chain-deposit-service.md`](../labs/7/7-multi-chain-deposit-service/README.md).

## Scale contract

Each blockchain lab carries a speed, a load, and an amount, per the
[lab brief contract](docs/contract.md#lab-brief-contract). Chain
throughput is fixed by the local validator, so the amount is expressed in
accounts, transfers, or slots indexed, and the speed is expressed against the
node's own rate rather than against a wall-clock target.

## Governing references

- [`docs/contract.md`](contract.md) — course-wide learning,
  evidence, source, cost, and repository contracts.
- [`lab-selection.md`](lab-selection.md) — the scored selection that
  produced the original core ten.
- [`low-level-track.md`](low-level-track.md) — the Rust and C track.
- [`labs/README.md`](../labs/README.md) — authoritative list and lifecycle status.
