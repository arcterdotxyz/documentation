---
description: How arcter-router settles each order against the better of a vault or a maker, and what it enforces before funds move.
---

# Order routing

{% hint style="info" %}
This page describes the planned design. Arcter is pre-launch and no program is deployed.
{% endhint %}

An order on Arcter can fill in two places: a market's vault, or a signed quote from a maker. The `arcter-router` program decides between them inside the transaction, so the result depends on state the program verifies itself and not on what an app claims.

## Picking the better fill

The router works through three steps for each market an order touches.

1. **Price the vault.** `arcter-router` calls into `arcter-vault`, which derives the quote from the verified report and the Market account in that same transaction. A vault price supplied by a client is ignored. The vault only competes if the regime allows quoting and the size fits the current clip.
2. **Check any maker quote.** A quote competes only if it was included in the transaction, the native Ed25519 program verified its signature, and `arcter-rfq` accepts its expiry and quote id. It must also sit inside the oracle band.
3. **Settle on the higher output.** The trader receives whichever source pays more. A transaction that names a maker quote worse than the vault fails, so an app cannot steer an order to a worse price.

Details of each lane are in [Vault liquidity](vault-liquidity.md) and [RFQ desk](rfq-desk.md).

## Orders between two assets

Vaults hold one asset against USDC, so selling TSLAx for NVDAx is two trades joined together:

| Step | Sold | Received | Filled by |
| --- | --- | --- | --- |
| 1 | The input asset | USDC | Its vault, or a maker |
| 2 | All USDC from step 1 | The output asset | Its vault, or a maker |

The lanes are chosen separately, which means a maker may fill one step while a vault fills the other. Each step answers to its own market's band and regime. The trader signs one transaction with one minimum output, measured on the full route. Solana transactions are atomic, so a failure at step 2 also undoes step 1. The fill record lists both steps with an itemized price.

## Fitting a route into one transaction

A two-step swap can be heavy. It may carry oracle reports, signature checks and the extra accounts that issuer transfer hooks demand, all inside a limit of 1,232 bytes and a compute budget.

The planned approach:

- Versioned transactions and address lookup tables shorten the account list.
- Hook accounts are resolved ahead of time, because one missing account fails the transaction.
- The quote service simulates the route first, so an ineligible wallet or a rejected hook produces a clear reason on the ticket.
- Compute budgets are measured per route during testing.

Whether the heaviest routes fit with every hook account included is an open discovery question. A route that does not fit will not be offered.

## Open to any caller

Nothing about `arcter-router` is reserved for Arcter's own app. Access depends on the wallet signing the transaction, so the front end submitting it makes no difference. A wallet, an aggregator or an agent can price an order through the planned [quote API and SDK](../system/developer-access.md) and build the instructions directly. The app at [arcter.xyz/platform](https://arcter.xyz/platform) receives no preferred treatment.

## Enforced on every fill

| Guard | Failure it stops |
| --- | --- |
| Attestation PDA for the signing wallet, plus any issuer hook | A wallet trading a market it may not access |
| Oracle band, per step | A fill far from the verified reference |
| Regime check, derived in the swap | Quoting in `HALTED`, or relying on stale regime state |
| Clip and daily volume cap | Size beyond what the market allows |
| Minimum output and expiry | Execution after conditions have moved |
| Exact transfer amounts under the owner's signature | Any movement of funds beyond this trade |

Arcter uses no standing approvals, so no allowance is left behind for a later attacker to drain. Each transaction moves funds with the user's signature and then ends. A blockhash also lapses after about 150 slots, which gives a ticket an outer deadline in addition to its own expiry, checked against the Clock sysvar.

## Ordering and MEV

No central operator orders Solana transactions. Leaders rotate on a published schedule, and the leader for a slot sees what it receives. Sandwiching exists on the network, and Jito's block engine is widely used, so Arcter treats ordering as an adversarial input.

Two limits contain the damage. The signed minimum output caps what anyone can extract at the tolerance the trader accepted, and the oracle band bounds each step regardless of position in a block. The app therefore defaults to a tight minimum output and a short quote window.

Inclusion is a separate concern. Dynamic priority fees are planned, while Jito and stake-weighted paths will be evaluated during testing. Any move to a private path would be reviewed for its effect on fills. Network failure modes are covered in [Network disruptions](../safeguards/network-disruptions.md).
