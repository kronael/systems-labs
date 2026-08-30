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
over the reading. Commit `c6ecc0f`. The rest of `S26` stays open.

2026-08-30: the seventeen-spec link gap is closed. Every neighbour in every
lab spec now carries a documentation URL that answered 200 when it was
written, and the nine bullets that named a category now name a product —
Aurora Serverless v2, Step Functions, Fargate, DynamoDB Streams, PostgreSQL
with a self-run Kafka, Cloud Run, API Gateway caching, Istio, GitHub Pages.
The banned opener left `7/2` and `7/6` in the same commit, `36e7e17`.

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
instead of naming or disclaiming the mechanism. Commits `67c2b49`, `fb3afaa`,
`fe3fbf2`. `7/2:83-85` stands: the sentence states a cost the evidence
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
"around" cannot land at the same boundary twice. Commits `6a1d717`,
`52eebf8`. What remains in `S26` is user-owned: the phase-4 corpus numbers
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
  `S22`'s Affected list also names `specs/0/1-lab-selection.md`, which contains
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
  `specs/0/1-lab-selection.md`, `specs/0/2-low-level-track.md`,
  `specs/0/4-search-and-retrieval-track.md`, `specs/index.md`,
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
  Commits `4f4a67a`, `c2e0da1`, `410e420`, `e7c2264`, `0cbb25a`.

## S23 — nine labs name a domain that does no work (2026-08-28, approved)

The learner should finish a lab knowing something true about its subject
matter, not only about its technology. A framing review of all 33 labs against
that standard found the catalog mostly sound and the failures concentrated:
phases 7 and 8 are grounded throughout, and phase 4's finance monoculture is
earned rather than lazy — one corpus, one generator, one recording pipeline,
with the record shape paid for once in `4/1` so four stores can be compared
against identical data.

Nine labs name a domain whose facts never reach the requirements. The worst is
`1/1`, the first lab a learner meets, which quotes nothing: no product, no
reason a quote expires, no rule deciding which of two quotes is better.

Every fix is a **framing** change — what the product is, what the records
mean, why a limit is that number. No quirk changes, no scale target changes,
no dependency is added, and the budget moves by at most an hour, because a
deepening that adds a parsing chore is the failure this bug exists to avoid.

| lab | today | grounded in |
|-----|-------|-------------|
| `1/1` + `2/5` | a quote of nothing | priced offers that expire; the pair must move together |
| `1/2` | an unnamed resource | a stay, whose half-open interval is the changeover day (RFC 5545) |
| `1/4` | four structural changes | the mandated unit price (Directive 98/6/EC Article 3) |
| `2/2` | "structured records" | interval meter readings, which also gives `2/2` the identity S24 shows it lost |
| `3/2` | routing as scenery | the real churn report's own products (Huston, APNIC) |
| `4/2` | freshness asserted | the venue's own last-entry-uncommitted rule |
| `6/1` | "records" to a stalled consumer | a host log shipper, whose ceiling exists because the memory belongs to the workload |
| `8/1` | homology stripped out | what a match is for (NCBI BLAST) |
| `7/2` | netting unexplained | why settlement nets at all (DTCC) |

Deliberately not taken, to hold the line against overdoing it: a second
`1/4` proposal, `1/3`'s cancellation race, `4/5`'s instrument alias table —
the closest of the three to a parsing chore — and `7/3`, which its reviewer
called droppable.

- **Severity:** low
- **Scope:** nine labs, framing only
- **Affected:** `specs/1/1-*.md`, `specs/1/2-*.md`, `specs/1/4-*.md`,
  `specs/2/2-*.md`, `specs/2/5-*.md`, `specs/3/2-*.md`, `specs/4/2-*.md`,
  `specs/6/1-*.md`, `specs/7/2-*.md`, `specs/8/1-*.md`
- **Source:** four-bucket framing review, 2026-08-28; each citation fetched
  during the review, and the Directive 98/6/EC and RFC 5545 quotes re-fetched
  independently before entry
- **Status:** closed 2026-08-28
- **Fix:** all nine shipped, one sub per lab, every citation fetched by the sub
  and the load-bearing ones re-verified independently before commit. The RFC
  5545 sentence turned out to live in section 3.6.1, not 3.8.2.2 where the
  brief sent it; the sub found that and was right. Commits `cd11dba`,
  `0e30c3b`, `e7c2264`.

  The standard this bug applied was not written down anywhere. It is now
  `CLAUDE.md`'s domain-grounding paragraph and step 4 of `Adding a lab`.

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
  `specs/index.md`, `specs/0/1-lab-selection.md`
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

## ✅ FIXED 2026-08-23 — S20 — phase 1 misses two dimensions, and `1/2` and `1/4` overlap (2026-08-23, fixed)

Signed off and shipped in two parts. The merge landed first: `1/4` folded into
`1/2`, whose falsified belief is now that a lease is not a lock, and `2/2` kept
the model contrast instead of a product pairing. The two missing dimensions
then landed as new labs on the numbers the merge freed —
`specs/1/4-uninterrupted-catalog-service.md` for change against live traffic,
and `specs/1/6-shared-budget-service.md` for work the store declines to
complete. Both are PostgreSQL-only, both quirk pages were fetched and quoted
again before being cited, and both citations are marked solution-bearing.
`1/4` also cites the explicit-locking page, because `ALTER TABLE` alone does
not establish that a waiting change stops the readers queued behind it.

The risk the proposal named survived scrutiny. `1/6` sits beside `1/2` and
stays distinct because the two ask opposite questions of the same store.
`1/2`'s invariant is one the store can be made to refuse at write time, so its
lesson is what a delivery lease does not promise. `1/6`'s invariant is an
aggregate over a set that no single write can be judged against, so the store's
answer under concurrency is a refusal to complete the unit of work, and
everything after that refusal — the boundary, the bound on attempts, the answer
the client already holds — belongs to the application. `1/6` runs no broker and
has no asynchronous work at all.

