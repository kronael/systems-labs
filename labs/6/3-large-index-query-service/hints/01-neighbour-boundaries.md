> Spoilers. Open only when stuck.

# What the neighbours do differently

- **LMDB** — maps the whole database and returns answers straight out of the
  mapping — and buys survival by mapping read-only by default, serializing
  writers, and copying pages on write, so the kernel's paging is the only
  cache it has.
- **RocksDB** — left memory-mapped reads behind after they bottlenecked
  read-heavy workloads larger than memory; it owns a block cache in user
  space and pays an explicit copy for control over what stays resident.
- **ScyllaDB** — rejected kernel paging outright and reads with asynchronous
  direct I/O, because an application that schedules its own I/O decides which
  page an eviction costs and when a device wait happens.
