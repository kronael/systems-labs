# Spatial query service

Design and build a system that answers spatial questions over a prepared map
extract: which features lie inside a given area, which features lie near a
given point and match a text filter, and which features in an area changed
since a given time.

Every answer must be exact with respect to the data the service has accepted.
A feature belongs in an area answer because its geometry lies in the area, and
a feature that merely comes close must never appear. A proximity answer ranks
by real distance on the ground, stated in metres, and means the same thing
beside the equator, at high latitude, and across the antimeridian. A change
answer names exactly the features created, modified, and deleted, and every
answer states the edit horizon it reflects.

The assignment is the whole service: the query API, what each prepared store
holds and when, the path that applies edits while queries run, and end-to-end
tests. What each store holds and computes, how the two stay honestly
consistent as edits arrive, and the coordinate and distance model behind every
answer are the learner's decisions. Reading the map format is not part of the
problem; the extract arrives loaded.

## What you are given

The supplied Compose stack starts PostGIS with the extract already loaded, an
empty OpenSearch, the feature and edit generator, the source replay for cached
OpenStreetMap extracts, and the fault controller. The generator produces a
seeded world-spanning extract and a continuous edit stream — creates,
modifications, and deletions at declared ratios — and every feature carries a
stable identity and version so verification can assert histories. The seed
deterministically places the boundary clusters the schedules name: features
straddling the antimeridian and features above 80° latitude.

The learner owns the query service, everything the search engine comes to
contain, and the edit application path. No cloud account is required.

## Requirements

The service exposes at most four public operations covering three query
classes: features inside a region, features near a point, and features changed
in a region since a given time.

Area answers contain exactly the features whose geometry lies in the queried
region, wherever on the globe that region sits — including regions that span
the antimeridian and regions above 80° latitude. Proximity answers treat the
radius as metres on the ground at any latitude and rank by true distance. The
text filter on proximity queries matches words and word prefixes,
case-insensitively, across the several languages present in the extract.
Change answers name exact feature identities, and an edit acknowledged at or
before an answer's stated horizon appears in that answer.

Edits keep arriving while queries run. The service does not stop accepting
edits to make a query correct, and each answer states the horizon it reflects
rather than silently serving an unstated mixture of old and new.

The scale target is 30 million features spanning latitudes to ±85° and
longitudes across the antimeridian, a sustained mixed load of 300 queries per
second from 100 concurrent clients, and a continuous 100 edits per second.
These three numbers size the problem — the speed and the amount together rule
out answering a query by examining every feature, and the extent rules out a
design that is only correct near the equator. They are not pass thresholds;
thresholds stay relative, calibrated, structural, or learner-declared.

The evidence must show the sustained query rate, latency by query class
against the learner's declared service level, the lag from an edit's
acceptance to its visibility in each query class, and the divergence between a
trusting answer and the exact answer at the named placements. How those
measurements are produced is the learner's choice; no telemetry stack is
required.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what the fast spatial lookup in each store actually returns, and what stands
  between that result and a correct answer;
- which store is authoritative for which fact, and what a reader sees at the
  moments when the two disagree;
- what a distance is in this service: where its unit comes from, how it is
  computed, and what changes far from the equator;
- how a region that crosses the antimeridian is represented, and why that
  representation returns the features inside it rather than the features
  outside it;
- what accepting an edit means: at what moment it becomes visible to each of
  the three query classes, and what the horizon statement promises between
  those moments;
- what each query class costs, and how that cost grows with extent, feature
  count, and accumulated edit history;
- what an in-flight query returns when a store restarts, and what a client may
  rely on immediately afterwards.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

Every answer matches the independently computed exact identity set for its
stated horizon at every query point, including the antimeridian region, the
high-latitude radius, and the pair around a named edit. The named near-boundary
features fall on the correct side in every run. A query in flight across a
store restart either completes exactly or fails loudly; no partial or silently
degraded answer is accepted, and the first query after recovery answers
exactly.

The evidence report includes the sustained mixed query rate, latency by query
class against the declared service level, the edit-to-visibility lag for each
query class, and, for each named placement, the feature identities a design
that trusted its fast lookup would have returned wrongly. It names the
residual limitation.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **MongoDB** — [documentation](https://www.mongodb.com/docs/manual/geospatial-queries/)
- **H3** — [documentation](https://h3geo.org/docs/)
- **Tile38** — [documentation](https://tile38.com/)

## What is outside the problem

The expected focused time is eighteen to twenty-two hours; phase 8 labs are
deliberately denser because the interaction between the search engine and the
domain store is the lesson, and three query classes each demanding exact
answers under antimeridian and high-latitude placements, held against a
continuously edited store, price out well above a phase 1-4 lab. The learner builds the query service, the content
and upkeep of the search engine, the edit application path, and the tests.
PostGIS, OpenSearch, the loaded extract, the generator, and the fault
schedules are prepared. Map rendering, tile serving, routing, and cartography
are outside the problem.

OpenStreetMap data is licensed under the ODbL: whoever alters or builds upon
the data may distribute the result only under the same licence, so a database
derived from an extract carries share-alike obligations. Real extracts —
per-region `.osm.pbf` files such as Geofabrik publishes — are opt-in through
`make source`, bounded, checksummed, cached under
`${PREFIX:-/srv}/data/systems-labs/sources/`, never redistributed by this
repository, and never fetched on a request path. CI and every required gate
use generated seeded input only and run locally with no cloud account.

Stuck? See `hints/`.
