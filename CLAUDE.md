# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Systems Labs is a curriculum of end-to-end systems labs: sixteen core labs
across phases 1 through 4, plus three separate catalogs. Today the
repository contains **specifications only**. There is no code, no `Makefile`,
no `git` repository, and no build, lint, or test command. Every command listed
under [Verification contract](#verification-contract-specified-not-implemented)
is specified, not implemented.

Do not report a build or test result. There is nothing to run.

## Layout

```text
specs/
  index.md                  authoritative list of specs and their status
  01-systems-labs.md        governing course spec — all shared contracts
  0/1-lab-selection.md      20 scored candidates, 10 keeps, 10 cuts
  0/2-low-level-track.md    second catalog, Rust and C, 5 specced
  0/3-blockchain-track.md   third catalog, Solana and Ethereum, 5 specced
  0/4-search-and-retrieval-track.md  fourth catalog, OpenSearch, 5 specced
  0/5-shared-scaffold.md    what all labs share, and the build order
  0/6-serverless-contrast-track.md  phase 2 paired against phase 1
  1/1 … 1/5  2/1 … 2/5  3/1 3/2  4/1 … 4/3 4/5   the sixteen core lab specs
  6/1 … 6/5                 the five low-level specs
  7/…  8/…                  the blockchain and retrieval specs
  <phase>/README.md         per-phase orientation: its technologies and docs
docs/cloud-access.md        optional cloud accounts, free tiers, zero-spend
.diary/YYYYMMDD.md          shipping log
```

The digit directory under `specs/` is the curriculum **phase**, not a version:
1 local runtime, delivery, and cross-system atomicity; 2 the same problems on a
serverless execution model; 3 real Internet streaming; 4 NoSQL, analytics, and
platform portability; 6 low-level; 7 blockchain; 8 search, retrieval, and
spatial.

**Phase 5 no longer exists**, and its single lab became `4/5`. A phase holding
one lab is a chapter heading. The gap stays open on purpose; do not renumber to
close it.

**Phase 2 was merged away and then reintroduced for a different reason.** The
old `2/1` became `1/5` and is not coming back. The current phase 2 is the
serverless contrast catalog: four phase 1 products rebuilt on an execution model
the learner cannot operate, plus one product that introduces it. See
`specs/0/6-serverless-contrast-track.md`.

## Four tracks, and why the others are not ports

Phases 1 through 4 are the accepted-in-principle catalog: sixteen labs whose
quirks live in databases, brokers, queues, and platforms. Phase 6 is a **separate** catalog in
Rust and C whose quirks live in the kernel — readiness and I/O completion,
durability, virtual memory, allocation, time. Phase 7 is a **separate** catalog
on Solana and Ethereum in four shapes only: validator enhancement, chain data
processing, end-to-end program development, and permissionless delivery.

Phase 8 is a **separate** catalog built on retrieval — OpenSearch paired with a
domain store over real public corpora: proteins, news, a bounded web crawl,
map extracts, and relevance evaluation. Its labs are deliberately denser than
the core phases, because the interaction between the search engine and the store is
the lesson. See `specs/0/4-search-and-retrieval-track.md`.

Never propose porting an existing lab to another language and calling it a new
lab. A port inherits the original's grader, failure schedule, and answer; the
learner reimplements a known solution under a stricter compiler. Each track's
labs must expose a failure the others cannot reach. See
`specs/0/2-low-level-track.md` and `specs/0/3-blockchain-track.md`.

## Ground labs in sources, do not invent them

Every candidate names the public document where its quirk is described, and the
track records carry that origination column. When adding a lab, find the real
reported behavior first — a kernel manual page, a post-mortem, a paper, a
vendor limit — and cite it. Do not invent a puzzle and then look for a source.
All such sources are *cited*: nothing is copied from them.

Each lab spec carries its own quirk sources as links in `Code pointers`, one
bullet per source with a single sentence on what that page establishes. Fetch
the page and confirm it says that; a URL recalled from memory is not a
citation. Where a source names the mechanism that solves the lab, mark it
solution-bearing so it lands in `HINTS.md` and never in `README.md`.

## Languages

- **Python** — tooling: fixtures, TOML, evidence reports, provenance, the
  grading runner, repository automation. `uv` projects and PEP 723 scripts.
- **Go** — anything that must keep time under load: the open-loop workload
  generator, fault controller, provider simulators, mechanical grader. Do not
  move these to Python; a generator that slows with the system under test
  destroys the measurement the labs teach.
- **Go and TypeScript** — the two supported learner starters for the core
  phases.
- **Rust and C** — the learner languages for phases 6–8.
- **HCL** — infrastructure. OpenTofu, Terraform-compatible. Pulumi and CDK are
  excluded by policy, so infrastructure is never written in Python or
  TypeScript.
- SQL and PL/pgSQL stay in the database boundary; Java is confined to the
  Flink job.

## The core labs

Phase 1, local: `1/1-resilient-quote-service.md` ·
`1/2-reservation-fulfillment.md` · `1/3-order-activity-dashboard.md` ·
`1/4-reliable-record-import.md` · `1/5-auditable-transfer-service.md`

Phase 2, serverless: `2/1-metered-billing-api.md` ·
`2/2-reliable-record-import.md` ·
`2/3-serverless-reservation-fulfillment.md` ·
`2/4-serverless-auditable-transfer.md` ·
`2/5-serverless-quote-aggregation.md`

Phases 3 and 4: `3/1-internet-route-observatory.md` ·
`3/2-recoverable-route-analytics.md` · `4/1-market-history-api.md` ·
`4/2-low-latency-market-api.md` · `4/3-exact-trade-analytics.md` ·
`4/5-portable-market-ingestion.md`

The original selection produced ten. `4/3` was added for the analytical store
that merges in the background. `1/5` and `4/5` arrived by merge — they are the
former `2/1` and `5/1`. Phase 2 was then built by pairing: `2/1` is the former
`4/4`, `2/2` is the former `1/4`, and `2/3` to `2/5` are new recasts of `1/2`,
`1/5`, and `1/1`. `1/4` was rewritten on a self-run broker so that phase 1 holds
no hosted-service emulator.

**Phase 1 runs software the learner operates; phase 2 runs an execution model
the learner cannot.** That split is the reason phase 2 exists, and it is the one
place a product may be repeated. A port across languages is still forbidden.

The core catalog in `specs/index.md` holds each lab's system, architecture
pressure, and prepared environment. Read it there; do not restate it elsewhere,
because a second copy drifts. `01-systems-labs.md` specifies the contracts every
lab shares and carries no per-lab detail.

## Naming rule — read before editing any link

Lab files carry **solution-neutral product names**. The file name states what
the product does, never the technology that solves it. `1/2` is
`reservation-fulfillment`, not `postgres-durable-jobs`.

`specs/index.md` carries the product names in both its core catalog and its
full specification list, and every internal link resolves. Before editing a
link, read the real file name from disk rather than copying one from a table.

## Teaching mode — no free solutions

When a learner works a lab in this repository, act as a teacher, not an answer
key. NEVER state the design, name the technique, list the edge cases, or
reproduce a hint from `HINTS.md` unless the learner has explicitly asked for a
hint or the solution **twice** — two separate, explicit requests, not implied
by frustration or a vague "I'm stuck". Before that threshold: ask guiding
questions, name a concept worth reviewing, or explain why a proposed approach
fails. Never write the solving code.

The failure schedule is the lab. `grader/` and the fault controller — with
each lab's seeded recipes compiled into its binary, never shipped as readable
files — hold the edge cases so the learner discovers them by running
`make fault`, after committing to a design; the readable schedule exists only
while a run is in flight. Never summarize the edge cases into `README.md` and never recite them
in conversation. The master spec's
[Teaching contract](specs/01-systems-labs.md) fixes which artifact holds what.

`specs/` is author-facing and is not part of the learner tree. The reasoning
about what is optimal and why belongs here.

## Spec conventions

Every spec starts with YAML frontmatter holding one key:
`status: draft | reference | accepted`. `01-systems-labs.md` and every lab are
`draft`. The four catalogs under `0/` and `docs/cloud-access.md` are
`reference`.

Every lab spec uses the same nine sections in this order, and no others:

`Brief` · `Prepared scaffold` · `Requirements` · `Architecture questions` ·
`Adversarial evaluation` · `Acceptance evidence` · `Neighbouring systems` ·
`Scope` · `Code pointers`

`Neighbouring systems` names the two or three technologies a practitioner would
have reached for instead and the one thing each does differently at this lab's
boundary, then points at their documentation. The lab never runs them. This is
where excluded technologies live: named as reading, not as dependencies.

`Scope` is renamed `Scope and cost` or `Scope and data` where that lab carries
a real cost or real-data boundary. `Code pointers` always ends with the line
"Implementation pointers do not exist while the spec is `draft`."

The prose follows a hard style: no marketing, no second person, no naming of
the solution. A lab spec states the invariant and the evidence, and states the
**categories** of decision the learner owns — the concurrency model, the data
layout, the delivery and recovery path, the process decomposition — naming no
candidate mechanism under any of them, not even to exclude it. A disclaimed
mechanism is still a named mechanism: "does not prescribe a worker pool" tells
the learner a worker pool is in play. Preserve that. Adding "use a semaphore"
to a spec breaks the curriculum, and so does adding "no semaphore is
required".

## Contracts every lab inherits

`specs/01-systems-labs.md` is the single source for all cross-lab rules. Read
it before changing any lab spec. It fixes:

- **Lab brief contract** — every lab reads "design and build a system that does
  X under these requirements". The brief fixes the product, public behavior,
  environment, limits, failures, and evidence. It never gives the service
  decomposition, schema, event keys, transaction boundaries, retry algorithm,
  cache policy, recovery mechanism, or deployment topology. Each submission
  carries an `ARCHITECTURE.md`; a pattern name is not an explanation.
- **One system, no solution** — a brief describes exactly one system and never
  splits it into named services. A second dependency belongs to the fixed
  environment, not to a second product. Writing "outbox" or "write-through
  cache" into a brief answers the lab's own question and breaks it.
- **Difficulty standard** — every lab is hard, and the difficulty comes from
  the quirk: a notification that never replays, a lease that is not a deadline,
  an index that is not yet consistent. Never make a lab hard through input
  formats, parsing chores, or boilerplate volume.
- **Scale target** — every lab states a speed, a load, and an amount. Those
  three numbers size the problem so a toy design fails on its own terms. They
  are not pass thresholds; thresholds stay relative, calibrated, structural, or
  learner-declared.
- **Telemetry is not universal** — require it only where the measurement is the
  evidence (load, cache, latency labs). Elsewhere state which numbers the
  report must contain and leave the method open. Observability as a deployable
  product belongs in its own lab, not as a tax on all of them.
- **Grading** — mechanical checks are deterministic and live in `grader/`.
  Judgment checks (architecture reasoning, evidence quality, residual limit)
  go to a language-model reviewer through `make grade`: a prompt plus a per-lab
  rubric, no grading framework. It refuses to run until the mechanical gates
  pass, and it never names the solution.
- **Standard laboratory scaffold** — one dependency-only Compose profile,
  identical across labs, runnable before any learner code exists. Specialized
  labs *extend* it (Flink cluster, ElasticMQ plus DynamoDB Local plus Lambda
  runner, `kind` plus OpenTofu). They never replace it.
- **Three gates** — product (60–120 min slice), failure (2–4 h deterministic
  falsification), evidence (1–2 h load or deploy run). All three must pass.
- **Scope budget** — one technology surprise, ≤2 learner processes, ≤4 public
  operations, one state model, one failure schedule, one evidence report.
- **Per-lab directory shape** — `README.md` (task, no solution), `HINTS.md`
  (architecture, solution-bearing citations), `app/` and `tests/`
  learner-owned, `starter/` with empty seams, plus `grader/ sources/ workload/
  infra/ evidence/`. The worked golden and rotten pairs live at repository
  root under `reference/`, excluded from the learner distribution by an
  explicit publish step; the fault schedules are seeded recipes compiled into
  the fault controller binary — never readable files in the learner
  distribution — that `make fault` materializes, runs, and removes, pinned by
  an aggregate digest.
- **Config** — first CLI argument is a TOML lab config, optional second is a
  secrets file. Environment variables carry injected secrets only.
- **Data** — generated seeded input is the default and the only CI input. Real
  RIPE RIS Live and Kraken recordings are opt-in, bounded, checksummed, cached
  under `${PREFIX:-/srv}/data/systems-labs/sources/`, and never fetched on a
  request path.
- **Cost** — every required gate runs locally with no AWS account. Optional
  cloud use is Lambda, SQS, DynamoDB on-demand and short logs only; MSK, EKS,
  NAT gateways, RDS, and ElastiCache are excluded by policy.
- **Licensing** — all prose, graders, fixtures, and workloads are original
  GPL-3.0. The Research ledger records each consulted source as *cited*,
  *adapted*, or *copied*; sources marked citation-only must contribute no
  copied text, code, or fixture. Update the ledger when you add a dependency.

## Verification contract (specified, not implemented)

When labs get built, each exposes this root vocabulary and no other:

```
make up         start the environment (make down stops it)
make            format, build, lint, fast test
make test       unit and contract tests, under five seconds
make test-all   local integration suite, what CI runs
make fault      deterministic failure and recovery scenarios
make bench      seeded load, invariant check, evidence output
make grade      language-model judgment review, only after the gates pass
make teaching-lint   fail on a mechanism name — prescribed, disclaimed, or
                     merely mentioned — a barrier name, neighbour product
                     name, dependency configuration-parameter name, or
                     solution-bearing citation in any lab README; CI runs it
make source     record bounded real data
make smoke      live cloud check, the only target that leaves the workstation
make clean      remove generated artifacts, keep cached source data
```

Correctness gates assert exact record identities and histories, never counts
alone. Performance gates never hard-code a number: use a relative baseline on
the same host, a calibrated warm-up capacity, a structural bound, or a
learner-declared SLO.

## Approval boundary

`01-systems-labs.md` blocks implementation while it is `draft`. Creating a lab
directory, pinning dependency versions, or authoring lab 01 needs the user to
move that spec to `accepted` first. Editing and expanding the specs is open
work.
