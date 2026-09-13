> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold, fault,
  evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../../../docs/blockchain-track.md) — track record
  and the origination of this candidate.
- [`3-finality-aware-transfer-index.md`](../../../../labs/7/3-finality-aware-transfer-index/README.md)
  — the neighbouring lab whose subject is what a receipt proves when the chain
  can withdraw it. There the movement names itself and the problem is that the
  chain takes it back. Here the movement never names itself and the chain
  never takes anything back. Reorganization belongs to that lab and is out of
  scope here.
- [`4-reliable-transaction-dispatcher.md`](../../../../labs/7/4-reliable-transaction-dispatcher/README.md)
  — the neighbouring lab whose subject is when a resubmission is the same
  payment. It holds the keys and can ask the chain what happened. This lab
  holds no key abroad and has nobody to ask.
- [NEAR chain signatures](https://docs.near.org/chain-abstraction/chain-signatures)
  — the signing network's real counterpart, and the source of this lab's
  central constraint: it is "a 'one way' solution to sign and execute outbound
  transactions happening on other blockchains", addresses are derived
  deterministically so they are ordinary addresses on the destination chain,
  and its documentation warns against signatures that can be "replayed on a
  chain you did not intend to interact with".
- [FDIC, Recordkeeping for Custodial Accounts, 89 FR 80135](https://www.govinfo.gov/content/pkg/FR-2024-10-02/html/2024-22565.htm)
  — the domain fact that makes the invariant and the daily close inevitable.
  Proposed 12 CFR 375.3(b) requires "Maintaining accurate balances of
  custodial deposit accounts with transactional features at the beneficial
  ownership level" and "Conducting reconciliations against the beneficial
  ownership records no less frequently than at the close of business daily";
  375.3(c)(1) requires "direct, continuous, and unrestricted access to the
  records … including in the event of business interruption, insolvency, or
  bankruptcy of the third party". The same document records why: after the
  Synapse bankruptcy, institutions "encountered significant difficulties in
  obtaining, reviewing, and reconciling Synapse's records", and "the deposits
  at the IDIs appear to be insufficient to cover the amounts owed by the
  fintech companies to their customers".
- Implementation pointers do not exist while the spec is `draft`.
