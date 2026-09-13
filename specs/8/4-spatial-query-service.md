---
status: draft
---

# Spatial query service — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

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
