---
status: draft
---

# Shared budget service

## Brief

Design and build a shared budget service. An organization holds budgets, each
with a spending limit; clients submit claims against them, and one claim may
draw on several budgets at once. No budget is ever overspent, and a decision
returned to a client is final.

Whether a claim can be placed depends on what the service found true of every
budget that claim names at the moment it decided, and many clients claim
against overlapping sets of budgets at the same time. The store may decline to
complete a unit of work rather than accept it, and the service still owes
every claim a decision.

The required environment is PostgreSQL as the system of record. The data
layout, where the spending rule is decided, what one unit of work covers,
where the boundary of that unit sits relative to the answer the client
receives, and the process decomposition are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts PostgreSQL, OpenTelemetry collection, the
fault controller, and an optional second application replica. It includes
generated clients, initial schema loading, a deterministic set of 100,000
budgets with 50 million retained claim records, and claim, limit-change, and
statement workloads whose overlap between claims is a configured profile
rather than an accident of the generator.

The learner owns the database design, application topology, decision
behaviour, and application Compose layer. Standard Make targets start the
environment, seed budgets and claim history, run feature tests, drive
overlapping claims at a declared concurrency, inject process death and
database restarts, and capture query plans and evidence.

## Requirements

Clients submit a claim under a claim identity, naming an amount and the
budgets it may draw on; settle a claim at a final amount not greater than the
amount claimed, or release it; change a budget's limit; and read a budget
statement. Resubmitting a claim identity with the same request returns the
same decision; resubmitting it with different data fails visibly.

No budget is overspent. At every point in the observed history, the total
committed against a budget is not greater than that budget's limit. This holds
while many clients claim against overlapping sets of budgets at once, not
merely in a quiet system, and it holds through every supported write path.

A claim that names several budgets draws its whole amount from among them; the
service chooses how the amount is divided and the decision states the
division. A claim that cannot be placed in full is refused with a stated
reason.

Every decision returned to a client is final. Nothing the service does
afterwards may contradict it, and one claim identity never receives two
different decisions.

Every accepted claim reaches a decision. No claim fails to reach one because
other claims keep conflicting with it; the `ARCHITECTURE.md` declares the
bound it holds and shows that bound holding under the stated overlap. The work
the service performs to decide one claim is bounded, the `ARCHITECTURE.md`
declares that bound, and it states what the client receives when a claim
reaches it.

A budget statement is a state the budget actually passed through: the claims
it lists and the amount it reports as remaining agree with each other and with
one point in that budget's history. A statement that cannot be produced that
way is not returned.

A budget's limit can be changed while claims against it are in flight. Once a
limit change is accepted, no claim is approved against the previous limit; a
limit lowered below what is already committed refuses new claims rather than
withdrawing decisions already given.

Database errors map to clear client outcomes. A claim the service cannot
decide is returned to the client with a reason rather than dropped or left in
flight, and retry must have a stated boundary.

The scale target is 100,000 budgets holding 50 million retained claim records,
a sustained 2,000 claim submissions per second from 500 concurrent clients, an
overlap profile in which one budget in a thousand receives a fifth of all
claims and three claims in ten name between two and four budgets drawn from
that same contended set, 5 percent of submissions repeating an
already-decided claim identity, and 10 limit changes per second against
budgets under contention. These numbers size the problem; they are not pass
thresholds. Decision latency is measured against the learner's declared
service level.

The evidence must state how many units of work the store declined to complete
and at what offered concurrency, how that count moved as the overlap share
rose, the distribution of attempts per decided claim, how many claims reached
the declared bound and what their clients received, and the offered rate
against the rate actually delivered. The overspend check is recomputed from
the retained claim records and never from a stored total. How that measurement
is produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what "committed against a budget" means, where that fact lives, and how it
  is checked without trusting a stored total;
- how a claim naming several budgets is decided, and what that decision is
  true of at the moment it is given;
- where the boundary of one decision sits relative to the answer the client
  receives, and what a client may observe if the service has to arrive at that
  decision more than once;
- how a claim is kept from waiting forever while other claims keep conflicting
  with it, what the declared bound is, and how the evidence shows it held;
- how the work performed to decide one claim is bounded, and what the client
  receives at that bound;
- what a budget statement is true of, and how the design defends that under
  concurrent claims and limit changes;
- how a limit change and a decision in flight are ordered, and what each
  observes;
- which indexes serve each public access pattern at the declared volume, and
  what each costs under the stated overlap;
