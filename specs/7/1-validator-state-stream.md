---
status: draft
---

# Validator state stream

## Brief

Design and build a system that streams account and slot updates out of a
running local Solana validator into a durable store and answers queries over
the result: the current state of a named account, the history of a named
account over a slot range, and the status history of a named slot. Every
answer names the slot it reflects, the commitment level of that slot, and the
age of the data behind it, and the system declares a staleness bound that its
answers keep.

The validator delivers updates by calling into a plugin that the learner
writes and the node loads. The node pushes and the plugin reacts on the node's
own path, so time spent inside a notification is paid for by the validator
itself. Updates arrive at processing time, before any commitment applies, and
among them are updates for slots the chain later abandons. An answer at
confirmed level or above must never reflect state from a slot the chain never
kept.

The assignment is the whole system: the plugin body, the path from
notification to durable state, the query API, the staleness bound, cold start
against a validator that is already running, and end-to-end tests. The data
layout, how updates move from notification to durable state, how commitment
is tracked and reconciled across the account and slot-status streams, the
cold-start recovery approach, and the process decomposition are the
learner's decisions.

## Prepared scaffold

The supplied stack starts a local `solana-test-validator` configured to load
a plugin library from a fixed path, PostgreSQL as the store, a seeded
transaction generator that drives transfers across a declared set of
accounts, and the fault controller. The generator names exact accounts,
amounts, and ordering, so checks can derive the expected history of every
account at every commitment level from the seed and the observed slot stream.
The stack preserves the validator's ledger across restarts within a run.

The course supplies the plugin loading configuration, a starter crate with
the plugin entry point and empty seams, a fault hook on the notification path
that the controller uses to inject a stall at a named barrier, and scenario
files. The learner owns the plugin, the store contents, the query API, and
the tests, all in Rust. The environment is fully local; no public RPC
endpoint, cloud account, or mainnet funds are involved.

## Requirements

The system answers the three queries above for any account and slot the
generator has touched. Each answer carries the slot it reflects, that slot's
commitment level, and the age of the data behind it. The system declares one
staleness bound; when it cannot answer a query within that bound, it says so
rather than silently answering from older state.

Histories are exact. Every update the validator delivered for an account
appears in that account's history exactly once, in slot order, and each entry
carries the commitment the chain ultimately gave its slot. The full
slot-status surface is in play, not the happy path: a slot's processed and
confirmed statuses arrive asynchronously to each other and in either order,
account updates arrive before any status exists for their slot, a slot can be
marked dead, and after a validator start the status stream can resume long
after account updates do. State from a slot that was skipped or marked dead
must never contribute to an answer at confirmed level or above.

The validator stays healthy while the plugin runs. With the store healthy,
the node's slot production stays within a declared factor of the same host's
baseline measured without the plugin. When the store stalls, whatever the
system gives up — growing staleness, refused answers, bounded memory spent —
must be visible at the API rather than silent, and the node's slot production
must remain within the declared factor for the length of the stall.

There is no way to ask the running validator for a snapshot. Started with an
empty store against a validator that already holds state, the system either
reaches a correct answer for a queried account or refuses with a stated
reason; it never presents a partial view as complete. On every start the
validator streams the accounts it restored, marked as startup accounts, and
signals the end of that stream; a validator restart must not duplicate or
corrupt any history.

The scale target is 100,000 distinct accounts under update, the generator
driving the node at the fastest rate the node sustains on the host, and one
run covering at least 5,000 slots and one million account updates while 50
concurrent queries run against the API. Speed is expressed against the node's
own rate, because chain throughput is fixed by the local validator. These
three numbers size the problem; they are not pass thresholds. Thresholds stay
relative, calibrated, structural, or learner-declared.

The evidence must show the age of answered data against the declared bound,
and the node's slot production with and without the plugin on the same host.
How that measurement is produced is the learner's choice; no telemetry stack
is required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what a notification proves at the moment it arrives, and what it cannot yet
  prove about commitment or survival of its slot;
- how much work happens on the validator's calling path, what bounds it, and
  what happens to that bound when the store stalls;
- which state an answer at each commitment level may draw from, and how state
  from a dead or skipped slot is kept out of confirmed answers;
- how account updates are reconciled with slot statuses that arrive later, in
  either order, or not at all;
- what the cold-start story is against a validator that is already running,
  and what the API says while it holds;
- what the staleness bound means operationally, how it is measured, and what
  the API does at the moment it is violated;
- what a validator restart does to the system, and why the startup account
  stream cannot duplicate or corrupt a history.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

