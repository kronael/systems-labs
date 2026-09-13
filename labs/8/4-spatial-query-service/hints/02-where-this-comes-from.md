> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../../../docs/search-and-retrieval-track.md)
  — the track record holding this candidate and its origination.
- [The `&&` operator](https://postgis.net/docs/geometry_overlaps.html) —
  returns `TRUE` when the 2D bounding box of one geometry intersects the 2D
  bounding box of the other, and that is all it tests.
- [Spatial indexing
  workshop](https://postgis.net/workshops/postgis-intro/indexing.html) —
  spatial indexes index the bounding boxes of features, not the features; an
  index-only query over one neighbourhood counts 49,821 people where the exact
  answer is 26,718. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- [`ST_Intersects`](https://postgis.net/docs/ST_Intersects.html) — the exact
  intersection predicate, which "automatically includes a bounding box
  comparison that makes use of any spatial indexes". Solution-bearing: this
  belongs in `hints/`, never in `README.md`.
- [`ST_Contains`](https://postgis.net/docs/ST_Contains.html) — containment
  means every point of B lies in A and the interiors share a point, with the
  same automatic index comparison. Solution-bearing: this belongs in
  `hints/`, never in `README.md`.
- [`ST_DWithin`](https://postgis.net/docs/ST_DWithin.html) — the radius
  predicate whose distance is in spatial-reference units for `geometry` and in
  metres for `geography`. Solution-bearing: this belongs in `hints/`, never
  in `README.md`.
- [`ST_Distance`](https://postgis.net/docs/ST_Distance.html) — `geometry`
  distance comes back in the units of the spatial reference system, so SRID
  4326 yields degrees, while `geography` yields geodesic metres.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [Geography
  workshop](https://postgis.net/workshops/postgis-intro/geography.html) — "a
  distance of 122 degrees" between coordinates "is a nonsense number", degree
  squares shrink toward the poles, and the shortest Cartesian route from Los
  Angeles to Tokyo crosses the Atlantic. Solution-bearing: this belongs in
  `hints/`, never in `README.md`.
- [EPSG:3857](https://epsg.io/3857) — pseudo-Mercator is "not a recognised
  geodetic system", is defined only between 85.06°S and 85.06°N, differs from
  true Mercator by errors of 0.7 percent in scale and up to 21 km on the
  ground, and exists for web mapping and visualisation, not measurement.
- [RFC 7946, sections 3.1.9 and
  5.2](https://datatracker.ietf.org/doc/html/rfc7946) — a geometry crossing
  the antimeridian should be cut in two, and a bounding box spanning it has a
  western edge numerically greater than its eastern edge. Solution-bearing:
  this belongs in `hints/`, never in `README.md`.
- [OpenSearch geodistance
  query](https://docs.opensearch.org/latest/query-dsl/geo-and-xy/geodistance/)
  — `distance_type` is `arc` by default; `plane` is "faster but inaccurate for
  long distances or points close to the poles". Solution-bearing: this belongs
  in `hints/`, never in `README.md`.
- [OpenSearch geoshape
  query](https://docs.opensearch.org/latest/query-dsl/geo-and-xy/geoshape/) —
  the spatial relations the search engine evaluates (`INTERSECTS`, `DISJOINT`,
  `WITHIN`, `CONTAINS`), and on geopoint fields only `INTERSECTS`.
- [OpenSearch refresh
  API](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/) —
  a written document "is not searchable until a refresh operation converts
  these in-memory structures into searchable segments on disk".
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [OpenStreetMap copyright](https://www.openstreetmap.org/copyright) — the
  data is ODbL: "If you alter or build upon our data, you may distribute the
  result only under the same license."
- [Geofabrik download server](https://download.geofabrik.de/) — per-region
  OpenStreetMap extracts in `.osm.pbf`, updated daily, licence ODbL 1.0.
- Implementation pointers do not exist while the spec is `draft`.
