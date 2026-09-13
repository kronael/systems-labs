> Spoilers. Open only when stuck.

# What the neighbours do differently

- **wrk** — saturates a fixed set of threads and connections and reports the
  latency of the requests it managed to issue; its command line fixes
  threads, connections, and duration but no rate, so the load it offers is a
  consequence of the target's speed rather than an input to the run.
- **tcpreplay** — holds a schedule faithfully — original timing, a fixed
  rate, or line rate — by never being a client at all: it pushes captured
  packets one way, expects no responses, and measures nothing, which is the
  opposite trade.
- **k6** — turns the coupling between one iteration's completion and the
  next one's start into a per-scenario configuration choice, so whether a
  test's offered load survives a slow target is decided in test design
  rather than by the tool.
