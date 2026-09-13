> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../../../docs/blockchain-track.md) — blockchain
  track rationale and candidates.
- [NSCC rule filing SR-NSCC-2023-007 (Release No. 34-98213)](https://www.sec.gov/files/rules/sro/nscc/2023/34-98213.pdf)
  — why settlement nets at all: NSCC estimates that in 2022 "netting through
  NSCC's continuous net settlement ('CNS') accounting system reduced the value
  of CNS settlement obligations by approximately 98% or $510 trillion from
  $519 trillion to $9 trillion".
- [Compute budget](https://solana.com/docs/core/fees/compute-budget) — a
  transaction may consume at most 1,400,000 compute units, an instruction
  defaults to 200,000, and a transaction that would exceed a limit is not
  included in a block; the page also names the instruction that requests a
  different limit. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- [Program limitations](https://solana.com/docs/programs/limitations) — the
  runtime bounds a deployed program beyond compute: a 64-frame call stack,
  cross-program invocation depth of 4, and no access to most of `std`.
- [Transactions](https://solana.com/docs/core/transactions) — a serialized
  transaction is at most 1,232 bytes, the IPv6 minimum MTU of 1,280 minus 48
  bytes of headers, and every referenced account address costs 32 of them.
- [Accounts](https://solana.com/docs/core/accounts) — account data is capped
  at 10 MiB, and every account must hold a lamport balance proportional to
  its size — `(bytes + 128) × 3,480 lamports per byte-year × 2 years` — to
  remain on chain.
- [`MAX_PERMITTED_DATA_INCREASE`](https://docs.rs/solana-program-entrypoint/latest/solana_program_entrypoint/constant.MAX_PERMITTED_DATA_INCREASE.html)
  — the maximum number of bytes a program may add to an account during a
  single realloc is 10,240.
- Implementation pointers do not exist while the spec is `draft`.
