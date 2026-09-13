# Shared budget service

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

The document must compare at least two viable decision designs without turning
a known library name into the argument, and state one residual limitation.

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **FoundationDB** — [documentation](https://apple.github.io/foundationdb/developer-guide.html)
- **MySQL with InnoDB** — [documentation](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html)
- **DynamoDB transactions** — [documentation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)

## What is outside the problem

The expected focused time is fourteen to twenty hours. PostgreSQL, the budget
and claim data, the overlap profile, the workloads, telemetry, and faults are
prepared. Multiple currencies, payment providers, approval workflows, cross-
organization budgets, and forecasting are outside the problem.

Stuck? See `hints/`.
