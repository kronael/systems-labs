> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`docs/low-level-track.md`](../../../../docs/low-level-track.md) — low-level track
  rationale and candidates.
- [fsyncgate](https://danluu.com/fsyncgate/) — the PostgreSQL thread showing
  that a failed `fsync` can clear the error and the dirty page, so the next
  `fsync` reports success over data that never reached the disk.
- [PostgreSQL error handling in fsync](https://lwn.net/Articles/752063/) — the
  kernel-side account of where the error is reported and where it is lost.
- Implementation pointers do not exist while the spec is `draft`.
