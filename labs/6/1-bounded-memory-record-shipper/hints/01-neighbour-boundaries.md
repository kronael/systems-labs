> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Fluent Bit** — caps buffered data with a per-input memory limit and
  pauses ingestion when the limit is reached; its backpressure page warns
  that some inputs "are prone to data loss" once that happens under
  memory-only buffering, and offers the filesystem as the buffer that
  survives.
- **Vector** — makes the full-buffer decision an operator setting on each
  destination: block until there is room, pushing the stall upstream, or
  drop the newest events — with a disk buffer as the variant that survives a
  restart.
- **Filebeat** — treats the log file itself as the buffer: when the output
  stalls it stops reading, keeps its position in a registry file, resumes
  when the output recovers, and delivers at least once — a shutdown mid-send
  re-delivers after restart.
