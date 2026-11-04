---
description: A register of what can go wrong on Arcter, who pays if it does, and which control limits the damage.
---

# How Arcter models risk

Tokenized stocks bring outside dependencies onto a blockchain: a home exchange, an issuer, a custodian and a price source. A venue that trades them inherits all four, on top of the usual software and network hazards. This page lists those hazards in one place, so that a depositor or trader can see what they are accepting.

{% hint style="info" %}
Arcter is pre-launch. No programs are deployed, no audit has been completed, and no public addresses exist. Every control below describes the intended design, and figures may change before launch.
{% endhint %}

Each hazard gets three answers: who loses money, what limits the loss, and where the mechanism is documented. A hazard with no honest answer to the second question stays on the list anyway. Missing entries count as documentation defects, so please send them to hello@arcter.xyz.

## The risk register

Rows are grouped by the party that carries the loss first.

| Hazard | Falls first on | What limits it | Full detail |
| --- | --- | --- | --- |
| Inaccurate, stale or replayed price data | Depositors and traders | Report verification, age limits, a Pyth cross-check, a 25% jump guard, `HALTED` | [Price data defenses](price-data-defenses.md) |
| Weekend and overnight gaps | Depositors | `CLOSED` regime: spread x3.0, clip x0.5 | [Session regimes](../execution/session-regimes.md) |
| Informed or one-way flow | Depositors | Inventory skew, clip limits, one-sided quoting at the inventory limit | [Vault liquidity](../execution/vault-liquidity.md) |
| Issuer controls, credit and legal terms | Every holder of an equity token | Mint review at listing, change monitoring, disclosure, caps, a multi-issuer catalog | [Issuer dependence](issuer-dependence.md) |
| Cluster halt, congestion, RPC failure | Depositors, plus traders with transactions in flight | Regime derived in each swap, one-hour restart window, time-based deadlines | [Network disruptions](network-disruptions.md) |
| Maker misbehavior or a stolen maker key | Takers | Quote expiry, replay protection, ed25519 verification, revocable approval, the oracle band | [RFQ desk](../execution/rfq-desk.md) |
| Program defect | Everyone | Planned independent reviews, fuzzing and invariant tests, a public bug bounty, launch caps | [Security practices](../system/security-practices.md) |
| Abuse of upgrade authority | Everyone | Squads multisig, a time lock, verified builds that match deployed code to source | [Security practices](../system/security-practices.md) |
| Wrong parameter | Depositors and traders | Time-locked changes with a written reason, launch caps, the oracle band | [Parameter governance](../accountability/parameter-governance.md) |
| Loss of the USDC peg, or a Circle freeze on a reserve | Everyone | A widely used settlement asset, depeg monitoring, a halt on the affected market, concentration shown openly | [Quote currency](../catalog/quote-currency.md) |
| Rule changes by regulators | Arcter as a venue | Strictest applicable access reading, attestations that can narrow a market | [Regulatory position](../legal/regulatory-position.md) |

## One control sits under most of the table

The `arcter-router` program refuses any fill that lands too far from the guarded oracle price. That limit is the oracle band, planned at 75 bps for Tier A at launch, with wider values for lower tiers. It applies to vault fills and maker fills alike.

Trace the failures above and most reach this check. A mistyped spread, a compromised maker key and a broken quote service all start in different places, and each one still has to produce a fill inside the band. An unbounded set of scenarios therefore becomes a loss per fill that can be calculated in advance. Launch caps on volume and deposits build on that number, so total exposure is also calculable while the design earns a track record.

The band has a limit of its own. It measures distance from the oracle, so it offers no help if the oracle itself is wrong. That is why price data has its own set of defenses, and why a suspect price halts the market and does not simply narrow it.

## What no control removes

Some risks stay with the person holding the position:

* **Price moves in the asset you hold.** Owning tokenized NVDA means carrying NVDA's price. Arcter prices trades and provides no insurance on holdings.
* **Issuer failure or intervention.** An issuer can become insolvent, and it may be able to freeze or move tokens. Caps and disclosure bound and reveal this exposure. They do not cancel it.
* **Real volatility.** Stale data and implausible jumps stop a market. A genuine fall below 25% in one update does not. Arcter passes on an accurate drop exactly as it happened.
* **A halt while you are exposed.** Network halts stop withdrawals as well as trades. Nothing on Solana can settle until the cluster resumes.
* **Tracker structure.** These tokens provide price exposure, not shareholder rights, and they can trade away from fair value. Oracle anchoring reduces that effect for vault fills without eliminating it.

## Keeping the register current

Once Arcter is live, any incident touching a row will get a published post-mortem. A resulting design or parameter change will pass through the Squads time lock like any other change. Incidents at other venues are also reviewed, and the conclusions are recorded in [Precedents](precedents.md). Arcter expects to be judged by the same standard.
