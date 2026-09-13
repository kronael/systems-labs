# Internet route observatory

Design and build an operational view of Internet routing observations. The
system consumes BGP update envelopes from many collectors and peer sessions
and answers what is observed for a prefix and from which vantage points, where
vantage points currently disagree, which prefixes have settled onto a changed
route, whether collectors and peer sessions are connected and fresh, and
whether processing is keeping up. Every answer carries the observation scope
it rests on: which vantage points contributed, and what the system could not
see from them.

The required pipeline uses Kafka and accepts the supplied RIPE RIS
Live-compatible record contract; envelope decoding is prepared, and BGP wire
formats are not learner work. A bounded real RIS recording is optional; the
deterministic generator proves every requirement. The data layout, how
observation state and scope are represented, how a settled change is told
apart from transient churn, the process decomposition, and the choice of
query store are the learner's decisions.

## What you are given

The supplied Compose stack starts Kafka, OpenTelemetry collection, a query-
store slot, and the fault controller. It includes generated multi-collector
BGP observations, an opt-in bounded RIS recorder, immutable replay, provenance
manifests, session-flap, divergent-vantage, and convergence-burst scenarios,
and broker inspection.

The learner selects the supported query store from the prepared PostgreSQL or
embedded persistent-store profiles and owns all application services and their
Compose layer. Standard Make targets start the chosen profile, record or replay
input, run faults and load, and collect evidence. CI never contacts RIPE.

## Requirements

Every API response states the observation scope it answers from and its
freshness. The system may not present collector data as one authoritative
global route. A prefix absent from one vantage point while another still
observes it is a disagreement between views, not a vanished route, and the
answer must say which the system is seeing. A reported routing change must be
distinguishable from path exploration: the transient paths a converging
router announces before settling may not be published as changes. Malformed
records have an explicit, testable outcome.

The product exposes observed origins and paths per vantage point, settled
changes, disagreement between vantage points, collector and peer-session
health, rejected input, and processing lag. A peer session that drops and
later returns changes what the system can claim while it is down, and the
affected answers must say so. Restart and consumer replacement lose no
accepted update. Memory and internal waiting work remain bounded under a
convergence burst or collector loss.

Generated and cached real input pass the same source contract. Provider time,
local observation time, raw-record checksum, schema version, and provenance
remain traceable through the result or its evidence.

The scale target is 200 million retained updates from 20 collectors and 400
peer sessions, a sustained 5,000 updates per second with bursts of 50,000, one
million distinct prefixes, and 1 percent of prefixes carrying half of all
update volume. These numbers size the problem; they are not pass thresholds.
Freshness and lag gates stay relative, structural, or declared by the
learner, not a fixed number.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what one routing observation means and which dimensions define its scope;
- what an answer claims when vantage points disagree about a prefix, and what
  the absence of a prefix from one vantage point is evidence of;
- how a withdrawal seen at one collector changes the answer without erasing
  what other collectors still observe;
- how a settled route change is distinguished from path exploration during
  convergence, and what evidence backs the distinction;
- what a dropped peer session makes unknowable, and how answers account for
  that gap after the session returns;
- how malformed or unsupported records remain inspectable;
- what the observatory can never conclude from its vantage points, however
  long it watches.

The architecture must compare at least two representations of observation
scope and settledness against the supplied vantage-point set and workload
distribution.

## Acceptance evidence

For every supplied schedule the API returns the exact observed origins and
paths per prefix per vantage point and the exact scope statement each answer
carried, never counts alone. The prefix withdrawn at one collector remains
reported as observed at the other; the convergence burst settles to the exact
final path with no transient path in the change history; answers touched by
the dropped peer session state the gap in what was observable. Accepted
identities survive restart, rejected identities have reasons, live-source
interruption is visible, and the system recovers from the burst without an
unbounded stale backlog.

The report records provenance, provider and observation times, Kafka identity
and position, per-vantage-point observation histories, disagreement windows
with their scope statements, quarantine reasons, processing lag, and API
freshness. It documents the limits of what the vantage points can support.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`HINTS.md`, because saying it here would point straight at the answer.

- **RouteViews** — [documentation](https://www.routeviews.org/routeviews/)
- **CAIDA BGPStream** — [documentation](https://bgpstream.caida.org/)
- **RIPE Atlas** — [documentation](https://atlas.ripe.net/docs/)

Stuck? See `HINTS.md`.
