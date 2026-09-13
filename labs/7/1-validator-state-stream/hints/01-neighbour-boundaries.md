> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Yellowstone gRPC (Dragon's Mouth)** — consumes the same plugin interface
  but immediately turns it into a filtered gRPC subscription service, moving
  consumers off the validator's boundary onto a network stream with
  per-subscription commitment filters — the back-pressure problem is solved
  once, inside the plugin, for all consumers.
- **RPC WebSocket subscriptions** (`accountSubscribe`) — these let the node
  apply the requested commitment level before a notification is delivered, so
  the subscriber never faces pre-commitment state — and a dropped connection
  has no replay and no snapshot to resume from.
- **JSON-RPC polling** (`getProgramAccounts`) — this returns current account
  state at a chosen commitment level in one shot, which makes cold start
  trivial and every state between two polls invisible.
