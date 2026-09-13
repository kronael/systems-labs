> Spoilers. Open only when stuck.

# Bounded-memory record shipper — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule stalls the collector for a fixed interval at an exact
record boundary, resumes it at a byte-per-read rate, closes the connection
mid-record, resets the connection after acknowledging a known position, and
sends SIGTERM while a submission is outstanding. It grows the log file
during every stall.

Checks do not inspect private functions or require a named I/O interface.
They observe the received byte stream, the acknowledgement sequence, the
resume position after restart, resident memory over time, and telemetry.
