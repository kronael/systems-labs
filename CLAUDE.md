# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# systems-labs

Curriculum of end-to-end systems labs. Each lab prompts a complete, useful
product that a learner designs, breaks, recovers, and defends with evidence.
31 labs across seven phases; sixteen are the core.

**Today the repository holds specifications only** — no code, no `Makefile`, no
lab directory. Every target below is specified, not implemented. Do not report a
build or test result; there is nothing to run.

`specs/01-systems-labs.md` is the governing spec and the single source for every
cross-lab rule. This file is its operational summary; where the two disagree,
the spec wins and this file is wrong.

## Teaching mode — no free solutions

When a learner works a lab here, act as a teacher, not an answer key. NEVER
state the design, name the technique, list the edge cases, or reproduce a hint
from `HINTS.md` unless the learner has explicitly asked for a hint or the
solution **twice** — two separate, explicit requests, not implied by frustration
or a vague "I'm stuck". Before that threshold: ask guiding questions, name a
concept worth reviewing, or explain why a proposed approach fails. Never write
the solving code.

## No worked solutions — the source is the ground truth

**No lab has a golden or rotten implementation.** Not in the lab directory, not
at repository root, not anywhere. A challenge has one correct answer, so a
worked reference is well defined. A systems lab admits many correct designs, so
no implementation is canonical, and a "rotten" one is only one of countless ways
to be wrong. Authoring a worked design would also answer every
`Architecture questions` bullet at once, in a document whose only protection is
that nobody reads it.

Two things stand in its place:

- **The cited source.** Every lab names the public document describing the real
  reported behaviour its quirk rests on — a manual page, a post-mortem, a paper,
  a vendor limit. That citation is the lab's ground truth. Fetch the page and
  confirm it says what you claim; a URL recalled from memory is not a citation.
  NEVER invent a quirk and then look for a source.
- **A described check.** `.claude/skills/verify/SKILL.md` states what must hold
  and where to look, for an agent to run. No grader binary, no framework, no
  fixtures proving the checker. A lab that needs elaborate checking machinery to
  be interesting has the wrong task.

Verification needs no oracle: it asserts invariants over observed histories
rather than comparing output to a reference run.

`template/` is the one exception, and it is not a lab. It carries a working
implementation of a trivial domain — one record type, one operation, one
invariant, one fault — as the scaffold's regression test, so a broken generator,
controller, or checker fails before any lab does.

## Never leak the failure schedule

The failure schedule IS the lab. A learner who reads it before designing has
been handed the design.

- Fault schedules are NEVER readable files in the learner distribution. Their
  seeded recipes are compiled into the shared Go fault controller.
- `make fault` materializes the schedule it runs, runs it, and removes it, so a
  readable form exists only while a run is in flight.
- A frozen aggregate digest over every materialized schedule is checked under
  `make test-all`, so a recipe cannot drift silently.
- The spec's `Adversarial evaluation` section NEVER propagates into `README.md`.

The standard is deterrence, not impossibility. The learner owns the machine and
can disassemble the controller or fetch the recipe sources from the public
source repository — each a deliberate act, like opening `HINTS.md`.

## README vs HINTS — never spoil the lab

- **`README.md`** states the task, invariants, limits, interfaces, scale target,
  and acceptance evidence. It carries nothing else. It must not name, describe,
  compare, or rule out any solution method. The ban imports the `challenges/`
  categories verbatim: techniques, data structures, memory orderings,
  recurrences, invariants, implementation shapes, failed strategies, naive or
  brute-force approaches, complexity analysis of candidate approaches, and
  phrases such as "the trick", "the trap", "the catch", or "the hard part".
  State the input limits without explaining how an approach behaves at them.
  The banned invariants are a design's internal ones; the product invariants the
  task must hold are the task itself.
- **Names are part of the prompt.** Lab titles, directory slugs, catalog rows,
  and source labels are solution-neutral too.
- **`HINTS.md`** holds architecture guidance, rejected designs, and
  solution-bearing citations. Opened by choice, never by default. Nothing in
  `README.md` summarizes it.
- Fixed wording, never paraphrased: every `HINTS.md` opens with the exact line
  `> Spoilers. Open only when stuck.` and every `README.md` ends with the exact
  line ``Stuck? See `HINTS.md`.``

`make teaching-lint` enforces this and runs in CI: it fails on a mechanism name
— prescribed, disclaimed, or merely mentioned — a barrier name, a neighbour
product name, a dependency configuration-parameter name, or a solution-bearing
citation in any lab `README.md`.

## The brief — what, never how

A brief reads "design and build a system that does X under these requirements."
It fixes the product, public behavior, environment, limits, failures, and
evidence. It NEVER gives the service decomposition, schema, event keys,
transaction boundaries, retry algorithm, cache policy, recovery mechanism, or
deployment topology.

**A disclaimed mechanism is still a named mechanism.** Writing "does not
prescribe a worker pool" tells the learner a worker pool is in play. A brief
states the **categories** of decision the learner owns — the concurrency model,
the data layout, the delivery and recovery path, the process decomposition, the
coordination between components — and names no candidate under any of them, not
even to exclude it.

