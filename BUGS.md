# Bugs

Record a defect here rather than fixing it in passing — see `CONTRIBUTING.md`,
"Filing a defect". A fix that changes a contract, control flow, or anything
cross-cutting is a redesign: record it as a proposal and get sign-off first.

## S39 — a lab on object storage has no hosted comparison (2026-09-20, proposed)

`4/6` publishes versioned dataset releases to S3-compatible object storage,
run locally as SeaweedFS. Its gate proves a declared model, and
`docs/contract.md` requires that such a gate be compared against the hosted
service by `make smoke`. S3 is not on the contract's optional-cloud list,
which names Lambda, SQS, DynamoDB on-demand and short logs.

Either the contract gains S3 on that list, or the lab ships with no hosted
comparison and says so. The cost of the comparison is real: at the advertised
$0.023 per GB-month of storage and $0.09 per GB transferred out, one release
of the size the lab's domain implies is roughly ten dollars a run.

This is a contract change and needs sign-off.

- **Severity:** medium — `4/6` ships with no hosted comparison
- **Scope:** `docs/contract.md`, optional-cloud list
- **Affected:** `4/6`
- **Source:** review of the object-storage proposal, 2026-09-20
- **Status:** proposed
- **Fix:**

## S41 — the catalog and the phase pages state the quirk (2026-09-21, proposed)

`docs/contract.md:429` requires `labs/README.md` to name every lab with its
"architecture pressure". That column states the lesson: `labs/README.md:43`
"Cache authority, eviction, stampede, degradation", `:44` "Background merges,
asynchronous mutation, insert frequency", `:91` "the spatial index is only a
filter", `:36` "a freeze that strands the publish".

The phase pages do the same. `labs/1/README.md:29` names a pause barrier and
`1/2`'s freeze past acknowledgement; `labs/2/README.md:94` gives the freeze
barrier and the controller layer for the whole phase; `labs/6/README.md:58`
names `epoll` and `io_uring` and `:74` links `mallopt(3)`.

Both files are learner-facing and opened by default, before any lab. The
contract binds the lab `README.md` and says nothing about either. No lab edit
repairs this, because the contract orders the column.

- **Severity:** high — the teaching boundary fails one tier above the lab
- **Scope:** `docs/contract.md`, `labs/README.md`, seven phase `README.md`
- **Affected:** every lab, before the learner opens one
- **Source:** red review, 2026-09-21
- **Status:** proposed
- **Fix:** decide what the catalog column may say. A column that orients
  without stating the pressure is the contract change; extending the ban list
  to the catalog and the phase pages is the rest of it.

## S42 — the clock layer cannot be built as specified (2026-09-21, proposed)

`docs/contract.md:410` requires per-container clock skew.
`time_namespaces(7)`: "time namespaces do not virtualize the CLOCK_REALTIME
clock", so a container has no wall clock of its own and stepping it steps the
host's, for the controller, the dependencies and every evidence timestamp at
once. `libfaketime` "cannot work with statically linked binaries" and relies
on a preload a Go program does not pass through; Go is the mandated starter.

`1/1/EVALUATION.md:18` is the worst case. It steps the clock "between the two
readings a named request's own quote comparison spans" and says a run where
the step never lands inside a comparison "fails rather than passes".
`1/1/README.md` never mentions a clock, and a correct design reads the wall
clock once, so the correct design fails. `HOWTO.md` sends every learner to
lab 01 first, and this is the first `make fault` they run.
`1/2/EVALUATION.md:17` repeats it. `6/5` shows the fix: the README states the
time-base requirement and the answer key names the layer.

- **Severity:** high — the first fault a learner runs fails a correct design
- **Scope:** `docs/contract.md:410`, `1/1`, `1/2`, `2/5`, `6/5`
- **Affected:** lab 01, and every lab whose answer key names the clock layer
- **Source:** learner and CTO reviews, 2026-09-21, both pages fetched
- **Status:** proposed
- **Fix:** drop the container clock step where a freeze longer than a real
  validity window does the same work, and where a backwards reading is the
  lesson, put the clock seam into the starter contract and state the
  time-base requirement in the brief.

## S43 — a barrier cannot hold a boundary the learner owns (2026-09-21, proposed)

