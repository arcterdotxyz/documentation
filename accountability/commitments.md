---
description: What Arcter will do at launch, and how anyone will be able to test each promise for themselves.
---

# Launch commitments

{% hint style="warning" %}
**None of this is in force yet.** Arcter is pre-launch. No programs are deployed, no audit has been completed and no public addresses exist. This page states what Arcter will do from launch. Each check below becomes possible once the first release ships.
{% endhint %}

A promise is only useful if an outsider can test it. So each row below pairs one commitment with one check, and most checks need nothing more than a block explorer. After launch, a commitment that fails its check will be handled as an incident.

## Custody and exits

| At launch, Arcter will | How to check |
| --- | --- |
| Never hold user funds. Each trade moves tokens under the trader's own signature, and no standing approval is ever requested. | Inspect a swap on Solscan or the Solana Explorer. The signer list and token movements are all there. |
| Keep vault reserves in accounts controlled by each Market PDA, with depositors holding a plain SPL share mint that redeems pro rata. | Read the Market account and its share mint supply, then compare your shares to the reserves. |
| Add no Arcter condition to a withdrawal in any regime, including `HALTED`, wherever a withdrawal is safe to process. | Look at a halted period on the Ledger. Withdrawals appear beside the halt record. The invariant tests will be public. |

Issuers sit outside that last promise. A frozen or paused mint can block the token side of a withdrawal, and a permanent delegate can move tokens out of any account, a vault included. The unaffected USDC side stays withdrawable. See [Issuer dependence](../safeguards/issuer-dependence.md).

## Change control

| At launch, Arcter will | How to check |
| --- | --- |
| Route every parameter change through the Squads multisig and its time lock, and publish the reasoning before execution. | The **Change history** on the Ledger lists the proposal, reasoning, waiting period and execution, and each entry links to Solana. |
| Hold program upgrade authority behind that same multisig and time lock, and never leave it with a single key. | Read the upgrade authority on each program account in an explorer. |
| Ship verified builds of every program. | Rebuild from the tagged public source and compare the result to the deployed program. |
| Charge the fee in force when your fill happens, and never change it afterwards. | The fee is written into each fill event. |

Solana programs can be upgraded, so Arcter will not describe its code as immutable. The protection is notice: a change is visible during its waiting period, and anyone can match the result to public source. Details are in [Parameter governance](parameter-governance.md).

## Pricing and routing

| At launch, Arcter will | How to check |
| --- | --- |
| Record mid, spread, skew and fee as separate fields in the event for each fill. | Open the fill in an explorer and reconcile each field against the ticket you signed. |
| Reject any fill beyond the oracle band, by program invariant and without discretion. | Read the invariant tests, then read the band margin the Ledger shows per fill. |
| Publish the gap between every fill and the reference price, with nothing filtered out. | Recompute any figure in [Fill reports](fill-reports.md) from on-chain events. |
| Take no payment for order flow, and give no liquidity source preference. The router sends each trade to the best price it can verify. | Use the oracle report and Market state from a fill to work out which lane was cheaper. |

## Disclosure

| At launch, Arcter will | How to check |
| --- | --- |
| Publish a listing record before any market opens, covering tier evidence, the mint extension review and issuer disclosures. | Find the market's listing record on the Ledger. |
| Release a written post-mortem within 14 days of any incident. | Read the incident log. A missing post-mortem counts as a breach by itself. |
| Keep issuer, gap and oracle risk in the main text of these docs. | Read the [Risk model](../safeguards/risk-model.md). Nothing there is held back for an appendix. |

## Promises Arcter will not make

Some assurances are absent because the design cannot enforce them:

* the best price across every market on Solana
* a profit on any trade or deposit
* continuous availability, since Solana can restart and RPC providers can fail
* protection from an issuer's own powers over its mint

Those limits are stated here so that the list above can be taken literally.
