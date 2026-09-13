---
status: draft
---

# Metered billing API — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule replays a scripted history against the public API and the queue
while firing faults at named barriers: it freezes the environment at the
freeze barrier following the response for invoice 4711, with work observably
outstanding; recycles the environment between the two requests of one logical
operation on account 2205; kills the invocation carrying subscription
118207 at the invocation ceiling; and redelivers the queue message carrying
usage event 771003 after its work is acknowledged — the queue's delivery
contract is another lab's subject, and the redelivery is an environment fact
here that the invariant must survive. It then closes a period under continued
traffic and reads every invoice.

Checks observe public responses, queue histories, and store contents. They
do not inspect private functions and do not require a named pattern. They
compare every invoice line against an independently computed answer derived
from the acknowledged input under the supplied rating rules.
