> Spoilers. Open only when stuck.

# What the neighbours do differently

- **FoundationDB** — neither reads nor writes block and conflicting
  transactions fail at commit time, but the client library owns the loop
  that repeats them, so the boundary this lab makes the design draw is drawn
  inside the data-access API and the application mostly never sees the
  refusal.
- **MySQL with InnoDB** — a conflicting writer blocks as soon as it tries to
  take a lock and does not proceed until the holder commits or rolls back, so
  contention arrives as waiting and lock ordering rather than as work handed
  back to be done again, and its cost shows up in latency instead of in
  refused units.
- **DynamoDB transactions** — cancels the whole grouped operation when it
  conflicts with another in flight, and caps one group at 100 items and 4 MB,
  so the set a single decision may read and write is bounded by the API
  rather than by the product's own rule.
