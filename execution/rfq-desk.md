---
description: How approved makers price large orders with ed25519-signed quotes that settle atomically inside the taker's transaction.
---

# RFQ desk

{% hint style="info" %}
Arcter is pre-launch and its programs are not deployed. The quote format and maker funding described here are planned, and can change before launch.
{% endhint %}

Vaults suit everyday sizes. A large order needs someone who can hedge it, so `arcter-rfq` lets approved makers compete with signed prices. It applies when an order exceeds the vault clip, or when a maker simply beats the vault.

Makers sign off-chain, which costs nothing, so they can quote continuously. Only the taker's transaction touches the chain, and the taker pays its network fees.

## From request to fill

1. The quote service sends the request to every connected maker.
2. Makers reply during a collection window of a few hundred milliseconds.
3. The best maker price appears beside the vault price, and the ticket offers the better one.
4. The taker's wallet signs a transaction containing the maker's signed quote.
5. The transaction settles both legs at once, or fails as a whole.

## The signed message

A quote is a fixed byte message signed with the maker's ed25519 key. All amounts are raw token units.

| Field | Meaning |
| --- | --- |
| `domain` | Binds the quote to the `arcter-rfq` program, so a signature cannot be replayed against any other deployment |
| `maker` | Maker public key, which must hold a `MAKER` attestation |
| `mint_in`, `mint_out` | The pair, with USDC on one side |
| `amount_in`, `amount_out` | Exact size and price. Partial fills are not allowed. |
| `taker` | The requesting wallet, or empty to allow any taker |
| `expiry` | Unix time in seconds. Quotes last seconds, not minutes. |
| `quote_id` | Unique per maker and usable once |

## How the signature is checked

A Solana program does not usually verify ed25519 itself, as doing so would use too much compute. The transaction instead includes an instruction for the native Ed25519 program, and a bad signature fails the whole transaction. `arcter-rfq` then reads that instruction through the Instructions sysvar.

This pattern is safe only if the reading is strict. An Ed25519 instruction proves that some key signed some bytes, not that the right maker signed this quote. The program must therefore confirm that:

* the instruction belongs to the Ed25519 program and sits at the expected position relative to the current instruction;
* its offsets and instruction indexes point at its own data, not at bytes elsewhere in the transaction;
* the verified public key equals `maker`;
* the verified message equals the quote being settled, byte for byte.

The instruction carries a 64 byte signature, a 32 byte public key and the message. That adds up against the 1,232 byte transaction limit, which matters most on a two-leg route.

## Settlement checks

Before any token moves, `arcter-rfq` confirms:

* the verified signature, as above;
* the maker's attestation, and the taker's access where the market requires it;
* that the quote identifier has not been used or cancelled;
* the expiry, against the Clock sysvar;
* the implied price, which must sit inside the [oracle band](quote-math.md).

The Clock check gives a quote its short life. Blockhash expiry adds a separate limit on the transaction, at about 150 slots. A quote's `expiry` is the tighter of the two.

`amount_in` and `amount_out` are raw, but the band check is made on scaled value using the multiplier in force. If a multiplier change lands between signing and settlement, the implied price moves and the band can reject the fill.

## Replay protection

Each filled quote creates a PDA derived from the maker and `quote_id`. A second attempt fails, because the account already exists. This holds across transactions and needs no shared counter, so one maker's quotes do not have to settle in sequence.

Cancelling works the same way. A maker can create the PDA for a single quote ahead of time, or raise a per-maker floor that voids every lower identifier at once. Creating a PDA costs a rent-exempt deposit. Who funds it, and whether it can be reclaimed after expiry, remain to be confirmed.

## How funds move

A quote signature proves agreement, but it is not token authority. An SPL transfer needs the account owner or a delegate to sign, and Arcter does not use standing approvals. The maker is also absent from the taker's transaction.

The planned answer is prefunded maker inventory. Each maker would hold balances in token accounts controlled by an `arcter-rfq` PDA. Only the maker can withdraw them, and only a valid signed quote can draw on them during settlement. This design is still to be confirmed, along with issuer clearance for those accounts under default account state and transfer hook rules.

On a successful fill:

* `mint_in` passes from the taker to the maker's balance;
* `mint_out` passes from the maker's balance to the taker;
* the RFQ fee, 2 bps of notional at launch, reaches the treasury PDA.

## When a fill fails

Settlement fails if the quote has expired, its identifier is spent or cancelled, or the price has left the band by the time of inclusion. Transfers also fail if a hook refuses them or an account is frozen.

A failed fill moves no tokens and costs the maker nothing on-chain, though any hedge it opened remains its own. The transaction's fee payer still pays the base fee and any priority fee.

## Why a maker would quote

1. **Hedgeable flow.** Each request shows size and direction, and each asset has a liquid underlying. A maker filling 200,000 USDC of tokenized NVDA during the session can offset the risk within seconds. Outside the session, it can build the gap into its quote.
2. **Skew income.** A vault's lean is [readable from its Market account](quote-math.md). A maker trading with the skew earns a spread for pushing that vault toward target.
3. **No queue and no venue charge.** A maker wins on price alone, and a losing quote costs nothing.

Primary access matters here. Whether makers can reach mint and redeem, directly or through partners, sets how tightly they can hold prices, and that is still an open discovery question.

## Limits on a bad quote

The vault path's protections apply here too:

* a mispriced quote cannot settle beyond the band;
* only attested makers can quote;
* short expiries bound how long a stale price can survive;
* atomic settlement means a failed transfer undoes the entire fill.

A compromised or hostile maker can do no worse than fill inside the band, which an aggressive honest maker could do anyway. Its own inventory is what it puts at risk. The operational cost is real, however. Early depth may rest on a small number of makers, and the desk is only as deep as they choose to be.

Onboarding and the full protocol are covered in [Market maker onboarding](../guides/market-maker-onboarding.md) and [Developer access](../system/developer-access.md).
