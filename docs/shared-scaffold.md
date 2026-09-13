---
status: draft
---

# Shared scaffold

This file is author-facing. Like everything under `labs/`, it is not part of
the learner tree; any learner-facing orientation is derived from it later.

## Decision

Thirty-four labs cannot each own a generator, a fault controller,
and an evidence format. They share one implementation of each, and a lab
contributes only its own declarative files. This document specifies those
shared components and the boundary between shared and per-lab.

The scaffold is the first thing built once `01-systems-labs.md` is `accepted`.
Nothing else can be authored against a contract that does not exist: a lab
written before the five invariant shapes are fixed will encode different words.

## Boundary

Shared code holds everything that is identical across labs. A lab holds
everything that names its own domain.

| Concern | Shared | Per lab |
|---------|--------|---------|
| Verification | the history vocabulary and its five invariant shapes | its `EVALUATION.md`, and the boundaries it checks |
| Workload | the open-loop generator, the seed and rate model, the replay engine | the record schema and the generator parameters |
| Faults | the controller with the recipe set compiled in, every injection mechanism, the digest | its seeded schedule recipes and their barriers |
| Evidence | the run manifest, the history format, the histogram format, the report schema | which measurements the lab requires |
| Environment | the Compose profile shape, health checks, network layout | which dependencies the profile starts |

No shared component may contain anything solution-bearing. A helper that
batches writes, coordinates a cache fill, or reconciles an offset with a
transaction has answered a lab's question inside the scaffold, and every lab
that imports it inherits the answer.

## Verification, described not built