`docs/contract.md:366` fails a run whose declared barrier is never mapped or
never reached. Twelve labs declare barriers inside schemas the learner
designs: `1/5/EVALUATION.md:13` holds "a named transfer's database commit"
while `1/5/README.md:21` gives the learner the schema;
`1/2/EVALUATION.md:12` freezes "a worker"; `2/3/EVALUATION.md:21` holds
attempts "at the contended resource" while `:28` requires no named item
layout. A proxy on the PostgreSQL or Kafka wire cannot tell which `COMMIT` or
`Produce` carries transfer 4711.

Two labs show the shape of a fix. `4/1/EVALUATION.md:24` reads keys the
store's own `DescribeTable` exposes. `4/6/EVALUATION.md:13` holds "the store's
acknowledgement of a write the service made", with no identity inside it;
its own scenarios at `:17-28` then key on release and increment identities
the learner lays out, so the shape holds at `:13-15` and not below it.

- **Severity:** high — the gate rests on a mechanism no file specifies
- **Scope:** `docs/contract.md:359-369`, twelve labs
- **Affected:** five of six phase 1 core labs
- **Source:** CTO, red and CEO reviews, 2026-09-21, agreeing independently
- **Status:** proposed
- **Fix:** for every barrier that names a record, name the public observable
  the adapter reads, or state that the record contract is course-fixed.

## S44 — teaching-lint as specified fails this repository (2026-09-21, proposed)

`docs/contract.md:817` scopes the lint to every course-authored learner-facing
text except `hints/` and `EVALUATION.md`, and fails on a barrier name, a
mechanism name "prescribed, disclaimed, or merely mentioned", and a
dependency's configuration-parameter name. Current hits: the catalog's
pressure column, `labs/4/README.md:94` "visibility timeouts",
`labs/8/2-news-aggregation-service/README.md:127` which prints schedule
identities, and a second `hints/` sentence in 36 of 36 lab READMEs against
`:296` "says nothing else about hints anywhere". The lint also keys on a
"Neighbouring systems" heading; the heading in every README is "What a
practitioner might have used instead". `docs/shared-scaffold.md:188` calls the
check "a scan rather than a judgment", and the mechanism vocabulary it scans
for does not exist.

- **Severity:** medium — the gate that protects the boundary cannot be written
- **Scope:** `docs/contract.md:817-831`, `docs/shared-scaffold.md:188`
- **Affected:** CI, and the four files named above
- **Source:** CTO and red reviews, 2026-09-21
- **Status:** proposed
- **Fix:** re-scope the lint, correct the heading it keys on, and write the
  vocabulary list before the contract calls the check cheap.

## S45 — the schedule is both unreadable and published (2026-09-21, proposed)

`docs/contract.md:265` says "A schedule is never a readable artifact at rest"
and `:961` says none is readable inside the learner tree. `:280` and `:725`
say the schedule is stated in `EVALUATION.md`, and every `EVALUATION.md` opens
a section called "The failure schedule".

`2/1/EVALUATION.md:13` prints invoice 4711,
accounts 2205 and 2206, subscription 118207 and usage event 771003, while
`1/5/EVALUATION.md:13` says only "a named transfer".

- **Severity:** medium — the contract contradicts itself on its own protection
- **Scope:** `docs/contract.md:265`, `:280`, `:725`, `:961`, `README.md:88`
- **Affected:** the compiled recipe, the digest, the publish step
- **Source:** CTO and red reviews, 2026-09-21
- **Status:** proposed
- **Fix:** define the schedule, which is shape and may be read, apart from the
  recipe, which is identities and timing and stays compiled. Then apply one
  identity policy to `2/1`, `4/6` and `8/2`.

## S46 — 31 answer keys do not say what a weak design gets wrong (2026-09-21, open)

`HOWTO.md:162` tells the learner that `EVALUATION.md` "states what a strong
design holds and what a weak one gets wrong", and `docs/contract.md:280`
requires it. Only `1/2`, `1/4`, `1/6`, `1/8` and `4/6` carry such a section.
The other 31 hold one `## The failure schedule` and nothing else, lab 01
included, so a learner who passes the schedule has nothing to check a design
against.

- **Severity:** medium — the answer key answers nothing in 31 of 36 labs
- **Scope:** 31 `EVALUATION.md` files
- **Affected:** every lab except the five named
- **Source:** learner review, 2026-09-21, count verified
- **Status:** open
- **Fix:** `1/2/EVALUATION.md:32` is the template, five sentences.

## S47 — 25 labs have no hint that unsticks (2026-09-21, open)

