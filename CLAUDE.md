# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# systems-labs

Curriculum of end-to-end systems labs. Each lab prompts a complete, useful
product that a learner designs, breaks, recovers, and defends with evidence.
34 labs across seven phases; seventeen are the core.

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
- **`EVALUATION.md`.** The answer key: what a strong solution holds, how to
  check it, and any independently computed result a check needs. Opened by
  choice, like `HINTS.md`. No grader binary, no framework. A lab that needs
  elaborate checking machinery to be interesting has the wrong task.

Verification needs no oracle: it asserts invariants over observed histories
rather than comparing output to a reference run.

`template/` is the one exception, and it is not a lab. It carries a working
implementation of a trivial domain — one record type, one operation, one
invariant, one fault — as the scaffold's regression test, so a broken generator,
controller, or evidence writer fails before any lab does.

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

## Four files, four audiences — never spoil the lab

One rule governs everything below: **a learner who wants the exercise must be
able to avoid the answer without effort.** That is why these are separate files
and never sections of one. Anything solution-bearing that lands in `README.md`
cannot be unseen.

| file | holds | opened |
|------|-------|--------|
| `README.md` | the task and the landscape | by default |
| `HINTS.md` | the design reading | by choice, when stuck |
| `EVALUATION.md` | the answer key | by choice, when checking |
| `ARCHITECTURE.md` | the learner's own reasoning | written, not read |

**`README.md` carries the requirements and the technology landscape, and that
is all.** The requirements: what the product does, the invariants that must
hold, the limits, the public interface, the scale target, and the acceptance
evidence. The landscape: the names of the two or three technologies a
practitioner would have reached for instead, with links to their documentation.

Naming a neighbour orients a learner who would otherwise not know the
alternatives exist, and a name alone solves nothing. Saying what that neighbour
does *differently at this lab's boundary* is the comparison, and it points
straight at the quirk — that sentence belongs in `HINTS.md`.

Beyond the task and the landscape, `README.md` must not name, describe,
compare, or rule out any solution method. The ban imports the `challenges/`
categories verbatim: techniques, data structures, memory orderings, recurrences,
invariants, implementation shapes, failed strategies, naive or brute-force
approaches, complexity analysis of candidate approaches, and phrases such as
"the trick", "the trap", "the catch", or "the hard part". State the input limits
without explaining how an approach behaves at them. The banned invariants are a
design's internal ones; the product invariants the task must hold are the task.

**Names are part of the prompt.** Lab titles, directory slugs, catalog rows, and
source labels are solution-neutral too.

**`HINTS.md`** holds the architecture guidance, the neighbour boundary
differences, the rejected designs, and the solution-bearing citations. Nothing
in `README.md` summarizes it.

**`EVALUATION.md`** is the answer key. It states what a strong design holds,
what a weak one gets wrong, the boundaries to observe, and any independently
computed result a check compares against. The ban does not bind it.

`HINTS.md` and `EVALUATION.md` are solution-bearing **by choice**, and that
choice is their only protection. Do not add secrecy machinery around them, and
do not weaken them to make them safe to open early — a hint that spoils nothing
is a hint that helps nobody.

Fixed wording, never paraphrased: every `HINTS.md` opens with the exact line
`> Spoilers. Open only when stuck.` and every `README.md` ends with the exact
line ``Stuck? See `HINTS.md`.``

`make teaching-lint` enforces this and runs in CI: it fails on a mechanism name
— prescribed, disclaimed, or merely mentioned — a barrier name, a neighbour
boundary-difference sentence, a dependency configuration-parameter name, or any
`Code pointers` citation in any course-authored learner-facing text other than
`HINTS.md` and `EVALUATION.md`. Every quirk source is solution-bearing, because
the page that reports the behaviour states it, so a `README.md` that cites the
page has handed over the reading. The only links `README.md` carries are the
neighbour documentation pointers and the dataset provenance.

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

Individual labs run six to twenty-five focused hours. The seventeen core labs
run to roughly 150 to 250 focused hours.

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

**The domain supplies the reasons, never the difficulty.** A lab names a real
subject, and that subject must explain why the invariant exists and why the
scale target's numbers are the size they are. A domain that could be swapped
for any other without changing a requirement is decoration: the learner
finishes knowing the technology and nothing else. Ground it in a public source
the same way a quirk is grounded — fetch the page, quote the sentence.

The opposite failure is worse, because it is invisible. The domain must never
become something to learn: no formats to parse, no vocabulary to memorize, no
regulation to interpret. One sentence of fact that explains an existing
requirement is the whole of it. If a learner would need the domain to pass
rather than to understand, cut it back.

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
  EVALUATION.md     the answer key, opened by choice
  sources/          provider adapters and provenance manifests
  workload/         seeded generators and bounded cached replay
  infra/compose/dependencies.yml   fixed external systems only
  infra/kubernetes/sandbox/  infra/opentofu/sandbox/
  evidence/         generated, gitignored
