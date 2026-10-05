# Changelog

## [v0.5.0] — 20261005

> systems-labs v0.5.0 — two more labs, sharper faults
>
> Two labs join the catalog, five existing labs gain a fault or a requirement,
> and the hints carry only the reading.
>
> • `1/8` streaming completion service — streamed answers from one engine over a fixed pool.
> • `4/6` dataset release service — versioned releases on object storage, fetched by identity.
> • `4/3` and `8/3` — a withdrawn trade and a removed page must be gone from every answer.
> • `1/8` and `4/1` — requests name their caller, and the evidence shows what one costs another.
> • `1/1` and `1/2` — one more scenario each in the failure schedule.
> • Phase 6 — the shared telemetry harness supplies what `6/1`, `6/3` and `6/4` must measure.
>
> Full notes: https://github.com/kronael/systems-labs/blob/master/CHANGELOG.md

### Added

- `labs/1/8-streaming-completion-service`: streamed answers from one vLLM
  engine on CPU, two declared bounds per request (S51 queues its gate).
- `labs/4/6-dataset-release-service`: 12 GiB releases on an S3-compatible store
  behind an admitted budget, with no `make smoke` (S39).
- `4/3` withdraws a trade and `8/3` removes a page. `1/8` and `4/1` name the
  caller on every request. `1/1` and `1/2` gain one failure scenario each (S42).
- `shared/telemetry/` supplies the phase 6 measurement for `6/1`, `6/3` and
  `6/4` (S50 queues what its Callgrind runner cannot isolate).

### Changed

- Hint files carry only the reading: the author markers are gone from every
  `where-this-comes-from.md` file.
- `7/6` states its naming intervals without saying what they decide.
- `docs/` is the author tree and `labs/` the learner tree. `README.md` owns the
  lab counts, and `TODO.md` holds one candidate and the rulings.

### Fixed

- Learner-facing text names no solution method in `1/8`, `4/6`, `4/3`, `6/3`,
  `7/6` and `8/3`. Quotes match their pages (`1/7`, `1/8`), and the `1/1`,
  `1/2`, `4/3` and `4/6` hints cite the pages their facts rest on.
- `BUGS.md` holds only what still stands: S39, S41 to S52.
