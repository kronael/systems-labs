# Bugs

## S26 — half the labs promise a landscape they do not carry (2026-08-29, proposed)

Three parallel hunts over all 33 existing lab specs. Findings are grouped by
what they break, not by phase. Everything below was re-verified against the
file text.

**The systemic one. Seventeen specs carry the sentence "The names and their
documentation links publish into `README.md`" and contain no documentation
link at all.** The `README.md` those specs generate would name neighbours with
nothing to read, which is the half of the landscape contract that does the
orienting. `1/1`, `1/3`, `1/5`, `2/1`, `2/2`, `2/3`, `2/4`, `2/5`, `6/2`,
`6/3`, `6/4`, `6/5`, and every phase-7 spec — `7/1`, `7/2`, `7/3`, `7/4`,
`7/6`. Four of them also name categories rather than products, which fails the
contract twice: `2/3` ("A relational store behind the same functions"), `2/4`
("A change-data stream from the store"), `2/5` (all three are categories), and
`7/1`, whose second and third neighbours are two API surfaces of the same
platform.

**Banned phrasing in learner-facing Briefs.** `CLAUDE.md` bans "the trick",
"the trap", "the catch", and "the hard part" from text that publishes into
`README.md`. `7/2:24` and `7/6:23` both open the paragraph that states the
lab's difficulty with "The hard part is". `0/3:15` and `0/3:72` use it too,
which is fine — the track record is author-facing.

**Unmarked solution-bearing citations.** `CLAUDE.md` step 6 requires every
`Code pointers` citation marked neutral or solution-bearing; unmarked defaults
to README-eligible. These name the answer and are unmarked:

- `4/3:143-153` — all three ClickHouse pointers, naming the table engine, the
  asynchronous mutation behaviour, and `parts_to_throw_insert` with its
  default. The engine name answers the lab's central data-layout question.
- `6/3:150-158` — the mmap paper and `mmap(2)`/`madvise(2)`, while the Brief
  makes the file-access strategy the learner's decision.
- `6/2:140-143` — fsyncgate and the LWN article, while the durability strategy
  is the learner's decision.
- `6/5:169-180` — the coordinated-omission source and both timing interfaces,
  while `wrk2` and HdrHistogram in the same section *are* marked.
- `6/4:158-160` — `malloc(3)`, while its three sibling bullets are marked.

2026-08-30: this finding is closed by contract change, not by marking each
bullet. Every `Code pointers` citation is solution-bearing now, every lab
spec's section says so in its own words, and `README.md` carries only the
neighbour documentation links and the dataset provenance. The page that
reports a behaviour states the behaviour, so citing it in `README.md` handed
over the reading. Commit `ec51a45`. The rest of `S26` stays open.

2026-08-30: the seventeen-spec link gap is closed. Every neighbour in every
lab spec now carries a documentation URL that answered 200 when it was
written, and the nine bullets that named a category now name a product —
Aurora Serverless v2, Step Functions, Fargate, DynamoDB Streams, PostgreSQL
with a self-run Kafka, Cloud Run, API Gateway caching, Istio, GitHub Pages.
The banned opener left `7/2` and `7/6` in the same commit, `fd293b7`.

