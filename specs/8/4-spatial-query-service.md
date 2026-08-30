---
status: draft
---

# Spatial query service

## Brief

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

## Prepared scaffold

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

## Architecture questions

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

`HINTS.md`-bound, because each presupposes a mechanism: what a store's fast
spatial lookup returns before a correct answer is formed, and which of two
stores is authoritative for which fact.

## Adversarial evaluation

Verification replays the seeded extract and edit schedule, then issues queries
whose answers it has computed independently with a geodesic implementation
that shares no code with either store. Correctness is exact feature
identities, never counts alone, and every answer is judged against the horizon
that answer states.

Hard cases and faults fire at named barriers, never on a timer and never at
random:

- an area query whose region crosses the antimeridian, placed so that named
  feature 810017 lies inside it and the feature at its mirrored longitude lies
  outside; a design that inverts the region returns almost the whole extract
  or almost nothing;
- a proximity query at 84° latitude whose radius includes named features that
  flat-map arithmetic excludes and excludes ones it includes; the placements
  are chosen so that every reasonable earth model agrees on the exact answer
  while planar and projected arithmetic disagree grossly;
- edit 4711 changes a named feature's geometry; when the acknowledgement of
  edit 4711 is observed, the same area query is issued twice, and the pair
  must be consistent with the horizons the two answers state;
- the spatial store, and in a separate scenario the search engine, is
  restarted while a named query is in flight.

Verification does not inspect private functions and does not require a named
index, projection, store role, or consistency mechanism.

## Acceptance evidence

Every answer matches the independently computed exact identity set for its
stated horizon at every query point, including the antimeridian region, the
high-latitude radius, and the pair around edit 4711. The named near-boundary
features fall on the correct side in every run. A query in flight across a
store restart either completes exactly or fails loudly; no partial or silently
degraded answer is accepted, and the first query after recovery answers
exactly.

The evidence report includes the sustained mixed query rate, latency by query
class against the declared service level, the edit-to-visibility lag for each
query class, and, for each named placement, the feature identities a design
that trusted its fast lookup would have returned wrongly. It names the
residual limitation.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **MongoDB** interprets GeoJSON on the WGS84 sphere through its `2dsphere`
  index, so the degrees-are-not-metres trap never arises — the store fixes the
  earth model instead of leaving it as a design choice, and offers a smaller
  predicate surface (`$geoWithin`, `$geoIntersects`, `$nearSphere`) in
  exchange. See the [geospatial queries
  documentation](https://www.mongodb.com/docs/manual/geospatial-queries/).
- **H3** replaces geometry with a hierarchical hexagonal grid: containment
  becomes cell membership, and because child cells are not entirely contained
  by their parents, that membership is approximate by construction — the
  filter is the final answer, carrying a stated error, and no exact predicate
  ever runs behind it. See the [H3 documentation](https://h3geo.org/docs/).
- **Tile38** turns proximity into an event: a geofencing server that pushes
  Nearby, Within, and Intersects notifications to webhooks and queues, moving
  the product from querying durable history to watching a live boundary. See
  the [Tile38 documentation](https://tile38.com/).

The lab does not run them.

## Scope

The expected focused time is eighteen to twenty-two hours; phase 8 labs are
deliberately denser because the interaction between the search engine and the
domain store is the lesson, and three query classes each demanding exact
answers under antimeridian and high-latitude placements, held against a
continuously edited store, price out well above a phase 1-5 lab. The learner builds the query service, the content
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

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/4-search-and-retrieval-track.md`](../0/4-search-and-retrieval-track.md)
  — the track record holding this candidate and its origination.
- [The `&&` operator](https://postgis.net/docs/geometry_overlaps.html) —
  returns `TRUE` when the 2D bounding box of one geometry intersects the 2D
  bounding box of the other, and that is all it tests.
- [Spatial indexing
  workshop](https://postgis.net/workshops/postgis-intro/indexing.html) —
  spatial indexes index the bounding boxes of features, not the features; an
  index-only query over one neighbourhood counts 49,821 people where the exact
  answer is 26,718. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- [`ST_Intersects`](https://postgis.net/docs/ST_Intersects.html) — the exact
  intersection predicate, which "automatically includes a bounding box
  comparison that makes use of any spatial indexes". Solution-bearing: this
  belongs in `HINTS.md`, never in `README.md`.
- [`ST_Contains`](https://postgis.net/docs/ST_Contains.html) — containment
  means every point of B lies in A and the interiors share a point, with the
  same automatic index comparison. Solution-bearing: this belongs in
  `HINTS.md`, never in `README.md`.
- [`ST_DWithin`](https://postgis.net/docs/ST_DWithin.html) — the radius
  predicate whose distance is in spatial-reference units for `geometry` and in
  metres for `geography`. Solution-bearing: this belongs in `HINTS.md`, never
  in `README.md`.
- [`ST_Distance`](https://postgis.net/docs/ST_Distance.html) — `geometry`
  distance comes back in the units of the spatial reference system, so SRID
  4326 yields degrees, while `geography` yields geodesic metres.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [Geography
  workshop](https://postgis.net/workshops/postgis-intro/geography.html) — "a
  distance of 122 degrees" between coordinates "is a nonsense number", degree
  squares shrink toward the poles, and the shortest Cartesian route from Los
  Angeles to Tokyo crosses the Atlantic. Solution-bearing: this belongs in
  `HINTS.md`, never in `README.md`.
- [EPSG:3857](https://epsg.io/3857) — pseudo-Mercator is "not a recognised
  geodetic system", is defined only between 85.06°S and 85.06°N, differs from
  true Mercator by errors of 0.7 percent in scale and up to 21 km on the
  ground, and exists for web mapping and visualisation, not measurement.
- [RFC 7946, sections 3.1.9 and
  5.2](https://datatracker.ietf.org/doc/html/rfc7946) — a geometry crossing
  the antimeridian should be cut in two, and a bounding box spanning it has a
  western edge numerically greater than its eastern edge. Solution-bearing:
  this belongs in `HINTS.md`, never in `README.md`.
- [OpenSearch geodistance
  query](https://docs.opensearch.org/latest/query-dsl/geo-and-xy/geodistance/)
  — `distance_type` is `arc` by default; `plane` is "faster but inaccurate for
  long distances or points close to the poles". Solution-bearing: this belongs
  in `HINTS.md`, never in `README.md`.
- [OpenSearch geoshape
  query](https://docs.opensearch.org/latest/query-dsl/geo-and-xy/geoshape/) —
  the spatial relations the search engine evaluates (`INTERSECTS`, `DISJOINT`,
  `WITHIN`, `CONTAINS`), and on geopoint fields only `INTERSECTS`.
- [OpenSearch refresh
  API](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/) —
  a written document "is not searchable until a refresh operation converts
  these in-memory structures into searchable segments on disk".
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [OpenStreetMap copyright](https://www.openstreetmap.org/copyright) — the
  data is ODbL: "If you alter or build upon our data, you may distribute the
  result only under the same license."
- [Geofabrik download server](https://download.geofabrik.de/) — per-region
  OpenStreetMap extracts in `.osm.pbf`, updated daily, licence ODbL 1.0.
- Implementation pointers do not exist while the spec is `draft`.
