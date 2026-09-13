> Spoilers. Open only when stuck.

# What a practitioner might have used instead

This lab's `README.md` names no neighbouring system: every genuine candidate
carries the answer in its name or on its landing page, so naming one here
would solve the lab instead of orienting you. All three — names, links, and
what each does differently at this lab's boundary — live only in this file.

- **CME Globex** — [documentation](https://cmegroupclientsite.atlassian.net/wiki/display/EPICSANDBOX/Cancel+on+Disconnect).
  The counterparty side of this product's problem, solved as a registered
  facility: when the venue detects a session lost, it voids that
  participant's resting orders itself, exempting the kinds explicitly marked
  to outlive the session, so the participant's own doubt never has to run —
  the counterparty was armed in advance.
- **Kubernetes** — [documentation](https://kubernetes.io/docs/concepts/architecture/nodes/).
  Its node controller acts on remote liveness evidence, and when that
  evidence implies every node failed at once it concludes the evidence itself
  is broken and stops evicting; the self-distrust suspends future authority
  but never voids anything already done.
- **Recursive DNS resolvers** — [documentation](https://www.rfc-editor.org/rfc/rfc8767.txt).
  Keep answering from expired data when the authority cannot be reached,
  bounded on the order of days, because an answer holds no standing authority
  once given and expires on its own — the opposite rule, for the opposite
  kind of output.

The lab does not run them.
