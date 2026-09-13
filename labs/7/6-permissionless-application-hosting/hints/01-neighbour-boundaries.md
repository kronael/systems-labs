> Spoilers. Open only when stuck.

# What the neighbours do differently

- **GitHub Pages** — static hosting behind a CDN makes availability an
  operator's obligation instead of the publisher's arrangement: the site
  stays up because the host keeps it up, the name moves the instant the
  operator says so, and the price is that nothing binds the name to
  particular bytes and the account is the availability — published sites may
  be no larger than 1 GB, bandwidth has a soft limit of 100 GB per month, and
  the host's terms decide what stays published.
- **Arweave** — makes permanence a purchase rather than an arrangement: one
  upfront fee, most of which enters a storage endowment that pays out only
  when block rewards fall short, priced on the expectation that storage keeps
  getting cheaper. Nothing needs the publisher after the transaction and
  nothing can be unpublished — and the mutable-pointer problem remains
  exactly as it is here.
- **ENS with `contenthash`** — puts the pointer itself on a chain: a resolver
  record holds a machine-readable multicodec content address, formerly
  standardized as EIP-1577, so moving the pointer is a transaction, the
  record persists with no republishing publisher, and every past value is
  public — at the cost of gas per move and of needing a chain client or a
  trusted gateway to read it.
