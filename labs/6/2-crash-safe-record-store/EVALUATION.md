> Spoilers. Open only when stuck.

# Crash-safe record store — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule kills the process and the backing device at exact record
boundaries, drops unflushed data on resume, returns one I/O error to a flush
call and then resumes normal behaviour while dropping the data that flush was
carrying, so a design that retries into a false success loses a record it
acknowledged, truncates the tail of a file, and flips
bytes inside a stored record, then restarts the store. Verification replays
the full client history against what the store serves.

Checks do not inspect private functions or require a named on-disk format.
They observe acknowledgements, served records, recovery duration, and
reported errors.
