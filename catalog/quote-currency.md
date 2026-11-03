---
description: Why every Arcter market settles in USDC, what that dependency means, and how units are converted across mints.
---

# Quote currency

Every Arcter market is priced and settled in USDC. Spreads, fees, vault values and caps are all stated in it, and an order between two assets passes through it.

## USDC on Solana

| Item | Detail |
| --- | --- |
| Issuer | Circle |
| Presence on Solana | Native, so settlement needs no wrapped token |
| Token program | Classic SPL Token program |
| Decimals | Read from the mint, 6 for USDC |
| Supply on Solana | Roughly $6.6-7.3B, September to October 2026, from third-party trackers |
| Share of all USDC | About 9-10%, of roughly $74B |
| Share of Solana stablecoins | About 44% of $16B, mid-September 2026 |
| Moving between networks | Circle's CCTP, by burning and minting |

## Why a single settlement asset

Each extra quote currency would split a market's depth into separate pools. One unit keeps prices comparable, makes vault value a single number, and gives a two-asset route a simple shape that programs can verify in one transaction. See [Order routing](../execution/order-routing.md).

USDC is the natural choice on Solana, as the dollar most traders, makers and pools already hold. It also spares most users a conversion step before they can trade.

## What this dependency costs

Every market relies on one issuer.

* **Freezes.** Circle can freeze USDC in any account, by design. A freeze on a vault's USDC reserve would stop that market and hold back the USDC side of withdrawals.
* **Peg.** USDC is assumed to be worth one dollar. Monitoring is planned, and a depeg would distort every quote until the market halts.
* **Redemption.** Who can redeem directly with Circle is Circle's decision. Most holders exit by selling.
* **Network movement.** CCTP depends on Circle's attestation service, which paused for Solana in May and June 2026. Those pauses affect moving USDC between networks, not settlement on Solana.

The [risk model](../safeguards/risk-model.md) lists this dependency.

## Adding another quote asset later

Nothing is planned. Any addition would have its own isolated markets, would need the same mint review as a listing, would have to show real liquidity on Solana, and would go through the Squads multisig and its time lock.

Settling in a yield-bearing dollar is a separate idea, held for later because it adds accounting complexity.

## Units and conversion

Solana mints declare their own decimals, and Arcter never assumes a value. Programs read decimals from each mint when a market is created, and vault math runs on raw units.

| Quantity | Handling |
| --- | --- |
| USDC balance | Raw units, using the decimals on the mint |
| Asset balance | Raw units, never scaled |
| Multiplier | Read from the mint per quote, or 1.0 where the asset has none |
| Price from `arcter-oracle` | USDC raw units per displayed token |

Vault value then comes to:

```
vault_value = usdc_raw + token_raw x multiplier x price / 10^token_decimals
```

Where a report's price already includes the multiplier, the formula must not apply it twice, and this is settled per asset. See [Splits and dividends](../execution/splits-and-dividends.md).

Rounding on shares favors the vault over the person transacting.
