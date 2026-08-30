---
status: draft
---

# Uninterrupted catalog service

## Brief

Design and build a catalog service that keeps answering while the shape of its
records changes. Merchants publish items; clients retrieve one item and list a
merchant's items under a filter. The service also accepts a declared change to
the shape of a catalog record and adopts it while it is serving.

The catalog lists packaged food and household goods, and European
price-indication law — Directive 98/6/EC — requires each offered product to
show a unit price, the price per litre or per kilogram, beside its selling
price, while exempting some products: those whose unit price would equal the
selling price, and those whose nature makes the indication useless. The duty
is why the record's shape changes, and it takes effect on a date, not when
the catalog is ready. What each item must carry under the new shape depends
on that item — its own computed value, or none at all — so while a change is
being adopted the catalog holds items in both shapes, and one shape never
replaces the other in a single step.

There is no maintenance window because the store never closes: the catalog is
the storefront of every merchant on it, and it keeps selling while the change
lands. For the whole time a declared change is being adopted, requests are
accepted, answered within a declared ceiling, and never answered from a
record caught between the two shapes. Every published item carries exactly
one shape, and the answer states which one.

The required environment is PostgreSQL as the system of record. The data
layout, where a record's published shape is decided, how a declared change
reaches the stored catalog, how requests in flight and a change in progress
are ordered against one another, and the process decomposition are the
learner's decisions.

## Prepared scaffold

The supplied Compose stack starts PostgreSQL, OpenTelemetry collection, the
fault controller, and an optional second application replica. It includes
generated clients, initial schema loading, a deterministic 25-million-item
catalog across 50,000 merchants, and read, write, and long-read workloads.

The stack supplies the declared changes themselves: four record-shape changes
carried in the lab's TOML configuration, which together take the catalog from
the shape it sold under yesterday to the shape the price-display duty
requires. Each states the shape the catalog must end in and the rule that
gives every existing item its value under that shape. Among them: a
representation of an existing attribute changes — the amount an item contains
becomes computable rather than merely displayable — and readers of both
shapes must understand it while the change runs; a value is computed when the
change is adopted from what each record already holds, differs for every
item, and is conditional — the stated rule decides which items must carry it
and which must not; a uniqueness rule becomes narrower, so a shopper
comparing per-unit prices never meets the same product twice under one
merchant; and an attribute the product stops publishing — the free-text
per-unit claim merchants wrote by hand, which cannot stand beside a computed
value it may contradict. The changes build on one another — the computed
value reads the representation an earlier change establishes — which is why
the declared order is part of the declaration. A declared change states an
outcome and never a step; how the catalog arrives at it is the learner's
decision. The learner never consults the law: each declared change states its
rule completely.

The stack also includes the request recorder: a prepared component outside the
learner's processes through which client traffic passes, timestamping every
request with its outcome and duration. That record is the observable boundary
of the uninterrupted claim — a request refused or delayed while a change is
being adopted is visible there whether or not the application noticed it.

The learner owns the database design, application topology, change-adoption
behaviour, and application Compose layer. Standard Make targets start the
environment, load the catalog, run feature tests, adopt the declared changes
under load, inject restarts, and capture query plans and evidence.

## Requirements

Clients publish an item under a merchant's item identity, retrieve one item,
and list a merchant's items under a declared filter with paging. A publication
carries a revision; republishing the same revision has one effect, and
publishing a stale revision fails visibly. An operator applies one declared
change and inspects the progress and outcome of a change in flight.

Every answer states the shape it carries, because under the new shape absence
is meaningful: an item without the per-unit value is either exempt or not yet
reached, and a listing that blurs that distinction misleads a shopper exactly
where the duty exists to let prices be compared. No answer mixes the two
shapes and no item is published in both at once. An item accepted before a
change is retrievable after it under the same identity, carrying the new
shape with the value — or the stated absence — the change's rule gives it.

For the whole duration of a declared change, no request is refused for a
reason the change caused and no request exceeds the submission's declared
latency ceiling. That ceiling is declared before the change is adopted, not
chosen afterwards to fit what happened.

Declared changes are adopted one at a time and in the declared order. A second
change submitted while one is in progress is refused visibly. A change
interrupted by process death or database restart leaves every item readable
and every acknowledged publication intact, and it can be carried to completion
or withdrawn; both outcomes are discoverable through the public operation, and
neither requires editing stored records by hand. Carrying it to completion
resumes from what the interrupted attempt already made true rather than
beginning again: adopting a change across 25 million live items takes real
time, the duty's date does not move, and a change that pays for its progress
twice after every interruption may never finish at all.

Reads and writes accepted during a change keep the meaning their
acknowledgement gave them. A publication acknowledged before a change is not
lost by it, and a publication acknowledged during one is retrievable
immediately in the shape its answer named.

The scale target is a catalog of 25 million packaged-goods items across
50,000 merchants, a
sustained 3,000 retrievals and listings per second alongside 300
publications per second from 400 concurrent clients, one listing request in a hundred
streaming a large result that stays open for 30 seconds, and all four declared
changes adopted inside one evidence run. These numbers size the problem; they
are not pass thresholds. Latency during a change is measured against the
learner's declared ceiling.

The evidence must state, for each declared change, how long its adoption took,
how many requests inside its window were refused or exceeded the declared
ceiling, the worst observed latency and where in the window it fell, and how
many items were published in each shape over time. How that measurement is
produced is the learner's choice, but the request recorder's history is the
authority for what clients saw.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what "the shape of a record changed" means to a client, how an answer
  states which shape it carries, and how a reader tells an item the rule
  exempts from an item the change has not reached;
- how every public answer stays correct while items exist in both shapes, and
  what that costs on the read path once a change is finished;
