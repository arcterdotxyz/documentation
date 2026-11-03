---
description: The tokenized equity products Arcter plans to list on Solana, how they are structured, and how their mint settings shape a market.
---

# Tokenized equities

Two issuers anchor Arcter's planned equity catalog: xStocks and Ondo Global Markets. Both offer US-listed stocks and ETFs as tokens on Solana. Neither gives you a share. Arcter does not issue tokens, and no market is live. Early candidates are the names easiest to hedge, such as SPY, QQQ, NVDA and TSLA, and [Admission criteria](admission-criteria.md) sets the conditions for each.

## The two products

| | xStocks | Ondo Global Markets |
| --- | --- | --- |
| Issuer | Backed Assets (JE) Limited, Jersey | Ondo Global Markets (BVI) Limited |
| Form | Tracker certificates under a base prospectus | Total return trackers |
| Backing | 1:1 by the underlying | Underlying assets plus any cash in transit, held through a bankruptcy-remote special purpose vehicle with a third-party security agent |
| Holder rights | No voting rights and no legal claim on the underlying shares | No voting, statutory information or other shareholder rights |
| Coverage on Solana | 100+ stocks and ETFs | 250+ stocks and ETFs; launched on 21 January 2026 |
| Value on Solana | About $430M, early November 2026 reports | About $300M, RWA.xyz snapshot, November 2026 |
| Distribution | Kraken | To confirm per channel |

Figures are dated snapshots and move quickly.

Kraken's guide describes xStocks as offering synthetic exposure to a stock's value, while also stating 1:1 backing. Readers should take both statements seriously: the token tracks a price, and your claim runs against the issuer's structure, never the company behind the stock.

## What the mints contain

Inspection of xStocks mints on an explorer shows Token-2022 with these extensions configured. Some may be set for future use, so being present does not prove one is active.

| Extension | What Arcter must do |
| --- | --- |
| Scaled UI Amount | Run vault math in raw units and read the multiplier fresh for each quote |
| Transfer hook | Carry the hook's extra accounts in every swap; confirm it accepts program-owned vault accounts |
| Default account state | Expect new token accounts to start frozen; seek issuer clearance for vault accounts |
| Permanent delegate | Disclose that the issuer can move tokens out of any account, including a vault |
| Confidential transfer | Check how it interacts with vault accounts before listing |
| Metadata pointer | Read name and symbol from the mint's declared metadata location |

Ondo's position on Solana is less settled. Its documents describe a display multiplier for supported wallets, and reports point to a plan for transfer hooks on eligibility. Arcter has not verified the extensions on each Ondo mint, so each must be checked directly before listing.

Each listing records the mint's configuration. A keeper watches for changes, and any change pauses the market until reviewed. Tokens may sit under either token program, so Arcter's code handles both.

## Who may hold them

Eligibility comes from the issuer, and each list varies by product and channel.

- **xStocks:** Kraken's availability pages exclude US persons, Canada, the UK, Australia and prohibited regions. EEA clients answer a questionnaire. Other channels may differ.
- **Ondo Global Markets:** US persons are barred, along with Canada and other listed jurisdictions. The UK, the EEA, Switzerland and several other places are open only to qualified or professional clients, under local tests.

Arcter applies the strictest applicable reading for each market and does not serve US persons. Where a hook decides, simulation shows the reason on the ticket before you sign. The mechanics are in [Access controls](../system/access-controls.md). Nothing here is legal advice, and counsel review comes before launch.

## Pricing

Each listing needs a verified reference. Chainlink Data Streams is the intended primary source, and a public xStocks Data Streams product covers price and corporate action verification. Pyth pull updates act as the cross-check. Coverage for each asset on Solana still needs confirming, so neither source should be assumed for a given token.

Session state then determines the [regime](../execution/session-regimes.md), and corporate actions follow [Splits and dividends](../execution/splits-and-dividends.md).

## Risks in every holding

Arcter discloses each of these and prices what it can.

**Issuer credit.** A holder depends on the issuer and its custody arrangements. If an issuer fails, recovery depends on that structure, and the stock's market price offers no guarantee of it. Vault depositors carry the same exposure on the asset side.

**Issuer control.** Issuers can pause tokens, freeze accounts and, through a permanent delegate, move tokens from any account. Terms may also change.

**Tracking.** A token can trade away from fair value. Oracle anchoring limits that for vault fills without removing it. An Ondo token's price can also drift from the stock price as dividends accumulate, because one token may come to represent more than one share.

**Thin primary access.** Whether makers can reach mint and redemption decides how tightly arbitrage holds prices. That is an open question for discovery.

[Issuer dependence](../safeguards/issuer-dependence.md) has the full treatment.
