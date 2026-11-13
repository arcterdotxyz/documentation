---
description: How Arcter plans to earn money, what that income must pay for, and how much volume the model needs.
---

# Revenue

Arcter is pre-launch and has no revenue. This page sets out the planned model, using launch parameters. Nothing here is a forecast.

## Income lines

| Line | Rate at launch | Who bears it | Grows with |
| --- | --- | --- | --- |
| Vault swap fee | 2 bps on notional | Trader | Volume |
| Maker lane settlement fee | 2 bps on notional | Trader | Larger order volume |
| Share of vault spread | 10% of realised spread | Depositors, taken from spread earnings | Volume and spread together |

Depositors keep the other 90% of spread, so the third line pays most when vaults earn most. Fee settings live in `arcter-config` and change only through the [7 day time lock](../accountability/parameter-governance.md). Tickets show the fee, and each fill event records it. The full cost breakdown is in [Costs](../execution/costs.md).

## Ways Arcter will not earn

* Payment for order flow
* Trading against its own users
* Selling user data

Token emissions are not planned either. Depositors are paid from fee share, so a vault's income is real trading income.

## Worked examples

These are illustrations of the arithmetic, not projections. Both assume a mean all-in half-spread of 12 bps, and 365 trading days because markets run through weekends.

| Daily volume | Annual fees (2 bps) | Annual spread share (10% of 12 bps) | Annual total |
| --- | --- | --- | --- |
| 10M USDC | About 730K USDC | About 438K USDC | About 1.2M USDC |
| 50M USDC | About 3.65M USDC | About 2.19M USDC | About 5.8M USDC |

Set those volumes against the market. Solana carried about $4.4B of tokenized stock volume in September 2026. Thirty days at 10M USDC a day is 300M USDC, or roughly 7% of that month. At 50M USDC a day the figure is roughly 34%. Launch caps sit below both cases, and the first case already implies a meaningful share of Solana's flow. Neither should be read as an expectation for the early phases.

## Where flow should come from

Arcter needs traders and integrators to choose it for practical reasons:

* **A tighter quote.** Oracle-centered vaults do not leak value to arbitrage the way curves do, so they can quote tightly and still pay depositors. Aggregators such as Jupiter route on price, so a good quote can win flow without a listing deal. Proprietary AMMs are strong competition here, and this advantage is unproven.
* **Priced closed sessions.** These tokens trade at night and at weekends. Arcter charges an explicit session spread for that risk and shows it on the ticket.
* **Existing holders.** Self-custodial holders of xStocks and Ondo tokens already sit in Solana wallets. Reaching them through wallets and aggregators avoids a separate onboarding push. See [Partners and integrations](../solana/partners-and-integrations.md).
* **Integrations that persist.** A wallet or app built on the [developer tools](../system/developer-access.md) keeps sending flow without further acquisition spend. Those tools arrive in the Platform phase.

## What the income pays for

Fees collect in the treasury PDA held by `arcter-config`. Planned spending, in rough order of priority:

* audits and the bug bounty
* oracle and market data access, whose price is still to be confirmed
* RPC, the data stream behind the read model, and SOL for keeper transactions and account rent
* any provider costs tied to attestations
* the team

Until volume covers these, the venue depends on outside funding. Sources and amounts are not established and are not stated here.