- how a request in flight is ordered against a change in progress, and what a
  writer observes at the moment the published shape changes;
- how a change's progress is measured, what the reported progress is honest
  about, and what it cannot say;
- how an interrupted change is carried to completion or withdrawn without any
  item becoming unreadable, and what makes the second attempt safe;
- how the declared latency ceiling was chosen, what it costs, and what
  evidence shows it held for the whole window rather than on average;
- what a client that retrieved an item before a change and publishes it back
  after must observe;
- which indexes serve each public access pattern, and what each costs while a
  change is in progress;
- which query answers "which shape is item X published in, and since when",
  and what it costs at the declared volume.

Two further questions presuppose part of a design and are solution-bearing;
they publish to `HINTS.md`, never to the task:

- why the time a declared change takes and the time it makes the catalog
  unavailable are not the same quantity, and which property of a declared
  change decides whether the catalog's size affects the second one;
- why a change that must wait for the catalog's longest-running reader also
  stops every request arriving after it, including requests that need nothing
  that reader is holding.

The document must compare at least two viable adoption designs without turning
a known tool name into the argument, and state one residual limitation.

## Adversarial evaluation

The failure schedule adopts each declared change while the full workload runs.
It starts one change at the moment a long-running listing request is open and
holds that request open across it; it kills the application process when a
named item identity has been published in the new shape and its neighbour has
not; it restarts PostgreSQL mid-adoption; it submits writes at named item
identities on both sides of that boundary while the change runs; it applies a
second change while one is in progress; it withdraws a change after it has
partly landed; and it repeats publications of already-acknowledged revisions
throughout.

At the end the full catalog is read. Every item is published in exactly one
shape, every acknowledged publication is present exactly once at its
acknowledged revision, and every value under the new shape matches the
declared change's stated rule — where the rule exempts an item, that means
the value's absence.

Checks observe only HTTP, SQL-visible state, the request recorder's history,
process lifecycle, query plans, metrics, and the submitted evidence. They do
not require a particular schema, adoption structure, or coordination
primitive.

## Acceptance evidence

Every declared change reaches its stated end shape, and after each one the
catalog answers every public operation correctly for every item. No request is
refused for a reason a change caused, and no request exceeds the declared
ceiling inside a change window. An interrupted change leaves the catalog
readable and is carried to completion or withdrawn through the public
operation alone. Failure is returned to the caller or exposed in queryable
status rather than only logged.

The report includes the request recorder's exact history across every change
window, each change's adoption duration, the refused and over-ceiling request
counts inside it, the latency distribution during the window against the
declared ceiling, the count of items published in each shape over time, and
`EXPLAIN (ANALYZE, BUFFERS)` for every required large-data query before and
after each change. It names the window in which the catalog holds two shapes
at once and the limit of the chosen adoption design.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **MongoDB** — [documentation](https://www.mongodb.com/docs/manual/data-modeling/).
  Records in one collection are not required to carry the same fields and a
  field's type may differ between them, so a shape change is never an
  operation against the store at all: it moves entirely into the application,
  which gains no moment at which the whole collection is known to have
  changed.
- **MySQL with InnoDB** — [documentation](https://dev.mysql.com/doc/refman/8.4/en/innodb-online-ddl-operations.html).
  The statement declares the interruption it is willing to cause and the
  server refuses the change outright when it cannot honour that declaration,
  so the operator learns before the change runs what this lab makes the design
  discover while serving traffic.
- **CockroachDB** — [documentation](https://docs.cockroachlabs.com/docs/stable/online-schema-changes).
  Runs the change as a background job that holds no locks on the table data
  and rolls out the new shape while the previous one is still in use, which
  moves the coexistence of two shapes out of the application and into the
  database's own machinery — and turns the change into a job whose duration
  the operator does not bound.

The lab does not run them.

## Scope

The expected focused time is fourteen to twenty hours. PostgreSQL, the catalog
data, the declared changes, the request recorder, telemetry, and faults are
prepared. A simple design fails this lab at its scale target: adopting a
declared change against 25 million items as one operation runs for minutes,
and every request arriving while it runs waits behind it, so the change a
small catalog absorbs invisibly becomes an outage the request recorder writes
down. Replication, cloud databases, search relevance, merchant
authentication, legal interpretation, and a browser front end are outside the
problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  PostgreSQL, evidence, and verification contracts.
- [Directive 98/6/EC](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:31998L0006)
  — Article 3: "The selling price and the unit price shall be indicated for
  all products referred to in Article 1, the indication of the unit price
  being subject to the provisions of Article 5", and "The unit price need not
  be indicated if it is identical to the sales price"; Article 5 lets Member
  States waive the indication where it "would not be useful because of the
  products' nature or purpose or would be liable to create confusion".
  Neutral: it explains why the change exists and why the new value is
  conditional, never how to adopt one, so it may publish in `README.md`.
- [PostgreSQL `ALTER TABLE`](https://www.postgresql.org/docs/current/sql-altertable.html)
  — the lock level differs per subform and "An `ACCESS EXCLUSIVE` lock is
  acquired unless explicitly noted", while "Adding a column with a volatile
  `DEFAULT` (e.g., `clock_timestamp()`), a stored generated column, an
  identity column, or a column with a domain data type that has constraints
  will cause the entire table and its indexes to be rewritten."
  Solution-bearing: the same page lists which subforms take a weaker lock, so
  it belongs in `HINTS.md`, never in `README.md`.
- [PostgreSQL explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html)
  — the mode taken by a change "Conflicts with locks of all modes", including
  the mode every plain read acquires, and "a transaction seeking either a
  table-level or row-level lock will wait indefinitely for conflicting locks
  to be released". Solution-bearing: this is why a waiting change stops
  arriving readers, so it belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
