> Spoilers. Open only when stuck.

# What the neighbours do differently

- **OpenZeppelin Relayer** — moves signing, nonce assignment, fee bumping,
  and resubmission behind an API, so the question this lab asks — when a
  resubmission is the same payment and when it is a second one — becomes a
  policy inside the relayer that the caller never sees.
- **Flashbots Protect** — submits Ethereum transactions outside the public
  mempool and includes them only if they do not revert, so one failure mode
  this lab must handle — a visible transaction stuck in the open pool — is
  traded for private inclusion the sender cannot observe in progress.
- **Temporal** — makes the dispatching process durable — a workflow resumes
  after a crash and retries its activities by configuration — which solves
  the process half of this lab and leaves the chain half untouched, because a
  retried activity is a new submission and the engine cannot know whether the
  abandoned one will still land.