```

Checks observe public boundaries only, never private application functions.
`starter/` contains no end-to-end path.

## Verification contract

Each lab exposes this root vocabulary and no other:

```
make up         start the environment (make down stops it)
make            format, build, lint, fast test
make test       unit and contract tests, under five seconds
make test-all   local integration suite, what CI runs
make fault      deterministic failure and recovery scenarios
make bench      seeded load, evidence report, observed history
make teaching-lint   fail on any solution leak in learner-facing text; CI runs it
make source     record bounded real data
make smoke      live cloud check, the only target that leaves the workstation
make clean      remove generated artifacts, keep cached source data
```

Correctness gates assert exact record identities and histories, never counts
alone. Performance gates NEVER hard-code a number.

**There is no grader binary and no verification skill.** Each lab carries
`EVALUATION.md` — the answer key, read by whoever checks the work. It states
what a strong solution holds, the boundaries to observe, the selectors a check
ranges over, and the independently computed result a check needs where no
invariant shape supplies one.

A skill was the wrong shape: an agent loads one by default, so it discloses
itself without the learner ever choosing to see it. A file is opened by an act
— the same standard `HINTS.md` has always had, and the only protection either
file gets or needs.

No part of the judgement is compiled. Two parts of the run still are, because
neither can be described away: the fault controller, which must fire at exact
barriers, and the workload generator, which must hold an offered rate under
load. The run stays reproducible; the judgement over it does not, and a lab
whose correctness cannot be stated clearly enough for that to be reliable has
an unclear invariant.

## Languages

- **Python** — tooling: fixtures, TOML, evidence reports, provenance
  manifests, repository automation. `uv` projects and PEP 723 scripts.
- **Go** — anything that must keep time under load: the open-loop generator,
  fault controller, provider simulators. NEVER move these to
  Python; a generator that slows with the system under test destroys the
  measurement the labs teach.
- **Go** — the learner starter, and every brief names one starter, never a
  menu. The exceptions are dictated by the environment, never by taste:
  **Rust and C** for phase 6, the low-level track; the chain's own language
  for an on-chain program; TypeScript for a browser bundle.
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
4. Name the domain and cite the fact that makes its numbers and its invariant
   inevitable. If the spec reads the same with the domain swapped out, the
   domain is decoration; ground it or drop the pretence.
5. Split `Architecture questions`. A question stated at the level of the
   property the design must defend publishes into `README.md`. A question that
   presupposes a mechanism is solution-bearing: mark it `HINTS.md`-bound, or
   rewrite it until it names only the property.
6. Every `Code pointers` citation is solution-bearing and lands in `HINTS.md`
   or `EVALUATION.md`. NEVER put a quirk source in `README.md`: the document
   that reports the behaviour states the behaviour, so citing it hands over
   the reading the learner is meant to do. `README.md` carries the
   requirements, the dataset provenance, and the neighbour documentation
   links, and no other citation.
7. Copy `template/` to the lab directory and remove its implementation. Write
   `EVALUATION.md`. NEVER write a worked solution — the answer key describes
   what a strong design holds, it does not implement one.
8. Add the fault schedule recipe to the controller, update the frozen aggregate
   digest, and confirm `make fault` materializes and removes it.
9. Run `make teaching-lint`. Add a row to the core catalog in `specs/index.md`.

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
expose a failure the others cannot reach; see the track files under `docs/`.

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

- Phases 1 and 2 — twelve specs. Ten were reviewed and released as `v0.1.0`;
  the import lab has since merged into `1/2`, phase 1 gained two labs from an
  orthogonality review (`S20`), and `1/7` arrived from `S22`.
- Phases 3, 4, 6, 7, 8 — 22 specs, drafted and reviewed against the contracts
  this file states. `7/7` is the newest: a deposit service holding capital at
  addresses it cannot sign for, where an observed movement names no request.
  Its first draft duplicated `7/3` and `1/5`; it was narrowed to the one
  property no other lab covers, and `7/3` and `7/4` now point back at it.
- The 2026-08-23/24 sweep closed `S8`, `S11`, `S14`, `S15`, `S16`, `S17`, `S18`,
  `S19`, `S20`. The 2026-08-28 sweep closed `S23` and `S24`: nine labs gained
  a domain that does work, and the teaching contract was restored across the
  specs that predate it. The 2026-08-29 hunt opened `S25` and `S26`: the
  governing spec and the shared scaffold contradict contracts they govern, and
  seventeen labs promise neighbour documentation links they do not carry. The
  open queue in `BUGS.md` is `S1`, `S9`, `S10`, `S25`, `S26` — all `proposed`,
  all needing sign-off. The 2026-08-30 sweep closed most of `S26`: every
  `Code pointers` citation is solution-bearing, each brief names one starter
  instead of a menu, every neighbour carries a fetched documentation link,
  the mechanism leaks are gone, and every fault fires at a named barrier.
  What is left in `S26` is user-owned — the phase-4 corpus numbers and the
  phase-2 pair drift.
- Every spec carries the nine sections in order, the neighbour publish-split
  sentence, `status: draft`, and a scale target. Seventeen carry that sentence
  without the links it promises — see `S26`. `1/7` and `7/7` are in
  `specs/index.md`'s full list; whether `1/7` joins the seventeen core labs is
  undecided.
- No code exists. The shared scaffold is specified in `specs/0/5-shared-scaffold.md`.
- Root `LICENSE` (verbatim GPL-3.0 from gnu.org), `README.md`, and
  `THIRD_PARTY.md` exist as of 2026-08-29, discharging the licensing contract
  in `01-systems-labs.md`. `THIRD_PARTY.md` lists only `LICENSE` itself;
  nothing else here is copied.
- `docs/cloud-access.md` is the single cloud-onboarding page: zero-spend budget,
  short-lived credentials, per-phase account table. Only phase 4 buys anything.

# Project Memory

- A disclaimed mechanism is still a named mechanism. This was the oldest and
  most pervasive leak in the repository, and this file used to require it.
- Emulators reproduce a hosted API and omit its limits, so a lab whose lesson
  lives in a limit needs the fault controller to supply the limit.
- Within a category the course picks the heavier tool, and the lab must then
  drive it into the regime where its quirk fires. A heavyweight dependency the
  scale target never stresses teaches configuration and nothing else.
