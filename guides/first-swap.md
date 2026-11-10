---
description: Step by step from an empty wallet to a settled Arcter swap, including what to do when a signature takes too long.
---

# Make your first swap

{% hint style="info" %}
This walkthrough describes the intended flow for the planned release. Arcter's programs are not live, so screens and labels may still change.
{% endhint %}

A swap on Arcter is one Solana transaction with one signature. There is no separate approval step, and nothing stays authorized after the transaction ends. This guide follows that transaction from connecting a wallet to reading the receipt.

## What to have ready

| Item | Why it matters |
| --- | --- |
| A Solana wallet | Phantom, Solflare and Backpack connect through Wallet Standard. If you prefer email or a passkey, the [app](https://arcter.xyz/platform) can create an embedded Solana wallet through Privy. [Connecting a wallet](../solana/connecting-a-wallet.md) has the detail. |
| A little SOL | Network fees are paid in SOL, whatever you trade. Sponsored fees on a first trade are under review, so do not count on them. |
| An asset to spend | USDC is the plainest start, because every market settles in it. An equity token you already hold also works, and the route through USDC happens for you. |
| A `TRADER` attestation, where the market asks for one | Open markets rely on issuer rules alone. A permissioned market also needs this credential, which you clear once. A KYC provider checks identity, sanctions and jurisdiction, and the result is stored in an Attestation PDA for your wallet with no personal data in it. Check your status under **Access** on the **Account** page, and read [access controls](../system/access-controls.md) for the mechanism. |

Each equity token has its own eligibility rules set by its issuer. If you are a US person, or you live somewhere an issuer excludes, you cannot use these markets. See [regulatory position](../legal/regulatory-position.md).

## Step 1: Fill in the ticket

Choose **Trade** in the left rail, select a market under **Listing**, and enter a figure in **You send**. **You get** updates as you type, and the ticket itemizes how it reached that figure.

| Line | Meaning |
| --- | --- |
| Oracle mid | The verified reference price, after its age and deviation checks. The report behind it is identified alongside. |
| Half-spread | What liquidity costs right now, in basis points and in USDC. The session multiplier is already applied. |
| Skew | A shift against whichever side would worsen the vault's inventory. It can help you if your trade rebalances the vault. |
| Venue fee | The protocol's own charge, kept apart from the spread. |
| Regime badge | `OPEN`, `EXTENDED` or `CLOSED`, with its spread multiplier. |

Regimes change the price, so read the badge before anything else. At `OPEN` you pay x1.0 on the spread with the full clip available. `EXTENDED` costs x1.5 with x0.75 of the clip. `CLOSED` costs x3.0 and allows x0.5 of the clip, because the home exchange is shut and the price can gap when it reopens. An order that can wait for the next regular session will usually be cheaper. See [session regimes](../execution/session-regimes.md) and [quote math](../execution/quote-math.md).

## Step 2: Set your floor

**Price tolerance** controls how much worse than the quote you will accept. **Least you will get** shows that tolerance as a hard minimum output, and the transaction carries this figure. The router enforces it on-chain.

Keep the tolerance tight. Block producers see transactions before they land, and sandwiching exists on Solana. A narrow floor limits what anyone can extract from your order.

## Step 3: Sign once

Before quoting, the app simulates the route, including any transfer hook on the token. If your wallet is not eligible for that asset, you should see the reason on the ticket and not a failed transaction.

When you press the trade button your wallet opens with its own simulation preview, which is normally a list of estimated balance changes. Check three things there:

1. The asset leaving your wallet and the amount.
2. The asset arriving, which should be at or above **Least you will get**.
3. Any token account being created.

The third item needs explaining. Solana holds each token in its own account, so a wallet that has never held a token needs a token account for it. The transaction creates that account for you, in the same signature, for a small refundable deposit of SOL. The deposit comes back if you later empty and close the account.

A wallet preview is an estimate made a moment before execution. Some tokens also use Scaled UI Amount, which makes the displayed balance differ from the raw balance, and at least one wallet bug has been reported showing an xStock balance at the wrong scale. If a preview looks wrong, trust the ticket's minimum, or cancel and check.

Your cost on-chain has two parts:

* a base fee of 5,000 lamports per signature
* an optional priority fee, bid per compute unit, which the app sets dynamically to help the transaction land

## Step 4: If the signature is slow or the trade fails

Every transaction names a recent blockhash, and that blockhash expires after about 150 slots, roughly a minute or a little more. Leave the wallet prompt open too long and the transaction can no longer land. Nothing moves and no fee is charged. Request a fresh quote and sign again.

| What you see | What happened | What to do |
| --- | --- | --- |
| The ticket says the quote expired | The quote window closed, or the blockhash ran out | Refresh and sign the new quote |
| The transaction failed on-chain | The price or vault moved past your floor, or a check did not pass | Your assets stay put, though the network fee is still charged. Review the ticket and try again. |
| The transaction never confirms | Congestion or an RPC failure, which are the usual day-to-day problems on Solana | Wait until the blockhash has expired before retrying, so that you do not trade twice |
| The market shows `HALTED` | A safety trigger fired | See the last section below |

## Step 5: Read the receipt

Slots last about 400 ms, so a settled swap usually appears quickly. The receipt is built from the event your transaction emitted. It repeats the breakdown you saw at quote time, names the source of your fill, and links to the transaction on Solscan and the Solana Explorer.

Your fills also collect under **Your trades** on the **Holdings** page. Arcter compares every fill with the oracle mid in its [fill reports](../accountability/fill-reports.md), which lets you measure execution yourself. To trace a fill by hand, see [check the record yourself](reading-the-explorer.md).

## Bigger tickets

When an order exceeds the vault clip, the router asks professional makers to price it through the [RFQ desk](../execution/rfq-desk.md). You keep the same ticket and the same single signature, and only the source of the fill changes. Maker quotes last seconds, and the app fetches a replacement when one lapses. A maker's price must also sit inside the oracle band.

Limit and time-weighted orders are held for later, so anyone working a large position over time must split it by hand for now. See [delivery phases](../plan/delivery-phases.md).

## What the token gives you

Owning an equity token does not make you a shareholder. These tokens provide price exposure, and they carry issuer risk.

* xStocks are issued by Backed Assets (JE) Limited of Jersey as 1:1 backed tracker certificates under a base prospectus.
* On xStocks, a split or dividend can appear as a change in the Scaled UI Amount multiplier. Your displayed balance moves and your raw balance does not.
* Ondo Global Markets tokens have their own issuer terms, and Ondo's FAQ describes 30% US withholding on dividends.

Read [tokenized equities](../catalog/tokenized-equities.md) and [issuer dependence](../safeguards/issuer-dependence.md) before you trade meaningful size. A short disclosure covering both appears on your first trade.

## When a market stops quoting

A market moves to `HALTED` when its oracle goes stale, when an issuer or feed pauses for a corporate action, when one update jumps more than 25%, during the one-hour grace window after a cluster restart, or when a guardian pauses it. The halt applies to that market alone, and the app states the trigger and what must happen before quoting resumes.

Arcter cannot freeze tokens in your wallet. Issuers are different: their mints can carry freeze and permanent delegate powers, and those powers sit outside Arcter's control. The [session regimes](../execution/session-regimes.md) page covers each trigger.
