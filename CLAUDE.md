# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# systems-labs

Curriculum of end-to-end systems labs. Each lab prompts a complete, useful
product that a learner designs, breaks, recovers, and defends with evidence.
34 labs across seven phases; seventeen are the core.

**Today the repository holds the labs as text only** — no code, no `Makefile`,
no scaffold. Every target below is described, not implemented. Do not report a
build or test result; there is nothing to run.

`docs/contract.md` is the governing contract and the single source for every
cross-lab rule. This file is its operational summary; where the two disagree,
the spec wins and this file is wrong.

## Teaching mode — no free solutions

When a learner works a lab here, act as a teacher, not an answer key. NEVER
state the design, name the technique, list the edge cases, or reproduce a hint
from `hints/` unless the learner has explicitly asked for a hint or the
solution **twice** — two separate, explicit requests, not implied by frustration
or a vague "I'm stuck". Before that threshold: ask guiding questions, name a
concept worth reviewing, or explain why a proposed approach fails. Never write
the solving code.

## No worked solutions — the source is the ground truth

**No lab has a golden or rotten implementation**, anywhere. A systems lab
admits many correct designs, so no implementation is canonical, and a worked
one would answer every `Architecture questions` bullet at once.

Two things stand in its place. **The cited source**: every lab names the public
document reporting the behaviour its quirk rests on, and that citation is the
lab's ground truth. Fetch the page and confirm it says what you claim; a URL
recalled from memory is not a citation, and NEVER invent a quirk and then look
for a source. **`EVALUATION.md`**: the answer key, opened by choice, no grader
binary and no framework.

Verification needs no oracle. It asserts invariants over observed histories
instead of comparing output to a reference run.

`template/` is the one exception and is not a lab. It implements a trivial
domain as the scaffold's regression test, so a broken generator, controller, or
evidence writer fails before any lab does.

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
source repository — each a deliberate act, like opening `hints/`.

## Four artifacts, four audiences — never spoil the lab

One rule governs everything below: **a learner who wants the exercise must be
able to avoid the answer without effort.** That is why these are separate
artifacts and never sections of one. Anything solution-bearing that lands in `README.md`
cannot be unseen.

| file | holds | opened |
|------|-------|--------|
| `README.md` | the task and the landscape | by default |
| `hints/` | the design reading, one file per hint | by choice, when stuck |
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
straight at the quirk — that sentence belongs in `hints/`.

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

**`hints/`** holds the architecture guidance, the neighbour boundary
differences, the rejected designs, and the solution-bearing citations, one file
per hint, indexed by `hints/README.md`. Nothing in `README.md` summarizes it.

**`EVALUATION.md`** is the answer key. It states what a strong design holds,
what a weak one gets wrong, the boundaries to observe, and any independently
computed result a check compares against. The ban does not bind it.

`hints/` and `EVALUATION.md` are solution-bearing **by choice**, and that
choice is their only protection. Do not add secrecy machinery around them, and
do not weaken them to make them safe to open early — a hint that spoils nothing
is a hint that helps nobody.

Hints are a directory, `hints/`, one file per hint, with `hints/README.md`
listing what each file answers and nothing more. Fixed wording, never
paraphrased: every hint file and that index opens with the exact line
`> Spoilers. Open only when stuck.` and every lab `README.md` ends with the
exact line ``Stuck? See `hints/`.``

**A failure mode is a hint.** A lab `README.md` never says how a design fails,
where it fails, or what a client sees when it does. Predicting that is the
learner's first task.

`make teaching-lint` enforces this and runs in CI: it fails on a mechanism name
— prescribed, disclaimed, or merely mentioned — a barrier name, a neighbour
boundary-difference sentence, a dependency configuration-parameter name, or any
`Code pointers` citation in any course-authored learner-facing text other than
`hints/` and `EVALUATION.md`. Every quirk source is solution-bearing, because
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

Every lab passes three: product, 60–120 min; failure, 2–4 h; evidence, 1–2 h.
`HOWTO.md` states what each delivers. A lab's own budget runs six to twenty-five
focused hours, and the seventeen core labs sum to 203 at the low end of their
stated budgets and 294 at the high end. Every one of these numbers is an
estimate: nothing has been built, so nothing has been measured.

