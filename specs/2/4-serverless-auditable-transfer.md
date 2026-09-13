---
status: draft
---

# Serverless auditable transfer — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule freezes the environment between the ledger write and the audit
publication, destroys environments between invocations, fails the queue while
the ledger stays healthy and the reverse, resubmits transfers with the same and
with altered payloads, replays delivered audit messages, and drives the hot
accounts past what the configured ceiling admits. It then replays the full client history
against balances and the audit product, and requires an audit view rebuilt
after the loss of its store to match the acknowledged transfer set by
identity.

Checks do not require a named publication mechanism. They observe HTTP,
store-visible state, queue traffic, invocation counts, and the submitted
evidence.
