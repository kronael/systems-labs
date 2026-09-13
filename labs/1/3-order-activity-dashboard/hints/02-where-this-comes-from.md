> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold, Kafka,
  evidence, and verification contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [KIP-429](https://cwiki.apache.org/confluence/display/KAFKA/KIP-429%3A+Kafka+Consumer+Incremental+Rebalance+Protocol)
  — under the eager protocol a healthy consumer revokes every assigned
  partition before rejoining the group, because no partition may be
  reassigned before it is revoked. Solution-bearing: this belongs in
  `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
