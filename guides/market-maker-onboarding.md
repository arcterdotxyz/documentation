---
description: What a professional firm needs to answer Arcter RFQs, how an ed25519 quote is signed and verified, and how fills are measured.
---

# Onboard as a market maker

{% hint style="info" %}
This walkthrough describes the intended flow for the planned release. Arcter's programs are not live, no program addresses exist, and the message layout on this page is illustrative until the specification is published.
{% endhint %}

Makers fill the orders that vaults are too small or too wide to serve. You price block requests on the [RFQ desk](../execution/rfq-desk.md), you beat the vault when your own price is sharper, and you can earn the skew in the [quote math](../execution/quote-math.md) by moving vault inventory back toward target. A losing quote costs you nothing. A winning one settles atomically inside the taker's transaction.

## Terms of entry

You need a `MAKER` attestation, stored in the Attestation PDA for your wallet. Professional trading firms get one through the standard verification flow. Once issued, it appears under **Access** on the **Account** page.

Launch terms are light. Nothing obliges you to quote, you pay no membership fee, and no minimum volume applies. Makers are ranked in one way only: reliable quoting moves you up the request routing order.

## Technical checklist

| Requirement | Notes |
| --- | --- |
| An ed25519 quoting key | Keep it in a service that signs quote messages only. Do not reuse it for transactions. |
| USDC and token inventory | Hold each mint you intend to quote. Mints may use the classic Token program or Token-2022, so handle both. |
| Cleared token accounts | With default account state, new accounts on a mint may start frozen. Ask the issuer about clearance before you quote that asset. |
| A gateway session | An authenticated WebSocket connection. [Developer access](../system/developer-access.md) holds the connection details. |
| An RPC connection or stream | You need fresh account state to read vaults, multipliers and regimes. |

This design does not ask you to leave a standing allowance in place.

{% hint style="warning" %}
**How your inventory reaches settlement.** A signed quote proves agreement but carries no token authority, and you are absent from the taker's transaction. The planned design therefore has you prefund balances in token accounts controlled by an `arcter-rfq` PDA. Only you can withdraw from them, and only a valid signed quote can draw on them during a fill. Issuer clearance for those accounts is still to be confirmed, so wait for the specification before building settlement tooling. See [the RFQ desk](../execution/rfq-desk.md).
{% endhint %}

## From request to settlement

**1. A request reaches you.** Each broadcast gives you the pair, the size and the side, along with the attestation class of the taker.

**2. You price it.** At `OPEN` you can usually hedge almost at once. At `CLOSED` your price is a view on how far the market might gap, and hedging may not be possible at all. That is also when the vault sits at x3.0 spread, so a sharper price is most likely to win.

Read the Scaled UI Amount multiplier fresh for any mint that uses one. Quote in raw units, and convert with care, because a price built on the wrong scale is a real loss.

**3. You sign and reply.** The collection window lasts a few hundred milliseconds, and the best valid price wins.

**4. The taker's transaction lands.** It carries an instruction to the native Ed25519 program that verifies your signature. `arcter-rfq` then confirms, through instruction introspection, that this verification occurred for the exact message it expected.

**5. Both legs settle or neither does.** With no fill, no tokens move.

## Signing a quote

The sketch below uses `@solana/web3.js` and `@noble/curves`. Field order and contents are an example, so do not treat them as the final layout. Amounts are raw `u64` values, and the program field stops a signature from one deployment being replayed against another.

```typescript
import { Ed25519Program, Keypair, PublicKey } from "@solana/web3.js";
import { ed25519 } from "@noble/curves/ed25519";

interface RfqQuote {
  program: PublicKey;   // binds the quote to one deployment
  maker: PublicKey;
  taker: PublicKey;
  mintIn: PublicKey;
  mintOut: PublicKey;
  amountIn: bigint;     // raw units, u64
  amountOut: bigint;    // raw units, u64
  quoteId: bigint;      // u64, single use
  expiresAt: bigint;    // unix seconds, i64
}

function encodeQuote(q: RfqQuote): Uint8Array {
  const bytes = new Uint8Array(5 * 32 + 4 * 8);
  const view = new DataView(bytes.buffer);
  let at = 0;
  for (const key of [q.program, q.maker, q.taker, q.mintIn, q.mintOut]) {
    bytes.set(key.toBytes(), at);
    at += 32;
  }
  for (const value of [q.amountIn, q.amountOut, q.quoteId]) {
    view.setBigUint64(at, value, true);
    at += 8;
  }
  view.setBigInt64(at, q.expiresAt, true);
  return bytes;
}

export function signQuote(quote: RfqQuote, signer: Keypair) {
  const message = encodeQuote(quote);
  const seed = signer.secretKey.slice(0, 32);
  const signature = ed25519.sign(message, seed);
  return { message, signature };
}

// Built by the taker's client, placed ahead of the settlement instruction.
export function verifyInstruction(
  maker: PublicKey,
  message: Uint8Array,
  signature: Uint8Array,
) {
  return Ed25519Program.createInstructionWithPublicKey({
    publicKey: maker.toBytes(),
    message,
    signature,
  });
}
```

The verification instruction repeats your message, your public key and your signature, so it uses a real share of the 1,232 byte transaction limit. Compact messages leave more room for hook accounts and oracle reports.

## What settlement checks

| Check | Failure result |
| --- | --- |
| The Ed25519 instruction matches your key and the expected message | Settlement fails |
| Your `MAKER` attestation is valid at fill time | Once revoked, your quotes stop settling |
| The nonce or quote ID is unused | A second attempt is refused, which blocks replay |
| The quote has not expired against the Clock sysvar | Settlement fails |
| The price sits inside the oracle band, tested on every fill | No fill occurs for either side |
| Both transfers complete | The entire transaction unwinds |

These checks protect both parties. A stolen key can extract only so much from takers, and a stale quote can cost you only so much.

## Expiry and cancellation

Replay protection uses a per-maker nonce or a quote ID PDA, and the final choice will appear in the specification. A per-maker nonce also lets you void many outstanding quotes in one on-chain call.

Treat cancellation as a backstop. A cancel is an ordinary transaction with no privileged lane, so under congestion it can be slow and it competes on priority fee like everything else. Short expiries are your real protection, so keep them to a few seconds at most. Quote expiry is checked against cluster time from the Clock sysvar, which can differ from your own clock, so leave a margin.

A taker's blockhash also limits a quote, since it expires after about 150 slots. Your quote should always lapse well before that.

A stale quote that drifts outside the band will fail and will not fill at a bad price. Your hedge assumptions inside the band stay your responsibility. Transfer hooks can fail a fill too, if a required account is missing or if a wallet is ineligible, so watch your settlement failure rate by mint.

## Trading against vault skew

Skew is public. Market accounts can be read by anyone, and a quote can be simulated without sending a transaction, so you can see which side each vault is improving to attract rebalancing flow, up to the maximum skew at the edge of its band.

Taking that side within the clip earns spread against the oracle mid. You keep hedging and basis risk, and this income shrinks as other makers compete for it. It is the rebalancing design working as intended, and it keeps vaults two-sided.

## Reviewing your results

Your fills sit with all the others in the [Ledger](reading-the-explorer.md). Makers appear anonymously unless they opt in to being named, though your settlement wallet is visible on-chain to anyone who looks. Authenticated API endpoints return your fill history and quote statistics, including win rate and improvement over the vault price. That improvement figure decides your routing priority.
