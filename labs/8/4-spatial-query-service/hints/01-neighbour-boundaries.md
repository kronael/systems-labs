> Spoilers. Open only when stuck.

# What the neighbours do differently

- **MongoDB** — interprets GeoJSON on the WGS84 sphere through its
  `2dsphere` index, so the degrees-are-not-metres trap never arises — the
  store fixes the earth model instead of leaving it as a design choice, and
  offers a smaller predicate surface (`$geoWithin`, `$geoIntersects`,
  `$nearSphere`) in exchange.
- **H3** — replaces geometry with a hierarchical hexagonal grid: containment
  becomes cell membership, and because child cells are not entirely
  contained by their parents, that membership is approximate by
  construction — the filter is the final answer, carrying a stated error,
  and no exact predicate ever runs behind it.
- **Tile38** — turns proximity into an event: a geofencing server that
  pushes Nearby, Within, and Intersects notifications to webhooks and
  queues, moving the product from querying durable history to watching a
  live boundary.
