---
description: How an Arcter market holds one asset and USDC, quotes around an oracle price, and accounts for depositor shares.
---

# Vault liquidity

{% hint style="info" %}
Arcter is pre-launch and its programs are not deployed. Parameters below are launch values, and the behavior described is the intended design.
{% endhint %}

Arcter has no shared pool. Each listed asset gets its own market in `arcter-vault`, stocked by depositors with that asset and USDC. Unless halted, the market shows a buy price and a sell price around the oracle mid, up to a size limit called the clip.

## What a market is made of

`create_market` adds a listing without new code. It produces:

| Account | Role |
| --- | --- |
| Market account (a PDA) | Parameters, state and accounting for that asset |
| Two reserve token accounts | The asset and USDC, controlled by a program PDA and not by any person |
| Share mint | A plain SPL mint for this market, redeemable pro rata |

A reserve may sit under the classic Token program or Token-2022. The code handles both, since USDC's token program is still to be confirmed. Before a market can open, an issuer using default account state must clear its reserve account, and a mint with a transfer hook must accept a program-owned account. Whether each issuer will do so is an open question for discovery.

Every swap writes to its Market account. Swaps on the same market therefore run one after another, while swaps on different markets can run in parallel under Sealevel. A two-leg swap locks both markets for its duration.

## Why reserves do not set the price

A curve-based pool prices from its own balances. When the outside price moves, that pool stays wrong until an arbitrageur corrects it and keeps the difference. Arcter reverses the order of events. The oracle sets the middle of the market, and reserves only decide how the spread leans. Depositors keep the spread and are not paying that same arbitrage.

The price of this design is full dependence on oracle data. [Price data defenses](../safeguards/price-data-defenses.md) sets out how that dependence is contained.

## Raw units and the multiplier

Token-2022 mints may use Scaled UI Amount. A split or dividend changes the multiplier, while raw balances stay exactly as they were. All vault accounting therefore runs on raw token units. The multiplier is applied only to convert for pricing and display.

```
token_ui    = token_raw / 10^token_decimals * multiplier
vault_value = usdc_balance + token_ui * mid
```

The multiplier is read fresh for each quote. The extension can schedule a new multiplier with an effective timestamp, so the program compares that time with the Clock sysvar and uses whichever value applies. Each listing must confirm whether its reference price is per scaled token, since an error here would misprice every fill. See [Splits and dividends](splits-and-dividends.md).

## Launch limits

A market aims for a set mix of its two assets, measured by value, and must stay inside a hard band around that mix. Tier A opens with:

| Parameter | Meaning | Tier A launch value |
| --- | --- | --- |
| `target_ratio_bps` | Intended share of vault value held in the asset | 5,000 bps (50%) |
| `inventory_band_bps` | How far the mix may stray before quoting turns one-sided | 2,000 bps (+/-20%) |
| `max_clip` | Largest single fill | 50,000 USDC |
| `daily_volume_cap` | Volume limit per market during the guarded launch | 2,000,000 USDC |

These values are set at listing, after [admission review](../catalog/admission-criteria.md). They live under `arcter-config`, and any change must pass the Squads multisig and its time lock.

## How one-way flow is handled

Suppose buyers keep arriving. Asset reserves fall, USDC builds up, and the mix drops below target. [Skew](quote-math.md) answers by making further buying more expensive and selling to the vault more attractive. Outside traders are thereby paid to refill the vault.

If buying continues until the mix reaches the band edge, the market stops selling the asset. Its risk is capped at that point. Selling to the vault stays open throughout, because that direction reduces the imbalance. The same logic runs in reverse when sellers dominate.

## Depositor shares

Share accounting is built on `vault_value`, using the guarded mid:

* A deposit mints shares at the current value per share.
* Spread collected stays in the reserves and raises that value.
* A withdrawal burns shares and pays a pro rata portion of both reserves, in kind.

Paying exits in kind means a departing depositor never forces the vault to trade. No Arcter pause blocks an exit. An issuer action can still interfere with one asset side, as the next section explains. [Earning as an LP](earning-as-an-lp.md) covers the economics.

## Market states

| State | Quoting | Depositor access |
| --- | --- | --- |
| `ACTIVE` | Both sides, within the current regime's limits | Deposits and withdrawals |
| `ONE_SIDED` | Only the side that reduces the imbalance | Deposits and withdrawals |
| `HALTED` | None | Continue where safe |
| `RETIRED` | None, permanently | Withdrawals only |

A market enters `HALTED` on any trigger listed under [session regimes](session-regimes.md). `RETIRED` follows a delisting.

## Issuer powers over reserves

Reserves hold real issuer-controlled tokens, and these controls apply to a vault as they would to any account:

* **Permanent delegate.** An issuer holding this power can move tokens out of any account, a vault included. Depositors carry that risk, and the listing page discloses it.
* **Freeze or pause.** While the asset cannot move, the market is `HALTED`. Withdrawing the unaffected USDC side continues, but the asset side may be stuck until the issuer acts.
* **Transfer hook.** An in-kind payout to a wallet must satisfy the hook, so a wallet the issuer rejects may be unable to receive the asset.

Deposits depend on a trustworthy mid to price new shares. Whether they stay open in a halt where the mid is in doubt must be confirmed before launch. For the wider picture, see [Issuer dependence](../safeguards/issuer-dependence.md).

## Required properties

Nothing here is yet proven on a live deployment. These are the requirements the programs must meet, to be tested with LiteSVM or Mollusk unit tests and Trident fuzzing against real Token-2022 configurations:

1. No fill settles beyond the oracle band, in any state or under any parameters.
2. The vault's spread on a fill is never negative, so a fill does not reduce value per share.
3. While `HALTED` or `RETIRED`, withdrawals are not blocked by Arcter's own logic.
4. At the band edge, the market stops offering the inventory that would widen the imbalance.
5. Share accounting gives the same result whatever path the trades took, and a multiplier change alone moves no raw balance.