There is no shared grader. Each lab carries `EVALUATION.md`, the answer key
whoever checks the work reads, as fixed by the
[verification section](contract.md#verification).

What the scaffold owes every lab here is **vocabulary**, so thirty-four answer
keys describe their checks the same way rather than each inventing a phrasing.
Five recurring shapes cover most lab invariants, and a key names the one it
means:

- **once** — an identity appears exactly once at the effect boundary the lab
  names as single-effect;
- **order** — identities appear in the order the lab declares, within the scope
  the lab declares that order to hold;
- **survives** — every acknowledged identity is present after a recovery point;
- **never** — a forbidden state does not occur;
- **bounded** — a quantity stays inside a declared bound for the whole run.

Each shape ranges over what that lab's own observable boundaries expose — an
HTTP response, SQL state, a provider's request log, a consumer position, the
evidence report. The lab names the boundary; the shape names what must hold
there. Nothing is compiled, no lab imports anything, and there is no event
schema: a typed multi-emitter log with clock domains and a merge rule is
machinery for a checker this curriculum does not build.

Two consequences follow, and both are stated rather than hidden. Where a check
depends on ordering across processes, the lab says which ordering it means and
over what scope, because no wall clock or monotonic clock is comparable across
the fault controller, the application, and a restart — only the lab knows which
order is the one its invariant needs. And where a lab's contract is not one of
the five — `2/1` compares invoice lines against an independently computed
answer, `6/5` a latency distribution, `7/2` a balance replay, `8/5` recomputed
scores — the answer key carries that computation directly. The five are a
convenience for the common cases, never a ceiling on what a lab may require.

## `shared/workload/`

One generator, parameterized per lab. It is Go, because it must hold an offered
rate while the system under test is failing to keep up.

It is **open-loop**: the schedule is computed in advance from the seed and the
rate, and a slow system produces a growing backlog rather than a reduced
offered rate. A generator that waits for a response before sending the next
request measures the system's own pace and reports it as capacity, which is
coordinated omission and the reason the rate-accurate replayer exists as a lab
in its own right.

Parameters come from the lab's TOML: seed, rate, burst shape, key skew,
duplicate ratio, late-arrival distribution, malformed-record ratio, disconnect
boundary, and cursor overlap. The same seed produces the same identities in the
same order on every host, because verification asserts histories against them.

The replay engine handles cached recordings under the same interface: preserve
original timing, scale time, or drive an open-loop rate. Recording is a
separate command from replay, and no application request path ever reaches a
provider.

## `shared/faults/`

One controller, driven by each lab's fault schedules, implementing the
mechanisms fixed by the
[fault injection contract](contract.md#fault-injection-contract):
process lifecycle including freeze and thaw, network shaping and asymmetric
partitions and a fault proxy, each dependency's own control surface, clock
skew, and the fault block device.

The contract states two things the controller's shape follows from: a barrier
holds execution rather than merely being noticed, and the controller supplies
the limit the environment omits, at the transport, process or clock layer. The
transport layer is the fault proxy; the process layer is freeze and thaw, held
apart from `SIGKILL`; the clock layer is the seeded time source the run
controls. Every lab names the layer its required gate depends on, so a gate
that depends on a limit no layer supplies is a gate that proves nothing.

A scenario is declarative and names a trigger, a target, and an effect. The
trigger is a **barrier**, not a time: an identity reaching a boundary. The
controller and the evidence writer therefore share the barrier vocabulary,
which is what lets a failure land at the same point on every run and lets the
history record where it landed.

`README.md` does not share it. The task is learner-facing text, and the
[teaching contract](contract.md#teaching-contract) forbids it from
naming the failure schedule. It names the boundaries the product exposes, and
the barrier is read out of the history at run time; the teaching lint fails on a
barrier name written into the file.

A schedule is never a readable artifact at rest, because a barrier name is an
edge case stated in words. Each lab's seeded recipes are compiled into the
controller binary — Go, per the language policy — rather than shipped as
readable files beside it, so what the learner receives is an executable that
produces the schedule, not a source that describes it. The recipe sources
live with the author tree, excluded from the learner distribution by the same
publish step that excludes `labs/`. Excluded is not
withheld: the controller is conveyed as object code under GPL-3.0, so the
learner distribution names the public source repository as the no-charge
route to its complete corresponding source, recipe sources included, per the
master spec's
[licence contract](contract.md#licence-and-corresponding-source).
`make fault`
— exactly as available to the learner as every other target — materializes
the schedule it runs, runs it, and removes it, so the readable form exists
only while the run is in flight. This is deterrence rather than
impossibility — a learner who disassembles the controller or fetches the
recipe sources from the source repository has made a
deliberate choice — and no readable schedule ships in the learner tree. A frozen
aggregate digest over every schedule the controller materializes is checked
under `make test-all`, so a recipe cannot drift without the change being
declared; the digest check adds no Make target of its own.

The block-device layer needs a privileged container. Its feasibility in the
target environment is unproven, and the low-level track depends on it. Proving
or replacing it is a prerequisite for phase 6, not a detail.

## `shared/telemetry/`

Available to every lab and required by few. It provides one histogram format
and one trace contract so evidence reports are comparable across labs.

Where a lab does not require instrumentation, this component still supplies the
evidence report writer, because the report schema is shared even when the
measurement method is the learner's choice.

## Teaching lint

`make teaching-lint` is the mechanical enforcement of the
[teaching contract](contract.md#teaching-contract), and CI runs it.
What the lint rejects is stated once, in the
[verification contract](contract.md#verification-contract).

The check is cheap by construction: the barrier vocabulary is shared, each
lab's neighbour boundary-difference sentences stay in its own spec, every `Code pointers`
bullet carries an explicit neutral or solution-bearing state, each lab's
dependency set bounds the parameter vocabulary to scan for, and the
mechanism vocabulary is one curriculum-wide list maintained with the lint,
so the check stays a scan rather than a judgment. It is
repository tooling, so it is Python per the language policy.

## `template/`

A complete lab directory that runs before any learner code exists. It is the
executable form of the
[per-lab directory shape](contract.md#repository-contract), with a
trivial domain: one record type, one operation, one invariant, one fault.

The template carries a working implementation of its trivial domain, and it
is the only one in the repository. It is the scaffold's regression test rather
than a lab: CI proves `make up`, `make test-all`, `make fault`, and `make bench`
all run end to end against it, so a broken generator, controller, or evidence
writer fails before any lab does.

No lab has an equivalent. A challenge has one correct answer, so a worked
reference is well defined; a systems lab admits many correct designs, so no
implementation is canonical and a "rotten" one is merely one of countless ways
to be wrong. What anchors a lab instead is the cited source that documents the
real reported behaviour its quirk rests on. A new lab starts as a copy of the
template with the implementation removed.

## Build order

1. The five invariant shapes, written down before anything else, because every
   lab's answer key uses their words. This is a page of prose, not a component,
   and it ships no directory.
2. The template, driven by hand, proving the directory shape and the Make
   targets against a trivial domain.
3. The workload generator, validated by the rate-accuracy property that phase 6
   later teaches.
4. The fault controller, mechanism by mechanism, cheapest first. The block
   device is proven or replaced here.
5. The evidence report writer and the shared telemetry contract.

Lab 01 is authored against the finished scaffold, not alongside it.

## Open questions

- Whether the fault block device can run in the target container environment,
  and what the low-level track does if it cannot.
- Whether the process freeze can land exactly at the barrier phase 2 needs —
  the point where the runtime and every extension have completed and no
  events are pending, which a returned response alone does not mark — on the
  Lambda-compatible runner.
  Every phase 2 failure schedule except the import lab's depends on that
  barrier; the proposal is `BUGS.md` S1 and needs sign-off.
- Whether the five invariant shapes survive contact with phases 7 and 8, whose
  identities are chain transactions and documents rather than records.
- Closed 2026-08-30: `template/` carries Go only, because the course now
  supplies one starter per lab and a second starter would double the surface
  the scaffold must keep passing.

## Governing references

- [`../01-systems-labs.md`](contract.md) — the contracts this
  scaffold implements.
- [`lab-selection.md`](lab-selection.md) — the scored selection.
- [`../labs/README.md`](../labs/README.md) — authoritative list and lifecycle status.
