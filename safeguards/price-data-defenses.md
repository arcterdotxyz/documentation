---
description: How Arcter verifies each pull report, and what happens when price data is old, replayed, contradicted or implausible.
---

# Price data defenses

Every quote on Arcter starts from a reference price, so a bad price is the most direct way to hurt a depositor. Solana oracles work by pull: nobody writes prices on a schedule, and the party that needs a price brings a signed report into the transaction. That model is fast, and it hands the transaction sender a choice about which report to bring. The defenses here are built around that fact.

{% hint style="info" %}
The `arcter-oracle` program is in development and not deployed. Report schema, access terms and per-asset coverage on Solana are confirmed during discovery. A listed asset is not assumed to have Data Streams coverage, and an asset that fails this check does not list.
{% endhint %}

## Two independent sources

| Source | Job | How it reaches a transaction |
| --- | --- | --- |
| Chainlink Data Streams, intended primary | Supplies the mid and session information | A signed report travels in the transaction, and Chainlink's verifier program checks it on-chain |
| Pyth pull updates, secondary | Independent cross-check on the mid | A signed update is posted and verified through Pyth's programs |
| Scaled UI Amount multiplier | Converts between raw units and displayed units | Read from the mint in each quote, never cached |

Pyth also offers 24/7 equity indices and Lazer low-latency data. Whether either has a role in Arcter is open. The closed-session comparison below depends on that decision.

## The checks, in order

`arcter-oracle` applies these steps before a price can reach a quote, a band check or a regime calculation.

1. **Authenticity.** The report must pass verification by Chainlink's verifier program. `arcter-oracle` pins the expected program address, so a look-alike verifier fails.
2. **Identity.** The report's feed identifier must equal the one stored on that Market account. A valid report for a different asset is rejected.
3. **Sanity.** A zero or negative price is rejected.
4. **Ordering.** The report's observation time must not be earlier than the newest one the Market has accepted.
5. **Age.** The gap between the Clock sysvar time and the report's observation time must fall within the session limit.
6. **Jump.** A move above 25% from the last accepted price halts the market.
7. **Agreement.** A Pyth price outside the configured deviation band halts the market.
8. **Pause state.** A paused issuer or feed halts the market for the duration of a corporate action.

Steps 1 to 4 fail only the transaction that carried the report. Steps 5 to 8 produce `HALTED`.

| Condition | Outcome | Way back |
| --- | --- | --- |
| Failed verification or wrong feed | Transaction fails | Send a valid report |
| Zero or negative price | Transaction fails | Send a valid report |
| Report older than the Market's newest | Transaction fails | Send a current report |
| Age above the session limit | `HALTED` | A fresh report arrives |
| Single move above 25% | `HALTED` | A time-locked review is published |
| Sources disagree beyond the band | `HALTED` | The sources converge and the cause is understood |
| Issuer or feed pause | `HALTED` | The corporate action ends |

A failed Solana transaction rolls back all of its state changes, so a swap that discovers a halt cannot record it. The swap simply refuses to fill. The record comes from the permissionless regime transition, which a keeper normally fires. Quotes stay safe in the gap, because every swap derives the regime for itself. See [Session regimes](../execution/session-regimes.md).

## Report age

A report carries its own observation time, and `arcter-oracle` compares it with the Clock sysvar. The limit depends on the session:

| Session | Regime | Maximum age | Pricing effect |
| --- | --- | --- | --- |
| Regular | `OPEN` | 1 hour | Published tier spreads |
| Extended | `EXTENDED` | 2 hours | Spread x1.5, clip x0.75 |
| Closed | `CLOSED` | 4 days | Spread x3.0, clip x0.5 |

These limits decide when a market must stop. They do not describe normal operation. Data Streams reports are designed to be sub-second, and the app fetches a new one for every ticket. A tighter age limit for fills is planned, with its value set once report behavior has been measured in testing.

During a closed session, the reference is the last verified level. The spread multiplier is the charge for the gap that may open before the home market returns. That charge appears on the ticket and goes to depositors, who carry the gap.

## Replay of older reports

A signature proves who produced a report. It says nothing about whether the report is the latest one. An old report stays cryptographically valid, and anyone can keep a copy. Without further checks, a trader could wait for the market to move, then attach an earlier report and trade against a price they know is out of date.

Arcter closes this in two ways:

* **A forward-only timestamp.** Each Market stores the observation time of the newest accepted report. Anything older fails, even inside the age limit. A swap already takes a write lock on its Market account, so the update adds no contention. Reports with an equal timestamp pass, because they carry the same price.
* **The age limit.** A trader cannot reach back past the session limit, or past the tighter fill limit once it is set.

Some choice remains. A sender can still pick among reports newer than the stored one. The value of that choice is capped by how far the price moved in that short interval, and it has to beat the spread to pay. Maker fills use the same checks, so an RFQ settlement cannot lean on an older reference.

## The cross-check

Chainlink and Pyth use separate publishers and separate signing, which is why a comparison between them means something. If they disagree beyond the configured band, Arcter does not guess which one is right. The market goes to `HALTED`, and it can only return once the disagreement has cleared. Pyth updates are planned to face their own age and ordering checks, so nobody can manufacture a disagreement by submitting an old update.

Closed sessions need more care. Two sources can legitimately differ when one holds a last close and the other tracks a weekend index. The rule for comparing them in `CLOSED` is to be settled during discovery, along with the band for each asset. Pyth coverage is also confirmed per asset, and the listing page will say which checks a market has.

## Pricing raw units correctly

Scaled UI Amount changes what a holder sees without moving a raw token. A report quoted per displayed share cannot be applied to raw balances until it passes through the current multiplier. Vault math runs on raw units, and the multiplier is read fresh for each quote. Whether a given report already reflects a pending multiplier change must be confirmed for each issuer, and a mismatch would misprice fills by the size of the change. Details are in [Splits and dividends](../execution/splits-and-dividends.md).

## One token, one feed

A Market prices exactly the mint it holds, using a feed for that mint. Arcter does not infer a price through a wrapper, a share token or an exchange rate between two tokens. The reasoning, and the incident behind it, is in [Precedents](precedents.md).

## Transaction limits

Reports, hook accounts and signature checks all compete for space in a 1,232-byte transaction and for its compute budget. A price update that does not fit means no fill, so size is a safety question as well as an engineering one. The router uses versioned transactions and address lookup tables, and the heaviest routes are measured during discovery.

## Changing a price source

Feed identifiers and deviation bands are stored per market in `arcter-config`. Replacing one needs a Squads multisig approval, the time lock and a published reason. The current feed, session, report age, multiplier and pause flag for each market are planned to appear on the ledger, so anyone can check them. See [Reading the explorer](../guides/reading-the-explorer.md).
