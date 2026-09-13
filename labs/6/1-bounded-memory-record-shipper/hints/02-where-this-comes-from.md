> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/low-level-track.md`](../../../../docs/low-level-track.md) — low-level track
  rationale and candidates.
- [Fluent Bit: Backpressure](https://docs.fluentbit.io/manual/administration/backpressure)
  — the real reported behaviour behind the brief: an agent limits how much a
  source "can buffer to memory", pauses the input when that limit is
  reached, and "some input plugins are prone to data loss after
  `mem_buf_limit` capacity is reached during memory-only buffering".
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [`epoll`](https://man7.org/linux/man-pages/man7/epoll.7.html) — the
  readiness half of the readiness-versus-completion distinction: an event
  says the "file descriptor is ready for the requested I/O operation", the
  application still performs the transfer itself, and the buffer never
  leaves its hands. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- [`io_uring`](https://man7.org/linux/man-pages/man7/io_uring.7.html) — a
  submitted operation owns its buffer until its completion is reaped, so
  in-flight work, not the application's queue, sets the memory floor.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [Efficient IO with io_uring](https://kernel.dk/io_uring.pdf) — the ring
  design and what submission and completion mean for ownership and ordering.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
