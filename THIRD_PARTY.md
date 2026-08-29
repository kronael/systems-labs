# Third-party material

This file lists every file or embedded asset in this repository that was
copied or adapted from an outside source, with its path, upstream revision,
copyright notice, and licence. It discharges rule 4 of the licensing contract
in [`specs/01-systems-labs.md`](specs/01-systems-labs.md).

## Copied or embedded files

| path | upstream | revision | copyright | licence |
|------|----------|----------|-----------|---------|
| `LICENSE` | [gnu.org/licenses/gpl-3.0.txt](https://www.gnu.org/licenses/gpl-3.0.txt) | retrieved 2026-08-29, sha256 `3972dc9744f6499f0f9b2dbf76696f2ae7ad8af9b23dde66d6af86c9dfb36986` | Copyright (C) 2007 Free Software Foundation, Inc. | Verbatim copying and distribution permitted; the licence text itself is not modifiable |

Nothing else in this repository is copied. The repository currently holds
specifications only — no code, no fixtures, and no workload generators exist
yet — and all specification prose is original.

## Cited, never copied

Every lab grounds its quirk and its domain in a public document, recorded in
that lab's `Code pointers` section and in the origination column of the
relevant track record under [`specs/0/`](specs/0/). Those documents are
**cited**: no prose, code, fixture, test, or diagram is copied from any of
them, and each lab's failure schedule and checks are original.

The sources cited across the catalog include standards and regulatory texts
(IETF RFCs, EU directives, US Code of Federal Regulations proposals, SEC and
FCC filings), vendor and project documentation (PostgreSQL, Kafka, NATS,
Valkey, ClickHouse, DynamoDB, AWS Lambda, Solana, Ethereum, IPFS, OpenSearch,
PostGIS, Flink), published papers and post-mortems, and public incident
reports. A source marked "citation only" contributes no copied material of any
kind.

## When this file changes

Any commit that copies or embeds an outside file must add a row above in the
same commit. A dependency that is merely *run* — a container image, a package
pulled at build time — is not listed here; it keeps its own licence and its
notice travels with distributed copies, as the licensing contract requires.
