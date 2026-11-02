---
description: What a vault depositor is paid for, which risks come with it, and where issuer controls limit a withdrawal.
---

# Earning as an LP

{% hint style="info" %}
Vault deposits are planned for release 1.1. No program is deployed and no vault is open. The terms below are design targets.
{% endhint %}

A depositor in an Arcter vault sells immediacy. Traders pay a spread to fill now, in a market with an outside reference price. Income comes from that spread, and the design tries to keep depositors from simply funding other people's arbitrage.

## What pays you

Each vault fill adds its realized spread, less a 10% [protocol share](costs.md), to the vault at the moment of the fill. Nothing else pays depositors. Token emissions are not planned, and there are no points.

Income is measured in the open. The [explorer guide](../guides/reading-the-explorer.md) shows how to check a market's reserves and fills on Solscan or the Solana Explorer, so a vault can be judged on recorded earnings.

## What can cost you

Read this table before depositing. A vault can lose money.

| Risk | How it reaches you | What limits it |
| --- | --- | --- |
| Market price | The vault holds the asset, so a falling share price reduces your value | You pick the market; each has its own reserves |
| Overnight and weekend gaps | A fill in `CLOSED` can sit well away from the next open | Spread x3.0 and clip x0.5 charge for this, yet a large gap can exceed both |
| Bad price data | A wrong mid gives wrong fills | Age checks, a Pyth cross-check, the 25% jump guard and the band on each fill, described in [Price data defenses](../safeguards/price-data-defenses.md) |
| One-way flow | Persistent selling leaves the vault heavy in the asset | Skew makes the crowded side dearer, and at the band edge the vault quotes one side only |
| Issuer credit | The asset side is a claim on an issuer | Disclosed per market; see [Issuer dependence](../safeguards/issuer-dependence.md) |
| Issuer controls | A permanent delegate can move tokens out of any account, including a vault, and a pause or freeze can block transfers | Mint settings are recorded at listing and disclosed; a change pauses the market |
| Program risk | Every market runs on the same programs, and no audit has been completed yet | Two independent reviews and a bug bounty are intended before launch |

Vaults use no leverage and cannot be liquidated. Reserves are held per market, and vaults do not lend to each other, so a loss on one asset does not draw on another. A flaw in shared program code is different, because it could affect every market at once.

## Depositing

A vault is one Market account holding one asset and USDC. To deposit, a wallet needs an `LP` attestation in `arcter-access`. It may supply USDC, the market's asset, or both. The contribution is valued at the verified mid, and the wallet receives vault shares at the current value per share.

| Item | Terms |
| --- | --- |
| Charge on entry or exit | None in Arcter fees; SOL network costs still apply |
| Minimum holding period | None |
| Share form | A plain SPL share mint per vault, redeemable pro rata |
| Deposit limit | The market's cap during launch |

Value per share is worked out as follows, with token balances in raw units:

```
value_per_share = (usdc_raw + token_raw_value) / total_shares
token_raw_value = token_raw x multiplier x mid / 10^token_decimals
```

`mid` is stated in USDC base units per displayed token. The `multiplier` is read fresh from the mint where the asset uses one, as explained in [Splits and dividends](splits-and-dividends.md).

A deposit depends on a verified mid. It cannot be priced while that mid is unavailable, so expect deposits to wait during a halt caused by missing or disputed data.

Because the share mint is a plain SPL token, the token program itself does not limit who can receive shares. Any restriction on share holders would have to sit in `arcter-vault` at redemption. The final rule is to be confirmed with counsel before launch.

## Withdrawing

Burning shares returns a pro rata slice of what the vault holds, in kind. A 1% redemption pays out 1% of each reserve.

This choice has three effects:

- Your exit does not sell anything, so it moves no price and leaves nothing to sandwich.
- Withdrawing needs no mid, which allows exits in `HALTED` and in `RETIRED`.
- Any inventory skew is reflected in your mix. An LP who wants only USDC can then sell the asset through a normal swap.

There is one real limit on this. The asset side of a withdrawal is a token transfer, and on Solana the issuer's rules apply to it.

| Condition at the mint | Effect on your exit |
| --- | --- |
| Issuer pause or freeze | The USDC side can still leave; the asset side waits for the issuer |
| Transfer hook rejects your wallet | The asset transfer fails until your wallet passes the issuer's rule |
| Default account state is frozen | Your token account may need issuer clearance before it can receive |

So a pause, a halt or a settings change at Arcter cannot trap capital, but an issuer's control can delay the asset side. The exact handling of a part-USDC, part-delayed redemption is still being specified, and each market page will disclose the controls found on its mint.

## Caps during launch

Each market opens with a deposit cap and a daily volume cap, set by [tier](../catalog/admission-criteria.md). These keep total exposure small while parameters are tested against real flow. Raising a cap goes through the Squads multisig and its time lock, with [fill reports](../accountability/fill-reports.md) as the evidence and [Delivery phases](../plan/delivery-phases.md) as the schedule.