Phase 1 now holds six labs and the core catalog seventeen. `specs/index.md` was
renumbered 01 to 17, the per-lab source map in `01-systems-labs.md` followed
it, and the phase 1 README's delivery-model and JetStream prose was corrected
to what the merge left behind.

The original entry is kept below.

Phase 1's five labs separate cleanly on one axis — who owns progress, and what
happens when it is lost. `1/1` has no durable state, `1/2` has a notification
nobody retains, `1/3` a consumer-owned position, `1/4` a broker-owned lease,
`1/5` a gap neither system can close alone.

Two dimensions are absent, and this was checked rather than recalled.

**Change over time.** No lab in the sixteen makes "the shape changed while the
system was running" its subject. Every lab is built once and never altered.
`grep` for schema change or migration across phases 1 to 4 returns `3/2`, where
it is one pressure among several in a Flink job, and one hit in `1/2` that is
scaffold plumbing rather than a lesson.

**Contention the database hands back.** The technology spine advertises "MVCC
and isolation, locks, deadlocks, serialization failures". `grep` for isolation
level, serialization failure, and deadlock across phases 1 to 4 returns
**nothing**. `1/2` has contention, but there the database enforces the
invariant; no lab teaches the opposite case, where the database refuses the
transaction and returns the problem to the application.

**`1/2` and `1/4` are the phase's weakest orthogonality claim.** Both read
"accept work, do it later, survive restart". They fail in opposite directions —
`1/2` risks losing work, `1/4` risks doubling it — but the products are close.

### Proposal, needs sign-off

**Merge `1/2` and `1/4` into one lab: reservation fulfillment whose fulfillment
work is leased.** One product, PostgreSQL plus the self-run broker, and one
falsified belief: *a lease is not a lock*. The lease expires while a worker
still holds a half-finished reservation, and the duplicate meets a database
invariant that refuses it. That belief needs both systems, so the second
dependency is earned rather than stacked.

A naive merge is wrong. Phase 1's spine is three delivery models — a
notification that never replays, a retained log, a lease on a timer — and
folding `1/4` into `1/2` deletes one of the three unless the merged lab keeps
the lease. It does. What the merge costs is `LISTEN`/`NOTIFY` as a required
mechanism, which leaves phase 1 entirely.

**Add two labs on the missing dimensions.** Both are local, both use a
dependency phase 1 already runs, and both quirks are documented.

- *Change against live traffic.* A shape change lands while the system serves
  requests, with old and new records coexisting and no window where either
  fails. Source, fetched 2026-08-23:
  [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html) —
  "An `ACCESS EXCLUSIVE` lock is acquired unless explicitly noted", and
  "Adding a column with a volatile `DEFAULT` ... will cause the entire table
  and its indexes to be rewritten." Solution-bearing: the same page lists the
  subforms taking weaker locks, so it belongs in `HINTS.md`.
