# systems-labs

A curriculum of end-to-end systems labs. Each lab gives you a brief, *design
and build a system that does X under these requirements*, and then it breaks
what you built. 34 labs across seven phases, seventeen of them the core.

Nothing here runs yet. Every lab exists as text and none of them as code: no
`Makefile`, no starter, no environment to bring up. You can read what every lab
will ask of you. You cannot do one yet. Read [Status](#status) before you plan
time around it.

## Who writes this, and how

I write this with a language model, and most of the prose you are reading came
out of one. The wanting is mine. I want these labs to exist, I use them to
organize what I am learning, and writing a lab forces me to think about a
system more carefully than reading about one ever did.

So treat this as a working draft by one person with a tool, not as a textbook.
Where a claim rests on a source I fetched the page and read it. Where it rests
on judgement, the judgement is mine and you may disagree with it. It gets
better as I go, and the parts I have argued with hardest are the ones I trust.

Doing a lab is the opposite case. Use a model to research and to scaffold, and
write the design and the code yourself, because a model will hand you a working
slice in ten minutes and the ten minutes are what you came for.
[`HOWTO.md`](HOWTO.md#using-a-model) says where I think the line sits.

## What a lab asks of you

A lab is not an exercise with an answer. The brief fixes the product, its
public behaviour, the environment, the limits, the failures it must survive,
and the evidence it must produce.

It does not give you the service decomposition, the schema, the retry
algorithm, the cache policy, or the recovery mechanism. Those are yours, and
getting them wrong is the teaching. Your first design is supposed to fail. The
redesign after it fails is the lesson.

Every lab passes three gates:

| gate | duration | you deliver |
|------|----------|-------------|
| product | 60–120 min | an end-to-end slice that accepts input, persists or transforms it, exposes a useful result, and survives restart |
| failure | 2–4 h | a deterministic scenario that falsifies your first design and forces the stated invariant to hold |
| evidence | 1–2 h | a load or deployment run recording the operational tradeoff and one remaining limitation |

A single lab runs six to twenty-five focused hours, and the seventeen core
labs come to 203 hours at the low end of their stated budgets and 294 at the
high end.

Those are estimates and nothing behind them was measured, because no lab has
been built or run yet. Read them as the size I am aiming at. Two of them I
already distrust: the product gate's hour assumes you know the dependency you
are wiring up, which is exactly what a first lab cannot assume, and the three
gate durations add to four hours at best and eight at worst, which leaves the
six-hour labs no room for the redesign that is supposed to sit between the
gates.

## Where the difficulty comes from

From the quirk of the system you are studying. A notification that never
replays. A lease that is not a deadline. An index that is not consistent yet.
Never from input formats, parsing chores, or boilerplate volume, because a lab
that is merely laborious has failed.

Every quirk is grounded in a public document describing real reported
behaviour: a manual page, a post-mortem, a paper, a vendor limit. That
citation is the lab's ground truth, and I fetch the page before I cite it.

## Keeping the answer away from yourself

If you want the exercise, you have to be able to avoid the answer without
effort. That is why each lab is four artifacts and not four sections of one.

| file | holds | you open it |
|------|-------|-------------|
| `README.md` | the task and the technology landscape | by default |
| `hints/` | the design reading, one file per hint | by choice, when stuck |
| `EVALUATION.md` | the answer key | by choice, when checking |
| `ARCHITECTURE.md` | your own reasoning | you write it |

No lab has a worked solution. A systems lab admits many correct designs, so no
implementation is canonical. `EVALUATION.md` says what a strong design holds
and how to check it. It does not implement one. There is no grader binary and
no score.

The failure schedule is never a readable file. Its recipes compile into the
shared fault controller, and `make fault` materializes a schedule only while a
run is in flight. The standard is deterrence, not impossibility: you own the
machine, and reading it is a deliberate act, like opening `hints/`.

## Where to start

[`HOWTO.md`](HOWTO.md) is the method. What you do in a lab, in what order,
what you write down, and how you know you are finished. Read it first.

There is no learner tree yet, so the only thing to read after it is the
catalog.

[`labs/README.md`](labs/README.md) lists the seventeen core labs in course
order, and lab 01 is the intended entry point. Read the rows to see what each
lab asks, then open that lab's `README.md`, which is the task and holds no
answer.

Two files in every lab directory do hold answers, and both say so on their
first line: `hints/` and `EVALUATION.md`. The schedule in `EVALUATION.md` names
the exact records at which the lab breaks your design, so opening it before you
have a design spends the lab.

Three things are worth doing while no lab runs, and none of them spoils
anything. Check that you meet [what you will need](#what-you-will-need). Read
one phase's rows in the catalog and pick the product you want to build first.
Read [`docs/cloud-access.md`](labs/cloud-access.md) if you expect to take the
optional cloud path. You are ready when you can start a container on your own
machine and name the lab you mean to do first.

Start with phase 1. The later phases assume it: three of the five phase 2 labs
recast a phase 1 product on an execution model you cannot operate, and each
one needs your phase 1 design and evidence to compare against. The other two
stand alone.

## Phases

The digit directory under `labs/` is the curriculum phase, not a version.

- **1** local runtime and delivery (7 labs)
- **2** the same problems on an execution model you cannot operate (5)
- **3** real Internet streaming (2)
- **4** NoSQL, analytics, portability (4)
- **6** low-level, in Rust and C (5)
- **7** blockchain, on Solana and Ethereum (6)
- **8** search, retrieval, spatial (5)

Phase 5 no longer exists. Its lab became `4/5`, and the gap stays open on
purpose.

## What you will need

Before the first lab you can write and run a program of a few hundred lines,
use a terminal, start a container, and read a vendor's own documentation
rather than a tutorial about it. You do not need to have operated a database,
a message broker, or a cluster. That is what the labs teach. A lab asks you to
design a system, so it assumes you can already build one that works.

You need a machine that runs Linux containers. Each lab states a scale target,
a speed and a load and an amount, and those numbers decide how much machine a
lab wants. The exact figures land with the scaffold.

Every required gate runs locally and no lab needs a cloud account to pass.
Labs name heavyweight dependencies, among them PostgreSQL, Kafka, DynamoDB
Local, ClickHouse, Valkey, and Flink, and each one is driven hard enough that
its quirk actually fires.

Go is the starter language unless the environment dictates otherwise: Rust and
C in phase 6, the chain's own language for an on-chain program, TypeScript for
a browser bundle.

[`docs/cloud-access.md`](labs/cloud-access.md) covers the optional cloud path.
Phases 2 and 4 are the only ones that can bill, and only there.

## Status

`docs/contract.md` is `draft`, and that blocks implementation.
Creating a lab directory, pinning dependency versions, and authoring the first
lab all wait on it moving to `accepted`. Every other spec reads `draft` too.

Open defects go in [`BUGS.md`](BUGS.md) rather than getting fixed silently.
The queue is empty today. The findings that stood in it came down to one
defect: four labs asked a local emulator for the limit their lesson lives in,
and an emulator reproduces an API and leaves its limits out. The fault
controller supplies the limit now, and every lab whose gate depends on one
says which layer supplies it.

## Contributing

[`CONTRIBUTING.md`](CONTRIBUTING.md) holds the authoring side: the teaching
contract every lab obeys, how to add a lab, the review queue, and the records
behind which labs exist and why.

## Licence

GPL-3.0. The full text is in [`LICENSE`](LICENSE).

All lab prose, fixtures, and workload generators are original work. External
sources are cited and never copied. Dependencies keep their own licences, and
their notices travel with distributed copies.
