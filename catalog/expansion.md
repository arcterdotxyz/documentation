---
description: Which assets Arcter plans to add after equities, what each must prove on Solana, and what stays off the venue.
---

# Expansion

Arcter widens its catalog when an asset can pass [Admission criteria](admission-criteria.md) on Solana. An announcement does not count, and neither does demand. The order below is a plan and it may change as discovery answers come in.

## Planned sequence

| Step | Assets | Depends on |
| --- | --- | --- |
| First release | Six to twelve equity and ETF markets, plus PAXG | Oracle coverage, vault clearance and mint review |
| Wider equities | Further Tier B and Tier C names, in batches | Confirmed data and review for each name |
| Yield-bearing dollars | Candidates such as USDY | Asset review and a price source for net asset value |
| Later classes | Anything else with an outside reference | The same conditions |

## Equities: from anchors to the long tail

The first markets are the ones easiest to hedge. Later batches extend through the catalog as each asset clears its review. Long-tail names are where a venue built around a reference price should be most useful, since thin general pools tend to misprice them. Their caps stay small for the same reason. Timing appears in [Delivery phases](../plan/delivery-phases.md).

## Gold

Gold has a spot reference, and tokens for it already exist on Solana. PAXG arrived in June 2026 from Paxos, via Sunrise. XAUt0, Tether Gold's omnichain token, is also present. Kamino opened a PAXG market on 27 July 2026.

Three matters need care:

- **Sessions.** Gold has no stock exchange hours, but spot markets do shut. Gold markets use the same closed-session protection during those periods, so the regime still applies.
- **Depth.** Solana gold pools are thin today. These markets may launch with low caps and tight clip sizes.
- **Route to Solana.** Each token gets a bridge review covering how supply is minted on Solana, what the custodian's attestations say, and how holders are affected by a pause on that route.

A dedicated price source for the exact token is still required.

## Yield-bearing dollars and treasuries

Tokenized treasuries passed $10B across all networks in February 2026, and a trader leaving a stock position may want somewhere to rest. Yield-bearing dollars and funds exist on Solana, with USDY as one example. None has been approved, and each must be confirmed before listing.

| Requirement | What it means here |
| --- | --- |
| Identified issuer and public terms | As for any listing, read together with who may hold the token |
| Mint review | Full inspection of extensions and authorities |
| Price source | A feed that tracks published net asset value for that exact token |
| Quoting | A narrow band around net asset value; closed-session rules would give way to the fund's own accrual, subject to review |
| Access | Where a token restricts holders, a market can require an attestation, and the issuer must clear the vault |

Where a token enforces eligibility at the mint, a wallet passes two checks: the issuer's rule and the Arcter attestation.

## What stays off the venue

Some assets are excluded at every stage:

- **Wrappers.** Receipt tokens and vault shares need a computed value, and Arcter prices only the token it holds. See [Precedents](../safeguards/precedents.md).
- **Assets with no outside reference.** Memecoins and volatile pairs give the design nothing to anchor to.
- **Unbacked synthetics.** A token without an identifiable regulated issuer and one-to-one backing fails review.
- **Undisclosed controls.** A mint carrying transfer restrictions the issuer has not disclosed does not list.

## Later classes

New kinds of real-world asset will reach Solana in time. Each goes through the same conditions. If it lacks a token-specific price, a session or valuation signal, or public issuer terms, it stays off Arcter regardless of interest.

US-eligible markets are a separate matter. They would need a different legal and product structure, and that decision is held for later.
