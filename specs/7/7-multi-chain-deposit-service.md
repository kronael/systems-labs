---
status: draft
---

# Multi-chain deposit service — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule fires at named barriers, never on a timer and never at
random:

- it stops the signing network at the barrier where a named depositor's
  withdrawal is accepted, and holds it stopped through the next accounting
  close, so the close must publish with the middle party silent;
- it kills the service between the request for a named depositor's
  authorization and the recording that the request was made, then restarts it
  against the same chains, so the service returns not knowing whether capital
  was authorized to leave;
- it delivers a named authorization's signed payload to the second deposit
  chain as well as the intended one, and includes it on both;
- it includes a named authorization's transaction after the accounting close
  that named that authorization unagreed has published, and at the same
  barrier sends a stranger's transfer of the same amount to the same address;
- it accepts a named depositor's full withdrawal at the moment that
  depositor's capital rests on an authorization the service has not
  attributed;
- it releases a named authorization's signature only after the accounting
  close that expected it has already published.

Evaluation observes the three chains through their own interfaces, the
service's public interface and portfolio view, process lifecycle, and exact
per-depositor and per-authorization histories. It does not require a named
recovery pattern, and it keeps watching after each close — a balance that
becomes unhonourable after it was stated is a failure regardless of counts.
