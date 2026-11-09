---
description: The planned quote API, live stream, TypeScript SDK and maker protocol for software that integrates with Arcter on Solana.
---

# Developer access

{% hint style="warning" %}
The API and SDK are planned and not yet available. No endpoint on this page is serving traffic, `@arcter/sdk` is unreleased, and no Arcter program is deployed. Paths, fields and method names may change until the first public specification. Developer access is scheduled for release 2.0, after the swap and vault releases.
{% endhint %}

Aggregators, wallets, agents and portfolio tools should get the same quote a person gets in the web client. The plan has one public surface, with no partner tier and no private route. Trades from every client reach the same `arcter-router` program, and the web client is meant to use only what this page describes.

## Rules for integrators

* **A quote is a preview.** The programs recompute or re-verify every term at settlement. A wrong quote can make a transaction fail, and it cannot fill outside the oracle band or below the signed minimum output.
* **The wallet signs everything.** The service holds no keys. There are no standing token approvals, so each trade moves funds under its own signature.
* **Amounts are raw.** Requests and responses use the mint's raw units. Equity tokens can carry a Scaled UI Amount multiplier, so convert for display only, and read the multiplier fresh.
* **Time is short.** A transaction's blockhash expires after about 150 slots, roughly a minute or a little more. A quote can expire sooner. Fetch a new one rather than holding an old one.

## Planned surface

| Path | Returns |
| --- | --- |
| `GET /v1/quote` | An itemized price and, when a wallet is supplied, a transaction ready to sign |
| `GET /v1/markets` | Listed markets with tier, parameters and current regime |
| `GET /v1/fills` | Trade history in pages, with the full price split per fill |
| `GET /v1/execution-quality` | The numbers behind [Fill reports](../accountability/fill-reports.md) |
| `GET /v1/eligibility/:wallet` | A dry run of a wallet's access checks |
| `wss://api.arcter.xyz/v1/stream` | Pushed fills, regime transitions and per-market price changes |

## Requesting a quote

The example asks to spend 2,000 USDC on NVDAx. All values are illustrative.

```
GET https://api.arcter.xyz/v1/quote?tokenIn=USDC&tokenOut=NVDAx&amountIn=2000000000&wallet=<BASE58_WALLET>
```

```json
{
  "lane": "vault",
  "amountOutRaw": "1126269000",
  "outDecimals": 8,
  "uiMultiplier": "1.0",
  "minOutRaw": "1124579000",
  "breakdown": {
    "mid": "177.40",
    "spreadBps": 10,
    "skewBps": -2,
    "feeBps": 2,
    "regime": "OPEN",
    "oracleReportSlot": 123456789
  },
  "rfq": null,
  "validUntil": 1793000000,
  "lastValidBlockHeight": 123456939,
  "transaction": "<BASE64_VERSIONED_TRANSACTION>"
}
```

Reading the response:

* `lane` names the winner, either `vault` or `rfq`. When a maker wins, `rfq` holds that maker's quote and ed25519 signature.
* `breakdown` gives each price term separately, so an integrator can display it or recompute it.
* `transaction` is a base64 serialized versioned transaction. The requesting wallet is the fee payer. It has the minimum output, oracle report, hook accounts, address lookup tables, compute budget and priority fee already in place, plus any token account creation the wallet needs, which takes a small refundable deposit.

Leave out `wallet` to receive a price with no transaction.

## Signing the returned transaction

This path suits a client that already has its own wallet code. It uses `@solana/web3.js` and any signer with a `signTransaction` method, including a Wallet Standard adapter.

```ts
import { Connection, VersionedTransaction } from "@solana/web3.js";

const connection = new Connection(process.env.SOLANA_RPC_URL!, "confirmed");

const params = new URLSearchParams({
  tokenIn: "USDC",
  tokenOut: "NVDAx",
  amountIn: "2000000000",
  wallet: wallet.publicKey.toBase58(),
});
const quote = await fetch(`https://api.arcter.xyz/v1/quote?${params}`).then((r) => r.json());

const tx = VersionedTransaction.deserialize(Buffer.from(quote.transaction, "base64"));
const signed = await wallet.signTransaction(tx);
const signature = await connection.sendRawTransaction(signed.serialize());

await connection.confirmTransaction({
  signature,
  blockhash: tx.message.recentBlockhash,
  lastValidBlockHeight: quote.lastValidBlockHeight,
});
```

Do not sign blindly. Decode the message, and check that its top-level instructions target the expected program IDs and that the minimum output matches what you showed the user. Until [Releases and addresses](releases-and-addresses.md) publishes real IDs, no address claiming to be Arcter's should be trusted.

## TypeScript SDK

`@arcter/sdk` is the shorter path. It is planned to wrap quoting, the access preflight, transaction checks, submission and fill decoding.

```ts
import { Connection } from "@solana/web3.js";
import { Arcter } from "@arcter/sdk";

const arcter = new Arcter({
  connection: new Connection(process.env.SOLANA_RPC_URL!, "confirmed"),
  signer: walletAdapter, // a connected wallet adapter or Wallet Standard signer
});

const quote = await arcter.quote({
  tokenIn: "USDC",
  tokenOut: "NVDAx",
  amountIn: 2_000_000_000n,
});

console.log(quote.breakdown); // mid, spread, skew, fee, regime

const { signature } = await arcter.swap(quote, { maxSlippageBps: 15 });
```

Releases will follow semantic versioning, each with a public changelog. Typed fill and regime events are planned for anyone consuming the stream.

## Live stream

`wss://api.arcter.xyz/v1/stream` is designed to push fills, regime changes and quote updates as fresh oracle data arrives. The web client has no private feed, so its ticket is planned to listen to this same stream.

## Maker protocol

An approved maker keeps an authenticated WebSocket session open, receives each request for quote, and answers with a message signed by its ed25519 key.

```
-> { "type": "rfq", "id": "...", "tokenIn": "USDC", "tokenOut": "NVDAx", "amountIn": "250000000000" }
<- { "type": "quote", "id": "...", "quote": { ...signed fields... }, "signature": "<BASE64_ED25519_SIGNATURE>" }
```

The chosen quote travels inside the taker's transaction. The native Ed25519 program verifies the signature, `arcter-rfq` confirms that result by instruction introspection, and a per-maker nonce or quote-id PDA blocks replay. The quote must also fall inside the oracle band.

Connecting requires a `MAKER` attestation, described in [Access controls](access-controls.md). There is no penalty for losing an auction and no quoting obligation at launch. A dependable record earns routing priority. [Market maker onboarding](../guides/market-maker-onboarding.md) covers message layout and nonces.

## Keys and limits

Quote endpoints will be open to anyone, with limits per IP address. An authenticated key raises those limits and unlocks the WebSocket. Keys exist so that we can identify an integrator when helping them or handling abuse. They have no effect on price, and anyone can confirm that by reading a Market account and recomputing the vault quote.

## Compatibility promise

Once `/v1` is public, it will not take breaking changes. Retiring anything will come with at least 90 days of notice in the changelog. Each SDK release targets one API version, and program IDs for each release will appear in [Releases and addresses](releases-and-addresses.md).
