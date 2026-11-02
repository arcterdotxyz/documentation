---
description: How Arcter plans to handle splits, dividends, mergers and delistings, built on raw balances, the Scaled UI Amount multiplier and verified event data.
---

# Splits and dividends

{% hint style="warning" %}
Issuer behavior differs by asset, and Arcter has not yet confirmed it mint by mint. Everything here is planned design. Each listing must establish how its own token behaves, and a token that cannot be established does not list.
{% endhint %}

A stock's issuer can change what one token means overnight. A venue that misses the change sells value to the first arbitrageur who notices. Arcter's plan rests on one split in the data: raw balances stay fixed, and everything that moves is converted at quote time from a multiplier and a verified price.

## Why units matter

Suppose a vault holds 100 displayed tokens with a multiplier of 1.0 and a mid of 500 USDC. Its asset side is worth 50,000 USDC.

After a 10-for-1 split, the raw balance has not moved. The multiplier is 10.0, the display reads 1,000 tokens, and the mid is 50 USDC. The value is still 50,000 USDC.

The danger is a mismatch. An old mid with the new multiplier values the position at 500,000 USDC, and a new mid with the old multiplier values it at 5,000 USDC. Either way, the vault would quote a price wrong by a factor of 10. These figures are an illustration, and the band and jump guard exist to stop such a fill.

## How the two issuers treat events

| | xStocks | Ondo Global Markets |
| --- | --- | --- |
| Mechanism seen or described | Scaled UI Amount, found on mint inspection; the multiplier moves for splits and dividends while raw balances stay put | Ondo describes a display multiplier for supported Solana wallets, with on-chain balances unchanged; other wallets reflect the change in the per-token price |
| Dividends | Balances adjust upward, net of applicable withholding, per Kraken's guide | Economics of reinvesting, net of withholding; Ondo's FAQ cites 30% US withholding |
| Splits | Multiplier moves; event timing to confirm | Not explained on Ondo's overview; to confirm |
| Pause behavior | Issuer or feed pause to confirm per event type | Ondo says trading can pause during corporate actions |

Ondo also notes that one token can come to represent more than one share, so its token price and the stock price can diverge. Arcter must therefore know, per asset, whether a reference price is per share or per token. Whether Ondo's multiplier is Scaled UI Amount on each mint needs confirming from the mint itself.

## What the programs do

Four rules follow from the above.

**Vault math stays in raw units.** Reserves, shares and accounting never store a scaled figure. A multiplier only converts for pricing and display.

**The multiplier is read fresh in every quote.** It comes from the mint in that transaction, compared against the Clock sysvar. Token-2022 can hold a scheduled new multiplier with an effective time, so reading the current value alone is not enough.

**The price and the multiplier must agree.** Whether a report's price already reflects the multiplier has to be settled per asset during discovery, so the factor is applied exactly once.

**An unexplained change halts.** The intended rule is that a multiplier move with no matching verified event record sends the market to `HALTED` until it is reconciled.

## The event sequence

Every event that changes the meaning of a token follows the same four stages.

1. **Stop.** A pause by the issuer or the feed moves the market to [`HALTED`](session-regimes.md). A swap derives this in-line, and a keeper can fire it, so no human step is needed. Vault quotes end, and maker fills stop in that market.
2. **Verify.** Arcter compares the event against verified corporate action data. Chainlink's xStocks Data Streams product offers corporate action verification. Coverage on Solana and for Ondo assets needs confirming, and an asset without a dependable source needs a stricter fallback or does not list.
3. **Reconcile.** The new multiplier and the new reference are checked together. The raw vault balance needs no adjustment.
4. **Resume.** Quotes return from the adjusted price under the current regime.

During a halt, deposits and withdrawals continue where safe. An issuer pause can block the asset side of a withdrawal, while the USDC side stays available. See [Earning as an LP](earning-as-an-lp.md).

## By event type

| Event | While it runs | Afterward |
| --- | --- | --- |
| Cash dividend | Multiplier moves; market may halt around the change, depending on the asset | Quoting continues from the reconciled price |
| Forward or reverse split | `HALTED` while paused | Quoting resumes after reconciliation |
| Merger or acquisition | `HALTED` | Resumes, or the market retires if the token ends |
| Ticker change | `HALTED` | Resumes once the feed and mint records match |
| Delisting of the underlying | `HALTED` | The market retires |

Depositors should note the ex-dividend moment. The share price falls while the multiplier rises, and any gap between the two is a brief mispricing of about the dividend's size. Whether a dividend needs a full halt depends on how closely those updates arrive, which remains to be measured in test drills and on live mints.

## Retiring a market

A market moves to `RETIRED` through the Squads multisig and its time lock when the underlying ceases to exist or the issuer ends the token. It never quotes again. Depositors withdraw their pro rata share in kind and may then pursue the issuer's redemption route, where they qualify for it. Arcter does not hold anyone's position through a wind-down, and each retirement is published with its reasons in [Parameter governance](../accountability/parameter-governance.md).

## Risk that remains

This design leans on a pause or a data flag arriving first. If a feed goes silent, the age limit halts the market anyway. If an issuer changes a multiplier without warning, the planned reconciliation rule should halt it.

The hard case is a confident, wrong feed through an event. The [oracle band](quote-math.md) and the 25% jump guard limit a single fill, and the Pyth cross-check may catch the error, but none of these removes the risk. A move smaller than 25% can still pass the jump guard, so a dividend-sized error relies on the band and the cross-check alone. More detail is in [Price data defenses](../safeguards/price-data-defenses.md).

Holders also keep the tracker risk: these tokens carry price exposure and no shareholder rights, so shareholder votes do not reach them.