**Named mechanisms in `Brief` or `Requirements`.** `2/3:56-58` asserts where
the partner lab's invariant lived and rules out a store-level constraint here;
`1/2:110-113` explicitly leaves that placement to the learner, so `2/3` both
leaks and misreports. `2/3:120-121` repeats it as an unmarked architecture
question. Also `4/1:46` ("table scans are not accepted"), `4/3:20` and
`:130-132` (names the store's merge behaviour outside the allowed sections),
`3/2:99-100` (names the exactly-once switch its own pointers mark
HINTS-only), `7/2:83-85` ("Pre-paying for storage the workload has not yet
needed fails this requirement"), `6/1:50-51`, `6/5:78-81`, `8/1:36-37`,
`8/1:100-101`, `8/1:110-111`, `8/4:81-82`, `4/2:72`.

2026-08-30: the mechanism leaks are closed. `4/2`, `6/5`, `8/1`, and `8/4`
now mark their mechanism-presupposing questions `HINTS.md`-bound in the
wording `7/7` uses; `4/1`, `3/2`, `6/1`, `4/3`, and `2/3` state the property
instead of naming or disclaiming the mechanism. Commits `be8f183`, `833409a`,
`bb8fdc7`. `7/2:83-85` stands: the sentence states a cost the evidence
reports, not a mechanism.

**Faults with no named barrier.** The contract says a scenario fires at a
named record, never on a timer. These do not: `4/2:81-85` (five of eight),
`2/2:102-105` (no identity anywhere, and one barrier reads "around its durable
effect", which cannot land at the same boundary twice), `1/1:86-89`,
`1/3:88-91`, `1/5:75-79`, `2/5:108-109` ("inside a measured window"),
`7/1:131`, `7/3:114-115`, `7/4:113`, `4/5:84-85`, `4/3:83`. Every spec does
declare what must hold after recovery; that half is clean.

2026-08-30: closed. Every scenario in all eleven specs now names the barrier
it fires at, and each ambiguous "around its durable effect" is split into one
fault immediately before the effect and one immediately after, because
"around" cannot land at the same boundary twice. Commits `2f32607`,
`90399b7`. What remains in `S26` is user-owned: the phase-4 corpus numbers
and the phase-2 pair drift, both design decisions.

**The phase-4 shared corpus is not shared.** `S23` records the intent as one
corpus and one generator so four stores compare against identical data. In
fact `4/1:48` says 100 million trades across 20 symbols, `4/2:52` says 100
million across 50, and `4/3:47` says 2 billion across 500. `4/1:22` names the
prepared generator as "a deterministic million-trade generator", 100x under
its own lab's target and 2000x under `4/3`'s. `4/2`'s scaffold names a loader
with no tie to the shared recording pipeline. `specs/4/README.md:125` claims
the phase-4 neighbour sections name Cloud Run and Step Functions; neither
appears in any phase-4 spec.

**Phase-2 pairs drifted from their partners.** `2/4:41-45` claims the product
is identical to `1/5` and then drops three of its invariants — value
conservation, history-and-balance agreement, and the conflicting-reuse rule —
without flagging any as platform-forced, though it does flag a fourth.
`2/3:14` generalizes `1/2`'s stay-boundary rule to "one resource for a
half-open time interval" and never restates the half that makes back-to-back
stays legal, so `S23`'s grounding of `1/2` did not propagate. `2/5`↔`1/1` is
clean.

**Smaller, verified.** `7/2:87-89` states a wall-clock speed where `0/3:136`
requires speed against the node's own rate. `0/3:112` claims `7/4`'s shape is
"end-to-end program development plus dispatcher" while `7/4:166` excludes
authoring a program, so `7/4` fits none of the four admitted shapes. `8/1:238`
cites sequenceserver.com as "the track record's origination source"; `0/4:32`
originates `8/1` from NCBI. `8/3:115` rests a load-bearing claim on an
evidence-window length the spec never states. `1/3`, `1/5` and `2/2` state no
earned-dependency sentence. `4/1`, `4/3` and `4/5` name four or five
neighbours where the contract says two or three, and `4/3:118` links only one
of a paired name. Five phase-8 specs and `0/4:11` carry "phases 1 to 5"
fossils. Grammar: `2/3:138` "They observes HTTP"; `2/5:40`; `8/2:92` says
"These three numbers" above four.

**Clean.** All 33 carry the nine sections in order and `status: draft` alone.
All 33 state a scale target with speed, load and amount. Every relative link
resolves. No spec references `4/4`, `7/5` or a phase-5 lab. Roughly sixty
citations were fetched across the three hunts and none contradicted its
claim; four are bot-blocked and were confirmed through mirrors or left marked
unreachable. Arithmetic recomputes throughout except the phase-4 corpus above.

### Proposal, needs sign-off

The neighbour-link gap is the one to fix first and is mechanical: seventeen
specs need two or three real documentation URLs each, fetched before they are
written. The phase-4 corpus numbers and the phase-2 pair drift are
design decisions, not typos — they change what a lab teaches — and need the
user before anything moves. The unmarked citations and banned phrases are
corrections that can ship together once approved.

- **Severity:** high — the neighbour-link gap silently breaks the landscape
  half of the teaching contract in half the catalog
- **Scope:** 24 lab specs plus `specs/4/README.md` and `specs/0/3`
- **Affected:** listed inline above
- **Source:** three parallel spec hunts, 2026-08-29; every finding
  re-verified against the file text, and the seventeen-spec link gap
  reconfirmed by a repository-wide count
- **Status:** proposed
- **Fix:**

## S25 — the governing spec and the shared scaffold contradict the contracts they govern (2026-08-29, proposed)

A cross-document hunt over `CLAUDE.md`, `specs/01-systems-labs.md`,
`specs/index.md`, the six `specs/0/` track files, `docs/cloud-access.md`, the
seven phase `README.md` files and `BUGS.md` found fourteen defects. Every one
was verified by reading both sides of the contradiction. None is a matter of
taste; each is a place where one document states something another denies, or
where a document denies itself.

**The two that break a contract rather than merely misstate one:**

- **`0/5` states a `make teaching-lint` rule that would fail every compliant
  `README.md`.** `specs/0/5-shared-scaffold.md:155` says the lint fails when a
  "`Neighbouring systems` product name" appears in learner-facing text.
  `specs/01-systems-labs.md:768` and `CLAUDE.md:131` say it fails on a
  "boundary-difference sentence". The teaching contract *requires* neighbour
  product names in `README.md` — that is the whole publish split — so `0/5`'s
  lint bans the text the contract mandates. `0/5` kept `S6`'s wording after the
  rule changed under it.
- **The governing spec still supplies a "verification skill" it deleted.**
  `specs/01-systems-labs.md:827` says "There is no grader binary and no grading
  framework, and there is no skill." The same file promises one at `:46` ("each
  lab describes its checks in an agent-run skill"), lists it among prepared
  material at `:112`, and makes the GPL licence contract enumerate it at `:876`
  and `:891`. `S17` removed the grader; these four survived.

**Contradictions between documents:**

- **The phase 1/2 pairing count is a fossil in four places.** `0/6:75` records
  that `2/2` lost its partner and that the true recasts are three: `1/1`→`2/5`,
  `1/2`→`2/3`, `1/5`→`2/4`. Against that, `01-systems-labs.md:138` and `:393`,
  `specs/2/README.md:12`, and `specs/1/README.md:128` all still say phase 2
  rebuilds **four** products. `specs/2/README.md` also denies itself: `:12`
  says four are rebuilt, `:29` says two of the five have no partner. It also
  says "Phase 1 built five systems"; phase 1 has seven.
- **`specs/index.md:55` calls `2/2` "the one phase 2 lab with no phase 1
  partner".** `specs/2/README.md:29` and `0/6:71` both say two are.
- **The governing spec asserts a deployment claim its own track file rebuts.**
  `01-systems-labs.md:404` says holding the phase 1 store fixed "would produce
  a shape no practitioner deploys". `0/6:26` says it "is deployable — functions
  with a relational store are a supported and documented shape" and rejects it
  for a different reason. Two documents carry the corrected reasoning; the
  governing one carries the falsified one.
- **The starter-language claim was scoped in `CLAUDE.md` and missed in the
  spec.** `01-systems-labs.md:436` and `:499` still say every lab supplies Go
  and TypeScript. `CLAUDE.md:281` scopes that to the core phases; `0/2` (Rust
  and C) and `0/3` (per-chain toolchains) agree with `CLAUDE.md`.
- **`CLAUDE.md:393` says "Only phase 4 buys anything".** `docs/cloud-access.md`
  licenses phase 2 an optional hosted run on on-demand DynamoDB, which bills
  with no perpetual free allowance.

**Stale state that misreports what is open:**

- **Nine `BUGS.md` entries carry a `FIXED` header and resolution prose while
  their own Status fields still read `open` or `proposed`:** `S20`, `S19`,
  `S18`, `S17`, `S15`, `S14`, `S8`, `S11`, `S2`. `CLAUDE.md:384` says that
  sweep closed them; this file says otherwise.
- **`S21` and `S22` read "approved, in progress" with empty Fix fields**, but
  both shipped — `1/7` exists and is wired, and `8/3` carries S21's breach
  consequence. `CLAUDE.md:387` names the open queue as `S1`, `S9`, `S10` only.
  `S22`'s Affected list also names `docs/lab-selection.md`, which contains
  no trace of `1/7`; that part never landed.
- **`S1` argues about `4/4-subscription-billing-api.md`, which does not
  exist** — the lab is now `2/1-metered-billing-api.md`, which `S10` argues
  about separately, neither entry referencing the other. This is the only
  reference to a nonexistent lab in the repository, and it sits in the entry
  `CLAUDE.md` names first in the open queue.
- **`0/5:9` says "Thirty-one labs"**; `0/5:42` says thirty-three, and disk
  holds 33. The `S24` count sweep missed it.
- **`0/1:9` says the catalog "has since grown to twelve"**; it is seventeen
  core labs. `0/1:213` still describes the superseded five-phase map in the
  present tense with no supersession note.
- **Four "phases 1 to 5" fossils survive** at `0/2:11`, `0/2:103`, `0/4:11`,
  and `specs/8/README.md:11`. `S24` fixed the identical defect in
  `6/README.md` and missed these.
- **`docs/cloud-access.md:7` over-claims its own scope** — "Cloud use exists
  only for the opt-in `make smoke` check" — while `:205` licenses public RPC
  recordings, which are `make source` work.

**Clean.** Every relative link and anchor in scope resolves. The Make
vocabulary is identical across `CLAUDE.md`, `01-systems-labs.md` and `0/5`.
Every file carries exactly one frontmatter key, and every status in
`specs/index.md` matches its file. Lab counts on disk are exactly 33
(7+5+2+4+5+5+5), the core catalog has exactly 17 rows, and the full list 33
entries.

### Proposal, needs sign-off

Two of these are redesign-shaped and need the user before anything ships:
the `0/5` lint rule (it changes what CI enforces) and the pairing-count fossil
(it decides whether the phase 2 contrast is described as three recasts plus two
originals, which is what `0/6` actually records). The rest are corrections to
text that is simply wrong and can ship as one commit once approved.

- **Severity:** high — `0/5`'s lint rule and the verification-skill survivors
  are in the two documents every other file defers to
- **Scope:** cross-document; no lab spec changes
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`,
  `docs/lab-selection.md`, `docs/low-level-track.md`,
  `docs/search-and-retrieval-track.md`, `specs/index.md`,
  `specs/1/README.md`, `specs/2/README.md`, `specs/8/README.md`,
  `docs/cloud-access.md`, `BUGS.md`
- **Source:** cross-document bug hunt, 2026-08-29; every finding re-verified
  against both files before entry
- **Status:** proposed
- **Fix:**

## S24 — the teaching contract drifted out of nine specs and four documents (2026-08-28, approved)

A framing review of all 33 labs surfaced defects that are not about any one
lab's design. They are places where the repository contradicts itself, so a
learner or an author reading one file is told something the next file denies.

Two are broken references into the curriculum graph, and both point at labs
that do not exist:

- `2/2` says "This lab is taken after the local import lab" and names it a
  prerequisite whose artifacts must be retained. `0/6` records that `2/2`
  "lost its partner when phase 1 merged the standalone import lab into `1/2`"
  and is "a model contrast rather than a recast". The prerequisite is
  unsatisfiable, and `specs/index.md` row 08 repeats the stale claim.
- `7/4` excludes cross-chain atomicity because "the last belongs to the
  cross-chain settlement audit candidate". `0/3` records that candidate as
  "cut: its finality lesson largely repeated `7/3`'s".

The rest are drift:

- **Counts.** `CLAUDE.md` says "32 labs" and "Phases 1 and 2 — eleven specs";
  disk holds 33 and twelve. `specs/index.md` says "all 32 labs";
  `specs/0/5-shared-scaffold.md` says "thirty-two answer".
- **`1/7` is unwired.** Absent from both tables in `specs/index.md`; absent
  from `specs/1/README.md`, which still opens "The six labs".
- **Section eight.** `CLAUDE.md` mandates nine sections "in order and no
  others", ending `Scope`. Fourteen specs write `Scope and cost` (five) or
  `Scope and data` (nine). The governing spec never names the sections, so
  `CLAUDE.md` is the only rule and the fourteen break it. It is drift, not
  design: `2/1` and `2/2` carry the suffix while `2/3`-`2/5` do not, in one
  phase with one cost profile.
- **The neighbour publish split.** Twenty-five specs carry the sentence that
  says the names and links publish into `README.md` while the boundary
  difference publishes into `HINTS.md`. Eight do not: `1/1`, `1/3`, `1/5`, and
  all five phase 2 specs — the oldest files, written before the rule existed.
  Without it an author has no instruction on the one split that decides
  whether a lab spoils itself.
- **`6/1`'s neighbours are mechanisms, not products.** `epoll`, `POSIX AIO`,
  and "a blocking thread per connection" are exactly what the ban lists, and
  the neighbour rule publishes names into `README.md`. Every other lab names
  products, where a name alone solves nothing. Here a name is the answer.
- **Languages.** `CLAUDE.md` says "**Rust and C** for phases 6-8". `0/2` is
  the low-level track and it is phase 6 alone; `7/1`, `7/2`, `7/4` and `7/6`
  name Rust, Go and TypeScript, and phase 8 names no language.
- **`6/README.md`** sends hourly-billed dependencies to "phases 4 and 5".
  Phase 5 does not exist.
- **Dropped words.** `2/3` "fault schedules, and are prepared"; `2/4`
  "history and checker are prepared"; `2/5` "fault and schedules are
  prepared".
- **Unmarked solution-bearing citations.** `6/4` marks `malloc_trim(3)` but
  not `mallopt(3)` or the hugepage page, which explain the same quirk; `6/1`
  leaves both `io_uring` pointers unmarked though they state the buffer
  ownership rule the lab is about.

- **Severity:** medium
- **Scope:** cross-cutting; nine specs and four documents
- **Affected:** `CLAUDE.md`, `specs/index.md`, `specs/0/5-shared-scaffold.md`,
  `specs/1/README.md`, `specs/6/README.md`, and the fourteen specs carrying a
  suffixed section eight
- **Source:** four-bucket framing review, 2026-08-28; every claim re-verified
  by grep against the working tree before entry
- **Status:** closed 2026-08-28
- **Fix:** all shipped. Section eight is `Scope` in all 33 specs; the
  publish-split sentence is in all 33; counts corrected in four documents;
  `1/7` wired into `specs/index.md` and `specs/1/README.md`; both dangling
  references cut; `6/1`'s neighbours are now Fluent Bit, Vector and Filebeat,
  with the readiness-versus-completion material moved to marked pointers; the
  language claim scoped to phase 6; dropped words and markers repaired.
  Commits `af59080`, `65035f5`, `d9ce4b3`, `6b08893`, `3f51aaf`.

## S22 — no lab teaches that an action must not outlive its evidence (2026-08-27, approved)

Every lab in the catalog asks its system to keep serving. None asks it to stop.
`7/3` labels finality but never withdraws, `3/1` reports what it could not see
but only observes, and `8/3` withholds fetches but loses nothing by doing so.
No lab holds a system that must void standing output because its own view
became untrustworthy, then resume when trust returns.

Grounded before proposing, not after. SEC Release 34-70694 finds that Knight
"did not have procedures in place to halt SMARS's operations in response to
its own aberrant activity" and "did not have a mechanism to test whether their
systems were relying on stale data". The counter-case is equally documented:
RFC 8767 serves stale DNS rather than fail, AWS static stability keeps the data
plane running through control-plane loss, and the 2015 DynamoDB cascade was
caused by servers disqualifying themselves. Both corners must therefore fail.

The falsified belief, scoped to stay singular: *no action may outlive the
evidence that justified it.* Knight's output-volume bounding is deliberately
excluded — that is a second belief.

### Approved 2026-08-27, in progress

New phase 1 lab at `1/7`. Four named barriers: a gap in the authority's
sequence, a severed session past a counterparty countdown, a process paused
between decision and application, and a heal that the system must resume
within a bounded number of records.

- **Severity:** medium
- **Scope:** phase 1, new lab
- **Affected:** `specs/1/7-*.md` (new), `specs/1/README.md`,
  `specs/index.md`, `docs/lab-selection.md`
- **Source:** grounding pass 2026-08-27; SEC 34-70694, RFC 8767, RFC 9309,
  Binance and Kraken API documentation, Kleppmann on fencing tokens
- **Status:** approved, in progress
- **Fix:**

## S21 — `8/3` rations a budget but breaching it costs nothing (2026-08-27, approved)

`8/3` already enforces a per-host ceiling and a corpus the budget cannot cover,
so "revisiting one page is always the choice not to visit another" is live. Two
things are missing. Breach has no consequence beyond a ledger failure, and the
budget is not contended by different kinds of work.

Both are documented. RFC 6585 defines 429 and `Retry-After`. Google reduces a
site's crawl rate on a significant number of 500, 503 or 429 responses, warns
that sustaining them beyond one to two days harms how the site appears, and
raises the rate again once errors fall. SEC EDGAR publishes a 10 requests per
second ceiling and blocks undeclared automated access outright. RFC 9309 makes
an unreachable `robots.txt` a complete disallow, so error responses poison the
permission to crawl and not merely its throughput.

### Approved 2026-08-27, in progress

Extend `8/3`'s requirements and failure schedule: an escalating consequence for
breach with a recovery path, and one budget contended by content fetches,
`robots.txt` refetches, and post-error retries. This deepens the model the lab
already carries rather than adding a second one.

- **Severity:** low
- **Scope:** phase 8, requirements extension
- **Affected:** `specs/8/3-web-crawl-and-index.md`
- **Source:** grounding pass 2026-08-27; RFC 6585, RFC 9309, Google crawl-rate
  documentation, SEC EDGAR access policy
- **Status:** approved, in progress
- **Fix:**

## S1 — `4/4` may not be falsifiable on the local Lambda emulator (2026-08-14, proposed)

`4/4-subscription-billing-api.md` teaches that the execution environment
freezes at the response, so work started before returning resumes under a
later caller or never runs at all. Its required gates run on the local runtime
emulator. It is unconfirmed whether that emulator reproduces freeze-at-response
or simply leaves the process running, in which case background work completes
locally and the lab's central belief is never falsified. A learner would pass
every local gate with a design that fails in production.

This is the same defect that disqualified the Firestore emulator from the
catalog: an emulator that is more permissive than the service it stands in for
turns a passing local gate into a false model. If it is confirmed, `4/4` needs
either a fault-controller mechanism that freezes the process at the response
barrier, or the lab must move its required gate to the opt-in cloud smoke run,
which breaks the no-account rule for phase 4.

### Feasibility, measured 2026-08-14

The freeze mechanism works in this environment. A container running a loop that
appends a line every 200 ms was frozen with the cgroup freezer through
`docker pause`, held two seconds, and thawed: 8 lines before the freeze, 8
after it, 16 after the thaw. Background work stops and resumes, which is the
behaviour the phase 2 labs depend on. An earlier run of this probe reported
success while producing no output at all, and a second reported failure because
this host's logging driver cannot be read; only the file-observed run counts.

That settles the mechanism and leaves one question: whether the local runner
exposes the moment the runtime and every extension have completed with no
events pending, so the controller knows when to apply it. Freezing at the wrong
instant falsifies a behaviour the service does not have. The remaining work is
a signal, not a mechanism.

### Proposal, needs sign-off

Do not move the required gate to a real account. That breaks the no-account
rule for phase 4, puts credentials in CI where seeded local input is meant to
be the only input, and destroys deterministic falsification: the grader asserts
exact histories at named barriers, and no barrier can be imposed on AWS's own
scheduler.

Reproduce the freeze locally instead. Lambda's freeze is a process freeze, and
process lifecycle with freeze and thaw is already the fault controller's
cheapest mechanism layer. The controller suspends the process at the barrier
and thaws it when the next invocation arrives; killing it instead models an
environment that was destroyed rather than reused.

The barrier is **not** "the handler returned its response", which is what an
earlier draft of this proposal said. The vendor page is precise: Lambda freezes
"when the runtime and each extension have completed and there are no pending
events". A handler can return while an extension is still running, so a
controller built to the response barrier would freeze too early and falsify a
behaviour the service does not have. The barrier is runtime-and-extensions
complete with no pending events, and the same correction applies to every phase
2 spec that says the environment freezes when the response is sent. Background work stops mid-flight, timers return late, and the wall
clock jumps, which are the observable consequences the lab is about.

This makes the emulator's permissiveness irrelevant rather than routing around
it, and `make smoke` keeps its real job: confirming the emulated freeze matched
the service. It is a scaffold change, so it lands in
`specs/0/5-shared-scaffold.md` and needs sign-off before it ships.

Severity rose with the restructure. Four of the five phase 2 labs freeze at
the response barrier in their failure schedules, not just the billing lab; the
import lab (`2/2`) instead depends on the runner's lease and batch contract.
The open question is now also recorded in `specs/0/5-shared-scaffold.md`; the
mechanism proposal below still needs sign-off.

2026-08-14, third review follow-up: `2/1` no longer asserts that the runner
enforces the freeze barrier. Its scaffold section states the lifecycle as a
capability the environment must provide and points at the `0/5` open
question, so no lab claims a capability the scaffold has not established.
The mechanism proposal is unchanged and still needs sign-off.

- **Severity:** high
- **Scope:** phase 2, fault controller
- **Affected:** all of `specs/2/`, `specs/0/5-shared-scaffold.md`
- **Source:** review of phase 1 and phase 4 dependencies, 2026-08-14
- **Status:** open
- **Fix:**
