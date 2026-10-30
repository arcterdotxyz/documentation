---
description: The four states an Arcter market can be in, what moves it between them, and what each one costs a trader.
---

# Session regimes

{% hint style="info" %}
Arcter is pre-launch and its programs are not deployed. Report fields named here depend on oracle terms still to be confirmed.
{% endhint %}

The US regular session fills roughly 32 of the 168 hours in a week. Solana runs through the rest, and equity tokens keep trading. So a quote has to reflect whether its reference market is open right now, and Arcter puts that charge on the ticket as a labeled line.

## The states

| Regime | Applies when | Spread | Clip |
| --- | --- | --- | --- |
| **`OPEN`** | The reference market is in regular hours | x1.0 | Full |
| **`EXTENDED`** | Pre-market or after-hours trading | x1.5 | x0.75 |
| **`CLOSED`** | The reference market is not trading | x3.0 | x0.5 |
| **`HALTED`** | Any safety trigger below is active | No quotes | None |

`arcter-oracle` keeps a regime per market, and every quote reads it. `arcter-config` stores the multipliers for each tier.

## How a regime is derived

Two independent signals feed the decision:

1. **Market status.** The Chainlink Data Streams report carried in the transaction is intended to include a session field. The exact schema and per-asset coverage on Solana must be confirmed in discovery.
2. **Report age.** The report's timestamp is measured against the Clock sysvar. The limit depends on the session: 1 hour regular, 2 hours extended, and longer when closed.

The strictest reading wins. Neither signal can loosen a market, so a fresh report whose status says closed still gives `CLOSED`.

Moving a market to its new regime is permissionless. A keeper normally calls the transition soon after a session changes. If a swap arrives first, it derives the correct regime in-line from the report it carries. A keeper failure therefore delays a stored update and cannot produce a cheap quote from the previous session.

The Clock sysvar's timestamp is a cluster estimate and not a precise wall clock. A small difference has little effect against limits measured in hours, and the market status signal carries session changes in any case.

## No calendar in the programs

The programs contain no holiday table. Early closes, holidays and unscheduled exchange shutdowns are expected to arrive through the oracle's market status. If that field cannot be confirmed for a given asset, the asset does not list on this basis. The aim is that a market never quotes into a session the data reports as shut, and never sits idle through one reported open.

## Why `CLOSED` still quotes

Two simpler policies were rejected:

* **Stop quoting on weekends.** Holders would be pushed to venues that pay no attention to the gap, on a network that does not close.
* **Keep weekday spreads.** Depositors would become the free counterparty to each Monday open.

Arcter takes the middle path. Weekend liquidity exists, in smaller sizes and at a stated premium. The x3.0 multiplier exists to charge for gap risk. It starts conservative, will be measured against real Monday opens, and can move only through the Squads multisig and its time lock.

## Halts

A market remains `HALTED` while any trigger is active, and quoting resumes once all have cleared.

| Trigger | Cleared by |
| --- | --- |
| Oracle data older than the session's limit | A fresh verified report |
| Issuer or feed pause during a corporate action | The pause ending (see [Splits and dividends](splits-and-dividends.md)) |
| A price jump above 25% in one update | Review through the time lock, with published findings |
| Chainlink and Pyth disagree beyond a set band | The sources converging again (band value to be confirmed) |
| The one-hour grace window after a cluster restart | The window ending (see [Network disruptions](../safeguards/network-disruptions.md)) |
| A guardian's manual pause | The pause being lifted, with a published incident note |

A pull oracle only updates when someone submits a report, so a quiet market can look stale without anything being wrong. The next verified report clears it.

The restart window has a Solana-specific reason. Cluster restarts are coordinated among validators, and they end with prices having moved while no transaction could land. How a program detects a restart reliably is still to be specified before launch.

## What keeps working during a halt

A halt stops quoting. It is not a freeze on funds:

* Deposits and withdrawals continue where safe. An issuer freeze can still hold the asset side while USDC remains withdrawable, as [vault liquidity](vault-liquidity.md) explains.
* A maker can still cancel its quotes.
* Accounts remain readable, and the open ledger shows the live reason for the halt from the same state the router uses.

An emergency pause can stop trading. It cannot move funds.
