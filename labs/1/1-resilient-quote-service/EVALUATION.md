> Spoilers. Open only when stuck.

# Resilient quote service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule applies a sustained latency increase to one provider at a
named request, sized so that the delay exceeds the validity left on the quote
that request returns, injects provider errors at named requests, drives a burst above
measured capacity beginning at a named request, cancels the client of a named
in-flight request, and sends SIGTERM once a named request has been
acknowledged and while its work is still active. On the clock layer it steps
the container's wall clock backwards between the two readings a named
request's own quote comparison spans, so the interval the design measures is
negative. It runs both open-loop and closed-loop traffic.

A negative interval is a real reading, not a malfunction to be filtered: the
design states what it does with one, and a run in which the step never lands
inside a comparison fails rather than passes.

Checks do not inspect private functions or require a named concurrency
pattern. They observe API behavior, the expiry stamped on each returned quote,
process lifecycle, traces, metrics, queue or admission state exposed by the
design, and resource use.
