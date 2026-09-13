# Metered billing API

Design and build a subscription billing service that manages customer
subscriptions, meters usage, and issues an invoice for every active
subscription at the close of each billing period — running entirely as
event-driven functions that keep their state in an external store, with no
long-lived process anywhere.

The period close is the lesson. This lab is taken after the recasts, so the
execution model's lifecycle is already a contract the learner has designed
against — the freeze, environment reuse, and the invocation ceiling arrive
here as environment facts, not as new material. What is new is a unit of work
that does not fit an invocation: every active subscription must be invoiced at
the close of a billing period, and the close is larger than the ceiling allows
one invocation to finish. Work still unfinished at the freeze stops with it
and resumes minutes later, under another request, or never. How the close is
divided, resumed, and proven complete is the learner's decision.

The assignment is the whole service: public behavior, state design, use of
the queue, the period-close path, and end-to-end tests. The function
decomposition, the store's data layout, the treatment of repeated deliveries,
the resumption of interrupted work, and the division of work larger than one
invocation are the learner's decisions.

## What you are given

The supplied Compose stack extends the standard dependency profile with a
Lambda-compatible function runner, an SQS-compatible queue, DynamoDB Local,
the workload generator, and the fault controller. The environment must
provide the platform's documented lifecycle: initialization once per
environment, one in-flight invocation per environment, a freeze once the
runtime and every extension have completed with no events pending, thaw on
reuse, environment recycling at named barriers, an invocation ceiling (the
platform's 15-minute maximum, scaled down by lab config so the boundary is
reachable in a test run), and the platform's payload ceilings. The local
runner does not freeze by itself. The fault controller's process layer
supplies the freeze, at the barrier the platform states: the runtime and every
extension complete with no events pending, which a returned response alone
does not mark. The controller holds the environment at that barrier, confirms
the suspension, and thaws it when the next invocation is assigned; the
mechanism is in the [shared scaffold](../0/5-shared-scaffold.md). Two further
platform behaviours are environment facts here rather than this lab's subject:
work above the fixed concurrency cap is rejected, and the queue that feeds
deferred work delivers batches at least once. Their contracts are the study of
other labs in this phase. A long-running container is not an accepted substitute,
because the lifecycle is the subject.

The rating rules are part of the supplied product definition, published with
the lab: the plan catalog and what each plan costs, how a mid-period plan
change prorates the split period, the rounding rule, and what constitutes an
invoice line. The independently computed answer and the learner's
invoices both derive from these published rules and the acknowledged input.
The arithmetic is deliberately not this lab's difficulty, and none of it is
secret.

The learner owns every function, the state design, and the use of the queue.
No cloud account is required.

## Requirements

The service exposes four operations: create or change a subscription, where a
plan change takes effect mid-period and both parts of the split period are
billed as the supplied rating rules prescribe; record usage events against a
subscription; query an
account's invoice for a period, including one still being assembled; and
close a billing period, after which every subscription active in that period
is invoiced exactly once.

An acknowledged operation is permanent. An acknowledged usage event appears
on exactly one invoice line, and an acknowledged plan change is billed
exactly once. Redelivery, a retried request, a killed invocation, and a
frozen environment may not double a charge or lose one. A query against an
invoice still being assembled says so rather than presenting a partial total
as final.

No account's data may appear in another account's response — including
through state that a previous invocation left behind in a reused environment.
An account whose invoice exceeds one response payload must still be fully
readable.

One belief is this lab's study: that the platform keeps executing a program
until its work is done. The frozen environment falsifies it, and what the
freeze leaves behind — process state visible to whoever arrives next — comes
with it. The rest are constraints the same schedule exercises rather than
separate studies: a close larger than one invocation must still complete, an
invocation killed at the ceiling must be survivable, and no reused
environment may leak one account's data into another's response.

The scale target is 250,000 active subscriptions across 5,000 accounts, 40
million usage events per billing period, and a sustained 200 requests per
second against the fixed cap of 128 environments, with the period close
running while that traffic continues. The amount is sized so the close cannot
finish inside the configured invocation ceiling. These numbers size the
problem; they are not pass thresholds. Latency is measured against the
learner's declared service level, cold and warm paths separately.

The evidence must report cold-start and warm latency, invocation counts,
concurrent executions over time, and the lag from the period-close trigger to
the last invoice. How the numbers are produced is the learner's choice; no
telemetry stack is required.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- how a period close larger than one invocation completes without repeating
  or skipping an account — across the loss of any environment between two
  invocations, and across the unit of work that is killed at the ceiling;
- which state, if any, is safe to keep inside the environment across
  invocations, and which is not;
- which operations are allowed to pay the cold path, which must not, and what
  each costs against the declared service level;
- what the freshness statement on an assembling invoice means operationally,
  and how it is derived;
- which platform behaviors the local runner cannot prove about the hosted
  service.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

Every subscription active in the closed period appears exactly once in that
period's invoices, by identity. Every acknowledged usage event and plan
change appears on exactly one line. The named faults leave no doubled charge,
no lost charge, and no cross-account response. The close completes across
many invocations and survives a killed one without repeating or skipping an
account.

The freeze in that schedule comes from the controller's process layer. The
required gate therefore proves a modelled freeze rather than the hosted
service's own behaviour, and `EVALUATION.md` states that. Where an account
exists, `make smoke` compares the modelled freeze against the service.

The report contains cold and warm latency distributions against the declared
service level, invocation counts, concurrent executions over time, and the
completion lag of the period close. It names the operations that pay the cold
path and the residual limit the design accepts.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Google Cloud Run** — [documentation](https://cloud.google.com/run/docs)
- **AWS Step Functions** — [documentation](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- **Temporal** — [documentation](https://docs.temporal.io/)

Stuck? See `hints/`.
