> Spoilers. Open only when stuck.

# Large index query service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule drives uniformly random point queries over the whole
keyspace until the touched data far exceeds resident memory and holds full
load through that transition. It starts range scans beside the point load,
raises the stream count to every available core, truncates a named retired
segment while queries against it are in flight, queries records appended
after startup at the declared visibility bound, and sends SIGTERM under full
load before restarting the service against a cold memory state. The harness
appends and seals segments throughout.

Checks do not inspect private functions or require a named access method.
They compare every answer against the generator's seeded truth and observe
per-class latency, resident memory over time, and the reported evidence.
