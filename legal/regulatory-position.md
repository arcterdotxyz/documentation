---
description: Who Arcter intends to serve, how access rules are planned to work on Solana, and which legal questions remain open.
---

# Regulatory position

{% hint style="warning" %}
**Not legal advice, and not yet reviewed.** This page describes Arcter's intended approach before launch. Counsel has not signed off on it. No statement here is a legal conclusion, and any part may change after review.
{% endhint %}

## Summary

Arcter is pre-launch and no programs are deployed. The plan is a self-custodial venue that does not serve US persons, follows the strictest applicable issuer rule for each market, and holds no personal data on-chain. Whether that design achieves the regulatory outcome Arcter intends is a question for counsel in each relevant jurisdiction.

## Who can reach each market

Issuers decide who may hold their products, and their lists differ by product and by distribution channel. Arcter's plan is to apply the strictest applicable reading per market. It does not write a looser list of its own.

| Issuer product | Published or reported limits | Caveat |
| --- | --- | --- |
| xStocks, issued by Backed Assets (JE) Limited of Jersey | Kraken's pages exclude US persons, Canada, the UK, Australia and prohibited regions. EEA clients answer a questionnaire. | The list varies by channel, and this one covers Kraken's channel only. |
| Ondo Global Markets | US persons are barred. Access in the UK, Switzerland and some of the EEA is limited to qualified or professional clients. | Check Ondo's current eligibility page before relying on this. |

Both rows reflect sources as of early November 2026, and each must be confirmed again before launch. Neither issuer's lists were written for Arcter, and how they apply to a venue like this one is among the questions for counsel.

## What these tokens are

xStocks are 1:1 backed tracker certificates issued under a base prospectus. A tracker certificate does not grant shareholder rights. Holders of equity tokens on Arcter should expect price exposure and issuer risk, not ownership of a share.

Ondo reaches Solana with its own terms, and its FAQ describes 30% US withholding on dividends. Read each issuer's documents directly. See [tokenized equities](../catalog/tokenized-equities.md).

## Two layers of control

Rules on Solana can live in the token itself, so Arcter plans for two layers.

**The mint layer belongs to the issuer.** Token-2022 extensions can enforce rules at transfer. Inspection of xStocks mints shows transfer hook, default account state and permanent delegate, and some of these may be configured for future use. Ondo has reportedly planned to use transfer hooks for eligibility, so each mint must be inspected and not assumed. Issuer registries remain the source of truth. Arcter cannot override anything at this layer.

**The venue layer belongs to Arcter.** `arcter-access` keeps an Attestation PDA per wallet, and the app adds its own screening and geo-fencing. Before quoting, the router simulates the trade, hook included, so that a blocked user sees a reason and not a failed transaction.

## Planned requirements

| Activity | Requirement |
| --- | --- |
| Using the app | Geo-fencing turns away visitors in restricted jurisdictions |
| Swapping | A `TRADER` attestation, covering completed KYC, a permitted location and eligibility under the asset's issuer rules |
| Depositing | An `LP` attestation, for professional clients only at launch |
| Quoting | A `MAKER` attestation, for professional trading firms |
| Receiving transferred vault shares | No program check, because a plain SPL mint has no hook. Counsel is to confirm whether gating entry at deposit is enough. |
| Redeeming vault shares | No Arcter permission check |

The last row is deliberate. A depositor should be able to leave using only their shares, because a venue-level power to block exits would look much like custody. It does not remove issuer powers: a frozen or paused token can still hold back the token leg of a redemption.

## How the controls work

| Control | Plan |
| --- | --- |
| Attestation | Once a wallet clears identity, sanctions and residency screening, a KYC provider's result is recorded in its PDA. The record has three fields: role, jurisdiction class and expiry. |
| Expiry | Every attestation lapses and must be renewed. A revoked one fails at the next attempted action. |
| Screening | The provider screens at issue and at each renewal. Sanctions screening also runs at the app layer. |
| Adjustable policy | The accepted attestation sources and policy rules are intended to be replaceable through `arcter-config`, so that tighter requirements can be adopted quickly. Changes pass through the Squads multisig and its time lock. |
| Open reads | Solana account data is public, so aggregators and explorers can read market state without permission. |

The KYC provider has not been named here, and neither the provider nor the policy design should be treated as final.

## SEC innovation exemption

The SEC issued its innovation exemption on 17 September 2026. Law firm and press summaries report it as a temporary five-year order for qualifying Tokenized Securities Venues that trade tokenized NMS stocks through permissioned AMMs. They report that it requires the same rights as the underlying stock and that it excludes synthetics. This page relies on those summaries, not on a reading of the order, and counsel should work from the primary text.

Arcter does not rely on this exemption and does not serve US persons. Tracker certificates are not shareholder rights, and how that sits against the reported same-rights condition is for counsel to confirm. US-eligible markets would need a different legal and product structure and are held for later.

## Europe and other jurisdictions

MiCA and MiFID review is required before launch, and counsel must sign off. Arcter takes no position here on how either applies. The classification of each token, of the venue and of Arcter's operating company in the EEA, the UK, Switzerland and elsewhere all remain open.

## What the design leaves out

The following limits are intended to keep the regulatory footprint narrow. Whether they do so is for counsel to assess.

* **No custody by Arcter.** Users sign every transaction, and no standing approvals exist. Vault reserves sit in program-controlled accounts on behalf of depositors, and how a regulator would view that is an open question.
* **No fiat ramp and no margin.** Arcter also issues no tokens.
* **No discretion over a fill.** Quotes come from public state and verified reports, and settlement is checked by program logic. Staff should have no means of improving, worsening or singling out a particular trade.

Two powers qualify those statements, and they are stated here on purpose. A guardian can pause a market, which stops trading for everyone in it but cannot move funds. Program upgrade authority sits behind a Squads multisig and a time lock, so program behaviour can change through that process. Upgrades and parameter changes are public and subject to delay, as described in [parameter governance](../accountability/parameter-governance.md).

The app is where users meet the operating company, so where authorization is needed, that company is the one expected to be assessed. See [entities and control](entities.md).

## Personal data

Nothing personal goes on-chain, and those details remain with the KYC provider. Arcter does not collect identity documents itself. An optional zero-knowledge credential route, letting a wallet prove eligibility without exposing its provider or any attribute, is an idea carried forward and is yet to be confirmed for Solana.

## When issuers change their rules

Access follows the issuer. A new restriction on an issuer's list should be matched at the attestation layer. Changes at the mint are watched directly: a keeper monitors configuration, and any change pauses the market until it has been reviewed. If a hook or default account state requires it, vault accounts will need issuer clearance, and whether issuers will grant that is unresolved. See [issuer dependence](../safeguards/issuer-dependence.md) and [expansion](../catalog/expansion.md).

## Open items for counsel

1. How each issuer's eligibility terms apply to a venue and to its vault depositors.
2. Whether the design supports the self-custodial position it intends.
3. MiCA and MiFID treatment of the venue and the operating company.
4. Whether any licence is needed, where, and by which entity.
5. How the rule on transferred vault shares should work.

## Sources

* [Kraken: xStocks availability](https://support.kraken.com/articles/xstocks-availability)
* [Ondo: Global Markets eligibility](https://docs.ondo.finance/ondo-global-markets/eligibility)
* [Cooley: summary of the exemption](https://www.cooley.com/news/insight/2026/2026-09-22-permission-to-innovate-sec-carves-out-path-for-on-chain-stock-trading)
* [The Block: report on the exemption](https://theblock.co/news/regulation/2026-09-17-sec-releases-innovation-exemption-415324)
