---
description: How Arcter behaves through cluster halts, coordinated restarts, congestion, RPC failures, expired blockhashes and skipped slots.
---

# Network disruptions

Solana is a Proof of Stake network with no single operator ordering transactions. Leaders rotate on a published schedule, and a slot lasts about 400 ms. That removes one central point of failure and leaves others. A venue that quotes at every hour has to plan for each of them, because the home market of a stock keeps moving whatever Solana is doing.

{% hint style="info" %}
Arcter's programs are not deployed. This page describes intended behavior. Restart handling is to be rehearsed in scripted drills before launch.
{% endhint %}

## Reliability so far

The last cluster-wide halt was on 6 February 2024, caused by a validator bug. Earlier halts occurred between 2021 and 2023. Solana reported 1,000 days without a network-wide outage on about 2 November 2026. That record is measured at cluster level, so it says little about congestion or a failing RPC provider. Those are the failures a user is most likely to meet in practice.

Depositors should still assume a restart can happen while they hold a position.

## Failure modes at a glance

| Event | What a user sees | What the funds do | Arcter's response |
| --- | --- | --- | --- |
| Cluster halt | Nothing confirms | Nothing moves, in or out | Wait, then enter the restart window |
| Congestion | Slow or failed transactions | Stay in place unless a fill lands within limits | Priority fees, in-line regime check |
| RPC failure | Errors or stale data in the app | Unaffected on-chain | Fall back to another provider |
| Expired blockhash | Transaction can no longer land | Stay in the wallet | Quote again, sign again |
| Leader skip | A short delay | Unaffected | Forward to later leaders, keep time-based checks |

## Cluster halts and coordinated restarts

In a halt, the cluster stops producing blocks. Restarting takes coordination among validators, who agree a restart point and return together. Nobody at Arcter takes part in this, and Arcter has no way to shorten it.

A halt has consequences that no program can avoid:

* No swap, deposit, withdrawal or guardian action can land. There is no alternative submission path outside the cluster.
* The stock's home market may keep trading, so vault prices fall behind.
* Depositors cannot leave during the event.

Activity after the agreed restart point may be discarded. A wallet should therefore check the status of any transaction that was not finalized before the halt, not assume it settled.

### The one-hour restart window

When the cluster returns, every market holds at `HALTED` for one hour. During the outage, prices moved and nobody could respond. The hour lets fresh reports arrive and gives depositors a chance to act before anyone can trade against a vault that froze mid-move.

Solana has no uptime feed for Arcter to read, so detection relies on time passing on-chain:

1. Keepers call a permissionless heartbeat instruction on a regular schedule. It stores the current Clock sysvar time.
2. Every heartbeat, and every swap, compares the Clock time with the last recorded heartbeat.
3. A gap above the configured threshold counts as a restart. The window starts at the first successful heartbeat or transition that sees the gap, and it runs for one hour.

The first heartbeat after a gap opens the window instead of overwriting the evidence. A swap that sees the gap first refuses to fill. Since a failed transaction rolls back, that swap records nothing, so the window still waits for a heartbeat or transition.

The method has known limits:

* **Keeper outages look like restarts.** If nobody calls the heartbeat for longer than the threshold, markets halt for an hour. Anyone can call it, so a keeper failure costs some availability and leaves funds unaffected. This error direction is deliberate.
* **Short halts can go unseen.** An outage briefer than the threshold does not open a window. The threshold is a parameter to be set from test measurements, and age limits on reports still apply.
* **Clock time is an estimate.** It comes from validator timestamps and is not a precise wall clock. How much of the real gap it shows right after a restart must be tested in restart drills, and the threshold should allow for that.

Withdrawals are available once transactions land again, during the window included. Quoting waits for the hour to end. Deposits are priced at the mid, so they wait for a usable price. Maker cancellations also work straight away.

Other regime signals still apply at the end of the hour. If reports are stale, or a jump exceeded 25%, the market stays `HALTED`.

## Congestion

Under heavy load, transactions compete for compute and for write locks on busy accounts. Each swap write-locks its Market account, so a popular market can congest before the wider network does. Priority fees, bid per compute unit, buy better placement. Stake-weighted quality of service and Jito are common inclusion paths, and Arcter is evaluating both during testing.

Arcter's safety does not depend on landing quickly:

* **The swap derives the regime.** A keeper transition may be delayed, but a stale regime cannot produce a cheap quote.
* **Reports are checked when the transaction lands.** A report that aged in a queue is judged against the Clock at execution.
* **The signed limits hold.** The minimum output and the oracle band bound a late fill. Failing is the expected result when a price has moved.

Costs still reach users. A failed transaction pays its fee, and a depositor leaving during a busy period may need to bid a higher priority fee. Makers should expect that a cancellation can be delayed too, which is why quotes are kept short-lived.

## RPC failure

Wallets and the app reach Solana through RPC nodes, and these fail far more often than the cluster does. A node can time out, lag behind the tip, return an outdated blockhash or simulate against old state. None of that alters on-chain funds, but it can mislead a user.

* The quote service treats simulation as an estimate. All enforcement happens on-chain at execution.
* The app is planned to use more than one RPC provider, with Helius as an option, and to show when its data is behind.
* A transaction that seems missing may still land. The app checks by signature before offering to retry.
* The read model comes from a Yellowstone gRPC stream, so a stream failure can delay the ledger display without changing settlement.

Withdrawals do not depend on Arcter's servers. Anyone can build the instruction and submit it through any RPC endpoint.

## Expired blockhashes

Each transaction references a recent blockhash, which expires after about 150 slots, roughly a minute or a bit more. An expired transaction can never land, so it gives every ticket a natural outer deadline. It fails safely: the funds stay in the wallet, and the user quotes again.

Three details matter for correct handling:

* **Rebuilding too early risks a double fill.** A wallet should wait until the blockhash has passed its last valid block height before signing a replacement. Resending the same signed transaction is safe at any time.
* **Blockhash lifetime is not a price guarantee.** The router enforces its own deadline using Clock time, and that deadline is usually shorter. This also covers durable nonce transactions, which skip blockhash expiry.
* **Slots stop during a halt.** A lifetime counted in slots may outlast a long outage in real time. The time-based deadline ends those tickets.

## Leader skips

A leader can miss its slots through a fault, poor connectivity or a late block. Transactions it held may have to reach a later leader, adding a short delay. Skips are routine and affect one leader at a time.

They are also why Arcter never converts slots into time. Slot length varies, and the count of slots is not a clock. Time locks, grace windows, report age and quote expiries all use the Clock sysvar's unix timestamp.

A leader can also leave a transaction out on purpose. Leaders rotate, so the next one can include it, which limits how long any single validator can delay a withdrawal or a maker's cancellation. No formal guarantee of inclusion exists, however, and collusion among leaders is outside what Arcter can control.

## Ordering and network changes

Block producers see transactions before they land, Jito's block engine is widely used, and sandwiching exists on Solana. Arcter does not count on favorable placement. A fill is bounded by the oracle band and the signed minimum output wherever it lands in a block. Keep the minimum tight and the quote window short.

The network itself is changing. Votor went live on Solana on 21 October 2026 (epoch 1064), replacing TowerBFT and targeting about 150 ms finality. Slot length did not change. Rotor is not scheduled yet, so Turbine still propagates blocks. Agave, Firedancer and Frankendancer all run on the network, which may help contain a bug in any one client. A consensus or client change that alters Arcter's timing assumptions triggers a review, published in the [parameter governance](../accountability/parameter-governance.md) log.

For background on how the network works, see the [Network primer](../solana/network-primer.md).
