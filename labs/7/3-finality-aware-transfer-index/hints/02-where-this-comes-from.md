> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`7-multi-chain-deposit-service.md`](../../../../labs/7/7-multi-chain-deposit-service/README.md) — the
  neighbouring lab that also publishes balances it may have to revise. There
  the movement never names the request that caused it and nothing is ever
  withdrawn; here the movement names itself and the chain takes it back.
  Reorganization belongs to this lab and is out of scope there.
- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  fault-injection, data, evidence, and verification contracts.
- [`docs/blockchain-track.md`](../../../../docs/blockchain-track.md) — the track
  record where this candidate and its origination are filed.
- Quirk origination:
  [Ethereum JSON-RPC API](https://ethereum.org/en/developers/docs/apis/json-rpc/)
  — fixes the log object's `removed` field, "`true` when the log was
  removed, due to a chain reorganization", and the block parameter's `safe`
  and `finalized` tags naming the latest safe head and the latest finalized
  block.
- [Execution API log schema](https://github.com/ethereum/execution-apis/blob/main/src/schemas/receipt.yaml)
  — the canonical specification carries `removed` on every log, so a
  withdrawn log is protocol surface, not a provider extension.
- [MetaMask `eth_getLogs` reference](https://docs.metamask.io/services/reference/ethereum/json-rpc-methods/eth_getlogs/)
  — the track record's cited page; a provider surface documenting the same
  `removed` semantics.
- [Proof-of-stake finality](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/)
  — the first block of each epoch is a checkpoint; a checkpoint that
  attracts two-thirds of staked ETH is justified, the justified checkpoint
  before it becomes finalized, and finality arrives roughly two epochs of 32
  twelve-second slots — about thirteen minutes — behind the head. Reverting
  a finalized block costs an attacker at least one-third of all staked ETH.
- Implementation pointers do not exist while the spec is `draft`.