`HOWTO.md:122` promises `hints/` holds "the architecture reading, what the
neighbouring systems do differently at this lab's boundary, and the designs
that were rejected". In 25 of 36 labs `hints/` holds the neighbour file and
the citation list and nothing else, and no lab in the repository holds a
rejected-designs hint. The stuck learner gets nothing, as in `1/1`, or the
whole answer at once, as in `3/2/hints/02:17`, which states all three
falsified beliefs verbatim. There is no tier between them.

One further leak sits inside the hint files themselves: 34
`where-this-comes-from.md` files send the learner to `docs/contract.md`, the
1095-line author contract.

- **Severity:** medium — `Stuck? See hints/` is the only exit the lab offers
- **Scope:** 25 `hints/` directories, 34 `where-this-comes-from.md` files
- **Affected:** every lab outside the eleven with a third hint
- **Source:** learner review, 2026-09-21, counts verified
- **Status:** open
- **Fix:** one guiding-question hint per lab in the shape of `1/2/hints/01` —
  the condition, the question, no design. Cut the contract link.

## S48 — hand-typed counts drift, and 1/7 prints its failure narratives (2026-09-21, open)

The lab counts are typed by hand in `CLAUDE.md:9`, `README.md:5` and
`README.md:127-133`, and `docs/shared-scaffold.md:12`, so every lab added
drifts them. `labs/1/7-spectrum-compliance-service/README.md:313-316` says
both failure narratives that follow are solution-bearing and then prints both.

- **Severity:** low — each is a sentence, but the pattern repeats every time a
  lab is added
- **Scope:** `README.md`, `CLAUDE.md`, `docs/shared-scaffold.md`, `1/7`
- **Affected:** every entry point a reader counts from
- **Source:** CEO and CTO reviews, 2026-09-21
- **Status:** open
- **Fix:** derive the counts from the directory listing in a check that CI
  runs, and move the two 1/7 narratives into `hints/`.

## S49 — two labs run technologies outside the spine (2026-09-21, proposed)

`docs/contract.md:503` fixes a small set of technologies.
`labs/1/8-streaming-completion-service/README.md:18` requires "one vLLM server
on CPU", and `4/6` runs SeaweedFS. Neither vLLM nor object storage appears in
the contract, the scaffold, or the selection record.

Two consequences follow. The phase pages introduce each phase's technologies,
so `labs/1/README.md` and `labs/4/README.md` cannot cover `1/8` and `4/6`
until the spine names what they run. And the vLLM CPU backend documents
`avx512f` as recommended and `avx2` as limited, while `README.md:146` promises
only "a machine that runs Linux containers".

- **Severity:** medium — two shipped labs sit outside the governing list
- **Scope:** `docs/contract.md:503`, `docs/lab-selection.md`, two phase pages
- **Affected:** `1/8`, `4/6`
- **Source:** CTO review, 2026-09-21, vLLM CPU page fetched
- **Status:** proposed
- **Fix:** amend the spine and the selection record for both, or gate the two
  labs until it is amended. Then state the CPU requirement `1/8` implies.

## S50 — the phase 6 cost runner cannot see the costs 6/3 asks about (2026-10-05, proposed)

`docs/shared-scaffold.md:167` supplies "a deterministic cost runner over
Callgrind, instruction counts rather than wall time, so a per-core cost
survives a noisy host", and `labs/6/3-large-index-query-service/README.md:68`
asks which costs of the access path grow with the number of cores and how the
evidence isolates them. Callgrind cannot isolate them. The Valgrind core manual
runs every thread under one lock "to serialise thread execution", and the
Cachegrind manual says of its simulation "It doesn't account for kernel
activity." The costs that grow with cores on a page-cache access path are
kernel costs under concurrency, so the runner reports a number the question is
not about.

- <https://valgrind.org/docs/manual/manual-core.html>
- <https://valgrind.org/docs/manual/cg-manual.html>

- **Severity:** medium — the harness promises a measurement its tool cannot make
- **Scope:** `docs/shared-scaffold.md`, `6/3`
- **Affected:** `6/3`; `6/1` and `6/4` name the same harness
- **Source:** release refine, 2026-10-05, both manual pages fetched
- **Status:** proposed
- **Fix:** keep Callgrind for the user-space instruction count and add a lane
  that measures the kernel cost under cores, or restate what the runner
  isolates. It changes the scaffold, so it needs sign-off.
