# systems-labs

A curriculum of end-to-end systems labs. Each lab hands you a brief — *design
and build a system that does X under these requirements* — and then breaks it.
34 labs across seven phases; seventeen are the core.

**Nothing here runs yet.** This repository holds the specifications for those
labs and no code: no `Makefile`, no lab directory, no starter. You can read
what every lab will ask of you, and you cannot yet do one. See
[Status](#status) before you plan time around it.

## What a lab asks of you

A lab is not an exercise with an answer. The brief fixes the product, its
public behaviour, the environment, the limits, the failures it must survive,
and the evidence it must produce.

It does not give you the service decomposition, the schema, the retry
algorithm, the cache policy, or the recovery mechanism. Those are yours, and
getting them wrong is the teaching. Your first design is supposed to fail —
the redesign after it fails is the part you are paying for.

Every lab passes three gates:

| gate | duration | you deliver |
|------|----------|-------------|
| product | 60–120 min | an end-to-end slice that accepts input, persists or transforms it, exposes a useful result, and survives restart |
| failure | 2–4 h | a deterministic scenario that falsifies your first design and forces the stated invariant to hold |
| evidence | 1–2 h | a load or deployment run recording the operational tradeoff and one remaining limitation |

A single lab runs six to twenty-five focused hours. The seventeen core labs run
to roughly 200 to 300 focused hours in total.

## Where the difficulty comes from

From the quirk of the system you are studying — a notification that never
replays, a lease that is not a deadline, an index that is not yet consistent.
Never from input formats, parsing chores, or boilerplate volume. A lab that is
merely laborious has failed.

Every quirk is grounded in a public document describing real reported
behaviour: a manual page, a post-mortem, a paper, a vendor limit. That citation
is the lab's ground truth.

## Keeping the answer away from yourself

A learner who wants the exercise has to be able to avoid the answer without
effort. That is why each lab is four files rather than four sections.

| file | holds | you open it |
|------|-------|-------------|
| `README.md` | the task and the technology landscape | by default |
| `HINTS.md` | the design reading | by choice, when stuck |
| `EVALUATION.md` | the answer key | by choice, when checking |
| `ARCHITECTURE.md` | your own reasoning | you write it |

No lab has a worked solution. A systems lab admits many correct designs, so no
implementation is canonical. `EVALUATION.md` says what a strong design holds
and how to check it; it does not implement one. There is no grader binary and
no score.

The failure schedule is never a readable file: its recipes compile into the
shared fault controller, and `make fault` materializes a schedule only while a
run is in flight. The standard is deterrence, not impossibility — you own the
machine, and reading it is a deliberate act, like opening `HINTS.md`.

## Where to start

Read [`specs/index.md`](specs/index.md). Its core catalog lists the seventeen
core labs in course order, and lab 01 is the intended entry point.

Phase 1 is where a learner starts. The later phases assume it: three of the
five phase 2 labs recast a phase 1 product on an execution model you cannot
operate, and each one needs your phase 1 design and evidence to compare
against. The other two stand alone and need no earlier lab.

## Phases

The digit directory under `specs/` is the curriculum phase, not a version.

- **1** — local runtime and delivery (7 labs)
- **2** — the same problems on an execution model you cannot operate (5)
- **3** — real Internet streaming (2)
- **4** — NoSQL, analytics, portability (4)
- **6** — low-level, in Rust and C (5)
- **7** — blockchain, on Solana and Ethereum (6)
- **8** — search, retrieval, spatial (5)

**Phase 5 no longer exists.** Its lab became `4/5`, and the gap stays open on
purpose.

## What you will need

Every required gate runs locally, and no lab needs a cloud account to pass.
Labs name heavyweight dependencies — PostgreSQL, Kafka, DynamoDB Local,
ClickHouse, Valkey, Flink — and each is driven hard enough that its quirk
actually fires.

Go is the starter language unless the environment dictates otherwise: Rust and
C in phase 6, the chain's own language for an on-chain program, TypeScript for
a browser bundle.

[`docs/cloud-access.md`](docs/cloud-access.md) covers the optional cloud path.
Phases 2 and 4 are the only ones that can bill, and only on the optional path.

## Status

Specification stage, and honestly so.

`specs/01-systems-labs.md` is `draft`, and that blocks implementation: creating
a lab directory, pinning dependency versions, and authoring the first lab all
wait on it moving to `accepted`. Every other spec reads `draft` too.

Open defects are recorded in [`BUGS.md`](BUGS.md) rather than fixed silently.
Seven are open as of 2026-09-12, and each one is a redesign waiting on a
decision. Four of them ask the same question: whether a lab's central failure
can be reproduced on the local dependency at all.

## Contributing

[`CONTRIBUTING.md`](CONTRIBUTING.md) holds the authoring side: the teaching
contract every lab obeys, how to add a lab, the review queue, and the records
behind which labs exist and why.

## Licence

GPL-3.0. The full text is in [`LICENSE`](LICENSE).

All lab prose, fixtures, and workload generators are original work. External
sources are cited and never copied; [`THIRD_PARTY.md`](THIRD_PARTY.md) records
anything copied or embedded. Dependencies keep their own licences, and their
notices travel with distributed copies.
