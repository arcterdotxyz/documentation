---
description: How to check Arcter's figures against Solana itself, using the Ledger, Solscan and the Solana Explorer.
---

# Check the record yourself

{% hint style="info" %}
This walkthrough describes the intended flow for the planned release. Arcter's programs are not live and no public addresses exist yet, so there is nothing to look up today.
{% endhint %}

Arcter's claim to be transparent is only worth what you can verify. The planned **Ledger** is the readable summary, while Solscan and the Solana Explorer show the underlying record. No wallet or sign-in is needed for any of them. When the Ledger and the chain disagree, the chain wins, and the mismatch should be reported as a bug.

## Start with the Ledger

Choose **Ledger** in the left rail. Three panels organize the page:

* **Trade record** lists itemized fills, and you can filter it by side and by source, vault or maker.
* **Parameters in force** shows the settings active now.
* **Change history** logs governance actions.

Open a market for its full picture:

| View | Contents |
| --- | --- |
| Current quote | Bid, ask and mid for this moment, itemized with the router's own pricing |
| Regime | The present state, its duration, and the market status and feed time that produced it |
| Inventory | Mix against target and band, with skew direction and size |
| Parameters | Tier, spreads, clip, caps and band, read from `arcter-config` |
| History | Itemized fills, spread revenue by regime, and each halt with its reason |
| Listing record | The tier rationale, the mint's extensions as reviewed, issuer disclosure, and reserve proofs where a feed exists |

A wider view covers the venue as a whole:

* Volume, fill count and revenue, split by market, by source and by regime.
* Total vault inventory exposed to each issuer, described in [issuer dependence](../safeguards/issuer-dependence.md).
* The distance of every fill from the oracle mid, the dataset behind the [fill reports](../accountability/fill-reports.md), available in summary and as a download.
* Parameter proposals, pending and executed, each with its reason, waiting period and execution record.

Every figure links to the transaction or account it came from.

## Read one transaction

A receipt or Ledger row links to a transaction signature. Solscan and the Solana Explorer both show the same chain data with different layouts. On either one, work through these fields:

| Field | What to check |
| --- | --- |
| Result | Success or failure. A failed transaction still pays its fee. |
| Slot and block time | When it landed. Compare this with the report time on the receipt. |
| Signers | The wallet that signed. A swap should show one signature, unless a fee payer sponsored it. |
| Fee | The base 5,000 lamports per signature plus any priority fee, in SOL. |
| Compute units | How much of the budget the route used. |
| Instructions | The top-level call to `arcter-router`. A versioned transaction may load some accounts from an address lookup table. |
| Inner instructions | The cross-program calls beneath it, such as vault, oracle, token and hook programs. |
| Token balance changes | The amounts that moved, per account. These should match your receipt. |
| Program logs | The events Arcter emits, along with any error text. |

For an RFQ fill, look for an Ed25519 program instruction ahead of the settlement call. That instruction holds the maker's signed quote.

## Mind raw and scaled amounts

Token-2022 assets can make two honest numbers look like a contradiction. Under Scaled UI Amount, the raw balance stays fixed and a multiplier sets what people see. Arcter's math runs on raw units, while an explorer or wallet may display either form.

Before reporting a mismatch, confirm which unit each side is using and which multiplier applied at that time. A split or dividend can change the displayed figure with no token moving.

## Inspect a mint

Search a mint address on either explorer and find its token extensions. xStocks mints show metadata pointer, confidential transfer, Scaled UI Amount, default account state, transfer hook and permanent delegate. Some of these may be configured for future use, so presence alone does not prove an extension is active. Compare what you find with the listing record. Any change to a listed mint should pause that market until it has been reviewed.

## Inspect the accounts and programs

Market accounts and Attestation PDAs are public, so an explorer can show their data, though raw bytes need the program's published layout to decode. Attestations contain only role, jurisdiction class and expiry.

On a program's page, check two things:

* **Upgrade authority.** It should resolve to the Squads multisig with its time lock.
* **Verified build.** This lets you match deployed code to public source.

Program, Market and multisig addresses will be listed in [releases and addresses](../system/releases-and-addresses.md) once they exist. Pending parameter changes will be visible through that multisig as well as in **Change history**.

## Rebuild it from scratch

If you would rather not trust any one site, take the data directly:

* Query `/v1/fills`, `/v1/markets` or `/v1/execution-quality` on the API, described in [developer access](../system/developer-access.md). The API arrives in a later release than the Ledger.
* Download the CSV exports. Signed-in depositors can also export full position history.
* Run the indexer yourself once it is published as open source, which is the plan. Arcter's read model is fed by a Yellowstone gRPC stream, so you need a similar stream or an RPC provider that offers one.

Academic and independent study of this data is welcome. A market in real-world assets should have a history that outsiders can reconstruct.
