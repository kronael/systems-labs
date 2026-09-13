> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold, fault,
  evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../../../docs/blockchain-track.md) — track record
  and the origination of this candidate.
- [`7-multi-chain-deposit-service.md`](../../../../labs/7/7-multi-chain-deposit-service/README.md) — the
  neighbouring lab in this phase. It holds no key on the chains it moves
  capital to and is never told what happened, so its question is what may be
  published about capital whose fate is unknown. This lab holds both keys and
  can ask. The two must not be run as one.
- [Transaction confirmation and expiration](https://solana.com/developers/guides/advanced/confirmation)
  — a transaction is valid while its blockhash sits within the 151 most
  recent, about 60 to 90 seconds; after expiry it will never be processed, an
  RPC node forwards a too-old transaction once and then drops it, and the
  `processed` commitment sits on a fork the cluster abandons for roughly 5
  percent of blocks.
- [Durable nonces](https://solana.com/docs/core/transactions/durable-nonces)
  — replaces the expiring blockhash with a stored value that a transaction's
  required first instruction advances, removing the validity window.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [Ethereum accounts](https://ethereum.org/en/developers/docs/accounts/) —
  only one transaction with a given nonce can be executed for each account,
  the replay protection that makes per-account order a hard constraint.
- [geth transaction pool options](https://geth.ethereum.org/docs/fundamentals/command-line-options)
  — replacing a pending transaction requires a price bump of 10 percent by
  default, and a non-executable transaction stays queued for up to three
  hours by default, so an abandoned transaction can still be mined long
  after its sender stopped watching.
- Implementation pointers do not exist while the spec is `draft`.
