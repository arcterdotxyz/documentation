---
description: How Arcter plans to divide responsibility between a foundation and an operating company, and how control is meant to move over time.
---

# Entities and control

{% hint style="warning" %}
**Not legal advice, and not yet reviewed.** This page sets out the intended structure before launch. Counsel has not signed off on it, and no part of it is a settled legal conclusion. No programs are deployed, so every arrangement below is planned.
{% endhint %}

Arcter is organized around two bodies with separate jobs: Arcter Foundation looks after the protocol, and Arcter Labs builds and runs the product around it. Neither issues the tokens traded on the venue. Those come from third parties such as Backed Assets (JE) Limited and Ondo.

## Who does what

| | Arcter Foundation | Arcter Labs |
| --- | --- | --- |
| Role | Steward of the protocol and its governance | Operating company |
| Holds | The Squads multisig behind `arcter-config` and program upgrades, and the guardian role, until both pass to a governance module | The team's employment, and any licence found to be required |
| Works on | Parameter stewardship and public rationale for changes | The platform, quote service, indexer and keepers |
| Faces users | No | Yes, through the app |

## Reasons for the split

**Governance stays away from commercial pressure.** A company that sells a product has incentives that should not reach the rules of a market. Placing governance in a foundation is meant to keep them apart.

**Liability follows the point of contact.** Users deal with a front end, and that front end is run by the operating company. It is the entity expected to seek authorization wherever one proves necessary, which is meant to limit the protocol's exposure. Whether the split achieves this in a given jurisdiction is for counsel to decide.

**The market should outlive the company.** The next section tests that aim.

## If Arcter Labs stopped operating

The aim is for the venue to keep working without its operating company, at a reduced level of service.

* Depositors could still redeem, because redemption carries no permission check from Arcter. Issuer controls on the token leg would continue to apply.
* Regime transitions are permissionless, so anyone could fire them in place of a keeper. A keeper failure degrades freshness and does not endanger funds.
* The quote service, keepers, indexer and front end are intended to be open source, which would let a third party take them over.

Vault quotes need more than on-chain state, because a fresh oracle report must be carried into each transaction. A successor would need its own access to those reports, and the terms of that access are unconfirmed. Without fresh reports, markets would halt through the staleness rule. Listings would slow, convenience would suffer, and trading could stop until someone took over. Funds would remain redeemable, subject to issuer controls.

## How control is meant to move

**First stage.** The foundation's Squads multisig, behind a time lock, is the only route for parameter changes and program upgrades. Each proposal is published with a written reason.

**Later stage.** Control of parameters passes to a governance module. No date has been set. These commitments are meant to carry across unchanged:

* the time lock
* a public reason for every change
* no access to user funds
* no ability to condition redemptions
* no ability to influence an individual fill

Upgrade authority makes these commitments different from fixed code. Solana programs can be replaced by whoever holds that authority, so the commitments rest on the multisig, the time lock and verified builds, which let anyone match deployed code to source. Users should treat the time lock as their window to review a change and leave. The guardian's emergency pause can stop trading, and it cannot move funds. See [parameter governance](../accountability/parameter-governance.md) and [commitments](../accountability/commitments.md).

## Still to confirm

This page states no place of incorporation, registration, licence or signer list, because none has been established here. The following need confirming with counsel before launch:

1. The formation and jurisdiction of each entity.
2. Which entity, if any, needs authorization, and where.
3. Who the multisig signers are, and how they relate to each entity.
4. What the governance module is, and when control can pass to it.

The rules on who may use the venue are covered in [regulatory position](regulatory-position.md).