Faults fire at named barriers, never on a timer. When the store has
acknowledged the write for account 4711's update at slot S, the store stops
accepting writes for thirty seconds while the generator keeps driving
transfers. When the update for account 4711 at slot S arrives, the fault hook
delays every subsequent notification by a fixed interval, so the plugin
itself is the slow party. When the update for account 4711 at slot S has been
delivered, the controller freezes the validator long enough that slots in the
frozen window are skipped when it resumes; verification reads the
slot-status stream to learn which slots were skipped or marked dead and
asserts that no confirmed answer ever reflected them. The controller also
kills the validator after a named slot is rooted and restarts it against the
preserved ledger, kills the learner's store-side path mid-write, and starts
the whole system with an empty store against a validator holding prior
state.

Verification queries at moments named relative to the notification stream:
after an account update arrives but before any status for its slot, between
a slot's processed and confirmed statuses, and after the root. It does not
inspect private functions or require a named design. It compares every
answer against the expected per-account history derived from the generator
seed and the observed slot stream, at exact account and slot identities,
never counts alone.

## Acceptance evidence

Every queried history matches the derived expected history at every query
point, across the store stall, the plugin stall, the skipped slots, both
kills, and the cold start. No answer at confirmed level or above ever
reflected a skipped or dead slot. The staleness bound held, or its violation
was visible at the API at the moment it happened. Node slot production with
the plugin loaded stayed within the declared factor of the same host's
baseline, including during the store stall.

The evidence report includes the distribution of data age at answer time
against the declared bound, slot production with and without the plugin,
what grew and what was refused during each stall, the exact account and slot
histories around each injected failure, and the cold-start behavior with its
stated cost. It ties the staleness bound to how the bound is enforced rather
than to an observed maximum.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Yellowstone gRPC (Dragon's Mouth)** —
  [documentation](https://github.com/rpcpool/yellowstone-grpc). Consumes the
  same plugin interface but immediately turns it into a filtered gRPC
  subscription service, moving consumers off the validator's boundary onto a
  network stream with per-subscription commitment filters — the back-pressure
  problem is solved once, inside the plugin, for all consumers.
- **RPC WebSocket subscriptions** (`accountSubscribe`) —
  [documentation](https://solana.com/docs/rpc/websocket). These let the node
  apply the requested commitment level before a notification is delivered, so
  the subscriber never faces pre-commitment state — and a dropped connection
  has no replay and no snapshot to resume from.
- **JSON-RPC polling** (`getProgramAccounts`) —
  [documentation](https://solana.com/docs/rpc/http). This returns current
  account state at a chosen commitment level in one shot, which makes cold
  start trivial and every state between two polls invisible.

Read their documentation on commitment filtering, delivery, and cold start.
The lab does not run them.

## Scope

The expected focused time is eighteen to twenty-four hours. The learner
builds the plugin, the store contents, the query API, and the tests. The
validator, the store, the generator, the fault hook, and the fault schedules
are prepared.
The environment is fully local; public RPC use is not part of the lab, and
where a learner consults one anyway it is opt-in, bounded, cached, and never
on a request path. Transaction and block streaming, multiple validators,
fork-choice analysis, and token-program decoding are outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/3-blockchain-track.md`](../0/3-blockchain-track.md) — blockchain
  track rationale and candidates.
- [Agave Geyser plugin docs](https://docs.anza.xyz/validator/geyser) — the
  validator calls the plugin during transaction processing and the plugin
  "should process the notification as fast as possible because any delay may
  cause the validator to fall behind"; processed and confirmed slot statuses
  arrive asynchronously to each other; startup accounts are streamed with a
  flag and an end-of-startup signal. The page's reference-plugin sections
  describe a working persistence design. Solution-bearing: this belongs in
  `HINTS.md`, never in `README.md`.
- [`SlotStatus`](https://docs.rs/agave-geyser-plugin-interface/latest/agave_geyser_plugin_interface/geyser_plugin_interface/enum.SlotStatus.html)
  — the full status set includes `Dead`, and processed-slot state "is not
  derived from a confirmed or finalized block".
- [Configuring state commitment](https://solana.com/docs/rpc) — a processed
  block "is the newest view, but it can still be rolled back".
- [solana-labs/solana#27842](https://github.com/solana-labs/solana/issues/27842)
  — after a start from an older snapshot, account updates resume before slot
  statuses do, leaving updates whose slots have no knowable status; closed
  without a fix.
- [solana-labs/solana#31242](https://github.com/solana-labs/solana/issues/31242)
  — a plugin cannot request account state from the validator outside the
  update stream; closed unimplemented. The discussion names the workaround.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
