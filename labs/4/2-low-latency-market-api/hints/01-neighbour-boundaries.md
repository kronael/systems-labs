> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Memcached** — a cache and nothing more — multithreaded, no persistence,
  no rich values — and manages memory in per-size slab classes, so eviction
  pressure lands within an item's size class rather than across the whole
  keyspace.
- **groupcache** — keeps hot entries inside each application replica,
  removing the network hop and the shared store; every replica then holds a
  private view, and coherence between replicas becomes the design problem.
- **Amazon ElastiCache** — runs the same engine as a managed service with
  failover, so a cache loss arrives as an empty replacement node on the
  provider's schedule; it is excluded from this course by cost policy.