A brief describes exactly one system and never splits it into named services. A
second dependency belongs to the fixed environment, not to a second product.

## The three gates

Every lab passes all three. They time the passes; the lab's budget is larger
than their sum, because the redesign between them is the teaching.

| gate | duration | delivers |
|------|----------|----------|
| product | 60–120 min | an end-to-end slice that accepts input, persists or transforms it, exposes a useful result, and survives restart |
| failure | 2–4 h | a deterministic scenario that falsifies the naive design and forces the stated invariant to hold |
| evidence | 1–2 h | a load or deployment run recording the operational tradeoff and one remaining limitation |

Individual labs run six to twenty-five focused hours. The sixteen core labs run
to roughly 150 to 250 focused hours.

## Difficulty, scale, and the earned dependency

**The task carries the lab.** Judge a brief by whether it sends the learner to
the primary documentation, to a post-mortem, to their own experiment on the
running system — and by whether the first design they commit to teaches them
something when it fails. A lab that needs elaborate checking machinery to be
interesting has the wrong task. Spend the effort there.

Difficulty comes from the quirk of the system under study — a notification that
never replays, a lease that is not a deadline, an index that is not yet
consistent. NEVER make a lab hard through input formats, parsing chores, or
boilerplate volume. A lab that is merely laborious has failed, and so has one
whose product path is obvious once the environment is running.

Every lab states a **scale target**: a speed, a load, and an amount. Those three
numbers size the problem so a toy design fails on its own terms. They are NOT
pass thresholds — thresholds stay relative, calibrated, structural, or
learner-declared, because absolute latency numbers flake across machines.

**A heavyweight dependency must be earned.** The lab must drive it into the
regime where its quirk actually fires and show that firing in the evidence. If
the naive small tool would pass the same gates at the same scale, the lab has
not earned its dependency and the scale target is too low.

## Faults fire at named barriers

A scenario says "when record 4711 is acknowledged, freeze the broker" — never on
a timer, never at random — so the failure lands at the same boundary on every
run and verification asserts an exact history. Random chaos proves nothing twice.
Every scenario declares what must remain true after recovery; a fault the system
survives with nothing to check is entertainment, not a gate.

## Per-lab directory shape

```text
NN-solution-neutral-name/
  README.md         task, no solution
  ARCHITECTURE.md   learner-owned
  HINTS.md          architecture reading, opened by choice
  lab.toml
  compose.yml       learner-owned application topology
  app/  tests/      learner-owned
  starter/          generated clients, contracts, empty seams, no product path
  .claude/skills/verify/SKILL.md   how an agent checks this lab
  sources/          provider adapters and provenance manifests
  workload/         seeded generators and bounded cached replay
  infra/compose/dependencies.yml   fixed external systems only
  infra/kubernetes/sandbox/  infra/opentofu/sandbox/
  evidence/         generated, gitignored
```

The verify skill names only public boundaries. `starter/` contains no end-to-end
path.

## Verification contract

Each lab exposes this root vocabulary and no other:

```
make up         start the environment (make down stops it)
make            format, build, lint, fast test
make test       unit and contract tests, under five seconds
make test-all   local integration suite, what CI runs
make fault      deterministic failure and recovery scenarios
make bench      seeded load, invariant check, evidence output
make teaching-lint   fail on any solution leak in a lab README; CI runs it
make source     record bounded real data
make smoke      live cloud check, the only target that leaves the workstation
make clean      remove generated artifacts, keep cached source data
```

Correctness gates assert exact record identities and histories, never counts
alone. Performance gates NEVER hard-code a number.

**There is no grader binary.** Each lab carries
`.claude/skills/verify/SKILL.md` — the procedure an agent follows to check that
lab against a completed run. It names the observable boundaries, the invariant
each must satisfy, and the identities to check, and it describes only what is
not obvious. It never names the failure schedule, a mechanism, or a design.

Only two things stay compiled, because neither can be described away: the fault
controller, which must fire at exact barriers, and the workload generator, which
must hold an offered rate under load. The run stays reproducible; the judgement
over it does not, and a lab whose correctness cannot be stated clearly enough
for that to be reliable has an unclear invariant.

## Languages

- **Python** — tooling: fixtures, TOML, evidence reports, the grading runner,
  repository automation. `uv` projects and PEP 723 scripts.
- **Go** — anything that must keep time under load: the open-loop generator,
  fault controller, provider simulators. NEVER move these to
  Python; a generator that slows with the system under test destroys the
  measurement the labs teach.
- **Go and TypeScript** — learner starters for the core phases; **Rust and C**
  for phases 6–8.
- **HCL** — infrastructure, OpenTofu. Pulumi and CDK are out, because a
  general-purpose language computes resources at run time instead of declaring
  them, and drift needs a plan that states every change in advance.
- SQL and PL/pgSQL stay in the database boundary; Java is confined to the Flink job.

## Config, data, and cost

