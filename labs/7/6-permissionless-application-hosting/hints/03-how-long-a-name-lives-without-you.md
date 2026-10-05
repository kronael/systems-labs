> Spoilers. Open only when stuck.

# How long a name lives without you

Current as of 2026-08-14, and the numbers decide the answer, which is why they
are here rather than in the task.

- A copy of a naming record placed in the DHT expires after 48 hours,
  whatever validity the record itself carries.
- The publishing node republishes every 4 hours by default, and only while it
  is running.

Put the two together and the window where a name resolves without its
publisher is fixed, and short.

A third number, how long a resolver caches the record, is stated three ways
by the vendor's own pages. The CLI reference gives the publish default as
`5m0s` while its own notes and the concept page say one hour, and the
implementation's `DefaultRecordTTL` is five minutes. Take the number from the
node you run, not from a page.

## What an upgrade does to a caller

An upgrade replaces the bytes at the same address, and the runtime documents
a visibility delay: the new bytes do not take effect in the slot that
deployed them. Read that before you decide what a client holding an older
interface may still do, and what it sees in the slot the upgrade lands in.

- <https://solana.com/docs/core/programs/program-execution> — "Visibility
  delay".