## Difficulty, scale, and the earned dependency

**The task carries the lab.** Judge a brief by whether it sends the learner to
primary documentation, to a post-mortem, or to their own experiment, and by
whether their first design teaches them something when it fails.

Difficulty comes from the quirk of the system under study. NEVER make a lab
hard through input formats, parsing chores, or boilerplate volume. A lab that is
merely laborious has failed, and so has one whose product path is obvious once
the environment runs.

**The domain supplies the reasons, never the difficulty.** The subject must
explain why the invariant exists and why the scale numbers are that size, and it
is grounded like a quirk: fetch the page, quote the sentence. It must never
become something to learn — no formats to parse, no vocabulary, no regulation to
interpret. One sentence of fact that explains an existing requirement is enough.

Every lab states a **scale target**: a speed, a load, an amount. They size the
problem so a toy design fails on its own terms. They are NOT pass thresholds;
thresholds stay relative, calibrated, structural, or learner-declared.

**A heavyweight dependency must be earned.** Drive it into the regime where its
quirk fires and show that firing in the evidence. If the naive small tool would
pass the same gates at the same scale, the scale target is too low.

## Faults fire at named barriers

A scenario says "when record 4711 is acknowledged, freeze the broker" — never on
a timer, never at random — so the failure lands at the same boundary on every
run and verification asserts an exact history. Random chaos proves nothing twice.
Every scenario declares what must remain true after recovery; a fault the system
survives with nothing to check is entertainment, not a gate.

## Per-lab directory shape

```text
labs/<phase>/NN-solution-neutral-name/
  README.md         task, no solution
  ARCHITECTURE.md   learner-owned
  hints/            one file per hint plus an index, opened by choice
  EVALUATION.md     the failure schedule and what it must leave true
  lab.toml
  compose.yml       learner-owned application topology
  app/  tests/      learner-owned
  starter/          generated clients, contracts, empty seams, no product path
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
make smoke      live cloud check, the only target that deploys to an account
make clean      remove generated artifacts, keep cached source data
```

Correctness gates assert exact record identities and histories, never counts
alone. Performance gates NEVER hard-code a number.

**There is no grader binary and no verification skill.** An agent loads a skill
by default, so a skill would disclose itself without the learner choosing to see
it; a file is opened by an act. `EVALUATION.md` states what a strong solution
holds, the boundaries to observe, the selectors a check ranges over, and any
independently computed result a check needs.

No part of the judgement is compiled. Two parts of the run are, because neither
can be described away: the fault controller, which fires at exact barriers, and
the workload generator, which holds an offered rate under load.

## Languages

- **Python** — tooling: fixtures, TOML, evidence reports, provenance manifests,
  repository automation. `uv` projects and PEP 723 scripts.
- **Go** — anything that must keep time under load: the open-loop generator,
  fault controller, provider simulators. NEVER move these to Python.
- **Go** — the learner starter, and every brief names one, never a menu. The
  environment dictates the exceptions: Rust and C in phase 6, the chain's own
  language for an on-chain program, TypeScript for a browser bundle.
- **HCL** — infrastructure, OpenTofu. Pulumi and CDK are out: a general-purpose
  language computes resources at run time instead of declaring them.
- SQL and PL/pgSQL stay in the database boundary; Java is confined to the Flink
  job.

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
2. Create the lab directory, `labs/<phase>/<number>-<name>/`, and write its
   `README.md`: the brief, what the environment gives, the requirements, the
   scale target, the architecture questions that state a property, the
   acceptance evidence, the neighbour names with their documentation links,
   and what is outside the problem. It ends with the fixed closing line.
3. Name the scale target — speed, load, amount — and check the earned-dependency
   rule: state what the naive small tool would fail at this scale. If nothing,
   raise the scale target or drop the dependency.
4. Name the domain and cite the fact that makes its numbers and its invariant
   inevitable. If the spec reads the same with the domain swapped out, the
   domain is decoration; ground it or drop the pretence.
5. Split `Architecture questions`. A question stated at the level of the
   property the design must defend publishes into `README.md`. A question that
   presupposes a mechanism is solution-bearing: mark it `hints/`-bound, or
   rewrite it until it names only the property.
