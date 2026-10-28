---
description: Why tokenized stocks need a venue of their own on Solana, what the 2026 market shows, and what the network demands in return.
---

# The case for Solana

Getting real-world assets onto a blockchain is largely done. Pricing them fairly is the open problem. Arcter exists because equity tokens now trade on Solana in serious size, while most of that flow still meets designs built for crypto pairs.

The argument rests on three questions. Is the flow real? Is the current execution actually weak? Does Solana suit a better design? The answers below are dated, and each one can change.

> Figures on this page are snapshots taken from public sources, and these markets move quickly. Data providers also define categories differently, so read each number as an approximate level. The [bibliography](../resources/bibliography.md) lists the sources.

## The flow is on Solana

| Measure | Reading | Date and source |
| --- | --- | --- |
| Distributed RWA value on Solana | About $5.19B, a record | 29 Oct 2026, RWA.xyz |
| Tokenized stock volume on Solana | About $4.4B in the month | Sep 2026, Solana roundup |
| Highest daily tokenized stock volume | $683M | Jun 2026, Solana roundup |
| Solana's share of 2026 tokenized equity volume | More than half | Sep 2026, Solana roundup |
| xStocks value on Solana | About $430M | Early Nov 2026 reports |
| Ondo Global Markets value on Solana | About $300M | Nov 2026, RWA.xyz snapshot |
| Stablecoins on Solana | Roughly $14-17B | Nov 2026 |

For wider context, tokenized equities across all networks came to about $2.5B in mid August 2026. All non-stablecoin RWAs came to about $38.7B in late August 2026. Tokenized treasuries had passed $10B in February 2026.

Supply has followed the same path. xStocks, from Backed Assets (JE) Limited of Jersey, covers more than 100 US-listed stocks and ETFs. Ondo Global Markets arrived on 21 January 2026 with over 200 instruments and has since passed 250 on Solana. PAXG launched in June 2026, and XAUt0 is present as well.

These tokens already do useful work for holders. Kamino held about $20.7M of xStocks collateral on 13 September 2026, and it opened a PAXG market on 27 July 2026. One report put Raydium's xStocks volume at $1.63B for the second quarter of 2026. Jupiter aggregates across these venues.

## Volume is not good execution

Equity tokens carry three facts that crypto market designs ignore. An outside market sets their price. That market closes. Their issuer can change the token.

| Weakness | What goes wrong |
| --- | --- |
| The curve has no reference | A constant-product pool moves only when someone trades against it. When a stock jumps on its home exchange, arbitrageurs collect the difference from depositors. |
| The calendar is ignored | US regular trading lasts 6.5 hours on five weekdays, about 32.5 of the 168 hours in a week. Roughly 80% of the time, pools carry gap risk at no explicit charge. |
| Issuer mechanics leak into balances | Scaled UI Amount changes the balance a holder sees while the raw amount stays put. A venue that confuses the two units can misprice a fill, and at least one wallet bug report describes an xStock shown at the wrong scale. |
| Costs stay out of sight | An aggregated route shows an output figure. It rarely shows the distance from the live reference price, or the reason for that distance. |

The [precedents](../safeguards/precedents.md) page collects earlier dislocations on tokenized venues.

## The alternatives, and what each one leaves open

* **Pools on Raydium and similar venues** offer open access and composability. Their curves do not read the reference price or the session.
* **Proprietary AMMs** are the most capable competition here, and they form a large part of Solana spot flow. Their pricing is private logic, so a holder cannot recompute a fill.
* **Jupiter** finds the best available route. It is an aggregator, so it inherits the quality of whatever sits beneath it. Arcter plans to register with it and to be one of those routes.
* **Centralized platforms** can show deep books, but trading there is custodial and off-chain.
* **Issuer channels** provide a way in and a way out. They are not a continuous market at every hour.

None of these has to fail for Arcter to work. The opening is narrower: a self-custodial, oracle-centered venue that prices the session openly and publishes how each fill compared with the reference. [Principles](principles.md) sets out the rules behind it.

## What Solana gives this design

| Property | Why it matters to Arcter |
| --- | --- |
| Slots of about 400 ms | With a signed report carried inside the transaction, little time separates a quote from its fill. |
| A base fee of 5,000 lamports per signature | Small tickets stay worth settling on-chain, although priority fees can add to this when blocks are busy. |
| Votor consensus, live on Solana since 21 Oct 2026 (epoch 1064) | It replaced TowerBFT and targets finality of about 150 ms. Slot length has not changed, and Rotor has no schedule yet, so Turbine still carries blocks. |
| Token-2022 extensions | Hooks, delegates and multipliers sit at the mint, so a venue can read issuer rules directly. |
| Open routing | Wallets and aggregators can reach any program with a good quote, so a better price can earn flow without a listing deal. |
| Local dollars | USDC is native to Solana and the network's largest stablecoin by the figures we have, so most users already hold the settlement asset. |

## What Solana demands in return

The same network imposes constraints, and they shape the design as much as the benefits do.

| Constraint | Effect on Arcter |
| --- | --- |
| Transactions are capped at 1,232 bytes | A two-leg swap with hooks, oracle reports and signature checks is heavy. The router relies on versioned transactions and address lookup tables, and whether the heaviest routes fit is still to be confirmed. |
| Each transaction has a compute budget | Compute use has to be measured route by route. |
| Blockhashes expire after about 150 slots | Each ticket has a natural outer deadline, so a stuck order lapses and does not linger. |
| Accounts must exist before use | A first-time holder needs an associated token account, which costs a small refundable deposit. |
| Block producers see transactions | Sandwiching exists. Tickets need a tight minimum output and a short quote window. |
| Issuers hold real powers | Transfer hooks, default account state and permanent delegates all reach into vault accounts. Whether each hook accepts program-owned vaults is an open question. |
| Inclusion is not guaranteed | Congestion and RPC failures are the common daily problems, so priority fees and stake-weighted or Jito paths are being evaluated. |
| Assets may not list | Oracle coverage on Solana must be confirmed per asset, and Solana gold pools are currently thin. |

The cluster itself also matters. Solana reported 1,000 days without a network-wide outage around 2 November 2026. The last cluster-wide halt came on 6 February 2024, after earlier halts between 2021 and 2023. A long record does not rule out another restart, so [network disruptions](../safeguards/network-disruptions.md) treats one as a planned-for event.

## What would prove the thesis wrong

Arcter's case weakens if issuers refuse to clear vault accounts, if oracle coverage proves thin, if heavy routes do not fit in a transaction, or if makers cannot reach primary mint and redeem. Each of these is a discovery question to settle before launch. The [delivery phases](../plan/delivery-phases.md) page shows when.

Nothing here is investment advice.
