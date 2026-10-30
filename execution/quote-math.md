---
description: The formula behind every Arcter vault quote, where each input comes from, and how to recompute one yourself.
---

# Quote math

{% hint style="info" %}
Arcter is pre-launch and its programs are not deployed. Values below are launch settings, and they will be readable from accounts once markets exist.
{% endhint %}

Vault quotes are arithmetic, not judgment. Each input will sit in a readable Solana account or in the oracle report carried by the transaction, so anyone can recompute any quote.

## The formula

Start with the half-spread, then let skew lean it to one side. Neither side may fall below zero, and the fee is added afterward:

```
half = base_half_spread_bps * regime_multiplier

ask_half = max(0, half - skew_term)
bid_half = max(0, half + skew_term)

ask = mid * (1 + (ask_half + fee_bps) / 10_000)
bid = mid * (1 - (bid_half + fee_bps) / 10_000)

skew_term = max_skew_bps * (current_ratio - target_ratio) / inventory_band
```

`skew_term` is capped at `max_skew_bps` in either direction. A positive value means the vault is long the asset.

## Where each input comes from

| Input | Source | What it pays for |
| --- | --- | --- |
| `mid` | `arcter-oracle`, after age and deviation checks | The asset's reference value |
| `base_half_spread_bps` | Tier setting in `arcter-config` | Holding inventory in regular hours, plus remaining oracle latency |
| `regime_multiplier` | The current [session regime](session-regimes.md) | Jump risk outside regular hours |
| `skew_term` | Market reserves, valued at `mid` | The cost or reward of moving the vault against or toward its target |
| `fee_bps` | `arcter-config` | The protocol fee, 2 bps at launch and shown on its own ticket line |

## From price to raw units

Quotes are stated per scaled token, but programs move raw units. Output is converted through the mint's Scaled UI Amount multiplier and rounded in the vault's favor:

```
ui_out  = usdc_in / ask
raw_out = floor(ui_out / multiplier * 10^token_decimals)
```

The multiplier in force comes from the mint at quote time, chosen using the Clock sysvar when a change is scheduled. Token-2022's standard conversion helpers use floating point and may not round-trip exactly. On-chain math is therefore planned as fixed-point integer arithmetic.

## Launch settings

Values below vary by [tier](../catalog/admission-criteria.md), and changing any of them requires the Squads multisig and its time lock.

| Tier | Example members | Base half-spread and regular-hours clip |
| --- | --- | --- |
| A | SPY, QQQ, AAPL, MSFT, NVDA and peers | 10 bps half-spread on 50,000 USDC clip |
| B | Very liquid single names beyond Tier A | 20 bps half-spread on 20,000 USDC clip |
| C | Remaining, less liquid equity tokens | 40 bps half-spread on 5,000 USDC clip |

The regime then scales each tier:

| Regime | Spread multiplier | Clip multiplier | Tier A half-spread | Tier A clip |
| --- | --- | --- | --- | --- |
| `OPEN` | 1.0 | 1.0 | 10 bps | 50,000 USDC |
| `EXTENDED` | 1.5 | 0.75 | 15 bps | 37,500 USDC |
| `CLOSED` | 3.0 | 0.5 | 30 bps | 25,000 USDC |
| `HALTED` | No quote | 0 | None | None |

With balanced inventory, a Tier A swap therefore pays about 10 bps from mid plus the fee on a weekday session. The same swap on a Saturday pays about 30 bps plus the fee, and the ticket shows why. These are starting points. [Fill reports](../accountability/fill-reports.md) will publish measured results, and any adjustment follows the same governed path.

## How skew rebalances a vault

Skew grows in a straight line as the mix drifts, reaching `max_skew_bps` at the band edge. That cap is 15 bps for Tier A at launch.

Its direction is what matters. Take a vault holding too much of the asset:

* Buying from the vault gets cheaper, since the ask half-spread narrows.
* Selling into the vault pays less, since the bid half-spread widens.

The same reasoning flips for a vault short the asset. Flow that repairs the mix earns a better price, and flow that worsens it pays more. At the band edge the harmful side stops quoting altogether.

Tier A shows why the zero floor matters. At launch settings in `OPEN`, a full 15 bps skew exceeds the 10 bps half-spread. Without the floor, the rebalancing side would be quoted through the mid at the vault's expense.

That is the entire rebalancing mechanism. There are no auctions, no keeper trades and no manual steps.

## The oracle band

Separately from the spread formula, `arcter-router` holds each fill to a fixed distance from the guarded mid. Tier A launches at 75 bps, and Tiers B and C use wider bands. The limit binds vault and RFQ fills alike.

The band is a backstop for failures upstream, such as a broken quote, a forged or abused maker signature, or a wrong parameter value. Whichever part went wrong, a fill priced outside the band fails the transaction.

## Reading the ticket

The ticket lays out each term before you sign:

* the mid, with the oracle report behind it;
* the spread, in bps and USDC, with skew visible;
* the protocol fee, in bps and USDC;
* the current regime;
* the all-in price, which is what your minimum output protects.

Signing fixes a minimum output in raw units. If reserves, the regime or the mid change before inclusion and the result falls below that amount, the program aborts and no tokens move. Network fees for a failed attempt are still spent, as explained in [Costs](costs.md).
