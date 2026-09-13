# Resilient quote service

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Envoy** — [documentation](https://www.envoyproxy.io/docs/envoy/latest/)
- **resilience4j** — [documentation](https://resilience4j.readme.io/docs/getting-started)
- **HAProxy** — [documentation](https://docs.haproxy.org/)

Stuck? See `hints/`.