6. Every `Code pointers` citation is solution-bearing and lands in `hints/`
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
9. Run `make teaching-lint`. Add a row to the core catalog in `labs/README.md`.

## Naming rule — read before editing any link

Lab files carry **solution-neutral product names**. The file name states what
the product does, never the technology that solves it. `1/2` is
`reservation-fulfillment`, not `postgres-durable-jobs`.

`labs/README.md` carries the product names in both its core catalog and its full
specification list. Before editing a link, read the real file name from disk
rather than copying one from a table.

## Phases

The digit directory under `labs/` is the curriculum **phase**, not a version:
1 local runtime and delivery; 2 the same problems serverless; 3 real Internet
streaming; 4 NoSQL, analytics, portability; 6 low-level; 7 blockchain;
8 search, retrieval, spatial. **Phase 5 no longer exists** and the `7/5` slot is
empty; both gaps stay open, so do not renumber to close them.

**Phase 1 runs software the learner operates; phase 2 runs an execution model
the learner cannot.** That is why phase 2 exists, and it is the one place a
product may be repeated: a port across languages inherits the original's answer,
a port across execution models confiscates it. Phases 6, 7 and 8 are separate
catalogs, and each track's labs must expose a failure the others cannot reach.

Where things live: every lab is a directory under `labs/`, and everything that
lab needs is inside it. `labs/README.md` is the core catalog and per-lab detail
lives there and nowhere else; `labs/<phase>/README.md` orients a phase;
`docs/contract.md` is the shared contract every lab obeys and
`docs/shared-scaffold.md` the environment it runs on; `docs/` also holds the
selection record and the track catalogs; `HOWTO.md` is the learner's method and
`CONTRIBUTING.md` its author-side counterpart; `BUGS.md` is the queue,
`TODO.md` the backlog, `.diary/` the log.

## Approval boundary

`docs/contract.md` blocks implementation while it is `draft`. Writing code into
a lab directory, pinning dependency versions, and building lab 01 need the user
to move it to `accepted` first. Writing and expanding the labs is open work.

## State of the repo

- Thirty-four labs exist under `labs/`: seven in phase 1, five in phase 2, two
  in phase 3, four in phase 4, five in phase 6, six in phase 7, five in phase 8.
  The seventeen core labs are the phase 1 to 4 rows of the core catalog in
  `labs/README.md`. `1/7` sits in phase 1 and outside the core; phases 6, 7
  and 8 are separate catalogs.
- Every lab directory holds `README.md`, `hints/` with its index, and
  `EVALUATION.md`. Each names a scale target and carries a fetched
  documentation link for every neighbour. `1/7` names no neighbour at all:
  every genuine candidate carries the answer in its name, so all three sit in
  `hints/`, and the lab records that deviation where it happens.
- `BUGS.md` holds `S33` to `S37`. The fault injection contract in
  `docs/contract.md`
  states the two rules every lab's gate now rests on: a barrier holds execution
  rather than being noticed once it has passed, and the controller supplies the
  limit the environment omits, at a transport, process, or clock layer. A lab
  whose required gate depends on a limit names its layer and says the gate
  proves a declared model, with `make smoke` as the comparison. `TODO.md` holds
  what does not exist yet.
- No code exists. The shared scaffold is specified in `docs/shared-scaffold.md`,
  and it is the first thing built once `docs/contract.md` is `accepted`.
- Root carries `LICENSE` (verbatim GPL-3.0 from gnu.org), `README.md`, and
  `CONTRIBUTING.md`. `LICENSE` is the one copied file here; the licensing
  contract in `docs/contract.md` adds a root ledger with the first commit
  that copies anything else.
- `docs/cloud-access.md` is the single cloud-onboarding page: zero-spend budget,
  short-lived credentials, per-phase account table. Phases 2 and 4 are the only
  ones that can bill, and only on the optional path.

# Project Memory

- A disclaimed mechanism is still a named mechanism, and it is the leak that
  hides best: ruling a candidate out names it as surely as prescribing it.
- Emulators reproduce a hosted API and omit its limits, so a lab whose lesson
  lives in a limit needs the fault controller to supply the limit.
- Within a category the course picks the heavier tool, and the lab must then
  drive it into the regime where its quirk fires. A heavyweight dependency the
  scale target never stresses teaches configuration and nothing else.
