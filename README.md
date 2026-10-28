---
description: Arcter is a self-custodial exchange on Solana for tokenized stocks, gold and yield-bearing dollars, with oracle-centered prices that anyone can recompute.
---

# What Arcter is

Arcter is a self-custodial exchange for real-world assets on Solana. You connect a wallet, choose two assets, read a price split into its parts, and sign once. Tokenized stocks come first, gold sits beside them, and yield-bearing dollars follow after review.

## At a glance

| Item | Detail |
| --- | --- |
| Network | Solana |
| Settlement asset | USDC, issued by Circle |
| First markets | Six to twelve planned, using xStocks and Ondo Global Markets tokens on names such as SPY, QQQ, NVDA and TSLA, plus PAXG |
| Later markets | A wider equity catalog by liquidity tier, then yield-bearing dollars once reviewed. See [Expansion](catalog/expansion.md). |
| Execution lanes | Oracle-centered vaults for everyday size and signed maker quotes for larger orders, with `arcter-router` in front of both |
| Price data | Chainlink Data Streams as the intended primary source, cross-checked against Pyth pull updates. Coverage per asset is still to be confirmed. |
| Custody | Arcter holds nothing for you. Your wallet signs each transaction, and there are no standing token approvals. |
| Charges | One itemized protocol fee, plus the SOL network fee for the transaction |
| Access | Not offered to US persons. Each market follows the strictest applicable issuer rules. |

## From ticket to fill

Reserve ratios decide the price in most automated pools. That suits a token whose price forms on-chain, and it suits a listed share poorly, because the share's real price forms on an exchange somewhere else. Arcter inverts the arrangement.

Each listing has one [vault](execution/vault-liquidity.md), a Market account holding the asset and USDC. Its quote starts from a verified reference price. A base spread set by the asset's tier is then multiplied by the current [session regime](execution/session-regimes.md), and an inventory skew makes the crowded direction dearer. When the home market shuts, the multiplier rises and the largest permitted swap, called the clip, gets smaller. The ticket states both changes ahead of your signature.

Orders too large for a vault go to the [RFQ desk](execution/rfq-desk.md). Approved makers sign short-lived ed25519 quotes off-chain, and the taker's transaction settles both legs atomically. The [router](execution/order-routing.md) compares the two lanes and shows a single price, itemized into mid, spread, skew and fee.

## Where Arcter departs from a pool

| Question | Constant-product pool | Arcter |
| --- | --- | --- |
| What sets the midpoint? | The ratio of the reserves | A verified oracle price, limited by a band around it |
| What changes on Sunday night? | Usually nothing visible | Spread x3.0 and clip x0.5, both stated on the ticket |
| How is a stock split handled? | The pool has no view of it | Math runs on raw units, the multiplier is read fresh for each quote, and the market halts during an issuer or feed pause |
| How does a depositor earn? | Fees, minus losses to arbitrage on each outside move | Quotes around the oracle mid, with skew paying arbitrage to rebalance the vault |
| What happens with a large order? | It walks down the curve | Makers compete on a signed quote inside the oracle band |
| Who may trade? | Whatever the mint itself enforces | Issuer rules, plus an attestation where a market requires one, checked by simulation before you sign |
| Where are the costs? | Inside the output figure | On separate lines, and in the on-chain record of the fill |

Pools remain a good fit for assets with no outside price. Proprietary AMMs already price crypto majors well, but their quoting logic is private. Arcter's position is that an asset with an outside reference deserves a public formula that any holder can recompute.

## Risks to know first

Arcter being self-custodial does not make these assets free of risk.

* Equity tokens are tracker certificates. They give price exposure, not shareholder rights.
* Issuers can pause tokens, and a permanent delegate can move tokens out of any account of that mint, a vault included. Vault depositors carry that risk.
* Programs can be upgraded through a Squads multisig and time lock. Verified builds are planned so that anyone can match deployed code to its source.
* A Solana cluster restart stops every market on the network until it recovers, and Arcter's markets then wait out a one-hour grace window.

The [risk model](safeguards/risk-model.md) covers each of these in more depth.

## What Arcter leaves to others

* It does not hold user funds, act on anyone's behalf, or run a fiat ramp in either direction.
* It does not issue assets or operate their mint and redemption. Regulated issuers do that work, and Arcter runs a secondary market above it.
* It offers no lending, margin or perpetual futures.
* It lists no memecoins or volatile pairs, since a market with no outside reference gives this design nothing to anchor to.

## Where to read next

| If you want to | Start with |
| --- | --- |
| Understand the reasoning | [The case for Solana](start/solana-thesis.md) and [Principles](start/principles.md) |
| Look up a word | [Terminology](start/terminology.md) |
| Follow a trade through the system | [Lifecycle of a swap](execution/lifecycle-of-a-swap.md), [Quote math](execution/quote-math.md) and [Session regimes](execution/session-regimes.md) |
| Build on the venue | [Blueprint](system/blueprint.md), [Programs](system/programs.md) and [Developer access](system/developer-access.md) |
| Judge the risks | [Risk model](safeguards/risk-model.md) and [Precedents](safeguards/precedents.md) |
| Try it after launch | [Your first swap](guides/first-swap.md) or [Your first deposit](guides/first-deposit.md) |
