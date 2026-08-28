---
status: draft
---

# Metered billing API

## Brief

Design and build a subscription billing service that manages customer
subscriptions, meters usage, and issues an invoice for every active
subscription at the close of each billing period — running entirely as
event-driven functions that keep their state in an external store, with no
long-lived process anywhere.

The execution model is the lesson. The platform runs each request in an
isolated environment that handles one invocation at a time, freezes when the
runtime and every extension have completed with no events pending — a handler
can return before that point — and thaws for a later invocation, the same
caller's or anyone's. Work still unfinished at the freeze stops with it and
resumes minutes later, under another request, or never. State the process
accumulated for one caller is still there for the next. Environments are
recycled even under continuous traffic, and an invocation is killed at its
ceiling. None of this is a fault; it is the documented contract.

The assignment is the whole service: public behavior, state design, use of
the queue, the period-close path, and end-to-end tests. The function
decomposition, the store's data layout, the treatment of repeated deliveries,
the resumption of interrupted work, and the division of work larger than one
invocation are the learner's decisions.

## Prepared scaffold

The supplied Compose stack extends the standard dependency profile with a
Lambda-compatible function runner, an SQS-compatible queue, DynamoDB Local,
the workload generator, and the fault controller. The environment must
provide the platform's documented lifecycle: initialization once per
environment, one in-flight invocation per environment, a freeze once the
runtime and every extension have completed with no events pending, thaw on
reuse, environment recycling at named barriers, an invocation ceiling (the
platform's 15-minute maximum, scaled down by lab config so the boundary is
reachable in a test run), and the platform's payload ceilings. Detecting the
freeze barrier on the local runner is an open question recorded in the
[shared scaffold](../0/5-shared-scaffold.md); this lab states the requirement
and does not assert that the scaffold has established the capability. Two further platform behaviours
are environment facts here rather than this lab's subject: work above the
fixed concurrency cap is rejected, and the queue that feeds deferred work
delivers batches at least once. Their contracts are the study of other labs
in this phase. A long-running container is not an accepted substitute,
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

## Architecture questions

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

## Adversarial evaluation

The failure schedule replays a scripted history against the public API and the queue
while firing faults at named barriers: it freezes the environment at the
freeze barrier following the response for invoice 4711, with work observably
outstanding; recycles the environment between the two requests of one logical
operation on account 2205; kills the invocation carrying subscription
118207 at the invocation ceiling; and redelivers the queue message carrying
usage event 771003 after its work is acknowledged — the queue's delivery
contract is another lab's subject, and the redelivery is an environment fact
here that the invariant must survive. It then closes a period under continued
traffic and reads every invoice.

Checks observe public responses, queue histories, and store contents. They
do not inspect private functions and do not require a named pattern. They
compare every invoice line against an independently computed answer derived
from the acknowledged input under the supplied rating rules.

## Acceptance evidence

Every subscription active in the closed period appears exactly once in that
period's invoices, by identity. Every acknowledged usage event and plan
change appears on exactly one line. The named faults leave no doubled charge,
no lost charge, and no cross-account response. The close completes across
many invocations and survives a killed one without repeating or skipping an
account.

The report contains cold and warm latency distributions against the declared
service level, invocation counts, concurrent executions over time, and the
completion lag of the period close. It names the operations that pay the cold
path and the residual limit the design accepts.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Google Cloud Run** serves the same scale-to-zero request shape from a
  container, and its CPU-allocation setting is this lab's boundary made
  configurable: request-based billing throttles the CPU once the response is
  sent, instance-based billing keeps it computing — for a price.
- **AWS Step Functions** is the vendor's own answer to work larger than one
  invocation: a Standard workflow records each step durably and runs for up
  to a year, so resumption stops being the function's problem and becomes a
  second orchestration surface to own.
- **Temporal** generalizes that answer: it records every effect in an event
  history and replays it after a crash, so code appears to run for months
  across process deaths — at the cost of operating a second stateful
  platform.

Read their documentation on lifecycle, state, and background work. The lab
does not run them.

## Scope

The expected focused time is fifteen to twenty-five hours. The learner builds
the functions, the state design, the queue usage, and the tests. The runner, the
queue, the store, the generator, and the fault schedules are prepared. Every required gate runs locally with no AWS account. The optional
smoke run behind `make smoke` uses Lambda,
SQS, DynamoDB on-demand, and short-retention logs only, with a stated
invocation and dollar ceiling; provisioned concurrency is excluded by the
cost contract, so a cold start cannot be bought away. Payment collection,
card networks, taxes, currency conversion, dunning, and API gateways are
outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, the
  Lambda execution-shape policy, cost, grading, and evidence contracts.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  why this lab comes first and has no phase 1 partner.
- [`2-reliable-record-import.md`](2-reliable-record-import.md) — the
  neighbouring queue-driven lab, whose subject is the queue's delivery contract
  rather than the execution model.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — the environment freezes when the runtime and each extension have completed
  and there are no pending events, which can be after the handler has
  returned, and thaws on the next invocation, so unfinished work resumes
  under another caller or never, and process state stays visible to whoever
  arrives next.
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
  — the 900-second ceiling, the payload limits, and the default account
  concurrency.
- [Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
  — one in-flight request per environment, and throttling once concurrency
  is exhausted.
- Implementation pointers do not exist while the spec is `draft`.
