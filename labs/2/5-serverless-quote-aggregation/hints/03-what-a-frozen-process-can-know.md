> Spoilers. Open only when stuck.

# What a frozen process can know about time

Read the runtime lifecycle page this lab cites, and read it for what happens
to the process during a freeze rather than for the lifecycle diagram.

Wall-clock time advances during a freeze. Nothing inside the process does. So
an expiry judged from anything the process itself recorded — a timer it set, a
deadline it computed, a monotonic reading it took before the freeze — is not
evidence about what time it is now.

That leaves the design question: what source of time survives a freeze, and
what does the service do for a caller when no source it trusts can tell it
whether a remembered quote is still valid?
