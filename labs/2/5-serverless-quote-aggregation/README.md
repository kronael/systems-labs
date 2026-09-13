# Serverless quote aggregation

Design and build the fare-search service from
[`../1/1-resilient-quote-service.md`](../../1/1-resilient-quote-service/README.md) again,
on an execution environment that runs one request per instance, admits work
against a ceiling the platform enforces, and bills per request and for the
duration each invocation executes. A traveller's search asks two independent
fare providers; each answers with a quote — a price honoured only until a
stated expiry, because seats are held behind it — and the service returns the
best quote still valid when the response leaves, with its provider and its
expiry. It must remain predictable when demand exceeds the ceiling or one
provider becomes slow.

A quote carries a deadline of its own, and this execution model can stop the
process holding it. An environment freezes with a quote in memory and thaws
minutes later, so a value that was fresh when it was stored can be stale when
it is read, and the process that would have expired it was not running. That
is this lab's subject: a deadline the design does not own, crossing a pause
the design does not control.

The required environment is the function execution model with an external store
available for shared state. How provider calls are arranged within an
invocation, how the shared store is used, how the time budget is divided, and
how the service decides a remembered quote is still honourable are the
learner's decisions.

This lab is taken after the local one. The product is identical on purpose.

## What you are given

The supplied environment starts the local function runtime, the provider
simulators, an external store, the fault controller, and the open-loop load
generator.

The simulators answer searches with seeded priced quotes, each carrying its
expiry, and follow a latency and failure schedule. The fault controller
destroys environments to force first-invocation latency, holds a provider past
the handler's remaining time, throttles above the declared ceiling, and freezes
an environment at the freeze barrier — the point where the runtime and every
extension have completed with no events pending — with a provider call
outstanding.

The local runner does not freeze by itself. The fault controller's process
layer supplies the freeze, at the barrier the platform states: the runtime and
every extension complete with no events pending, which a returned response
alone does not mark. The controller holds the environment at that barrier,
confirms the suspension, and thaws it when the next invocation is assigned.

The course supplies the provider protocol and the fault schedules behind
`make fault`. The learner owns the handler and its tests. No cloud account is
required.

## Requirements

The service returns the best valid quote under the same rule as the local lab:
the lowest-priced quote whose expiry has not passed when the response is sent,
between equal prices the one that stays bookable longer. A quote past its
expiry is never returned — the provider no longer honours it — and the rule
holds while a provider is slow, failing, or answering with quotes already near
their expiries. When no valid quote exists, the caller receives a typed
non-2xx response. A provider that exceeds its budget must not extend the
response beyond what the design promises.

Process memory survives inside a reused environment and is visible to whoever
arrives next, but no request is guaranteed to land in a reused environment, so
memory can neither be relied on nor treated as private. Any state the design
needs to be visible across requests lives outside the instance, and the
submission states what a read of that state proves and what it does not.

A quote remembered across a freeze must never be returned past its expiry. The
design states how it establishes that a remembered quote is still valid, and
what it does when it cannot establish that.

The declared concurrency ceiling is an environment fact here rather than this
lab's subject: the platform rejects work above it, and the design states what a
caller observes. Fan-out is the part the design still owns — one request per
instance means provider load scales with environment count rather than with a
pool the service sizes, and the evidence run records that multiplier.

A handler has a maximum run time, and a provider call that outlives it is
terminated rather than completed. The design states what the caller sees in that
case and what happens to the outstanding call.

Configuration comes from the standard TOML contract. Instrumentation is required
here, because the measurement is the evidence.

The scale target is a sustained 2,000 requests per second of open-loop offered
traffic against a declared concurrency ceiling sized so that rate is several
times what the ceiling admits, 500 concurrent client connections, one million
requests per evidence run, and a provider latency schedule that drives at least
one provider past its budget for a sustained interval. The offered rate is
fare search's to absorb — the traffic the booking path never sees — and the
ceiling does not grow because a fare sale started. These numbers size the
problem; they are not pass thresholds. Latency and rejection are judged
against the declared ceiling and the learner's stated service level.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what replaced the local design's admission control, and what changed about the
  caller's experience;
- what state, if any, is visible across requests, where it lives, what a read
  of it proves, and what it costs per request;
- how the provider fan-out changes when it happens per instance rather than per
  process, and what that does to provider-side load;
- how the timeout budget is divided between the providers and the handler's own
  ceiling;
- what a first invocation costs, how large a share of the tail it is, and
  whether the design tries to reduce it or to absorb it;
- how much of a quote's validity is already spent by the time the caller sees
  it, and how first invocations change that share;
- which measurement separates a slow provider from a throttled service, given
  that both appear to the caller as a failure to answer;
- at what offered rate this design becomes more expensive than the local one,
  and why.

The document must compare the local design and this one directly.

## Acceptance evidence

The service answers within its stated service level while a provider is slow,
and its behavior above the ceiling matches what the design declared. No response
outlives the promised budget, and no returned quote is past its expiry at the
moment the response is sent. Provider-side load stays inside the stated bound
while environments are being created and destroyed.

The freeze in that schedule comes from the controller's process layer. The
required gate therefore proves a modelled freeze rather than the hosted
service's own behaviour, and `EVALUATION.md` states that. Where an account
exists, `make smoke` compares the modelled freeze against the service.

The admitted fraction of offered traffic is reported and defended against the
measured local capacity. A design that rejects nearly everything has not met
the scale target, and the evidence must make that visible.

The evidence report includes latency as an HDR histogram separated into first
and reused invocations, offered rate against admitted and rejected rate, provider
request count per client request, shared-state read cost per request, and the
share of the tail attributable to first invocations. It compares its own
throughput and latency against the local lab's recorded baseline on the same
host, and names the offered rate at which this design stops being the cheaper
one.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Google Cloud Run** — [documentation](https://cloud.google.com/run/docs)
- **Amazon API Gateway caching** — [documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-caching.html)
- **Istio** — [documentation](https://istio.io/latest/docs/)

## What is outside the problem

The expected focused time is six to nine hours. The learner builds the handler
and its tests. The runtime, provider simulators, store, load generator, and fault
schedules are prepared. Booking, payment, seat selection, authentication,
provider onboarding, billing, and multi-region routing are outside the
problem.

The local quote lab is a prerequisite, and its artifacts must be retained: its
`ARCHITECTURE.md` and its recorded baseline, produced on the same host where
this lab's evidence run happens. The acceptance evidence compares against that
baseline; without it the required evidence cannot be produced.

Stuck? See `hints/`.
