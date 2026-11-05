---
description: Past failures in tokenized markets and on Solana itself, and the design requirement Arcter takes from each.
---

# Precedents

A blockchain changes how a trade settles. The structure of the market stays the same: a stock still has a home exchange, an issuer and a reference price. Tokenized venues have repeatedly run into trouble by forgetting that. Arcter reads these episodes as engineering requirements.

Arcter itself has no operating record, since its programs are not deployed. These are lessons from other systems. They do not prove the design works.

## What the record asks for

| Recurring failure | Requirement for Arcter |
| --- | --- |
| A company held the assets | Self-custody: fills are atomic, and vault exits pay in kind |
| A price had no outside reference | Every fill is bounded by a verified oracle price |
| A number was calculated where it should have been read | Each Market prices only its own mint, from its own feed |
| The network was assumed to be always on | Halts and restarts are treated as expected events |

These requirements leave issuer, gap, network and program risk in place. [How Arcter models risk](risk-model.md) covers what remains.

## Custody: FTX and CM-Equity (2020 to 2022)

FTX offered tokenized stocks in a custodial structure involving CM-Equity, a German firm. FTX filed for bankruptcy in November 2022. Afterwards, people asked who had really held the underlying shares, and what token holders were entitled to. Reports raised doubts about the backing, and a clear answer was not available to holders when they needed one.

The requirement: a tokenized share should not inherit an exchange's solvency.

Arcter holds no user funds in a company account. Assets stay in a user's wallet, or in vault accounts controlled by program-derived addresses, and depositors exit by redeeming shares. Two risks remain. Program upgrades sit behind a Squads multisig and a time lock, so depositors depend on those signers for as long as upgrades remain possible. An in-kind exit also depends on the issuer allowing the transfer, as [Issuer dependence](issuer-dependence.md) explains.

## Thin pools: early tokenized stocks on Solana (mid 2025)

When tokenized stocks reached Solana, the first trading happened in small DEX pools. Market commentary in July 2025 described liquidity as extremely thin. A pool price comes from its reserves and refers to nothing outside the pool, so even modest orders could move a token away from its stock.

The requirement: a reserve ratio has no concept of fair value, so reserves must not set the price.

Every Arcter quote begins at the verified mid. The oracle band is a program rule, so thin liquidity narrows the available size and leaves the price anchored.

## A slow leak: passive curves on repriced assets (continuing)

No single date applies here. When a reference price moves elsewhere, an arbitrageur trades the pool to the new level and keeps the difference. Researchers call this cost loss-versus-rebalancing. A curve holding an asset repriced all day by another exchange pays it continuously, and depositors fund it.

The requirement: where price discovery happens elsewhere, read the result.

Arcter's vaults quote around the oracle, so depositors earn spread on actual flow. Rebalancing still costs something, and that cost is visible as skew on the ticket.

## Derived values: a wrapper rate at Edel Finance, July 2026

On 1 July 2026 a lending protocol, Edel Finance, took about $403K of bad debt. Reports say the oracle for tokenized Google stock stayed correct. The attacker instead pushed the exchange rate of a wrapper token, wGOOGLx, to about 78 times its proper value, then borrowed real assets against the inflated collateral.

The requirement: any value computed from the oracle price is a separate attack surface, so a venue should keep as few of them as possible.

Each Market stores the feed identifier for its own mint, and `arcter-oracle` rejects any other. Arcter does not open markets on wrappers, on its own vault shares or on derived rates.

One conversion cannot be avoided on Solana. Scaled UI Amount sits between raw units and displayed units. The issuer sets that multiplier at the mint, so a trader cannot push it the way a wrapper rate can be pushed. It is still a derived step, which is why it is read fresh in each quote and why a multiplier change during a session is one of the planned test drills.

## Time: weekend gaps on round-the-clock tokens

US regular sessions cover about 32.5 hours a week, while a token trades at any time. It drifts while the home market is shut, then reprices sharply at the open. Two simple answers have fallen short. Weekday spreads through the weekend leave depositors absorbing Monday's gap. Shutting the venue sends weekend trades elsewhere, often to custodial books.

The requirement: the asset sets the hours, and the venue prices around them.

In `CLOSED`, Arcter applies a spread of x3.0 and a clip of x0.5, shows both on the ticket, and reads session state from oracle data where the schema allows it. See [Session regimes](../execution/session-regimes.md).

## Network: Solana's halts (2021 to February 2024)

Solana halted several times between 2021 and 2023, and again on 6 February 2024 after a validator bug. Each recovery took a coordinated restart among validators. Solana reported 1,000 days without a network-wide outage on about 2 November 2026.

The requirement: treat a clean record as evidence, not as a guarantee. Stocks keep trading through a halt, so a vault can come back holding a price that no longer exists.

Arcter records a heartbeat, detects a gap against the Clock sysvar, and holds all markets at `HALTED` for one hour after a restart. No program can offer an exit during a halt. Depositors should size positions with that in mind. See [Network disruptions](network-disruptions.md).
