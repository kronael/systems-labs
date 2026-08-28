---
status: draft
---

# Resilient quote service

## Brief

Design and build a network service for air-travel fare search. A traveller's
search asks two independent fare providers; each answers with a quote — a
price the provider honours only until a stated expiry, because seats are held
behind it. The service returns the best quote still valid when the response
leaves, with its provider and the moment it stops being bookable. The service
must remain predictable when demand exceeds capacity or one provider becomes
slow.

The assignment is the whole service: public API, internal boundaries,
concurrency model, failure policy, telemetry, lifecycle, and end-to-end tests.
How concurrent work is organized, how overload is admitted or refused, and how
the service divides into processes are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts two deterministic fare-provider simulators —
each answers a search with seeded priced quotes carrying their expiries —
OpenTelemetry collection, a metrics backend, and the fault controller. The
course also supplies generated API types, an open-loop load generator, HDR
histogram output, and the fault schedules behind `make fault`.

The learner owns the application services and their Compose layer. The common
Makefile starts dependencies, validates scaffold health, runs tests, injects
faults, runs load, and collects evidence. No cloud account is required.

## Requirements

The system accepts a trip search and returns the best valid quote with its
provider and its expiry. The best valid quote is the lowest-priced one whose
expiry has not passed when the response is sent; between equal prices, the
quote that stays bookable longer wins. A quote past its expiry is never
returned: the provider no longer honours it, so a caller acting on it goes
after a fare that no longer exists — worse than an honest refusal the caller
can retry. When no valid result exists, the caller receives a typed non-2xx
response. Health distinguishes a live process from a service ready to accept
work.

The service has explicit limits for in-flight work, memory growth, queued work,
and request duration. It propagates cancellation, stops accepting work during
shutdown, and accounts for admitted requests before exit. Configuration comes
from the standard TOML contract.

Telemetry distinguishes offered traffic, accepted traffic, completed work,
rejection, timeout, cancellation, work waiting to start, local execution, and
provider time. The system must make overload visible to clients and operators.

The scale target is a sustained 2,000 requests per second of open-loop offered
traffic, 500 concurrent client connections, and one million requests per
evidence run, with bursts that drive the offered rate above the measured local
capacity. The numbers are the shape of fare search: the search path absorbs
the traffic the booking path never sees — every price displayed upstream lands
here as a search, and a fare sale turns into a burst with no warning — while
each quote stays bookable for minutes, not hours. Latency and rejection are
judged against the calibrated capacity and the learner's declared service
level, not a fixed number.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- where the design decides whether to accept new work, and what resource that
  decision protects;
- how provider calls run, cancel, and combine partial results;
- whether work waits, rejects, sheds, or degrades when capacity is exhausted;
- how deadlines interact across the client, service, and providers;
- how the design guarantees the returned quote is still valid at the moment
  the response is sent, given that a provider's answer can arrive close to
  that quote's expiry;
- how shutdown accounts for accepted and unfinished requests;
- which measurements distinguish slow execution from waiting to execute;
- how the capacity bound was derived from evidence.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

The failure schedule applies a sustained latency increase to one provider at an exact
request boundary, injects provider errors, drives a burst above measured
capacity, cancels clients, and sends SIGTERM while work is active. It runs both
open-loop and closed-loop traffic.

Checks do not inspect private functions or require a named concurrency
pattern. They observe API behavior, the expiry stamped on each returned quote,
process lifecycle, traces, metrics, queue or admission state exposed by the
design, and resource use.

## Acceptance evidence

The product must answer valid requests, surface complete failure, recover after
the provider returns to normal, and shut down without silently discarding
accepted work. No returned quote is past its expiry at the moment the response
is sent, even while one provider is slow. Under sustained overload, memory and
waiting work remain within the submitted structural bounds.

The evidence report includes offered and achieved throughput, p50/p95/p99,
provider time, pre-execution delay, rejection, timeout, peak in-flight work,
and peak resident memory. It explains the difference between open-loop and
closed-loop results and ties the architecture decision to the measurements.

The admitted fraction of offered traffic is reported and defended against the
calibrated local capacity. A design that rejects nearly everything has not met
the scale target, and the evidence must make that visible.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Envoy** moves admission control, retry budgets, and outlier detection into
  a proxy in front of the service, so the overload policy lives in
  configuration and the application never learns its own capacity.
- **resilience4j**, and Hystrix before it, packages the failure policy as named
  per-call-site primitives, which fixes the isolation boundaries before any
  measurement has shown where the actual bottleneck sits.
- **HAProxy** caps connections and request rates at the edge, so excess traffic
  is refused before it reaches the service rather than handled inside it.

Read their documentation on admission control, circuit breaking, and load
shedding. The lab does not run them.

## Scope

The expected focused time is six to ten hours. The learner builds the quote
service and its tests. Provider simulators, telemetry, workload, and fault
injection are prepared. Booking, payment, seat selection, and everything after
the search are outside the problem, as are databases, caches, retries,
Kubernetes, and browser UI.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [`../0/6-serverless-contrast-track.md`](../0/6-serverless-contrast-track.md) —
  the serverless recast of this product and what it removes.
- [Duffel API reference — Offers](https://duffel.com/docs/api/offers) — the
  reported behaviour the product's expiry rests on: "An offer is only
  available to create an order for a limited time by the traveller before it
  expires, typically within 30 minutes", and past its stated expiry an offer
  "can no longer be used to create an order".
- [wrk2](https://github.com/giltene/wrk2) — a closed-loop generator sends its
  next request only after the previous response arrives, so it coordinates
  with the server and stops measuring during exactly the slow periods it
  exists to find. Solution-bearing: this belongs in HINTS.md, never in
  README.md.
- Implementation pointers do not exist while the spec is `draft`.
