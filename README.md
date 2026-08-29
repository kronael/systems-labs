# systems-labs

A curriculum of end-to-end systems labs. Each lab prompts a complete, useful
product that a learner designs, breaks, recovers, and defends with evidence.
34 labs across seven phases; seventeen are the core.

**This repository holds specifications only.** There is no code, no
`Makefile`, and no lab directory. Everything below is specified, not
implemented. Nothing here builds or runs yet.

## What a lab is

A lab is not an exercise with an answer. It is a brief: *design and build a
system that does X under these requirements.* The brief fixes the product, its
public behaviour, the environment, the limits, the failures it must survive,
and the evidence it must produce. It never gives the service decomposition,
the schema, the retry algorithm, the cache policy, or the recovery mechanism —
those are the learner's decisions, and getting them wrong is the teaching.

Every lab passes three gates:

| gate | duration | delivers |
|------|----------|----------|
| product | 60–120 min | an end-to-end slice that accepts input, persists or transforms it, exposes a useful result, and survives restart |
| failure | 2–4 h | a deterministic scenario that falsifies the naive design and forces the stated invariant to hold |
| evidence | 1–2 h | a load or deployment run recording the operational tradeoff and one remaining limitation |

Individual labs run six to twenty-five focused hours. The seventeen core labs
run to roughly 150 to 250 focused hours.

## Where the difficulty comes from

From the quirk of the system under study — a notification that never replays,
a lease that is not a deadline, an index that is not yet consistent. Never
from input formats, parsing chores, or boilerplate volume.

Every quirk is grounded in a public document describing real reported
behaviour: a manual page, a post-mortem, a paper, a vendor limit. That
citation is the lab's ground truth. No quirk is invented and then given a
source.

Faults fire at named barriers — *when record 4711 is acknowledged, freeze the
broker* — never on a timer and never at random, so the failure lands at the
same boundary on every run and verification asserts an exact history.

## The four files

A learner who wants the exercise must be able to avoid the answer without
effort. That is why these are separate files and never sections of one.

| file | holds | opened |
|------|-------|--------|
| `README.md` | the task and the technology landscape | by default |
| `HINTS.md` | the design reading | by choice, when stuck |
| `EVALUATION.md` | the answer key | by choice, when checking |
| `ARCHITECTURE.md` | the learner's own reasoning | written, not read |

No lab has a worked solution. A systems lab admits many correct designs, so no
implementation is canonical. `EVALUATION.md` describes what a strong design
holds and how to check it; it does not implement one. There is no grader
binary.

The failure schedule is never a readable file in the learner distribution: its
recipes are compiled into the shared fault controller, and `make fault`
materializes a schedule only while a run is in flight. The standard is
deterrence, not impossibility — the learner owns the machine.

## Phases

The digit directory under `specs/` is the curriculum phase, not a version.

- **1** — local runtime and delivery (7 labs)
- **2** — the same problems on an execution model the learner cannot operate (5)
- **3** — real Internet streaming (2)
- **4** — NoSQL, analytics, portability (4)
- **6** — low-level, in Rust and C (5)
- **7** — blockchain, on Solana and Ethereum (6)
- **8** — search, retrieval, spatial (5)

**Phase 5 no longer exists.** Its lab became `4/5`, and the gap stays open on
purpose. The numbering is not renumbered to close gaps.

## Reading order

- [`specs/index.md`](specs/index.md) — the authoritative catalog and every
  spec's status. Start here.
- [`specs/01-systems-labs.md`](specs/01-systems-labs.md) — the governing spec.
  Every cross-lab contract lives here and nowhere else.
- [`specs/0/`](specs/0/) — the selection record and the four track catalogs.
- [`specs/<phase>/README.md`](specs/) — what a phase is about and which
  technologies it uses.
- [`docs/cloud-access.md`](docs/cloud-access.md) — cloud onboarding. Every
  required gate runs locally with no account; only phase 4 buys anything.
- [`BUGS.md`](BUGS.md) — the open review queue.
- [`CLAUDE.md`](CLAUDE.md) — the operational summary of the governing spec,
  for agents working in this repository.

`specs/` is author-facing and is not part of the learner tree.

## Status

Specification stage. `specs/01-systems-labs.md` is `draft`, which blocks
implementation: creating a lab directory, pinning dependency versions, or
authoring the first lab all wait on it moving to `accepted`.

Known open defects are recorded in [`BUGS.md`](BUGS.md) rather than fixed
silently. At the time of writing, `S26` records that seventeen specs promise
neighbour documentation links they do not yet carry.

## Licence

GPL-3.0. The full text is in [`LICENSE`](LICENSE).

All lab prose, fixtures, and workload generators are original work. External
sources are cited and never copied; [`THIRD_PARTY.md`](THIRD_PARTY.md) records
anything copied or embedded. Dependencies keep their own licences, and their
notices travel with distributed copies.
