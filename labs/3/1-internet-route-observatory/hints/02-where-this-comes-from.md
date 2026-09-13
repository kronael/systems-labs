> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold, Kafka,
  real-data, source, and evidence contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- Quirk origination. The falsified belief is that the collector's view is the
  Internet's state and that an update means the routing changed. Both halves
  have primary sources:
  - [RIS Live manual](https://ris-live.ripe.net/manual/) — a message's
    timestamp is the collector's receipt time, ordering is guaranteed only
    within one peering session, and the stream carries collector metadata
    (`RIS_PEER_STATE` session-state messages) beside relayed BGP messages.
  - [RIS route collectors](https://ris.ripe.net/docs/route-collectors/) — the
    fleet is about two dozen collectors, most peering at one exchange point
    plus a few multihop collectors, so each collector's view is fixed by
    which peers connect to it, and some collectors are explicitly regional.
  - [RFC 4271, section 3](https://www.rfc-editor.org/rfc/rfc4271) — a BGP
    speaker advertises to its peers only the routes it uses itself, so a
    collector's peer reveals one selected path per prefix, never everything
    the peer knows.
  - [RouteViews](https://www.routeviews.org/routeviews/) — an independent
    collector fleet with its own peer population of over a thousand peers;
    the vantage-point coverage contrast in the neighbouring-systems reading.
  - Solution-bearing, for `hints/` and never `README.md`: [Labovitz, Ahuja,
    Bose, and Jahanian, *Delayed Internet Routing Convergence*, SIGCOMM
    2000](https://conferences.sigcomm.org/sigcomm/2000/conf/paper/sigcomm2000-5-2.pdf)
    — measured failovers averaged three minutes, oscillations ran up to
    fifteen minutes and tens of minutes at worst, and the rate-limiting
    advertisement timer shapes the bursts, so a burst of updates is
    exploration through transient paths rather than change.
  - Solution-bearing, for `hints/` and never `README.md`: [Oliveira, Pei,
    Willinger, Zhang, and Zhang, *Quantifying the Completeness of the
    Observed Internet AS-level Structure*, UCLA TR-080026,
    2008](https://web.cs.ucla.edu/~lixia/papers/08completeness-TR.pdf) — the
    public view from RouteViews and RIS vantage points reveals the full peer
    connectivity of only 4 percent of ASes, because export policy bounds what
    any vantage point can see; this is what the observatory can never
    conclude.
  - Solution-bearing, for `hints/` and never `README.md`: [Sermpezis et
    al., *Bias in Internet Measurement Infrastructure*, RIPE
    Labs](https://labs.ripe.net/author/pavlos_sermpezis/bias-in-internet-measurement-infrastructure/)
    — RIS and RouteViews collect feeds from roughly 300 and 500 peering ASes
    out of more than 70,000, skewed toward large networks and exchange
    points, so the sample is biased as well as small.
- Implementation pointers do not exist while the spec is `draft`.
