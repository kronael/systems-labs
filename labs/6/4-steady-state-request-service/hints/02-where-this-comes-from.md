> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`docs/low-level-track.md`](../../../../docs/low-level-track.md) — low-level track
  rationale and candidates.
- [`mallopt(3)`](https://man7.org/linux/man-pages/man3/mallopt.3.html) —
  `M_TRIM_THRESHOLD` releases only contiguous free space at the top of the
  heap, `M_MMAP_THRESHOLD` rises dynamically as large blocks are freed, and
  the arena count grows with lock contention. Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [`malloc_trim(3)`](https://man7.org/linux/man-pages/man3/malloc_trim.3.html)
  — only whole free pages can be released, and thread heaps ignore the pad.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) — the
  main heap grows through `sbrk`, and additional arenas appear when mutex
  contention is detected.
- [Transparent hugepage support](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html)
  — a 2 MB page can back a region of which one byte is touched, so resident
  size rises without any new allocation. Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
