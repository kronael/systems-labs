> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Apache Cassandra / ScyllaDB** — partition by key in the same way but let
  a partition grow without bound and degrade instead of throttling, which
  turns the hot-key problem from a provider quota into an operator's tail
  latency.
- **MongoDB** — can index any field after the fact, so access patterns need
  not be fixed before the data exists — and the same skew returns later as
  the choice of shard key.
- **PostgreSQL** — would answer every query in this lab from one node with
  ordinary indexes and no key design at all, and pays with a vertical ceiling
  in place of a partition limit.
