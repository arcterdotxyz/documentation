---
description: The conditions a token must pass before Arcter opens a market, the tier values that follow, and how a market leaves.
---

# Admission criteria

Existing on Solana does not qualify an asset for Arcter. Admission is a pass or fail test against published conditions. Once a market has its tier, its settings follow from a table, so there is nothing left to negotiate.

No market has been admitted yet. The values here apply at launch and pass through the usual governance if they change.

## The five conditions

An asset must pass all five. A single failure means no tier, and there is no override.

1. **A verified price for this exact token.** The reference has to cover the token held in the vault, including how its price relates to any multiplier. Chainlink Data Streams is the intended primary, with Pyth as the cross-check. Coverage is confirmed asset by asset. Prices derived through a wrapper are refused.
2. **A dependable session signal.** The [regime](../execution/session-regimes.md) needs an authoritative view of whether the underlying market is open. Report fields for this need confirming during discovery.
3. **A real issuer and real backing.** The issuer must be identifiable and regulated, with public terms and one-to-one backing. Tracker structures qualify when these hold. An unbacked synthetic with no such issuer does not.
4. **A mint review.** We read the live mint and its token program, record every extension and authority, and publish the result. The review covers transfer hook, permanent delegate, default account state, pause and freeze powers, confidential transfer and Scaled UI Amount.
5. **Behavior the vault can hold.** Vault accounts must be able to hold and move the token, so the hook has to accept them and the issuer must clear them where accounts start frozen. A token charging a transfer fee is refused, since it breaks exact settlement. Multiplier changes through Scaled UI Amount are accepted, provided they can be reconciled with the feed.

Two practical tests sit under condition 5. The heaviest route must fit inside transaction size and compute limits with all hook accounts included, and a simulated fill must succeed against mock mints that copy the real extensions.

## What each tier sets

| Setting | Tier A | Tier B | Tier C |
| --- | --- | --- | --- |
| Starting half-spread | 10 bps | 20 bps | 40 bps |
| Inventory skew limit | 15 bps | 25 bps | 50 bps |
| Band around the oracle | 75 bps | 150 bps | 300 bps |
| Largest vault fill in `OPEN` | 50K USDC | 20K USDC | 5K USDC |
| Launch cap, daily volume | 2M USDC | 750K USDC | 200K USDC |
| Launch cap, vault TVL | 5M USDC | 2M USDC | 500K USDC |

Regime multipliers then apply on top, so the largest fill is x0.75 in `EXTENDED` and x0.5 in `CLOSED`.

## Which tier applies

| Tier | Typical holdings |
| --- | --- |
| A | Major index funds and mega-cap names such as SPY, QQQ and NVDA, with deep home-market liquidity and fast data |
| B | Liquid single stocks below mega-cap size |
| C | Everything else that passes the five conditions |

Three measures decide placement:

- liquidity at home, based on average daily volume and the usual spread on the primary exchange
- how often its reference data updates
- how widely the token is held across wallets on Solana

Tier C exists on purpose. Slower data and wider home-market spreads justify a wider band and spread, and caps hold the exposure at a proportionate size. Serving these names well is part of Arcter's reason for existing.

The same underlying from two issuers makes two separate markets. Each has its own mint, its own review and its own tier, because the issuer's terms and controls differ.

The tier and its evidence are published for every market. A move between tiers passes through the Squads multisig and its time lock, like other settings.

## Controls found during review

A mint review can find controls that do not disqualify an asset but still matter. A permanent delegate is the clearest example. Findings go on the listing page and can lead to a lower tier or tighter caps. Vault depositors should read them first.

## After admission

Admission holds only while the conditions do. A keeper compares the mint's settings with the recorded ones, and any change pauses the market until someone reviews it.

An asset that stops qualifying loses its market. Likely causes include a withdrawn feed, an issuer ending the token, or a new mint control that fails review. The market moves to `RETIRED` through the time lock with public reasons, and from then on it only processes withdrawals. See [Splits and dividends](../execution/splits-and-dividends.md).

## Assets beyond equities

Gold and yield-bearing dollars face the same five conditions, each with one addition. An asset priced from net asset value needs a feed that follows that value and trades in a narrow band around it. An asset that reaches Solana through a bridge or omnichain route needs a bridge review, described in [Expansion](expansion.md).
