> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold, fault,
  evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../../../docs/blockchain-track.md) — track record
  and the origination of this candidate.
- [IPFS persistence and pinning](https://docs.ipfs.tech/concepts/persistence/)
  — the network guarantees content is discoverable, not persistently
  available; data persists only while pinned, and pinning services run nodes
  that pin data for a fee.
- [Pinning and garbage collection in practice](https://docs.ipfs.tech/how-to/pin-files/)
  — direct, recursive, and indirect pins; collecting garbage removes every
  unpinned object from the node.
- [IPNS](https://docs.ipfs.tech/concepts/ipns/) — a name is the hash of a
  key; DHT copies of a record expire after 48 hours regardless of the
  record's validity; Kubo republishes every 4 hours by default while it runs;
  resolution is slower than a content address because multiple records must
  be found. Solution-bearing for the pointer design: this belongs in
  `hints/`, never in `README.md`.
- [Deploying programs](https://solana.com/docs/programs/deploying) — the
  upgrade authority defaults to the deploying wallet and can update or close
  the program; `--final` removes it; once a program is immutable it can never
  be updated or closed, and a closed program's address can never be reused.
- [Solana name service records](https://guide.sns.id/domain-name/records.html)
  — the record convention the prepared registry follows: web3 record types
  including an IPFS content identifier and an Arweave address bound to a
  name. Solution-bearing for the pointer design: this belongs in `hints/`,
  never in `README.md`.
- [ENSIP-7 contenthash](https://docs.ens.domains/ensip/7/) — the on-chain
  pointer as practiced on Ethereum: a multicodec content address in a
  resolver record, formerly EIP-1577.
- [Arweave storage endowment](https://www.arweave.com/blog/endowment-with-arweave)
  — pay once, store forever: most of the fee enters an endowment released
  only when block rewards cannot cover storage, on the assumption of
  declining storage costs.
- [GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)
  — the hosted neighbour's terms: 1 GB site, 100 GB per month soft bandwidth
  limit, ten builds per hour.
- Implementation pointers do not exist while the spec is `draft`.
