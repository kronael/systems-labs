> Spoilers. Open only when stuck.

# Serverless quote aggregation — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule drives offered load several times the declared ceiling,
runs the provider latency schedule including a sustained slow interval and a
hard failure at named requests, destroys the environment that served a named
request so the next named request pays first-invocation latency, freezes the
environment with a named request's provider call outstanding, and holds a
named request's provider call past the handler's maximum run time.

Checks do not require a named cache or rejection mechanism. They observe
the public API, the expiry stamped on each returned quote, provider-side
request traffic, invocation and throttle counts, telemetry, and the submitted
evidence.
