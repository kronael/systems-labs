> Spoilers. Open only when stuck.

# What the neighbours do differently

- **RouteViews** — operates a second, independently peered collector fleet —
  over a thousand peers at exchange points and partner networks — published
  as MRT dumps on a schedule; the same prefix can look different there than
  from RIS, because coverage is a property of the peer population, not of the
  prefix.
- **CAIDA BGPStream** — merges archived dumps and live collectors behind one
  programming interface, which answers the replay-versus-live question in the
  toolchain instead of in the pipeline.
- **RIPE Atlas** — observes from probes hosted in roughly 3,300 of the more
  than 70,000 ASes and measures what packets actually do, where a route
  collector records what a few hundred peering ASes announce; each is a
  sample with its own bias, and neither is the Internet.
