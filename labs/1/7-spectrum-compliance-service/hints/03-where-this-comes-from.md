> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  workload, fault, and evidence contracts.
- [47 CFR § 96.39(c)(2)](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-D/part-96)
  — the origination. "A CBSD must receive and comply with any incoming
  commands from its associated SAS about any changes to power limits and
  frequency assignments. A CBSD must cease transmission, move to another
  frequency range, or change its power level within 60 seconds as instructed
  by an SAS." The mandated discontinuance is the product obligation, not a
  design; neutral.
- [47 CFR § 96.15(a)(4)](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-D/part-96)
  — "Within 300 seconds after the ESC communicates that it has detected a
  signal from a federal system in a given area, or the SAS is otherwise
  notified of current federal incumbent use of the band, the SAS must either
  confirm suspension of the CBSD's operation or its relocation to another
  unoccupied frequency, if available." The environment's behaviour — the
  coordinator suspends when an incumbent returns; neutral.
- [WINNF-TS-0016-V1.2.5 § 8.6](https://winnf.memberclicks.net/assets/work_products/Specifications/WINNF-TS-0016-V1.2.5%20SAS%20to%20CBSD%20Technical%20Specification.pdf)
  — the lease, the resume right, and the revocation in one section. "If the
  transmit expiration timer expires prior to reception of a HeartbeatResponse
  object, the CBSD shall discontinue transmission for the Grant within 60
  seconds after the value of the transmitExpireTime parameter expires, in
  accordance with part 96.39(c)(2)"; the CBSD "is not authorized to transmit
  until it receives a successful HeartbeatResponse object"; after "a
  HeartbeatResponse object with the response parameter set to
  SUSPENDED_GRANT, the CBSD is authorized to transmit and may initiate radio
  transmission any time after receiving this HeartbeatResponse object"; and
  "SAS to CBSD connectivity is considered to be lost when during a seven-day
  period there is no successful Heartbeat procedure between the SAS and the
  CBSD." Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [47 CFR § 15.711](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-H/section-15.711)
  — the same obligation in the neighbouring white-space regime, at the
  opposite timescale: "If a device fails to successfully contact the white
  space database, it may continue to operate until no longer than 120 minutes
  after the last successful contact, at which time it must cease operations
  until it reestablishes contact with the white space database and
  re-verifies its list of available channels and associated maximum power
  levels." Solution-bearing.
- [SEC Release 34-70694](https://www.sec.gov/litigation/admin/2013/34-70694.pdf)
  — the canonical post-mortem of standing output outliving its evidence.
  Knight Capital "did not have procedures in place to halt SMARS's operations
  in response to its own aberrant activity" and "did not have a mechanism to
  test whether their systems were relying on stale data"; the 2011 incident
  the order records was remediated by "changing the control so that this
  system would stop providing quotes after receiving an execution".
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [Binance, how to manage a local order book correctly](https://raw.githubusercontent.com/binance/binance-spot-api-docs/master/web-socket-streams.md)
  — the gap barrier's origination. The update procedure opens: "If the event
  first update ID (`U`) is greater than the update ID of your local order
  book + 1, you have missed some events. Discard your local order book and
  restart the process from the beginning." A missed record invalidates the
  whole derived view, not the record. Solution-bearing: this belongs in
  `hints/`, never in `README.md`.
- [CME Globex Cancel on Disconnect](https://cmegroupclientsite.atlassian.net/wiki/display/EPICSANDBOX/Cancel+on+Disconnect)
  — "If a lost connection is detected, COD cancels all resting futures and
  options orders for the disconnected registered iLink user", excluding GTC
  and GTD orders; a connection that answers no test request within a
  heartbeat interval "is assumed to be stale and the socket is closed".
  Solution-bearing.
- [Deribit Cancel On Disconnect](https://docs.deribit.com/) — "all orders
  created via this connection will be automatically cancelled when the
  connection is closed", while a graceful logout cancels nothing: the void is
  for doubt, not for departure. Solution-bearing.
- [Bybit Disconnect Cancel All](https://bybit-exchange.github.io/docs/v5/order/dcp)
  — the protection window is configured and bounded, "[3, 300], unit:
  second", and fires when the client has not reconnected and resumed
  heartbeats within it. Solution-bearing.
- [Kleppmann, How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
  — "You cannot fix this problem by inserting a check on the lock expiry just
  before writing back to storage", because "GC can pause a running thread at
  any point, including the point that is maximally inconvenient for you"; the
  side holding the resource checks "a fencing token", "simply a number that
  increases … every time a client acquires the lock". Solution-bearing.
- [RFC 8767](https://www.rfc-editor.org/rfc/rfc8767.txt) — serving stale DNS
  data "under the metaphorical assumption that 'stale bread is better than no
  bread'", with staleness "capped on the order of days to weeks, with a
  recommended cap of 604,800 seconds (7 days)". Solution-bearing.
- [RFC 9309 §2.3.1.4](https://www.rfc-editor.org/rfc/rfc9309.txt) — "If the
  robots.txt file is unreachable due to server or network errors … the
  crawler MUST assume complete disallow", with an escape hatch only after "a
  reasonably long period of time (for example, 30 days)". Solution-bearing.
- [Kubernetes node controller](https://kubernetes.io/docs/concepts/architecture/nodes/)
  — when all zones are completely unhealthy, "the node controller assumes
  that there is some problem with connectivity between the control plane and
  the nodes, and doesn't perform any evictions". Solution-bearing.
- [AWS DynamoDB service disruption, 2015-09-20](https://aws.amazon.com/message/5467D2/)
  — storage servers unable to confirm their membership "temporarily
  disqualify themselves from accepting requests", and that self-
  disqualification turned a network blip into a region-wide outage: the
  counter-case that makes the resumption requirement a gate. Solution-bearing.
- [Amazon Builders' Library, static stability](https://aws.amazon.com/builders-library/static-stability-using-availability-zones/)
  — "the data plane maintains its existing state and continues working even
  in the face of a control plane impairment": the counter-case for the
  answers this service must keep serving. Solution-bearing.
- [RIPE-378](https://www.ripe.net/publications/docs/ripe-378/) — route-flap
  damping deployed and then recommended against, because "a simple prefix
  withdrawal can result in the appearance of a major flap event a few AS hops
  away", suppressing perfectly valid routes: withdrawal itself can cascade.
  Solution-bearing.
- Implementation pointers do not exist while the spec is `draft`.
