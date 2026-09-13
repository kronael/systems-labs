> Spoilers. Open only when stuck.

# Why a merely slow worker sees its work delivered again

Read the acknowledgement page in the NATS JetStream documentation cited by
this lab, and read it for one thing: what the server does when the
acknowledgement deadline passes and nothing has acknowledged.

The deadline is not a lock. It is the server's estimate of how long the work
should take, and when it passes the server assumes the work was lost and hands
the same message to another consumer. A worker that is merely slow has not
lost anything, so two workers now hold the same unit of work and neither is
wrong.

That is the condition. What makes the second delivery harmless is yours to
design, and the question to hold on to is this: what has to be true of the
effect so that applying it twice leaves the same state as applying it once,
and where does that property have to live to survive both workers running at
the same moment?
