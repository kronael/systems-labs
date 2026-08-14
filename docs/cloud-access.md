---
status: reference
---

# Cloud access

Every required gate in this curriculum runs locally with no cloud account.
Cloud use exists only for the opt-in `make smoke` check, and the permitted
services are AWS Lambda, SQS,
DynamoDB on-demand, and short-retention logs. This document covers the three
things a learner reaching that optional step needs: getting an account, not
getting billed, and knowing which services each phase touches. The course
never promises that an account is free of charge; the guardrails below are
what keep the bill at zero in practice.

All numbers below were read from the cited vendor pages on 2026-08-14. AWS
free-tier plans and allowances have changed before (most recently the
credit-based plan structure) and will change again; the cited page is
authoritative on the day it is read, not this file.

## Before anything else

A zero-spend budget and a billing alarm come before the first deployed
function. The order is fixed: activate the account, secure the root user,
create the budget, enable the billing alarm, and only then create
credentials or resources.

The shortest correct path to the budget: open the Billing and Cost
Management console, choose **Budgets**, then **Create budget**, then **Use a
template (simplified)**, then **Zero spend budget** — "a budget that notifies
you after your spending exceeds AWS Free Tier limits" — enter an email
address, and choose **Create budget**
([budget templates](https://docs.aws.amazon.com/cost-management/latest/userguide/budget-templates.html),
checked 2026-08-14). Creating the first budget also enables Cost Explorer;
its graph takes up to 24 hours to appear
([creating a budget](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-create.html)).

The billing alarm is a second, independent tripwire. In the Billing console,
under **Billing Preferences → Alert preferences**, the learner enables
**Receive CloudWatch Billing Alerts**; billing metric data lands in the US
East (N. Virginia) Region and represents worldwide charges, updated several
times daily. A CloudWatch alarm on the `EstimatedCharges` metric (created in
us-east-1, threshold 1 USD, notification to an SNS email topic) then fires
when actual — not projected — charges cross the threshold
([billing alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html),
checked 2026-08-14).

## Account creation

Account creation itself costs nothing. Signing up requires an email address
and root password, contact information, a payment method (credit or debit
card), phone-number verification by an automated PIN, and a support plan
choice (Basic is free). Activation usually completes within minutes but can
take up to 24 hours
([create an AWS account](https://aws.amazon.com/resources/create-account/),
checked 2026-08-14).

During signup the account chooses a plan. On the **Free account plan** "you
will not incur any charges during this period until you upgrade to a paid
account plan"; the plan ends after six months or when credits are exhausted,
whichever comes first, and the account then closes on its own unless
upgraded
([Free Tier documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html),
checked 2026-08-14). For this curriculum's bounded smoke runs, the free
account plan is the right choice.

The root-user protection step follows immediately: the learner enables MFA
on the root user and stops using it for everyday work. AWS's own guidance is
to safeguard root credentials like other sensitive personal information and
to require MFA wherever a root or IAM user exists
([IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html),
checked 2026-08-14).

## Credentials without long-lived keys

AWS's stated best practice is that human users access AWS with temporary
credentials through federation, and it recommends IAM Identity Center for
that
([IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)).
The practical path for a single learner account: enable IAM Identity
Center, create a user with a least-privilege permission set, then run
`aws configure sso` once and `aws sso login` per session. The CLI caches an
SSO token under `~/.aws/sso/cache` and uses it to retrieve short-lived AWS
credentials for the profile's role, renewing them automatically while the
session token is valid
([CLI with IAM Identity Center](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html),
checked 2026-08-14). Nothing long-lived ever lands on disk in plaintext.

Where that path is unavailable — a tool that cannot speak the SSO flow —
the fallback is an IAM user with an access key, and AWS's guidance treats
this as the exception: least-privilege permissions, MFA on the user, and
keys updated or removed when no longer needed
([IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)).

This repository consumes credentials one way: the optional second CLI
argument, a TOML secrets file, which `make smoke` reads from the conventional
`keys.toml` path. The secrets file names the profile or carries the key
material, lives outside version control, and is never committed. Environment
variables carry runtime-injected secrets only, where a platform requires
them. A credential in a tracked file is a policy violation regardless of the
account it opens.

## What each phase actually needs

Required gates never need a cloud account in any phase. The column "cloud
account" below answers whether the phase's *optional* smoke step can use
one at all.

| Phase | Subject | Cloud account | Optional services touched | Local substitute satisfying required gates |
|-------|---------|---------------|---------------------------|--------------------------------------------|
| 1 | Local runtime, delivery, and cross-system atomicity | No | None. Every dependency is software the learner runs | PostgreSQL, Kafka, NATS JetStream in Compose |
| 2 | The same products on a serverless execution model | Optional | Every lab: a hosted run confirms that the emulated freeze and concurrency behaviour match the service. Permitted services only — Lambda, SQS, DynamoDB on-demand, short-retention logs | Lambda-compatible runner, ElasticMQ, DynamoDB Local in Compose |
| 3 | Real Internet streaming | No | None (RIPE RIS Live is a public stream; recording is opt-in and needs no account) | Kafka, Flink, PostgreSQL, checkpoint storage in Compose; generated or cached recordings |
| 4 | NoSQL, analytics, and platform portability | Optional | Lab 4/1: bounded hosted DynamoDB on-demand smoke, short-retention logs. Lab 4/5: the same permitted services through the bounded OpenTofu account shell | DynamoDB Local, Valkey, ClickHouse, Kafka, ElasticMQ in Compose; `kind`, Lambda-compatible runner, OpenTofu local sandboxes (ElastiCache is excluded, so the cache lab is local-only) |
| 6 | Low-level (Rust and C, kernel quirks) | No | None | The local Linux kernel is the laboratory |
| 7 | Blockchain (Solana and Ethereum) | No AWS; optional non-AWS RPC | Public RPC provider free tiers, opt-in, bounded, cached | `solana-test-validator` and a local Ethereum development node |
| 8 | Search, retrieval, and spatial | No | None | OpenSearch and PostGIS in Compose; public corpora downloaded once and cached |

Phase 5 was merged into phase 4 (its lab is now 4/5) and the gap at 5 is
deliberate. The original one-lab phase 2 became 1/5; the phase 2 above is the
serverless contrast catalog that later took the number.

## Free tier, with the real numbers

Each number below was read on 2026-08-14 from the page cited next to it.
AWS states plans and offers change; treat these as a snapshot.

**Account credits.** A new account receives USD 100 in credits at creation,
regardless of plan, and can earn up to USD 100 more by completing activities
— "up to $200 over 6 months". Over 30 services are "always free within
monthly usage limits on both the Free and Paid plans"; usage beyond an
always-free limit draws down credits automatically
([aws.amazon.com/free](https://aws.amazon.com/free/)). The free account plan
never charges; it ends at six months or credit depletion, whichever comes
first
([Free Tier documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html)).

**Lambda.** The free tier is one million requests and 400,000 GB-seconds of
compute per month
([Lambda pricing](https://aws.amazon.com/lambda/pricing/)). The pricing page
does not label the offer's duration; the Free Tier documentation files
Lambda's free requests under the always-free category
([Free Tier documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html)).

**SQS.** "All customers can make 1 million Amazon SQS requests for free each
month." Every SQS action counts as a request, each 64 KB chunk of a payload
bills as one request, and unused free usage does not roll over
([SQS pricing](https://aws.amazon.com/sqs/pricing/)).

**DynamoDB.** The perpetual monthly free allowance is 25 GB of storage, 25
WCUs, and 25 RCUs, and the page ties those benefits to provisioned capacity
on the Standard table class — on-demand mode does not carry the same
perpetual request allowance. On-demand requests bill at $0.125 per million
reads and $0.625 per million writes; storage beyond 25 GB bills at $0.25 per
GB-month
([DynamoDB on-demand pricing](https://aws.amazon.com/dynamodb/pricing/on-demand/)).
A bounded smoke run's request bill is a fraction of a cent, and the signup
credits cover it; an idle on-demand table bills storage only.

**CloudWatch Logs.** The always-free allowance is 5 GB per month covering
ingestion, archive storage, and data scanned by Logs Insights queries
combined. Beyond it, ingestion starts at $0.50 per GB and compressed archive
storage at $0.03 per GB
([CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)). Short
retention keeps stored volume far below the allowance.

**Cost Explorer.** Viewing costs in the console is free; each paginated API
request costs $0.01, and Cost Explorer cannot be disabled once enabled
([Cost Explorer documentation](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)).

## Staying at zero

- **On-demand tables only.** An on-demand DynamoDB table bills per request
  and per stored GB; nothing accrues while it sits idle. Provisioned
  capacity, by contrast, meters by the hour whether or not it is used.
- **Short log retention.** Every log group created by a smoke run gets an
  explicit retention of a few days. The 5 GB monthly allowance covers
  ingestion and storage together, so unbounded retention is the slow leak.
- **Delete what was created.** The smoke run's OpenTofu shell applies
  ownership and expiry tags and tears its resources down; a smoke run that
  ends without a destroy is unfinished. Lambda functions, queues, tables,
  and log groups are the complete inventory to check.
- **Why the exclusions exist.** MSK, EKS, NAT gateways, RDS, and ElastiCache
  are excluded by policy because they bill by the hour whether or not the
  learner uses them. A forgotten cluster or gateway turns a free curriculum
  into a monthly invoice; a forgotten on-demand table does not.
- **Check spend afterwards.** Cost Explorer shows the current month with
  roughly a 24-hour delay and refreshes at least daily
  ([Cost Explorer documentation](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)),
  so the day after a smoke run is the right time to look. The zero-spend
  budget and the billing alarm cover the case where nobody looks.

## Blockchain access

Phase 7 runs locally first. `solana-test-validator` "starts a full-featured,
single-node cluster on the developer's workstation" with no RPC rate limits
and no airdrop limits; `solana airdrop 10` funds a local wallet from thin
air ([Anza test validator docs](https://docs.anza.xyz/cli/examples/test-validator),
checked 2026-08-14). Ethereum development uses a local blockchain instance —
a private development network that allows much faster iteration than any
public testnet
([ethereum.org networks](https://ethereum.org/en/developers/docs/networks/),
checked 2026-08-14). Neither costs anything, and neither needs an account.

Public RPC providers are the opt-in path, and their free tiers carry the
bounded recording workloads this track allows. Helius's free plan for Solana
is $0 per month with 1M credits, 10 requests per second, and 1
`sendTransaction` per second
([helius.dev/pricing](https://www.helius.dev/pricing), checked 2026-08-14).
Alchemy's free tier for Ethereum includes 30M compute units per month at 25
requests per second and 500 CU/s
([alchemy.com/pricing](https://www.alchemy.com/pricing), checked
2026-08-14). Both plans change at the provider's discretion. Per the track
contract, public RPC access is opt-in, bounded, cached under the shared data
prefix, and never on a request path.

Test value comes from faucets, not purchases. The Solana Foundation faucet
distributes free devnet and testnet SOL — a maximum of 2 requests every 8
hours without authentication — and "does not distribute mainnet SOL"
([faucet.solana.com](https://faucet.solana.com/), checked 2026-08-14).
Testnet ETH "is supposed to have no real value" and "most people get testnet
ETH for free from faucets"; Sepolia is the recommended default testnet for
application development
([ethereum.org networks](https://ethereum.org/en/developers/docs/networks/)).
No mainnet funds are ever used in this curriculum: no lab buys, holds, or
transfers tokens with monetary value, and nothing in any grader depends on
a funded mainnet account.

## Sources

All fetched and read on 2026-08-14.

- <https://aws.amazon.com/free/> — signup credits ($100 plus up to $100
  earned), free versus paid plan, 30+ always-free services.
- <https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html>
  — free account plan mechanics: no charges until upgrade, six-month or
  credit-depletion end, always-free versus short-term-trial offer classes.
- <https://aws.amazon.com/lambda/pricing/> — 1M requests and 400,000
  GB-seconds per month free.
- <https://aws.amazon.com/sqs/pricing/> — 1M requests per month free; 64 KB
  chunk billing; no rollover.
- <https://aws.amazon.com/dynamodb/pricing/on-demand/> — 25 GB / 25 WCU /
  25 RCU free allowance (provisioned, Standard class); on-demand $0.125 per
  million reads, $0.625 per million writes, $0.25 per GB-month.
- <https://aws.amazon.com/cloudwatch/pricing/> — 5 GB free Logs allowance;
  $0.50 per GB ingestion and $0.03 per GB archive storage beyond it.
- <https://docs.aws.amazon.com/cost-management/latest/userguide/budget-templates.html>
  — the Zero spend budget template and its console path.
- <https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-create.html>
  — budget creation; first budget enables Cost Explorer.
- <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html>
  — enabling billing alerts, us-east-1 metric residency, alarm on actual
  charges.
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html> —
  temporary credentials via Identity Center for humans, long-lived keys as
  exception, root-user safeguards and MFA.
- <https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html>
  — `aws configure sso`, `aws sso login`, cached auto-refreshed short-lived
  credentials.
- <https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html>
  — Cost Explorer free in console, $0.01 per API request, ~24-hour data
  delay.
- <https://aws.amazon.com/resources/create-account/> — signup steps: email,
  contact details, credit or debit card, phone PIN verification, support
  plan, activation time.
- <https://docs.anza.xyz/cli/examples/test-validator> — local single-node
  Solana cluster, no rate or airdrop limits.
- <https://faucet.solana.com/> — free devnet/testnet SOL, 2 requests per 8
  hours unauthenticated, no mainnet SOL.
- <https://ethereum.org/en/developers/docs/networks/> — testnet ETH is free
  and valueless by design, Sepolia as default testnet, local development
  networks.
- <https://www.helius.dev/pricing> — Solana RPC free plan: 1M credits per
  month, 10 req/s, 1 sendTransaction/s, $0.
- <https://www.alchemy.com/pricing> — Ethereum RPC free tier: 30M compute
  units per month, 25 req/s, 500 CU/s.
