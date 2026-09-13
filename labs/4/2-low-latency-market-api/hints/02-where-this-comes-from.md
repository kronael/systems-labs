> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold, Valkey,
  evidence, and failure contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [Key eviction](https://valkey.io/topics/lru-cache/) — at `maxmemory` Valkey
  evicts by the configured policy, and its LRU and LFU are approximations
  that sample a handful of keys per decision rather than track exact recency.
  Solution-bearing: this belongs in hints/, never in README.md.
- [`EXPIRE`](https://valkey.io/commands/expire/) — expired keys are reclaimed
  on access plus a background sampling cycle that tolerates a fraction of
  expired keys lingering in memory, so a TTL is not an exact deadline.
  Solution-bearing: this belongs in hints/, never in README.md.
- [Get OHLC data](https://docs.kraken.com/api/docs/rest-api/get-ohlc-data) —
  "The last entry in the OHLC array is for the current, not-yet-committed
  timeframe": the venue's own contract makes the newest candle provisional,
  so an answer's age states how settled it is.
- Implementation pointers do not exist while the spec is `draft`.
