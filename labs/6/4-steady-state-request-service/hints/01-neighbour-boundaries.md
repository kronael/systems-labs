> Spoilers. Open only when stuck.

# What the neighbours do differently

- **jemalloc** — makes returning memory a scheduled policy rather than a side
  effect: unused dirty pages decay along a curve over roughly ten seconds and
  are purged with `madvise`, so residency follows load with a lag instead of
  holding a high-water mark.
- **tcmalloc** — releases from its page heap at a configured background rate,
  never on the release call itself, and its tuning guide states the cost this
  lab measures: released memory must be faulted back, and fine-grained
  release breaks up hugepages.
- **mimalloc** — purges rather than unmaps: it decommits unused pages after a
  delay it exposes as a first-class option, keeping the address ranges while
  shrinking residency.
