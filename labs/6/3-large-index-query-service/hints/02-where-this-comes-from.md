> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`docs/low-level-track.md`](../../../../docs/low-level-track.md) — low-level track
  rationale and candidates.
- [Are You Sure You Want to Use MMAP in Your Database Management System?](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf)
  — CIDR 2022. Blocking page faults with no asynchronous path, uncontrollable
  write-back, single-threaded eviction, page-table and shootdown cost that
  grows with cores, and measured read bandwidth far below the device.
- [`mmap(2)`](https://man7.org/linux/man-pages/man2/mmap.2.html) — access
  past the end of the mapped file raises `SIGBUS`, and a shared mapping
  reaches the file at a time the process does not choose.
- [`madvise(2)`](https://man7.org/linux/man-pages/man2/madvise.2.html) —
  the access-pattern advices are hints the kernel may ignore.
- Implementation pointers do not exist while the spec is `draft`.