First CLI argument is a TOML lab config; optional second is a secrets file.
Environment variables carry injected secrets only.

Generated seeded input is the default and the only CI input. Real RIPE RIS Live
and Kraken recordings are opt-in, bounded, checksummed, cached under
`${PREFIX:-/srv}/data/systems-labs/sources/`, and never fetched on a request
path. Decimals stay decimal.

Every required gate runs locally with no AWS account. Optional cloud use is
Lambda, SQS, DynamoDB on-demand, and short logs only. MSK, EKS, NAT gateways,
RDS, and ElastiCache are excluded — each bills continuously.

## Adding a lab

1. Find the real reported behavior first — a manual page, a post-mortem, a
   paper, a vendor limit — and record it in the track record's origination
   column. NEVER invent a puzzle and then look for a source. Fetch the page and
   confirm it says what you claim; a URL recalled from memory is not a citation.
2. Write the spec under `specs/<phase>/`, using the nine sections in order and
   no others: `Brief` · `Prepared scaffold` · `Requirements` ·
   `Architecture questions` · `Adversarial evaluation` · `Acceptance evidence` ·
   `Neighbouring systems` · `Scope` · `Code pointers`. Frontmatter carries one
   key, `status:`.
3. Name the scale target — speed, load, amount — and check the earned-dependency
   rule: state what the naive small tool would fail at this scale. If nothing,
   raise the scale target or drop the dependency.
4. Split `Architecture questions`. A question stated at the level of the
   property the design must defend publishes into `README.md`. A question that
   presupposes a mechanism is solution-bearing: mark it `HINTS.md`-bound, or
   rewrite it until it names only the property.
5. Mark each `Code pointers` citation neutral or solution-bearing. Solution-
   bearing ones land in `HINTS.md` and never in `README.md`.
6. Copy `template/` to the lab directory and remove its implementation. Write
   `.claude/skills/verify/SKILL.md` — only what is not obvious. NEVER write a
   worked solution.
7. Add the fault schedule recipe to the controller, update the frozen aggregate
   digest, and confirm `make fault` materializes and removes it.
8. Run `make teaching-lint`. Add a row to the core catalog in `specs/index.md`.

## Naming rule — read before editing any link

Lab files carry **solution-neutral product names**. The file name states what
the product does, never the technology that solves it. `1/2` is
`reservation-fulfillment`, not `postgres-durable-jobs`.

`specs/index.md` carries the product names in both its core catalog and its full
specification list. Before editing a link, read the real file name from disk
rather than copying one from a table.

## Phases

The digit directory under `specs/` is the curriculum **phase**, not a version:
1 local runtime and delivery; 2 the same problems serverless; 3 real Internet
streaming; 4 NoSQL, analytics, portability; 6 low-level; 7 blockchain;
8 search, retrieval, spatial. **Phase 5 no longer exists** — its lab became
`4/5`, and the gap stays open on purpose. Do not renumber to close it.

**Phase 1 runs software the learner operates; phase 2 runs an execution model
the learner cannot.** That is why phase 2 exists, and it is the one place a
product may be repeated. A port across languages stays forbidden — it inherits
the original's checks, failure schedule, and answer. A port across execution
models confiscates the answer.

Phases 6, 7, and 8 are **separate catalogs**, not ports. Each track's labs must
expose a failure the others cannot reach; see the track files under `specs/0/`.

The core catalog in `specs/index.md` holds each lab's system, architecture
pressure, and prepared environment. Read it there; do not restate it elsewhere,
because a second copy drifts. `01-systems-labs.md` specifies the shared
contracts and carries no per-lab detail except the source map that indexes its
own research ledger. `specs/0/` holds the selection record and the four track
catalogs; `specs/<phase>/README.md` orients a phase; `BUGS.md` is the review
queue; `.diary/` is the shipping log.

`specs/` is author-facing and is NOT part of the learner tree. The reasoning
about what is optimal and why belongs there.

## Approval boundary

`01-systems-labs.md` blocks implementation while it is `draft`. Creating a lab
directory, pinning dependency versions, or authoring lab 01 needs the user to
move that spec to `accepted` first. Editing and expanding the specs is open work.

## State of the repo

- Phases 1 and 2 — ten specs, reviewed and released as `v0.1.0`.
- Phases 3, 4, 6, 7, 8 — 21 specs, drafted; their briefs still carry the
  disclaimed-mechanism enumerations this file now forbids (`S14` in `BUGS.md`)
  and gate-sum hour budgets (`S11`).
- No code exists. The shared scaffold is specified in `specs/0/5-shared-scaffold.md`.

# Project Memory

- A disclaimed mechanism is still a named mechanism. This was the oldest and
  most pervasive leak in the repository, and this file used to require it.
- Emulators reproduce a hosted API and omit its limits, so a lab whose lesson
  lives in a limit needs the fault controller to supply the limit.
- Within a category the course picks the heavier tool, and the lab must then
  drive it into the regime where its quirk fires. A heavyweight dependency the
  scale target never stresses teaches configuration and nothing else.
