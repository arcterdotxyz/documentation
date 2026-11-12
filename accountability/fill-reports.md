---
description: How Arcter will score each fill against the reference price, what it plans to publish, and how to read the results.
---

# Fill reports

{% hint style="info" %}
No report exists yet, because no programs are deployed. This page describes the reporting planned for the first release. The targets below come from launch parameters, not from measured trading.
{% endhint %}

A venue that sells execution should measure it in public. Arcter plans to score all fills, leave none out, and use a benchmark it cannot quietly swap.

## The measure

The score for one fill is a signed distance in basis points:

```
score_bps = (fill_price - mid) / mid x 10,000, signed so that positive means the trader paid more than mid
```

Here `mid` is the verified reference price used to validate that same fill, and the fill event names the oracle report it came from. Because the signed report travels inside the transaction, the benchmark sits on Solana next to the trade. Arcter gets no second chance at choosing it.

| Reading | Meaning |
| --- | --- |
| Positive | The trader paid over mid. This is the usual case, and the amount is the cost of immediacy. |
| Zero | The fill matched mid. |
| Negative | The trader beat mid, which can happen when a maker quote improves on the vault. |

Two details matter for anyone recomputing these numbers:

* **Units.** Mints with Scaled UI Amount need the multiplier that applied at that fill. Raw and displayed amounts differ, and mixing them gives a wrong score.
* **Scope.** The score covers mid, spread, skew and the Arcter fee. SOL network fees and priority fees sit outside it, because they go to the network and vary by wallet.

## Planned datasets

| Report | What it will hold |
| --- | --- |
| Rolling view | Median, 90th percentile (p90) and worst fill, cut by market, lane and regime |
| Daily fill file | One row per fill, with no minimum size |
| Session comparison | `OPEN` and `CLOSED` costs side by side, which turns the weekend premium into a measured number |
| Maker improvement | The total by which maker quotes beat the vault quote available at that moment |
| Rejection record | How often trades fail on the minimum output or the oracle band |

The Ledger in the app is designed to carry the same score, through a **Cost vs mid** column on recent trades and an **Average cost vs mid** figure.

The rejection record has a known blind spot. A failed transaction lands on Solana and can be counted. A transaction that expires with its blockhash, or never reaches a leader, leaves no on-chain trace. Arcter can count those only from its own client and quote service, so the two sources will be published separately and labeled.

## What healthy results look like

These are launch expectations, to be replaced by real data:

1. **Tier A in `OPEN`.** The median fill should cost about 12 bps from mid: the 10 bps half-spread plus the 2 bps fee. A lasting rise above that suggests a tuning problem, and any time-locked change should cite this data.
2. **Tier A in `CLOSED`.** The same fill should cost about 32 bps, since the half-spread is multiplied by 3.0, and outcomes should spread wider. Quiet sessions should still sit well inside the 75 bps band.
3. **Band margin.** Many fills drifting toward the band edge at once is the earliest sign of an oracle fault. Arcter's alerting is planned around these same public figures.

Tier parameters are listed in [Admission criteria](../catalog/admission-criteria.md), and the formula is in [Quote math](../execution/quote-math.md).

## The case for keeping bad fills

Filtered data is marketing. A costly Saturday trade and a rejected order are exactly what a reader needs:

* a depositor sizing up a vault
* a maker setting a price
* a trader deciding whether a Sunday order can wait until Monday

All of them should see one record, and Arcter should be judged against it. The same record is planned as the input to Arcter's own parameter reviews.

## Getting the data

The Ledger is planned for the first release, with each aggregate linking down to its fills. See [Reading the explorer](../guides/reading-the-explorer.md). Programmatic access is planned for the Platform phase at `/v1/execution-quality`, through the [developer tools](../system/developer-access.md).
