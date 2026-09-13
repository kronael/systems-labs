# Validator state stream

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Yellowstone gRPC (Dragon's Mouth)** — [documentation](https://github.com/rpcpool/yellowstone-grpc)
- **RPC WebSocket subscriptions** — [documentation](https://solana.com/docs/rpc/websocket)
- **JSON-RPC polling** — [documentation](https://solana.com/docs/rpc/http)

Stuck? See `hints/`.
