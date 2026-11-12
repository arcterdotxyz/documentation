---
description: The four planned phases from first build to a public platform, the test each one must pass, and what stays off the plan.
---

# Delivery phases

Arcter is a pre-launch concept as of November 2026. Nothing is live, no programs are deployed and no audit has been completed. Everything below is a plan.

## Plan at a glance

| Phase | Timing | Outcome |
| --- | --- | --- |
| Discovery and build | Months 0-2 | Open questions answered, Anchor programs tested against mock mints that copy real extensions, public specification |
| Launch | Months 3-5 | Audited programs, six to twelve markets, swap, open ledger and maker lane |
| Depositors and holdings | Months 5-8 | Public vaults, wallet view with corporate action accounting, wider catalog by liquidity tier |
| Platform | Months 8-12 | Quote API, SDK, aggregator and wallet integrations |

Months are targets. Each exit test below is what decides whether a phase ends, so a date can slip while a test cannot be waived.

## Discovery and build

This phase settles the questions the design cannot answer from documents:

* which extensions are active on each target mint, and whether hooks accept program-owned vault accounts
* whether issuers will clear vault accounts, and on what terms
* which assets have Data Streams reports on Solana, with what session fields and at what cost
* whether makers can reach mint and redeem
* whether the heaviest routes fit inside size and compute limits
* how a market should behave if Circle freezes a vault's USDC reserve

**Exit test:** each question has a written answer, the programs pass their tests against mock mints, and the specification is public. An asset without confirmed oracle coverage does not move forward. Every later release is rehearsed the same way before it ships. See [Releases and addresses](../system/releases-and-addresses.md).

## Launch

Launch stays narrow on purpose:

* six to twelve markets where hedging is easiest, such as SPY, QQQ, NVDA and TSLA, plus PAXG
* swap through USDC, with launch caps from [Admission criteria](../catalog/admission-criteria.md)
* the maker lane, with two or three makers already active on tokenized equities
* the open ledger, and the dataset behind [Fill reports](../accountability/fill-reports.md) from the first fill

**Exit test before launch:** two independent reviews published, a public bug bounty open, and scripted drills passed for stale reports, disagreeing oracles, multiplier changes and cluster restarts.

**Exit test to the next phase:** a 60-day record in which the median `OPEN` fill meets its tier target, with no band or withdrawal invariant event.

## Depositors and holdings

Public vaults open here, each with a plain SPL share mint. Availability will depend on the [regulatory](../legal/regulatory-position.md) review for each jurisdiction. Holdings arrives alongside them, with scaled balances, cost basis and corporate action accounting. The catalog widens by liquidity tier, in batches. Cap increases pass through the time lock and cite the record so far.

**Exit test:** vaults in each tier hold two-sided quotes with no manual help through at least one long weekend and at least one high-volatility week.

## Platform

Developers get a REST and streaming quote API plus a typed TypeScript SDK. See [Developer access](../system/developer-access.md). Aggregator and wallet integrations follow. Registering with Jupiter and other aggregators lets a competitive quote reach users Arcter has not had to win directly.

## How progress will be judged

Five measures, tracked from launch:

1. 30-day volume
2. measured execution against the reference price
3. vault deposits
4. share of Solana's tokenized equity volume
5. active integrations

Solana Foundation programs, Colosseum hackathons and aggregator partnerships are the expected early channels. None of these is a confirmed partnership.

## Held for later

These ideas have no phase and no date:

* settling in a yield-bearing dollar, which adds accounting complexity
* limit and time-weighted orders on top of vaults, once core pricing has proven itself
* US-eligible markets, which would need a different legal and product structure

## Off the plan

Leverage, margin, perpetuals, lending and a fiat ramp are not coming. Neither are memecoins. Other protocols can serve those needs. Every phase here points at one goal: the most transparent place on Solana to trade a tokenized stock at any hour.
