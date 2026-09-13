# How to work a lab

This is the method. [`README.md`](README.md) says what the curriculum is, and
[`CONTRIBUTING.md`](CONTRIBUTING.md) says how a lab is authored. This file says
what you do, in what order, and how you know you are finished.

Nothing runs yet, so read this as the shape of the work you are preparing for.
[Status](README.md#status) says where the repository actually is.

## What a lab gives you, and what it keeps

A lab is a brief: *design and build a system that does X under these
requirements.* The brief fixes the product, its public behaviour, the
environment, the limits, the failures it must survive, and the evidence it must
produce.

It does not give you the service decomposition, the schema, the retry
algorithm, the cache policy, the recovery mechanism, or the deployment shape.
Those are the exercise. Your first design is meant to fail at a boundary you
did not think about, and the redesign after it fails is the lesson.

You get four artifacts, and you choose which of them you open.

| file | holds | you open it |
|------|-------|-------------|
| `README.md` | the task and the technology landscape | by default |
| `hints/` | the design reading, one file per hint | by choice, when stuck |
| `EVALUATION.md` | the failure schedule and what it must leave true | by choice, when checking |
| `ARCHITECTURE.md` | your own reasoning | you write it |

Open `hints/` and `EVALUATION.md` whenever you decide to. They are not
locked, and nothing marks you down for reading them. They are separate
artifacts so that avoiding the answer costs you nothing.

## The loop

Every lab runs the same seven steps.

1. **Predict.** Before you build, write down how you expect the named
   technology to behave at the boundary case the brief points at. Name the
   state that would prove you right. Then list the ways your design can fail,
   all of them, not the one you find first. Beside each, write what you would
   show someone to prove it happened: which record, which stored state, which
   number moved. A failure mode you cannot demonstrate is a guess, and you
   will not notice when it is the one that bites you.
2. **Build.** Make one thin path that is complete end to end: input arrives,
   state changes, something useful comes out, and it survives a restart.
3. **Test.** Test that path across the real public boundary with the real
   dependencies. Do not test private functions.
4. **Break.** Run the failure scenario. It fires at a named record, not on a
   timer, so it lands at the same place every run. Then play with it. You
   cannot move the barrier, because the schedule is compiled, but you can
   change your own side between runs: weaken a limit, widen a timeout, take
   out the guard you just added, run the load target underneath it. The
   barrier stays put and the outcome moves, which is how you learn what your
   design was actually doing. One run tells you the gate passed. Several tell
   you how the failure manifests.
5. **Inspect.** Read the evidence the dependency itself keeps, not the logs
   your own code wrote: its stored state, its record of what it has handed
   out, and its own measurement of what it did.
6. **Correct.** Change the design. Do not hide the technology behind a
   different one, and do not remove the pressure the lab applies.
7. **Prove.** Show the invariant holds after recovery, and say what the
   platform still cannot do.

## The three gates

| gate | time | you deliver |
|------|------|-------------|
| product | 60–120 min | the end-to-end slice from step 2, surviving restart |
| failure | 2–4 h | the deterministic scenario, with the stated invariant holding |
| evidence | 1–2 h | a load or deployment run recording one tradeoff and one remaining limitation |

A lab takes six to twenty-five focused hours. The gates add up to between four
and eight, and the rest is the redesign between them, which is not overhead but
the lab itself. Treat all of these numbers as estimates: nothing has been built
yet, so nothing has been measured.

## What you write down

You write `ARCHITECTURE.md`. Nobody hands it to you and nobody reads it to you.

Write the first version before you build: the decisions you own, what you chose
for each, and the prediction from step 1. Write the second version after the
failure gate: what the scenario falsified, what you changed, and why the new
design holds. Keep both. The distance between them is the evidence that you
learned something.

## Working the commands

Each lab exposes the same small vocabulary, and no other. The verification
contract in [`docs/contract.md`](docs/contract.md) is where
this list is defined; that file is the governing spec and carries no lab's
failure schedule.

```
make up         start the environment (make down stops it)
make            format, build, lint, fast test
make test       unit and contract tests, under five seconds
make test-all   the local integration suite, what CI runs
make fault      the deterministic failure and recovery scenarios
make bench      seeded load, the evidence report, the observed history
make source     record bounded real data
make smoke      the live cloud check, the only target that needs an account
make clean      remove generated artifacts, keep cached source data
```

`make fault` writes the failure schedule out, runs it, and deletes it, so the
schedule exists in readable form only while a run is in flight. You own the
machine and you can go and read it. Doing so hands you the design, the same way
opening `hints/` does, so decide deliberately rather than by accident.

## When you are stuck

In this order:

1. Re-read the brief. Most stuck designs are answering a requirement the brief
   did not make.
2. Read the primary documentation of the dependency, not a tutorial about it.
   The behaviour that breaks your design is usually stated plainly on the
   vendor's own page.
3. Run a small experiment against the running system and look at the native
   state. The lab is built around real reported behaviour, so the system will
   tell you.
4. Open `hints/`. It holds the architecture reading, what the neighbouring
   systems do differently at this lab's boundary, and the designs that were
   rejected.

## Using a model

Use one for research and for scaffolding. Do not use one for the design or for
the code.

Research is where it pays. Ask which vendor document describes a behaviour,
then open that document and read the sentence yourself. Ask for a summary of a
post-mortem you are about to read. Ask what a dependency's failure modes are
called, so that you know what to search for. Check every answer against the
primary source, because a confident wrong answer costs you a whole redesign.

Scaffolding is the other safe use: a Compose file, a client stub, a test
harness, the boring shape around the work. Difficulty here never comes from
boilerplate, so handing the boilerplate away costs you nothing.

The design and the code are the exercise. A model will hand you a working
slice in ten minutes and you will have skipped the part that teaches. The
design that fails at the failure gate has to be yours, or nothing is falsified
when it breaks. `ARCHITECTURE.md` is the same: you cannot defend a decision you
did not make, and the gate asks you to defend it.

Nobody enforces this and nobody is checking. It is your time.

## When you are finished

All three gates pass, and:

- the invariant the brief states holds after the failure and the recovery;
- `ARCHITECTURE.md` carries the prediction, what falsified it, and the
  redesign;
- the evidence run records one operational tradeoff and one limitation that
  remains;
- you can say what the platform cannot do, and why that is the platform's
  property and not your design's.

Then open `EVALUATION.md` and check yourself against it. It states what a
strong design holds and what a weak one gets wrong. There is no grader binary
and no score, because a systems lab admits many correct designs. Nobody is
keeping count except you.

## Choosing your first lab

Start at phase 1 and take lab 01. The later phases assume it: three of the five
phase 2 labs recast a phase 1 product onto an execution model you cannot
operate, and each one needs your phase 1 design and evidence to compare
against.

The catalog is in [`labs/README.md`](labs/README.md). Read the rows. Warning:
the specifications those rows link to are author material, and each one carries
the failure schedule, so opening a specification spends that lab for you.
