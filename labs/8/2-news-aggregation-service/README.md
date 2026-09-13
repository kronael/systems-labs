# News aggregation service

Design and build a system that ingests many news feeds, groups items that
report the same event, and serves a curated ranked view with a stated
freshness.

The obvious model is that two stories are either duplicates or they are not.
They are not either. Verbatim wire copy, a syndicated reprint with a new
headline, a lightly localized rewrite, and a follow-up that quotes the
original sit on a continuum, and no point on that continuum is the true
boundary. Where the system draws the line is a product decision with a
measurable cost in both directions: draw it tight and the same story floods
the view three times, draw it loose and a distinct story disappears inside a
cluster nobody opens.

The feeds compound the problem. Publication timestamps are unreliable and
sometimes move backwards. An article edited in place changes its content
under a stable identifier, so the item the system grouped an hour ago is not
the item the feed serves now. The system's own history — what it saw, when it
saw it, and what it served — is the only history it can stand behind, because
the publisher's history is not on offer.

The assignment is the whole service: ingestion path, the grouping decision
and its declared policy, the layout of the index and the durable store, the
ranked view, and end-to-end tests. How two items are judged to report the
same event, how that judgment holds up as the retained corpus grows, the
layout of the index and the durable store, and how the ranking weighs a match
against currency are the learner's decisions. Feed parsing is prepared work,
not learner work; the difficulty of this lab lives entirely in the grouping
decision and its history, never in feed formats.

## What you are given

The supplied Compose stack starts OpenSearch, PostgreSQL, the feed harness,
and the fault controller. The feed harness serves generated feeds over HTTP
in both RSS 2.0 and Atom shapes, replays cached real feed recordings, and
drives the scenario schedules. A starter adapter delivers every item as a
versioned internal record carrying the source feed, the identifier exactly as
the feed gave it, the publisher timestamps exactly as given, title, body, and
a raw checksum.

The generator emits verbatim reprints under different identifiers, edited
syndicated copies, follow-ups, in-place edits, and timestamp regressions at
declared ratios, and every item carries a stable harness identity so
verification can assert exact histories.

The learner owns the ingestion service, the grouping decision, the layout of
both stores, the ranking, and the query API. No cloud account is required.

## Requirements

The service exposes three public operations: a ranked view of current
stories, a full-text search over stories, and a story detail that lists the
member items and the story's membership history. A story is a group of items
reporting the same event, shown once with one representative; a query that
matches three reprints of one story returns one result, not three.

The learner declares a clustering policy in `ARCHITECTURE.md`: what makes
two items the same story, stated in falsifiable terms, and what each error
direction costs the reader. The policy is the contract verification holds the
system to. A policy that cannot be checked against an observed grouping is
not a policy.

Every served answer carries a freshness statement: what the view reflects
and as of when. The statement derives from the system's own observation
history, because publisher timestamps cannot support it; a feed replaying
old items with regressed timestamps must never move the freshness statement
backwards. The learner declares a bound on the time from an item's
acceptance to its visibility in the served view, and the evidence measures
it, because in this environment a write and its searchability are separate
events.

An article edited in place under a stable identifier must be reflected: the
served view shows the current content, the grouping is reconsidered under
the declared policy, and the story's membership history records the
transition. The system retains what it served before the edit as its own
history. Ingestion continues while queries run, and the ranking must express
both what a query matched and how current the story is, so a stale exact
match cannot sit above a fresh one by default.

The scale target is 2,000 feeds, a sustained ingest of 200 items per second
with bursts of 2,000 per second, 20 million retained items, and 100
concurrent readers across the view and search operations. At that size,
comparing each arrival against every retained item is arithmetic nobody can
afford, so the cost of the grouping decision is part of the design, not an
implementation detail. These numbers size the problem; they are not
pass thresholds. Thresholds stay relative, calibrated, structural, or
learner-declared.

The evidence must show sustained ingest rate, the acceptance-to-visibility
distribution against the declared bound, and the grouping error measured in
both directions against the declared policy. How the measurement is produced
is the learner's choice; no telemetry stack is required.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- the declared clustering policy: what makes two items the same story, what
  each error direction costs, and which case on the continuum the policy
  deliberately gets wrong;
- what the arrival of one item costs, against how much of the retained
  corpus it is effectively compared, and how that cost changes as the corpus
  grows;
- which store is authoritative for group membership and what the other
  reflects, and what a reader sees between a membership change and its
  visibility in the served view;
- how an in-place edit is detected, and what happens to the item's grouping
  and to the record of what was previously served;
- what the freshness statement means operationally, which clock it derives
  from, and why a regressed publisher timestamp cannot move it;
- how the system's own history is kept: what was served, when, and under
  which version of the item;
- what the ranking expresses, and what it deliberately ignores;
- which grouping errors appear under sustained burst, and why that is
  acceptable or not.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

Correctness is asserted as exact membership histories for named items, never
as counts alone. A named item shows the full transition: grouped, served,
edited in place, regrouped or deliberately retained under the declared
policy, and the served view reflects the outcome within the declared
visibility bound. `syn-0114` appears exactly once in the served view,
grouped with the story it reprints. The replay from `F-208` leaves the
system's observation history intact and never moves a freshness statement
backwards. After the restart at `wire-5000`, every item in flight appears
exactly once with an exact history — nothing dropped, nothing doubled.

The evidence report includes the sustained ingest rate, the
acceptance-to-visibility distribution against the declared bound, the
grouping error in both directions against the declared policy over the
labeled schedule, and the behavior of view latency and grouping quality
through the burst. It names the point at which the grouping decision would
have to be cheapened to hold the ingest rate. No gate hard-codes a number:
performance gates are relative to a recorded baseline on the same host,
calibrated during warm-up, structural, or learner-declared.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **pgvector** — [documentation](https://www.postgresql.org/docs/current/textsearch-intro.html)

Stuck? See `hints/`.