- what holding the invariant costs in delivered throughput — measured, not
  asserted — and where that cost is paid.

Two further questions presuppose part of a design and are solution-bearing;
they publish to `HINTS.md`, never to the task:

- why a store's refusal to complete a unit of work is a different kind of
  failure from a write it rejects as invalid, and what the application must
  own for the first kind that it does not own for the second;
- why a decision that reads a set of records and writes into that same set can
  be refused even when every individual write it made was legal on its own.

The document must compare at least two viable decision designs without turning
a known library name into the argument, and state one residual limitation.

## Adversarial evaluation

The failure schedule drives many clients at one contended budget set at named
claim identities. It accepts a limit change at the exact moment a named
claim's decision is in flight; it kills the application process after a named
claim is decided and before its answer leaves; it restarts PostgreSQL while
decisions are in flight; it repeats submissions of already-decided claim
identities; it introduces a set of claims constructed so that their decisions
conflict pairwise and cannot all proceed; it holds a statement read open
across a burst of claims and limit changes; and it raises the overlap share
until refused work dominates and holds it there.

At the end every budget and every retained claim is read. No budget is
overspent at any point in the recorded history, every decision returned to a
client is present and unchanged, every accepted claim identity carries exactly
one decision, and every settled claim's committed amount matches its
settlement.

Checks observe only HTTP, SQL-visible state, exact request and decision
histories, process lifecycle, query plans, metrics, and the submitted
evidence. They do not require a particular schema, transaction structure, or
coordination primitive.

## Acceptance evidence

No budget is overspent at any point in the recorded history, recomputed from
the retained claim records rather than from any stored total. Every returned
decision is final and unchanged after every schedule, every accepted claim
identity carries exactly one decision, and a repeated submission returns that
same decision without a second effect. Every accepted claim reaches a decision
under the stated overlap, and none is starved past the declared bound. Failure
is returned to the client or exposed in queryable status rather than only
logged.

The report includes exact claim and decision histories; how many units of work
the store declined to complete, at what offered concurrency, and how that
count moved with the overlap share; the distribution of attempts per decided
claim and its worst case; the number of claims that reached the declared bound
and what their clients received; the offered rate against the delivered rate;
decision latency against the declared service level; and
`EXPLAIN (ANALYZE, BUFFERS)` for every required large-data query. It names
what the design gives up to hold the invariant, and the limit of the chosen
boundary.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **FoundationDB** — [documentation](https://apple.github.io/foundationdb/developer-guide.html).
  Neither reads nor writes block and conflicting transactions fail at commit
  time, but the client library owns the loop that repeats them, so the
  boundary this lab makes the design draw is drawn inside the data-access API
  and the application mostly never sees the refusal.
- **MySQL with InnoDB** — [documentation](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html).
  A conflicting writer blocks as soon as it tries to take a lock and does not
  proceed until the holder commits or rolls back, so contention arrives as
  waiting and lock ordering rather than as work handed back to be done again,
  and its cost shows up in latency instead of in refused units.
- **DynamoDB transactions** — [documentation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html).
  Cancels the whole grouped operation when it conflicts with another in
  flight, and caps one group at 100 items and 4 MB, so the set a single
  decision may read and write is bounded by the API rather than by the
  product's own rule.

The lab does not run them.

## Scope

The expected focused time is fourteen to twenty hours. PostgreSQL, the budget
and claim data, the overlap profile, the workloads, telemetry, and faults are
prepared. A simple design fails this lab at its scale target: one process
deciding claims one at a time holds the invariant but cannot hold 2,000
submissions per second from 500 clients, while a design that decides claims
concurrently on what it has read has its work handed back at a rate that
climbs with concurrency, so its delivered rate stops following its offered
rate well below the target. Multiple currencies, payment providers, approval
workflows, cross-organization budgets, and forecasting are outside the
problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  PostgreSQL, evidence, and retry contracts.
- [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
  — the page carries both halves of the quirk. A transaction refused for a
  dependency between what it read and what a concurrent transaction wrote
  reports `ERROR: could not serialize access due to read/write dependencies
  among transactions`; its guidance on a serialization failure is that an
  application "should abort the current transaction and retry the whole
  transaction from the beginning", and that "applications must not depend on
  results read during a transaction that later aborted; instead, they should
  retry the transaction until it succeeds". It also fixes that a sequential
  scan "will always necessitate a relation-level predicate lock", which raises
  the rate of refusals. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
