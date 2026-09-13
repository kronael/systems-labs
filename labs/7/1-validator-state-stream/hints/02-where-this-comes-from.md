> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../../../docs/blockchain-track.md) — blockchain
  track rationale and candidates.
- [Agave Geyser plugin docs](https://docs.anza.xyz/validator/geyser) — the
  validator calls the plugin during transaction processing and the plugin
  "should process the notification as fast as possible because any delay may
  cause the validator to fall behind"; processed and confirmed slot statuses
  arrive asynchronously to each other; startup accounts are streamed with a
  flag and an end-of-startup signal. The page's reference-plugin sections
  describe a working persistence design. Solution-bearing: this belongs in
  `hints/`, never in `README.md`.
- [`SlotStatus`](https://docs.rs/agave-geyser-plugin-interface/latest/agave_geyser_plugin_interface/geyser_plugin_interface/enum.SlotStatus.html)
  — the full status set includes `Dead`, and processed-slot state "is not
  derived from a confirmed or finalized block".
- [Configuring state commitment](https://solana.com/docs/rpc) — a processed
  block "is the newest view, but it can still be rolled back".
- [solana-labs/solana#27842](https://github.com/solana-labs/solana/issues/27842)
  — after a start from an older snapshot, account updates resume before slot
  statuses do, leaving updates whose slots have no knowable status; closed
  without a fix.
- [solana-labs/solana#31242](https://github.com/solana-labs/solana/issues/31242)
  — a plugin cannot request account state from the validator outside the
  update stream; closed unimplemented. The discussion names the workaround.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
