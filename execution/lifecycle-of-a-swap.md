---
description: What happens to one Arcter swap, from the first simulated price to a confirmed Solana transaction.
---

# Lifecycle of a swap

{% hint style="info" %}
Arcter is pre-launch and its programs are not deployed. This page describes the intended design.
{% endhint %}

A swap on Arcter is one Solana transaction, signed once by the trader. Nothing is deposited beforehand and nothing is collected afterward. Either the whole trade lands, or no balance changes.

Three pieces share the work:

* `arcter-router`, the only entry point. It enforces the trader's limits and composes the route.
* `arcter-vault`, whose [oracle-priced markets](vault-liquidity.md) serve everyday sizes.
* `arcter-rfq`, which settles [signed maker quotes](rfq-desk.md) for larger orders.

## Stage 1: price

Take a request to buy tokenized NVDA with 2,000 USDC. The quote service simulates the path the programs will later run. It takes a verified reference price, finds the current [session regime](session-regimes.md), reads the market's reserves and applies [the quote formula](quote-math.md). Larger requests also go out to makers.

The result is a single price, split into mid, spread, skew and fee. Simulation includes any transfer hook on the mint, so a trade that would be refused shows a reason in place of a failed transaction.

## Stage 2: sign

The trader signs a versioned transaction containing the trade, a minimum output and a deadline. Solana has no standing token approval here. The input moves under that one signature, in that one transaction, and the permission ends with it.

Three things are decided at this point and cannot change later:

* **The account list.** Solana transactions declare every account up front, so the route is fixed when the wallet signs.
* **The cost of inclusion.** The fee is 5,000 lamports per signature, plus an optional priority fee priced per compute unit, paid in SOL. Sponsoring a first trade through a relayer is under review.
* **The outer time limit.** A recent blockhash expires after about 150 slots, roughly a minute or a bit more.

If the wallet has no token account for the output asset, the transaction creates one. That costs a small refundable deposit. Where a mint uses default account state, a new account may start frozen, and the trade cannot complete until the issuer has cleared it.

## Stage 3: checks inside the transaction

The programs trust nothing computed off-chain. In order, they:

1. compare the deadline with the Clock sysvar;
2. check market access against the wallet's Attestation PDA, where the market requires one;
3. verify the oracle report carried in the transaction and derive the regime from it;
4. recompute the vault price from that report and the Market account, or, on the RFQ path, confirm the maker's signature through the native Ed25519 program;
5. hold the fill to two limits, namely the trader's minimum output and the [oracle band](quote-math.md).

The program checks the deadline for itself because blockhash expiry is not a guarantee it can rely on. A transaction built with a durable nonce does not expire that way.

## Stage 4: settle

Token movements happen through cross-program invocations inside the same transaction. The input leaves the trader, the output arrives from the vault or the maker, and the fee reaches the treasury PDA. Mints with a transfer hook need their extra accounts present, and the router resolves those ahead of time.

The transaction emits one fill event with an itemized breakdown. Anyone can open it on Solscan or the Solana Explorer. The event is proof of the fill only once the transaction has reached the commitment level the reader requires. With Votor live since 21 October 2026, finality is targeted at about 150 ms, while slots remain about 400 ms.

## When a swap fails

A failed swap moves no tokens, but the network still charges for the attempt. The base fee and any priority fee are spent. The common causes are a price that moved past the minimum output, an expired blockhash, a regime change, or congestion and RPC failures that stop the transaction from landing at all.

A transaction that never lands costs nothing and can be signed again with a fresh quote. Block producers can see transactions, and sandwiching exists on Solana, so the ticket should carry a tight minimum output and a short window.

## Which lane fills the order

Both lanes compete, and the better price wins:

| Situation | Lane | Reason |
| --- | --- | --- |
| Size within the vault clip, no better maker quote | Vault | A formula sets the spread, with no counterparty to wait for |
| Size above the clip | RFQ | A maker hedging in the underlying market can price size more tightly |
| Maker beats the vault, at any size | RFQ | Price decides, so the clip is a guide and not a boundary |

A maker may win a small order whenever its quote is better. Because accounts are fixed at signing, the comparison runs before the wallet signs, and [order routing](order-routing.md) covers what is checked again on-chain.

## Swapping between two listed assets

Every Arcter market is priced in USDC, so tokenized AAPL into tokenized SPY runs as two USDC legs in one transaction. The same holds for an equity token into gold once a gold market exists. The trader sees one price and one minimum output, while the fill event records each leg.

Each leg must pass its own band and regime checks. If either leg fails, the transaction fails and both are undone. Two legs carrying hook accounts, oracle reports and possibly maker signatures are heavy against the 1,232 byte transaction limit. The router therefore uses address lookup tables, and compute budgets are measured per route. Whether the heaviest routes fit is an open question for discovery.

## Design limits

* **No shared pools.** A market holds one asset plus USDC, so a depositor chooses exposure one market at a time.
* **No discretion over fills.** Vault prices can be recomputed from public data, and the RFQ path checks a signature. Arcter staff cannot favor or delay a given fill.
* **No deferred settlement.** Trades are not netted or batched for later. A finalized transaction leaves nothing outstanding.
* **No claim of immutability.** Funds sit inside program-controlled accounts only during the transaction, but the programs themselves can be upgraded. Upgrade authority is planned to sit with a Squads multisig and a time lock, with verified builds that let anyone match deployed code to public source.

## Where to go next

1. [Vault liquidity](vault-liquidity.md): how a market holds inventory.
2. [Quote math](quote-math.md): the formula and its inputs.
3. [Session regimes](session-regimes.md): how the clock and halts change quoting.
4. [RFQ desk](rfq-desk.md): signed quotes for larger orders.
5. [Order routing](order-routing.md): how the parts connect.
6. [Earning as an LP](earning-as-an-lp.md): the depositor's view.
7. [Splits and dividends](splits-and-dividends.md): corporate actions.
8. [Costs](costs.md): everything a trade pays.
