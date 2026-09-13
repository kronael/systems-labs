> Spoilers. Open only when stuck.

# Serverless auditable transfer — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule freezes the environment at the freeze barrier with the
audit publication for a named transfer still outstanding, destroys environments between invocations, fails the queue while
the ledger stays healthy and the reverse, resubmits transfers with the same and
with altered payloads, replays delivered audit messages, and drives the hot
accounts past what the configured ceiling admits. It then replays the full client history
against balances and the audit product, and requires an audit view rebuilt
after the loss of its store to match the acknowledged transfer set by
identity.

Checks do not require a named publication mechanism. They observe HTTP,
store-visible state, queue traffic, invocation counts, and the submitted
evidence.
