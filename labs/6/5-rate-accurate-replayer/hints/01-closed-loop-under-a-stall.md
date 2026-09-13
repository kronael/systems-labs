> Spoilers. Open only when stuck.

# What a sender that waits for each answer reports instead

Read the coordinated-omission thread and the `wrk2` documentation this lab
cites. Both describe the same failure, and it is a measurement failure rather
than a system one.

A sender that waits for each answer before sending the next stops sending
while the target is stalled. The requests it would have issued during the
stall are never issued, so their latency is never measured, and the stall
shows up as a short gap in the results instead of as the worst latency in the
run. The measurement gets better the worse the target behaves.

The declared rate is a promise about when a request should have been sent, not
about when the sender got round to it. Hold on to that sentence, then ask what
your replayer has to record at send time for the report to survive a target
that stops answering.
