> Spoilers. Open only when stuck.

# What the neighbours do differently

- **SQLite** — in WAL mode makes commit durability a configuration choice:
  with `synchronous=NORMAL` a committed transaction may roll back after a
  power loss while the database stays consistent, and `synchronous=FULL`
  syncs the write-ahead log on every commit.
- **LMDB** — copies pages on write and never overwrites active data, so it
  needs no special recovery procedure after a system crash; one write
  transaction is active at a time, commit flushes to disk by default, and the
  `MDB_NOSYNC` flag trades that away — a system crash can then corrupt the
  database or lose the last transactions.
- **RocksDB** — journals every update to a write-ahead log, but by default a
  write returns once the data reaches the operating system: a process crash
  loses nothing, while a machine crash can lose the last updates unless the
  write asked for a sync.
