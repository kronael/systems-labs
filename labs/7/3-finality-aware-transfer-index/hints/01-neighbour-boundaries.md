> Spoilers. Open only when stuck.

# What the neighbours do differently

- **The Graph** — moves the reorganization into the framework: `graph-node`
  reverts a subgraph's entities automatically inside a configured reorg
  threshold that defaults to 250 blocks, and a deeper fork can leave the
  store inconsistent — the query answer carries no finality label.
- **TrueBlocks** — refuses the fast view: its Unchained Index answers from
  blocks roughly 28 behind the head and calls anything younger unripe, so it
  never withdraws an answer and never gives a fresh one.
- **Etherscan** — moves the whole boundary to a third party: a hosted index
  answers at `latest` or a recent block number with no finality label, and
  its reorg handling is invisible behind the service.
