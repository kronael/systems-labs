> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/low-level-track.md`](../../../../docs/low-level-track.md) — low-level track
  rationale and candidates.
- [Tene, mechanical-sympathy, 2013](https://groups.google.com/g/mechanical-sympathy/c/icNZJejUHfE)
  — the original account of the omission: a generator that waits for each
  answer stops sampling exactly during the slow interval, and the reported
  high percentiles understate the real ones by orders of magnitude.
- [`clock_nanosleep(2)`](https://man7.org/linux/man-pages/man2/clock_nanosleep.2.html)
  — relative sleeps drift, intervals round up to clock granularity, and an
  absolute sleep on a settable clock returns early when the clock is stepped.
  It also states the delay no service script can model: "after the sleep
  completes, there may still be a delay before the CPU becomes free to once
  again execute the calling thread", and that an interval that is not an exact
  multiple of the clock's granularity "will be rounded up to the next
  multiple".
- [`timerfd_create(2)`](https://man7.org/linux/man-pages/man2/timerfd_create.2.html)
  — a read returns the number of expirations since the last read, so missed
  periods are countable rather than lost.
- [wrk2](https://github.com/giltene/wrk2) and
  [HdrHistogram](https://github.com/HdrHistogram/HdrHistogram) — the
  corrective generator and the correction it records. Solution-bearing: both
  belong in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