- *Refused transactions.* The product stays correct under concurrent
  conflicting writes that the database declines to serialize, and the learner
  owns the retry boundary. Source, fetched 2026-08-23:
  [Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
  — "ERROR: could not serialize access due to read/write dependencies among
  transactions", and "When an application receives this error message, it
  should abort the current transaction and retry the whole transaction from
  the beginning." Solution-bearing: that second sentence is the answer, so it
  belongs in `HINTS.md`.

Result is six phase 1 labs: admission, lease-versus-lock, retained log,
cross-system gap, change against live traffic, refused transactions.

### Risk, not papered over

The refused-transactions lab sits close to the merged `1/2`: both put
concurrent conflicting writes against PostgreSQL. The distinction is that one
lets the database enforce an invariant and the other makes the application
handle a refusal. That is real but thin, and it is the part of this proposal
most likely to be wrong. If it does not survive scrutiny, the change-over-time
lab stands on its own and the phase lands at five.

- **Severity:** medium
- **Scope:** phase 1, curriculum structure
- **Affected:** `specs/1/2-reservation-fulfillment.md`,
  `specs/1/4-reliable-record-import.md`, `specs/1/README.md`,
  `specs/index.md`, `specs/01-systems-labs.md`,
  `specs/0/6-serverless-contrast-track.md` (the `1/4`↔`2/2` pairing)
- **Source:** orthogonality review of phase 1, 2026-08-23; both quirk sources
  fetched and quoted the same day
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## ✅ FIXED 2026-08-23 — S19 — the checks lost their independent producer (2026-08-23, fixed)

Resolved by `EVALUATION.md`. Four labs check by comparing against an
independently computed answer — `2/1` invoice lines, `6/5` a latency
distribution, `7/2` a balance replay, `8/5` recomputed scores — and none of
those is once, order, survives, never, or bounded. With no grader and only a
verification skill bound by the task's ban, an agent would have had to derive
that answer from the learner's own implementation: a grader reconstituted per
run, unaudited, different every time.

`EVALUATION.md` is the answer key and is solution-bearing by choice, so it
simply carries the independently computed result. The owner's framing settles
it: the file is protected the way `HINTS.md` is — a learner who wants the
exercise does not open it — and by nothing else.

The original entry is kept below.

Four labs require the check to derive the answer
itself rather than assert a shape over a history. `2/1` compares "every invoice
line against an independently computed answer derived from the acknowledged
input under the supplied rating rules"; `6/5` requires the reported percentiles
to match an independently computed distribution; `7/2` replays the accepted
settlement stream into an independent balance model; `8/5` recomputes every
reported score from the accepted inputs named in the report's lineage. None of
those is `once`, `order`, `survives`, `never`, or `bounded`. An agent asked to
do them at the stated scale will write throwaway checking code, most cheaply by
reusing the learner's own implementation — which reproduces a grader, once per
run, unaudited. This is the load-bearing thing the removal cut.

**Coverage.** A verify skill can omit a lab's hardest requirement and still read
as complete. The grader could not diverge from what a lab declared, because the
declaration and the checkers were the same artifact. `make teaching-lint` scans
a skill for leaks; nothing compares it to the `README.md` it is supposed to
check.

**Regression.** `grader/histories/` held hand-written accept and reject fixtures
per checker — the thing that proved a check discriminates without any design
existing. It went with the grader. `template/` proves the targets run against
one trivial domain; nothing proves a lab's check rejects a run that should
fail. CI now demonstrates that the scaffold executes, not that it decides.

**Fault proof.** The frozen aggregate digest proves a recipe has not drifted. It
does not prove the barrier was reached or the effect applied. A run whose fault
silently never fired is indistinguishable from one the design survived, and the
grader's exact-history assertion was what separated them.

**The performance bar.** The verification contract permits a learner-recorded
baseline or a learner-declared SLO, and nothing arbitrates a deliberately slow
baseline or a trivial SLO. This one predates the removal; the removal took away
the party that would have caught it.

### Proposal, needs sign-off

- Keep course-owned expected-value producers for the four labs whose contract is
  arithmetic over accepted input, as declarative oracle data or a seeded
  generator, and state plainly that they are not a worked design: they compute
  the product rule, never the architecture that must satisfy it.
- Give every public invariant a stable identifier in the lab spec, publish those
  identifiers in `README.md`, and require the verify skill to carry the same
  set. CI asserts set equality — a syntactic check, but it catches the omission.
- Restore hand-authored accept and reject histories per invariant shape, outside
  any lab directory. They are fixtures for the check, not implementations of the
  lab, and `S16`'s reasoning against worked solutions does not reach them.
- Make the controller fail the run when a barrier is not reached or an effect
  not applied, and record requested, applied, and observed for each.
- Say who owns the performance bar, or drop the learner-declared SLO as a gate
  form.

- **Severity:** high
- **Scope:** verification, shared scaffold, fault injection, CI, per-lab authoring
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`,
  `specs/2/1-metered-billing-api.md`, `specs/6/5-rate-accurate-replayer.md`,
  `specs/7/2-settlement-program-and-client.md`,
  `specs/8/5-relevance-evaluation-service.md`
- **Source:** codex critique of the restructure, verified against the files,
  2026-08-23
- **Status:** proposed
- **Fix:**

## ✅ FIXED 2026-08-24 — S18 — the history vocabulary cannot express two of its own five shapes (2026-08-23, fixed)

Resolved by cutting the claim rather than building the schema. The finding was
correct on every point: the record had no field for `never` or `bounded` to
range over, "ordered" had no cross-emitter rule, nothing produced the history at
the boundaries that matter, and build-order item one had no directory while
every other item did.

The proposal answered it with a versioned event schema, clock domains, a merge
rule, observation adapters, and a classification of all 31 acceptance contracts.
That is machinery for a compiled checker, and `S16` and the grader removal
deleted the checker. Building it now would be the over-engineering the owner
ruled out.

What the scaffold owes is vocabulary, not a data format. The five shapes stay as
words an answer key uses. Each ranges over whatever that lab's own observable
boundaries expose, so `never` and `bounded` need no record field. Where a check
depends on ordering across processes, the lab states which ordering it means and
over what scope — the spec now says plainly that no clock is comparable across
the fault controller, the application, and a restart, so only the lab knows.
Where a contract is none of the five, the answer key carries the computation
directly; the five are a convenience for common cases, never a ceiling. Build
order item one is now a page of prose that ships no directory.

The original entry is kept below.

`0/5` makes the history the one thing the scaffold still owes every lab, and the
build order makes it item 1 "because every other component and every lab spec
depends on it". As specified it does not hold.

**The record has no room for two of the shapes.** An event carries "a record
identity, a boundary name, a wall-clock and a monotonic timestamp, and the
observation site". `never` is defined over "a forbidden state" and `bounded`
over "a quantity"; the record has no state, no outcome, no value, and no unit.
Two of five shapes have nothing to range over.

**"Ordered" has no cross-process rule.** The history is called an ordered log and
timestamped with a monotonic clock. Monotonic clocks are not comparable across
the fault controller, the evidence writer, the application processes, a second
host, or a restart — which is exactly the set of emitters a lab has, and exactly
the boundary `order` and `survives` are asserted across. Nothing gives a run
identifier, an emitter identifier, a clock domain, a per-emitter sequence, or a
merge rule.

**Nothing produces it at the boundaries that matter.** `0/5` says the fault
controller and the evidence writer emit histories. The `Verification` section
names the observable boundaries as HTTP responses, SQL state, a provider's
request log, consumer positions, and the evidence report. No component turns
those five into history events, and `Planned repository boundaries` has no entry
for a history at all — every other build-order item has a directory
(`shared/workload/`, `shared/faults/`, `shared/telemetry/`, `template/`); item
one has none.

**The five shapes are admitted to be incomplete.** `0/5` says they cover "almost
every" lab invariant and leaves phases 7 and 8 as an open question, while `S19`
lists four current labs whose contract none of the five expresses. Building the
vocabulary before classifying all 31 acceptance contracts against it will build
the wrong one.

### Proposal, needs sign-off

- Specify a versioned event schema: run identifier, emitter identifier,
  per-emitter sequence, clock domain, boundary name, record identity, event type
  and outcome, and typed observed values with units. Define the merge rule that
  makes a multi-emitter history ordered, and say plainly where it is only
  partially ordered.
- Name `systems-labs/shared/history/` in `Planned repository boundaries` as its
  home, and name the course-owned observation adapters that emit from each of
  the five boundaries.
- Classify all 31 labs' acceptance contracts against the five shapes before the
  vocabulary is built, and add the shapes the classification demands rather than
  the ones that read well.

- **Severity:** high
- **Scope:** shared scaffold, verification, repository boundaries
- **Affected:** `specs/0/5-shared-scaffold.md`, `specs/01-systems-labs.md`
- **Source:** codex critique of the restructure, verified against the files,
  2026-08-23
- **Status:** proposed
- **Fix:**

## ✅ FIXED 2026-08-24 — S17 — lab and phase files still describe the removed grader (2026-08-23, fixed)

Resolved across every phase. The count was also wrong in scope: the entry read
as phases 3 to 8, but phases 1 and 2 carried twenty-seven references of their
own that the grader removal never touched.

The fix was not a rename. "Grader" did the fault controller's job in one
sentence and the check's in the next, so each was read and given the actor that
performs it: the failure schedule injects, kills, freezes and replays; checks
observe, compare and do not require. Prepared-scaffold lists no longer promise a
grader binary, `make grade` is gone from `8/2`, and the four track records under
`0/` are clean.

The original entry is kept below.

The restructure removed the grader binary, `make grade`, the per-lab `grader/`
directory, and `shared/grader/` from every governing contract. The lab specs
were not touched. Every `Prepared scaffold` section still lists "the black-box
grader" among the components the course supplies; every `Adversarial evaluation`
section still narrates its schedule as "the grader freezes…", "the grader
replays…"; `specs/8/2` still cites `make grade` as the target that "reviews
whether the policy itself is defensible", which the governing spec now says does
not exist; and `specs/index.md`'s row 05 still names a "history checker" as lab
05's prepared environment. 122 occurrences on 121 lines across 35 files under
`specs/1` to `specs/8`, plus four catalogs under `specs/0/` and the core
catalog row itself.

The fix is not a rename. "Grader" does two jobs in those sections and the jobs
now have different owners: where it drives a fault or holds a rate it is the
fault controller and the workload generator, both still compiled; where it
observes and asserts it is that lab's verify skill, which is prose, may not name
a barrier, and after this review may not name a record identity either. A single
substitution would flatten the distinction and hand every lab a component that
no longer exists. `8/2` needs a decision rather than a substitution, and the four
labs in `S19` need an expected-value producer before their sections can be
rewritten at all.

Recorded rather than changed, matching `S11`'s and `S14`'s treatment of the same
kind of drift: the reviewed scope was the governing documents, and this is 35
files of per-lab prose.

- **Severity:** high
- **Scope:** all lab specs, phase READMEs, track catalogs, core catalog
- **Affected:** `specs/1/1`–`1/5`, `2/1`–`2/5`, `3/1`–`3/2`, `4/1`–`4/3`, `4/5`,
  `6/1`–`6/5`, `7/1`–`7/4`, `7/6`, `8/1`–`8/5`, the phase READMEs of 4, 6, 7,
  and 8, `specs/0/1`, `0/2`, `0/3`, `0/6`, and `specs/index.md`
- **Source:** refinement pass and codex critique after the grader removal,
  2026-08-23
- **Status:** open
- **Fix:**

## ✅ FIXED 2026-08-23 — S16 — S3's resolution is superseded: no lab has a worked solution (2026-08-23, fixed)

`S3` moved each lab's `golden/` and `rotten/` out of the lab directory to a
repository-root `reference/` tree excluded by the publish step. That was
deterrence, and it left the underlying problem: a worked systems design answers
every `Architecture questions` bullet at once.

The owner's position closes it properly — labs have no worked solution at all.
The reasoning is that a challenge has one correct answer, so a golden reference
is well defined, while a systems lab admits many correct designs, so no
implementation is canonical and "rotten" is one of countless ways to be wrong.
`golden/` was never load-bearing here as it is in `challenges/`, where
`make bench` runs it as the oracle for expected output; this grader asserts
invariants over observed histories and needs no oracle.

Two things replace it. The cited source documenting the real reported behaviour
is the lab's ground truth. `grader/histories/` holds synthetic accept and reject
fixtures per checker, written by hand against the history model and never
recorded from a fault run, which prove the grader works without any design
existing. `template/` keeps a working implementation because it is the
scaffold's regression test and not a lab.

**Superseded in part, 2026-08-23.** The verification restructure removed the
grader itself, and `grader/histories/` went with it, so this entry's second
replacement no longer exists. Nothing now proves a check discriminates before a
design exists; that gap is `S19`.

- **Severity:** high
- **Scope:** repository contract, shared scaffold, per-lab authoring
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`,
  `CLAUDE.md`
- **Source:** owner decision, 2026-08-23
- **Status:** fixed
- **Fix:** `reference/` removed from the repository contract, planned
  boundaries, licence, teaching, and fault contracts; `grader/histories/` added
  to the per-lab shape; `0/5` template section rewritten; `CLAUDE.md` golden
  rule replaced, 2026-08-23

## ✅ FIXED 2026-08-24 — S15 — nothing enforces the earned-dependency rule (2026-08-23, fixed)

Closed by decision rather than by machinery. `S16` removed worked
implementations and the grader removal deleted the checker, so there is no
simple design left to run and time — the executable route `challenges/` uses is
gone, and rebuilding one would be exactly the checking machinery the owner ruled
out with "the task carries the lab, not the grader".

The rule therefore stands as an authoring check, and both governing documents
now carry it. `CLAUDE.md`'s Adding a lab procedure makes it step 3: name the
scale target, then state what a single-process single-store design would fail
at. If no specific sentence can be written, the dependency is unearned and
either the scale target rises or the dependency goes. Each lab's `Scope` carries
that sentence; `1/2` and `1/6` are the worked examples.

This is honestly weaker than a gate, and the entry is closed saying so rather
than claiming otherwise.

The original entry is kept below.

`01-systems-labs.md` states that if the naive small tool would pass the same
gates at the same scale, the lab has not earned its dependency and the scale
target is too low. Nothing checks it. `make teaching-lint` catches solution
leaks in learner-facing text; CI proves the scaffold's targets run end to end
against `template/`. Neither establishes that a single-process, single-store
design would fail this lab's scale target, so the rule is an author's promise
rather than a gate.

`challenges/` solved the same problem executably: `rotten/` must pass the small
suite and time out on every generated large case, and `make sys-rotten` enforces
that contract at the root. That route is closed here — `S16` removed worked
implementations from labs, so there is no simple design to run and time.

What remains is an authoring check rather than a gate: each lab spec states,
in `Scope`, what a single-process single-store design would fail at this scale
target, and a reviewer confirms the claim is specific enough to be wrong. If
no such sentence can be written, the dependency is unearned. Whether that is
enough, or whether the rule should be dropped rather than left unenforceable,
is the open question.

- **Severity:** medium
- **Scope:** verification contract, shared scaffold, per-lab authoring
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`,
  `CLAUDE.md`
- **Source:** review of the gates against the `challenges/` model, 2026-08-23
- **Status:** open
- **Fix:**

## ✅ FIXED 2026-08-24 — S14 — phase 3–8 briefs still carry disclaimed-mechanism enumerations (2026-08-14, fixed)

Resolved in all twenty-one phase 3 to 8 briefs. Each "does not prescribe …"
enumeration is replaced with a statement of the categories of decision that lab
leaves to the learner, naming no candidate under any of them. It mattered most
in phase 6, where the kernel interface names are themselves the answers.

One borderline case was caught rather than shipped: `3/1` replaced its
enumeration with "materializer topology", which names a mechanism shape rather
than a category, and is now the process decomposition.

The original entry is kept below.

The S12 rule — a brief states the categories of decision the learner owns and
names no candidate mechanism even to exclude it — was applied to the ten
phase 1 and 2 briefs the fourth review covered. Every phase 3–8 lab brief
still ends with a "does not prescribe" enumeration of candidate mechanisms —
3/1, 3/2, 4/1, 4/2, 4/3, 4/5, 6/1–6/5, 7/1–7/4, 7/6, and 8/1–8/5 — which the
amended brief contract and teaching lint now forbid. Same reasoning, outside
the reviewed scope, so the drift is recorded here rather than changed,
matching S11's treatment of the hour budgets.

- **Severity:** medium
- **Scope:** phases 3–8, brief contract
- **Affected:** all lab specs under `specs/3/`, `specs/4/`, `specs/6/`,
  `specs/7/`, `specs/8/`
- **Source:** fourth external review's S12 applied 2026-08-14, same reasoning
  extended
- **Status:** open
- **Fix:**

## ✅ FIXED 2026-08-14 — S12 — learner-facing briefs name solution methods the teaching contract forbids (2026-08-14, fixed)

Resolved across all ten phase 1 and 2 briefs and both governing documents.
Each brief's "does not prescribe" enumeration is replaced with a statement of
the decision categories that lab leaves to the learner — concurrency and
admission for `1/1`, data layout and the fulfillment path for `1/2`, event
layout and rebuild for `1/3`, and so on — naming no candidate mechanism. The
brief contract in `01-systems-labs.md` now says plainly that a disclaimed
mechanism is still a named mechanism and that a brief states categories only.
`CLAUDE.md`'s spec-conventions rule reversed from requiring the enumeration
to forbidding it. `make teaching-lint` (in `01` and `0/5`) now fails on a
mechanism name in any learner-facing artifact whether it is prescribed,
disclaimed, or merely mentioned, against one curriculum-wide mechanism
vocabulary. The phase READMEs carry no such enumerations and are marked
author-facing; `0/6` keeps its mechanism names as deliberate author-facing
pairing analysis and now opens by saying so. The same pattern in phases 3–8
is outside the reviewed scope and recorded as S14.

The original entry is kept below.

The governing spec says a lab brief never names the mechanism that solves it,
and the repository contract says learner-facing task text may not name,
describe, compare, or rule out a solution method. The lab `Brief` sections
systematically do exactly that in their "does not prescribe" clauses: worker
pools, semaphores, schemas, partition keys, offset policies, relays, cache
policies, retry rules, and similar implementation shapes. `CLAUDE.md` also
requires these enumerations, so the authoring rule contradicts the governing
teaching contract rather than merely violating it in one lab. If `Brief`
publishes into `README.md`, the task leaks the design vocabulary; if that
sentence is stripped, the publication mapping is unspecified.

- **Severity:** high
- **Scope:** teaching contract, phases 1 and 2
- **Affected:** `CLAUDE.md`, `specs/01-systems-labs.md`, all ten lab specs in
  `specs/1/` and `specs/2/`
- **Source:** `specs/01-systems-labs.md:127`,
  `specs/01-systems-labs.md:584`, `CLAUDE.md:189`, and each lab's `Brief`
- **Status:** fixed
- **Fix:** in-tree edits to all ten briefs under `specs/1/` and `specs/2/`,
  `specs/01-systems-labs.md` (brief contract, teaching-lint), `CLAUDE.md`
  (spec conventions, verification contract), `specs/0/5-shared-scaffold.md`
  (teaching lint), and `specs/0/6-serverless-contrast-track.md`
  (author-facing marker), 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S13 — GPL distribution contract withholds the controller's corresponding source (2026-08-14, fixed)

Resolved by specifying the source route rather than reopening the learner
tree. `01-systems-labs.md` gained a "Licence and corresponding source"
section: the source repository — recipe sources, `reference/`, and `specs/`
included — is public under GPL-3.0 in its entirety, so the publish step's
exclusions decide which tree a file lands in, never whether it is published;
the learner distribution carries the GPLv3 text in a root `LICENSE` and, in
the same place, names the source repository as the no-charge
equivalent-access route to the complete corresponding source of the compiled
fault controller. The fault-injection and teaching contracts and `0/5`'s
`shared/faults/` now state excluded-is-not-withheld and add fetching the
source repository as a third deliberate act beside disassembly and mid-run
reads, replacing "the readable-source path is closed". The planned
repository boundaries gained `systems-labs/LICENSE`. The root `LICENSE` file
itself is still an artifact to create, alongside the promised root `README`,
`NOTICE`, and `THIRD_PARTY.md`.

The original entry is kept below.

The repository declares itself GPL-3.0, but the planned learner distribution
conveys a compiled Go fault controller while excluding the seeded recipe
sources compiled into that binary and says the readable-source path is closed.
GPLv3 section 6 requires a conveyed object-code work to be accompanied by, or
offered equivalent access to, its machine-readable Corresponding Source; the
recipe inputs are part of the preferred form needed to generate and modify the
controller. The current tree also declares GPL-3.0 without carrying a root
copy of the license. The default learner tree may omit recipes for accidental-
disclosure deterrence, but recipients must have a clear GPL-compliant route to
the complete source, including those recipes.

- **Severity:** high
- **Scope:** licensing, learner publish step, fault controller
- **Affected:** `specs/01-systems-labs.md`,
  `specs/0/5-shared-scaffold.md`, planned root licensing files
- **Source:** `specs/01-systems-labs.md:26`,
  `specs/01-systems-labs.md:198`, `specs/0/5-shared-scaffold.md:105`, GNU
  GPLv3 sections 1 and 6
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` (licence and
  corresponding source, teaching contract, fault injection contract, planned
  repository boundaries) and `specs/0/5-shared-scaffold.md`
  (`shared/faults/`), 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S7 — `2/1` states an invariant its own schedule no longer falsifies (2026-08-14, fixed)

Resolved by the first option: the redelivery scenario returned to `2/1`'s
adversarial schedule as an environment fact rather than a subject. The grader
now redelivers the queue message carrying usage event 771003 after its work is
acknowledged, with the schedule stating in place that the queue's delivery
contract is another lab's subject and the redelivery is an environment fact
the invariant must survive. The Requirements' no-doubled-charge clause is
falsified again without making the delivery contract this lab's study.

Narrowing `2/1` to one primary model removed its redelivery fault scenario,
because the queue's delivery contract is `2/2`'s subject rather than this
lab's. Its Requirements still say a redelivered message may not double a
charge, so the lab now asserts a property nothing in its own adversarial
schedule tests. Either the scenario returns as an environment fact rather than
a subject, or the invariant moves to where it is falsified.

- **Severity:** medium
- **Scope:** phase 2
- **Affected:** `specs/2/1-metered-billing-api.md`
- **Source:** external review follow-up, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edit to `specs/2/1-metered-billing-api.md` Adversarial
  evaluation, 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-24 — S8 — the concurrency-ceiling standard is applied inconsistently (2026-08-14, fixed)

Resolved by stating the rule the two cases were already following. The scale
target contract in `01-systems-labs.md` now says the lab fixes an admission
ceiling when the ceiling is an environment fact needed to guarantee the regime —
the offered rate must provably exceed what is admitted, and a learner free to
raise it could make the pressure disappear — and the learner declares it when
choosing it is the design decision the lab is about, with the admitted fraction
required in evidence so a ceiling chosen to dodge the problem is visible. `2/3`
and `2/4` are the first case, `2/5` the second. A phase may hold both; what it
may not hold is two labs of the same kind disagreeing.

The original entry is kept below.

`2/3` and `2/4` now fix a concurrency ceiling in lab configuration, 32 and 64
environments, so the offered rate provably exceeds what the platform admits.
`2/5` keeps a learner-declared ceiling with only a relational sizing rule. The
argument for the exception is that `2/5`'s brief makes the declared ceiling a
design surface, and the admitted-fraction evidence requirement guards the
gaming risk. That may be right, but one phase should not carry two standards
without saying which applies when.

- **Severity:** low
- **Scope:** phase 2, scale-target contract
- **Affected:** `specs/2/5-serverless-quote-aggregation.md`, `specs/01-systems-labs.md`
- **Source:** external review follow-up, 2026-08-14
- **Status:** open
- **Fix:**

## S9 — cut `2/5` or fold its unique content elsewhere (2026-08-14, proposed)

Both external reviewers ranked `2/5` last and proposed cutting it. Admission
control — the local quote lab's central subject — is confiscated by the
platform, so the learner's buildable artifact reduces to one handler plus a
declared ceiling. Its distinctive lessons — throttling above the ceiling,
cold-start cost, state outside the process — are already exercised by `2/1`,
`2/3`, and `2/4`. Its one unique item is fan-out amplification per instance:
provider-side load multiplied by environment count rather than shared through
a process-wide pool, which fits elsewhere as a required evidence number.

### Proposal, needs sign-off

Cut `2/5`, or fold the fan-out-amplification measurement into another phase 2
lab as a required evidence number, and record the cut in the serverless
contrast track. Cutting a lab is a curriculum decision; nothing moves until
the user decides.

- **Severity:** medium
- **Scope:** phase 2, curriculum structure
- **Affected:** `specs/2/5-serverless-quote-aggregation.md`,
  `specs/2/README.md`, `specs/0/6-serverless-contrast-track.md`,
  `specs/01-systems-labs.md`
- **Source:** two external reviews, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## S10 — `2/1` opens the phase with a two-variable jump (2026-08-14, proposed)

`2/1` is the phase's largest lab — its honest budget is fifteen to twenty-five
hours — and it introduces the execution model and an unfamiliar product at
once. That is exactly the two-variable jump the phase's own ordering rationale
("the recasts that follow vary one thing rather than two") exists to avoid.
Both external reviewers flagged the opening slot.

### Proposal, needs sign-off

Move `2/1` out of the opening slot: open the phase with a recast whose product
the learner already owns, and let the metered billing lab land once the
execution model is familiar. Reordering labs is a curriculum decision; nothing
moves until the user decides.

- **Severity:** medium
- **Scope:** phase 2, lab ordering
- **Affected:** `specs/2/README.md`, `specs/2/1-metered-billing-api.md`,
  `specs/0/6-serverless-contrast-track.md`
- **Source:** two external reviews, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
- **Fix:**

## ✅ FIXED 2026-08-24 — S11 — phase 3–8 time budgets still price the redesign loop at zero (2026-08-14, fixed)

Resolved in all twenty-one specs and in the selection record, whose criterion
still read "three to eight focused hours" and now reads six to twenty-five.

Figures are sized to real difficulty rather than scaled uniformly. `7/6` lands
highest at twenty to twenty-five hours for an on-chain program plus a four-node
network and key custody; `4/5` at eighteen to twenty-four for two full
deployments of one domain contract; `6/3` at sixteen to twenty-two. The old
numbers summed the three gates and charged nothing for the falsify-and-rebuild
loop, which is where most of the hours actually go.

The original entry is kept below.

The phase 1 and 2 Scope budgets were restated honestly on 2026-08-14: the old
figures summed the three gates and priced the falsify-and-rebuild loop —
the pedagogy itself — at nothing, and were short by roughly a factor of two.
Every phase 3–8 lab still carries a gate-sum figure (four to twelve declared
hours), and `specs/0/1-lab-selection.md` still states "fits into three to
eight focused hours" as a selection criterion. The same reasoning applies. The
master's corrected total (150 to 250 focused hours for the sixteen core labs)
anticipates the correction; restating the phase 3–8 figures was outside the
reviewed scope, so the drift is recorded here rather than changed.

- **Severity:** low
- **Scope:** phases 3–8, selection record
- **Affected:** `specs/3/`, `specs/4/`, `specs/6/`, `specs/7/`, `specs/8/`,
  `specs/0/1-lab-selection.md`
- **Source:** external review of phases 1–2 applied 2026-08-14, same reasoning
  extended
- **Status:** open
- **Fix:**

## ✅ FIXED 2026-08-14 — S3 — the worked solution ships inside the lab directory (2026-08-14, fixed)

Resolved as proposed. `golden/` and `rotten/` moved out of the per-lab tree to
a repository-root `reference/<lab>/` pair, excluded from the learner
distribution by an explicit publish step — the same step that excludes
`specs/`. The waiver sentence is replaced with the `challenges/` position: the
worked solution is never shown to the solver. The proposed per-lab shape
dropped both directories, and `0/5`'s `template/` now proves the passing and
the failing reference against `reference/template/`, so CI stops enforcing
the leaky layout while still checking both.

The original entry is kept below.

`specs/01-systems-labs.md` places `golden/` at lab top level, beside the
learner's `app/`, and says in as many words that public availability does not
reduce the learner's required authorship. For an algorithmic exercise that
position is survivable — the golden answer is fifteen lines. For a systems lab
it is not: a golden submission is a complete worked design, which answers every
`Architecture questions` bullet at once. `rotten/` compounds it, because
diffing the two yields the fix and names which contract is the trap.

`specs/0/5-shared-scaffold.md` locks the shape into CI by requiring `template/`
to ship both.

The sibling `challenges/` repository is stricter than this on a far smaller
risk: its own guide says the golden solution is never shown to the solver.

### Proposal, needs sign-off

Move `golden/` and `rotten/` out of the lab directory to a repository-root
`reference/` tree, omitted from the learner distribution by an explicit publish
step. Replace the waiver sentence with the `challenges/` position. Update the
proposed per-lab shape and the `template/` requirement so CI stops enforcing
the leaky layout.

- **Severity:** high
- **Scope:** repository contract, shared scaffold
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` (repository contract,
  per-lab shape, planned boundaries) and `specs/0/5-shared-scaffold.md`
  (`template/`), 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S4 — the failure schedule ships as readable files (2026-08-14, fixed)

Strengthened 2026-08-14 after the third external review, which judged the
first fix nominal: the seeds and generator parameters still shipped readable
under `shared/faults/`, so the schedule was reconstructable from source. The
recipes are now compiled into the Go fault controller binary, the recipe
sources are excluded from the learner distribution by the same publish step
that excludes `reference/` and `specs/`, and what the learner receives is an
executable that produces the schedule rather than a source describing it.
`make fault` is unchanged; the standard remains deterrence rather than
impossibility — disassembling the controller is a deliberate act — but the
readable-source path is closed. Stated in the teaching, fault injection, and
repository contracts of `01-systems-labs.md` and in `0/5`'s `shared/faults/`.

The first resolution is kept below.

Resolved as proposed. The fault schedules are seeded recipes held with the
shared fault controller outside the lab directory, under solution-neutral
names. `make fault` — exactly as available to the learner as before —
materializes the schedule it runs, runs it, and removes it, so the readable
form exists only while a run is in flight. A frozen aggregate digest over
every materialized schedule is checked under `make test-all`, adding no Make
target. Stated in the teaching, fault injection, and repository contracts of
`01-systems-labs.md` and in `0/5`'s `shared/faults/`.

The original entry is kept below.

`scenarios/` holds declarative fault schedules whose triggers are named
barriers, and a barrier name is an edge case stated in words. The master spec
concedes the problem rather than solving it: reading them early is possible,
nothing prevents it, and nothing advertises it. A learner who reads them before
designing has the failure schedule, which the teaching contract itself equates
to being handed the design. The files must be present, because `make fault` is
the learner's own command.

`challenges/` already solved this exact problem and systems-labs has not
adopted the mechanism. Its adversarial fixtures are gitignored; the recipes
live at repository root under neutral names; `make bench` materializes one case
under `tmp/`, runs it, and unlinks it; and an aggregate digest stops the
recipes drifting.

### Proposal, needs sign-off

Adopt that mechanism for `scenarios/`: seeded recipes at repository root, one
schedule materialized per run and removed afterwards, with a digest check.
`make fault` stays exactly as available to the learner; the schedule stops
being a readable artifact.

- **Severity:** high
- **Scope:** teaching contract, fault controller
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` and
  `specs/0/5-shared-scaffold.md`, 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S5 — the task text states the quirk it is meant to hide (2026-08-14, fixed)

Resolved as proposed. The `challenges/` ban list is imported into the
repository contract's `README.md` bullet with its categories quoted verbatim,
including the clause that input limits are stated without explaining how an
approach behaves at those limits. The four announced quirks are stripped to
their properties: `1/2` lost the read-then-write sentence outright, `1/4`'s
timer-lease sentence became plain at-least-once delivery, `2/3`'s
primary-key-condition sentence is gone, and `2/4`'s atomicity-propagation
sentence became an order-and-grouping robustness requirement. The teaching
contract now states where `Architecture questions` publishes — property-level
questions into `README.md`, mechanism-presupposing ones into `HINTS.md` — and
all ten phase 1 and 2 labs were reviewed against the split: `HINTS.md`-bound
questions marked in `1/2`, `1/4`, `2/4`; property rewrites in `1/1`, `1/3`,
`2/1`, `2/3`; the questions in `1/5`, `2/2`, and `2/5` already name only
properties or the announced environment.

The original entry is kept below.

`Requirements` is README content by the repository contract, and in four labs it
announces the falsified belief outright: that change records from one atomic
write may interleave so the consumer must not assume atomicity; that conditions
apply to items named by primary key and transactions cannot be conditioned on a
query; that the acknowledgement window expires on a timer so a slow consumer is
handed a duplicate; and that a design which reads then writes must explain the
interleaving. The last one names the naive approach and its failure, which
`challenges/` bans explicitly.

`Architecture questions` compounds it. The section is learner-facing by
construction, and several of its bullets are leading questions that presuppose
the answer — how startup avoids a missed-change window, how workers establish
ownership, where an exhausted record lives, where the progress of a period
close lives. The teaching contract lists four artifacts and never says which
one holds this section.

### Proposal, needs sign-off

Import the `challenges/` ban list into the repository contract, including its
clause that limits are stated without explaining how an approach behaves at
them. Strip the quirk from the four `Requirements` sections. Split
`Architecture questions`: property-level questions stay with the task,
mechanism-naming ones move to `HINTS.md` or are rewritten to name only the
property.

- **Severity:** high
- **Scope:** teaching contract, phases 1 and 2
- **Affected:** `specs/1/2`, `specs/1/4`, `specs/2/3`, `specs/2/4`, `specs/01-systems-labs.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` and to `1/1`, `1/2`,
  `1/3`, `1/4`, `2/1`, `2/3`, `2/4`; the questions in `1/5`, `2/2`, and `2/5`
  were reviewed and already name only properties or the announced
  environment, 2026-08-14 (no git in this repository)

## ✅ FIXED 2026-08-14 — S6 — nothing enforces the teaching contract (2026-08-14, fixed)

Resolved as proposed. `make teaching-lint` joined the verification contract
in `specs/01-systems-labs.md` and gained a section in
`specs/0/5-shared-scaffold.md`: it fails if any lab `README.md` contains a
scenario barrier name, a `Neighbouring systems` product name, or any citation
marked solution-bearing, and CI runs it. The check stays cheap because every
`Code pointers` bullet in phases 1 and 2 already carries an explicit neutral
or solution-bearing state.

The original entry is kept below.

No gitignore rule, CI check, naming lint, or publish step exists anywhere in the
specification. Every protection in the teaching contract is a sentence asking an
implementer to behave. `S3` is worse than unenforced — the waiver makes
non-enforcement the stated policy.

### Proposal, needs sign-off

Add one root target, `make teaching-lint`, run in CI: fail if any lab
`README.md` contains a scenario barrier name, a `Neighbouring systems` product
name, or any citation marked solution-bearing. It is cheap once every citation
carries an explicit neutral or solution-bearing marker, and it is the single
change that turns the contract from aspiration into a gate.

- **Severity:** medium
- **Scope:** verification contract
- **Affected:** `specs/01-systems-labs.md`, `specs/0/5-shared-scaffold.md`
- **Source:** structural audit against `challenges/`, 2026-08-14
- **Status:** fixed
- **Fix:** in-tree edits to `specs/01-systems-labs.md` (verification
  contract) and `specs/0/5-shared-scaffold.md` (Teaching lint), 2026-08-14
  (no git in this repository)

## ✅ FIXED 2026-08-14 — S2 — `1/4` is a cloud-native lab filed in a local open-source phase (2026-08-14, fixed)

Resolved by the phase 1 and phase 2 split rather than by either option below.
`1/4` moved to `2/2`, and phase 1 gained a new `1/4` on NATS JetStream, so the
local phase now holds no hosted-service emulator at all. The split itself is
recorded in `specs/0/6-serverless-contrast-track.md` and stated in
`specs/01-systems-labs.md`.

The original entry is kept below because its reasoning produced the split.



The catalog follows an unwritten split. Phases 1, 3, 6, 7, and 8 run the actual
software — PostgreSQL, Kafka, Flink, OpenSearch, PostGIS, a local validator, a
local Ethereum node, the kernel itself — so there is no emulator and no gap,
and a cloud version would be the same software under someone else's operation.
Phase 4 is the one phase where a hosted service's semantics are the lesson, so
it is the only phase with an emulator gap and the only one that needs an
account.

`1/4-reliable-record-import.md` violates that split. It requires the SQS API,
the Lambda batch event shape, and the DynamoDB API, which makes it three
emulators — two of them JVM — and phase 1's only cloud touchpoint.

### Proposal, needs sign-off

First, state the split in `01-systems-labs.md`, because it is a real
organising principle that nothing currently records.

Then fix `1/4`, one of two ways:

1. Move it to phase 4. Cleanest boundary, but phase 1 loses the leased-queue
   delivery model, and the contrast between an ephemeral notification, a
   retained log, and a lease is the phase's spine.
2. Reground it on NATS JetStream, which runs natively and keeps the lesson.
   Phase 1 keeps its three delivery models and drops two emulators; phase 4
   keeps the SQS and Lambda versions, where hosted semantics belong.

RabbitMQ is not a substitute, despite being the obvious candidate. SQS holds a
timer lease: the visibility timeout expires on the clock, so a merely slow
consumer is handed the same message again while it is still working. RabbitMQ
redelivers on consumer liveness — a channel drop or a nack — so slowness costs
nothing, and `consumer_timeout` kills the channel rather than expiring one
message's lease. That is a different failure model, and `1/4` would falsify a
different belief.

JetStream keeps it: `AckWait` is a real timer that triggers redelivery on
expiry, and `MaxDeliver` caps attempts. Dead-lettering is assembled rather than
configured — `MaxDeliver` stops redelivery but leaves the message in the
stream, and the queue is built by consuming the max-deliveries advisory
subject.

The result store does not need to be NoSQL. It only has to make valid records
queryable with a per-identity fate, which PostgreSQL already does and phase 1
already runs. NoSQL data-model lessons are contract-shaped and belong to
phase 4.

Recommendation: the second. It fixes the principle rather than relocating the
gap.

- **Severity:** medium
- **Scope:** phase 1, phase 4, curriculum structure
- **Affected:** `specs/1/4-reliable-record-import.md`, `specs/01-systems-labs.md`, `specs/1/README.md`
- **Source:** review of local versus hosted dependencies, 2026-08-14
- **Status:** proposed (redesign, needs sign-off)
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
