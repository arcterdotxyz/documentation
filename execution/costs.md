---
description: Every cost of using Arcter, from protocol fees and spreads to SOL network fees, priority fees and refundable token account rent.
---

# Costs

A trade on Arcter has two kinds of cost. Arcter's own charges pay the protocol and depositors. Solana's network costs pay for inclusion and account storage, and none of that reaches Arcter. The ticket is designed to show both before you sign. Order flow is not sold.

## Arcter charges

| Charge | Rate at launch | Who bears it | Applies to |
| --- | --- | --- | --- |
| Vault swap fee | 2 bps of notional, shown on its own line and included in the all-in price | Trader | Each vault fill |
| RFQ settlement fee | 2 bps of notional | Trader | Each maker fill |
| Protocol share of spread | 10%, or one tenth, of the vault's spread income | Depositors, out of spread | Each vault fill |
| Vault deposit or withdrawal | Free | | |

Each leg of a two-asset order is a fill. Selling TSLAx for NVDAx therefore pays the fee on both legs.

The spread itself is also a cost to the trader. It depends on the market's tier and on the regime: x1.0 in `OPEN`, x1.5 in `EXTENDED` and x3.0 in `CLOSED`. See [Quote math](quote-math.md) and [Admission criteria](../catalog/admission-criteria.md).

## Solana network costs

These are paid in SOL, so a wallet needs a small SOL balance unless a transaction is sponsored. Sponsored fees on a first trade are an option under review.

| Cost | Amount | Refundable | Notes |
| --- | --- | --- | --- |
| Base fee | 5,000 lamports per signature, or 0.000005 SOL | No | One signer pays one base fee; each further signature, including one checked by the Ed25519 program, adds another |
| Priority fee | Optional; a bid per compute unit, multiplied by the compute limit requested | No | Rises with demand for the accounts and the slot |
| Token account rent | A rent-exempt deposit, set by account size | Yes, on closing the account | Needed the first time a wallet holds a given token |

Some details are easy to miss:

- **A failed transaction still costs fees.** If it lands and then fails, the base and priority fees are spent. A transaction whose blockhash expires before inclusion costs nothing.
- **Hooks raise compute use.** A route with transfer hooks and oracle reports needs a larger compute limit, so its priority fee can exceed a simple transfer's at the same price per unit. Measured figures per route will come from testing.
- **Rent is a deposit.** A token account must exist before it can receive a token, and creating one locks SOL. Token-2022 accounts with extensions are larger and need more. Closing an empty account returns the deposit to the wallet you choose.
- **Inclusion tips are extra.** A Jito tip is a separate, voluntary payment. Whether Arcter uses that path is being evaluated.

Vault shares are a token too, so a first deposit may also need a token account for the share mint.

## The ticket and the receipt

Before you sign, the ticket lists mid, spread and fee separately, in bps and in USDC. The plan is to add the estimated SOL cost and any new token accounts beside them.

After settlement, the same price breakdown is recorded with the fill, and the transaction on Solscan shows the fee actually paid. The quote and the record can be compared line by line.

## Rate changes

Every rate is a setting in `arcter-config`. A change must pass through the Squads multisig and its time lock with a published reason, and it only affects later fills. A fill always pays the rate in force when it executed. The change appears in the [governance log](../accountability/parameter-governance.md) before it can reach a ticket.

## Where the money goes

Arcter's fees collect in the treasury PDA held by `arcter-config`. The intended uses are security work such as audits and a bug bounty, oracle and infrastructure costs, and running the venue. Spending is subject to the same multisig and time lock as settings.

## A revenue illustration

These figures are hypothetical. They are not a forecast, and no market is live. The example assumes 10M USDC of daily volume, and 12 bps as the mean all-in half-spread.

| Source | Calculation | Per year |
| --- | --- | --- |
| Trading fees, vault and RFQ | 10M USDC × 2 bps × 365 days | About 730K USDC |
| Protocol share of spread | 10M USDC × 12 bps × 10% × 365 days | About 438K USDC |
| Total | | About 1.2M USDC |

The same arithmetic at 50M USDC a day gives about 5.8M USDC a year. [Revenue](../resources/revenue.md) goes further into the model.
