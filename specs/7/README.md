---
status: reference
---

# Phase 7 — Blockchain

## What this phase is about

Phase 7 is a separate catalog on Solana and Ethereum whose quirks live in
the chain's own contracts: what a validator notification proves, what the
runtime bounds before a program's logic does, what a receipt means, and what
deployment actually buys. The track admits four shapes only — validator
enhancement, chain data processing, end-to-end program development, and
permissionless deployment and delivery — and every candidate is one of
them. Each lab is grounded in a documented, publicly reported behavior
recorded in the [track record](../../docs/blockchain-track.md).

## The labs

- [Validator state stream](1-validator-state-stream.md) — a system that
  streams account and slot updates out of a running local validator into a
  durable store and answers queries over the result, with a declared
  staleness bound.
- [Settlement program and client](2-settlement-program-and-client.md) — an
  on-chain settlement program and the client that drives it end to end:
  registering participants, executing multi-party settlements, and
  answering balance and status queries.
- [Finality-aware transfer index](3-finality-aware-transfer-index.md) — a
  token-transfer index over the laboratory's Ethereum node that serves
  balance and history queries with an explicit finality label on every
  answer.
- [Reliable transaction dispatcher](4-reliable-transaction-dispatcher.md)
  — a payment dispatch service that lands a queue of payments on two
  chains, each exactly once, and reports the outcome of every one.
- [Permissionless application hosting](6-permissionless-application-hosting.md)
  — a system that publishes an application and delivers it to users with no
  server, domain, or account the publisher operates.
- [Multi-chain deposit service](7-multi-chain-deposit-service.md) — a
  deposit service holding customer capital at addresses on two chains it
  cannot sign for, where an observed movement names no request and the
  service must still say who owns what.

## The technologies

**Solana** is one of the two chains, run as `solana-test-validator` —
Agave's full-featured single-node cluster on the developer's workstation,
with airdropped lamports and a ledger that persists across restarts. The
phase chooses it because the entire chain runs locally with no rate limits
and no funds at risk, so the runtime's published limits and the validator's
own boundaries can be driven to the point where they matter. Documentation:
[solana.com/docs](https://solana.com/docs),
[Agave test validator](https://docs.anza.xyz/cli/examples/test-validator).

**Ethereum** is the other chain, present as a local development node — a
private blockchain instance with block-interval mining and real fee
ordering. The phase chooses it as the counterpart because its account
model, fee market, and finality ladder differ from Solana's at every
boundary these labs sit on, and the contrast between the two chains is
itself part of the curriculum. Documentation:
[ethereum.org networks](https://ethereum.org/en/developers/docs/networks/).

**Rust** is the program language: the validator plugin in the state-stream
lab — the node is Rust, and the plugin loads into its process — the
on-chain programs in the settlement and hosting labs, and a supported
starter for the dispatcher. Documentation:
[doc.rust-lang.org](https://doc.rust-lang.org/).

**TypeScript** is the client language: the client that drives the
settlement program, the interface bundles of the hosting lab, and a
supported starter for the transfer index. Documentation:
[typescriptlang.org/docs](https://www.typescriptlang.org/docs/).

**Go** is the other supported starter where the learner builds an off-chain
service against a chain: the transfer index and the dispatcher.
Documentation: [go.dev/doc](https://go.dev/doc/).

**PostgreSQL** is the prepared durable store behind the validator state
stream: the destination the plugin must keep feeding while the node keeps
its own pace. Documentation:
[postgresql.org/docs](https://www.postgresql.org/docs/).

**Kubo**, the reference IPFS implementation, forms the prepared four-node
private content network of the hosting lab. The phase chooses it because a
content address names the bytes rather than a host, so an application can
be delivered with no server the publisher operates — and what that
actually guarantees is the lab's subject. Documentation:
[docs.ipfs.tech](https://docs.ipfs.tech/).

**Docker Compose** still carries the scaffold: the validator and the
Ethereum node, the content network, generators, and the fault controller.
Documentation:
[docs.docker.com/compose](https://docs.docker.com/compose/).

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names what a practitioner would
have reached for instead — Yellowstone gRPC, RPC subscriptions and polling,
an EVM settlement contract, Stellar Soroban, The Graph, TrueBlocks,
Etherscan, OpenZeppelin Relayer, Flashbots Protect, Temporal, static
hosting behind a CDN, Arweave, ENS, NEAR Intents, Wormhole Portal,
THORChain — and the one thing each does
differently at that lab's boundary. They appear as reading rather than as
dependencies: the labs never run them, and knowing what each trades away is
part of defending a design.

## Cloud

This phase needs no cloud account and touches no AWS service: every
required gate runs on the local `solana-test-validator` and a local
Ethereum development node, with no mainnet funds anywhere in the
curriculum. The one optional path is non-AWS — public RPC provider free
tiers for bounded recordings — and it is opt-in, bounded, cached, and
never on a request path. See [cloud access](../../docs/cloud-access.md).
