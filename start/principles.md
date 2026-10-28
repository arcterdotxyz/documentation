---
description: Nine rules that each Arcter feature has to satisfy, and how a reader can check that the venue follows them.
---

# Principles

Arcter's design decisions come back to nine rules. A feature that breaks one does not ship. Each rule below is paired with what it means in practice, so you can hold the venue to it once it is live.

> Arcter is pre-launch and no programs are deployed. Where a rule depends on deployed code, read it as the intended behavior.

## 1. Your keys, your signature

Arcter never takes possession of what you own. Keys remain with you, and nothing settles without your signature.

Solana helps here, because there are no standing token approvals for a venue to hold. Each transaction moves funds with the signature it carries. A depositor's inventory sits in token accounts controlled by that market's PDA, and no team member holds a key to it. Maker fills pass directly between maker and taker.

Two limits apply. Programs can be upgraded, which is why the upgrade authority sits behind a Squads multisig and a time lock. And an issuer's permanent delegate can move tokens out of any account of that mint, so self-custody here means freedom from Arcter, not from the issuer.

## 2. The market that matters is elsewhere

The fair value of a listed share forms on its home exchange. Arcter does not claim to discover it, and reserves never move the midpoint. A verified oracle reference starts each quote, and a band around that reference limits each fill.

The same reference is the benchmark for the planned [fill reports](../accountability/fill-reports.md). Arcter's service is carrying that value to a wallet at 3am on a Sunday, and charging openly for doing so.

## 3. No number stays folded inside another

A quote shows mid, spread, skew and fee on separate lines. The session state sits on the ticket itself, in plain view, while the decision is being made.

Arcter sells no order flow and adds no hidden markup. A revenue line that works only while it stays hidden is one we will not build.

## 4. The clock is part of the price

An hour of exposure on a Tuesday afternoon is a smaller risk than an hour at 3am on a Sunday. The current [regime](../execution/session-regimes.md) therefore sets the spread multiplier, the clip and whether a market quotes at all.

No person picks the regime. It comes from verified oracle data and the clock, and the exact report fields are to be confirmed during discovery. Anyone may call the transition, and a swap also derives the regime in-line, so stale state cannot produce a cheap quote. Each signal can only tighten a market.

## 5. The mint is part of the market

An equity token's behavior is defined at its mint. Arcter reads each mint before listing and treats what it finds as a design input.

Scaled UI Amount means vault math runs on raw units, with the multiplier read fresh for each quote. A transfer hook means each swap must carry the accounts the hook requires. Default account state may leave a vault account frozen until the issuer clears it. Mint configuration is recorded at listing, a keeper watches for changes, and any change pauses the market until it has been reviewed.

## 6. Access is settled before you sign

Issuers set the jurisdictions, and their lists differ by product and channel. Arcter applies the strictest applicable reading to each market. Sanctions screening runs in the app, issuer registries remain the source of truth, and Arcter does not collect identity documents itself. `arcter-access` keeps an Attestation PDA per wallet for markets that require one.

The router simulates a trade, including any transfer hook, before it quotes. An ineligible user gets a clear reason up front, and avoids paying network fees on a transaction that was going to fail. The [regulatory position](../legal/regulatory-position.md) page has the wider picture.

## 7. One market's trouble stays in that market

Each listing has its own Market account, reserves, share mint, parameters and depositors. A halt, a dislocation or a delisting in one is contained there, and liquidity is never pooled across listings.

This isolation is economic. All markets run on the same programs and settle in USDC, so a program bug or a USDC failure would reach beyond a single market.

## 8. Claims come with a way to check them

Each fill links to its transaction on Solscan and the Solana Explorer. Parameters sit in public accounts that any RPC client can read. Changes pass through the time lock with a written rationale, and verified builds are planned so that deployed programs can be matched to public source.

The [explorer guide](../guides/reading-the-explorer.md) exists so that a skeptic can do this work alone. The [commitments](../accountability/commitments.md) page lists what can be checked, and how.

## 9. Assume the network will have bad days

Solana can restart at cluster level while outside markets keep moving. Congestion and RPC failures are more common and much smaller. Oracle reports can stop arriving. A corporate action can remove a usable price for a while.

Each case has a written response in the [risk model](../safeguards/risk-model.md), whether that is to widen, to halt or to wait out the one-hour window after a restart. An expired blockhash simply means the transaction never lands. A guardian's emergency pause can stop trading, and it cannot move funds. Deposits and withdrawals continue where safe, although an issuer pause can still block the affected token.

## What these rules exclude

| Decision | Reason |
| --- | --- |
| No token emissions | Depositors are paid from what trading earns, in spread and fee share. |
| Spot trading only | Borrowing, margin and liquidation would add risks that the venue exists to avoid. |
| No hand-set prices | Quotes come from the published formula. Parameters change only through the time lock. |
| No markets without a reference | A memecoin gives the design nothing to anchor to. |
