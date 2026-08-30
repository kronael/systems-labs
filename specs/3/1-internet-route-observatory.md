---
status: draft
---

# Internet route observatory

## Brief

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

## Prepared scaffold

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

## Architecture questions

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

## Adversarial evaluation

The failure schedule drops a named peer session after a named update is
acknowledged and restores it later, withdraws a prefix at one collector while
a second collector still announces it, replays a convergence burst at a named
prefix whose transient paths must not surface as routing changes, sends
malformed paths, kills processing at a named Kafka offset mid-stream, and
disconnects the optional recorder. Every fault fires at a named barrier, never
on a timer and never at random.

Checks observe source and API contracts, Kafka-visible identities, queryable
state, scope statements, traces, freshness, lag, resource bounds, and exact
accepted histories. They do not require a particular database schema or
streaming framework.

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

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **RouteViews** — [documentation](https://www.routeviews.org/routeviews/).
  Operates a second, independently peered collector fleet — over a thousand
  peers at exchange points and partner networks — published as MRT dumps on
  a schedule; the same prefix can look different there than from RIS,
  because coverage is a property of the peer population, not of the prefix.
- **CAIDA BGPStream** — [documentation](https://bgpstream.caida.org/).
  Merges archived dumps and live collectors behind one programming
  interface, which answers the replay-versus-live question in the toolchain
  instead of in the pipeline.
- **RIPE Atlas** — [documentation](https://atlas.ripe.net/docs/). Observes
  from probes hosted in roughly 3,300 of the more than 70,000 ASes and
  measures what packets actually do, where a route collector records what a
  few hundred peering ASes announce; each is a sample with its own bias, and
  neither is the Internet.

The lab does not run them.

## Scope

The expected focused time is ten to fourteen hours. Kafka, data sources,
telemetry, store profiles, workloads, and faults are prepared, but the scope
model, settling criterion, and process decomposition are not, and a first
design that treats a collector's view as ground truth is falsified and
rebuilt at least once against the supplied vantage-point set. Implementing
BGP, decoding MRT or BGP wire formats, RPKI validation, global anomaly
verdicts, alert delivery, and a map UI is outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, Kafka,
  real-data, source, and evidence contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- Quirk origination. The falsified belief is that the collector's view is the
  Internet's state and that an update means the routing changed. Both halves
  have primary sources:
  - [RIS Live manual](https://ris-live.ripe.net/manual/) — a message's
    timestamp is the collector's receipt time, ordering is guaranteed only
    within one peering session, and the stream carries collector metadata
    (`RIS_PEER_STATE` session-state messages) beside relayed BGP messages.
  - [RIS route collectors](https://ris.ripe.net/docs/route-collectors/) — the
    fleet is about two dozen collectors, most peering at one exchange point
    plus a few multihop collectors, so each collector's view is fixed by
    which peers connect to it, and some collectors are explicitly regional.
  - [RFC 4271, section 3](https://www.rfc-editor.org/rfc/rfc4271) — a BGP
    speaker advertises to its peers only the routes it uses itself, so a
    collector's peer reveals one selected path per prefix, never everything
    the peer knows.
  - [RouteViews](https://www.routeviews.org/routeviews/) — an independent
    collector fleet with its own peer population of over a thousand peers;
    the vantage-point coverage contrast in the neighbouring-systems reading.
  - Solution-bearing, for `HINTS.md` and never `README.md`: [Labovitz, Ahuja,
    Bose, and Jahanian, *Delayed Internet Routing Convergence*, SIGCOMM
    2000](https://conferences.sigcomm.org/sigcomm/2000/conf/paper/sigcomm2000-5-2.pdf)
    — measured failovers averaged three minutes, oscillations ran up to
    fifteen minutes and tens of minutes at worst, and the rate-limiting
    advertisement timer shapes the bursts, so a burst of updates is
    exploration through transient paths rather than change.
  - Solution-bearing, for `HINTS.md` and never `README.md`: [Oliveira, Pei,
    Willinger, Zhang, and Zhang, *Quantifying the Completeness of the
    Observed Internet AS-level Structure*, UCLA TR-080026,
    2008](https://web.cs.ucla.edu/~lixia/papers/08completeness-TR.pdf) — the
    public view from RouteViews and RIS vantage points reveals the full peer
    connectivity of only 4 percent of ASes, because export policy bounds what
    any vantage point can see; this is what the observatory can never
    conclude.
  - Solution-bearing, for `HINTS.md` and never `README.md`: [Sermpezis et
    al., *Bias in Internet Measurement Infrastructure*, RIPE
    Labs](https://labs.ripe.net/author/pavlos_sermpezis/bias-in-internet-measurement-infrastructure/)
    — RIS and RouteViews collect feeds from roughly 300 and 500 peering ASes
    out of more than 70,000, skewed toward large networks and exchange
    points, so the sample is biased as well as small.
- Implementation pointers do not exist while the spec is `draft`.
