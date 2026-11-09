---
description: The off-chain services planned around Arcter's programs, what each does, and what happens to the venue when one stops.
---

# Supporting services

Arcter's programs are planned to make every decision that affects money. The services around them exist for speed and convenience. Each one is designed so that its failure costs comfort, and possibly some access to the best prices, but never custody.

{% hint style="info" %}
These services are planned alongside programs that are not yet deployed. Descriptions here state intended behavior.
{% endhint %}

## Summary

| Service | Runs on | Holds keys? | Effect of an outage |
| --- | --- | --- | --- |
| Quote service | Stateless servers | No | Previews and maker auctions stop |
| Keepers | Scheduled workers | A fee-paying wallet only | State updates arrive late |
| Read model | Yellowstone gRPC into Supabase | No | Ledger and history go stale |
| Fee relayer, if adopted | Server wallet | A fee payer key only | Sponsored first trades stop |

## Quote service

For any swap, the service answers one question: what would this cost now?

It works in four steps. The service first simulates the route using current oracle and Market data. Next it sends a request to connected makers, and it accepts signed replies for a few hundred milliseconds. Finally it returns the best outcome with mid, spread, skew and fee itemized. It also resolves the hook accounts a transaction will need and flags an access problem before the user signs. Both the app and the public API read from it.

You do not need to trust this service, for three reasons.

* It has no wallet keys or balances, so there is nothing to steal.
* Pricing is recalculated on-chain at fill time. A wrong preview collides with the signed minimum output and the transaction fails.
* It cannot steer a trade away from the vault. If the vault beats every maker quote, the router uses the vault.

### What happens without it

Vault swaps remain possible for anyone able to build the transaction, at the same on-chain price. That person takes on the service's chores, including fetching a fresh signed price report, listing hook accounts, and fitting everything inside the size limit.

The report needs care. Chainlink's Data Streams access terms on Solana are unconfirmed, so a user with no entitlement may be unable to fetch a primary report independently. How far this limits a do-it-yourself swap has to be settled in discovery. Withdrawals are simpler, as a pro rata redemption needs no price.

## Keepers

Keepers send calls that are open to everyone: regime transitions, halts when data goes stale, and the end of the post-restart grace window. They also watch each listed mint for configuration changes and run heartbeat checks.

A late keeper cannot produce a mispriced fill, because a swap derives the regime in-line. Signals can only tighten a market. The worst outcome is a ledger page showing a state change a little late.

A keeper wallet holds SOL for fees and has no privilege in any Arcter program. The plan is to publish reference keeper code, so that anyone can run one without permission.

## Read model

A Yellowstone gRPC stream, built on Solana's Geyser plugin interface, follows program activity and writes it to Supabase. Helius RPC is an option alongside it. From that store come the [ledger](../guides/reading-the-explorer.md), [fill reports](../accountability/fill-reports.md), market statistics and portfolio history.

Every displayed figure is meant to link to its underlying transaction on Solscan or the Solana Explorer. A skeptical reader can rebuild any number from the chain and skip the read model entirely.

## Transaction inclusion

A correct transaction still has to land. The app sets priority fees dynamically from recent network conditions, and Jito and stake-weighted paths are being evaluated during testing. A blockhash expires after about 150 slots, so a transaction that fails to land simply dies and can be requoted. It cannot sit around to execute later at an old price.

RPC capacity falls under the same heading. The public Solana RPC endpoint is rate limited, so production services need private providers, with at least one fallback.

## Fee sponsorship

Paying a new user's first network fee through a relayer is under review. The relayer would act only as fee payer. The user still signs, and the relayer key could not move the user's funds. If this ships, an outage means the user supplies their own SOL.

## Maker connections

Makers run their own gateways and hold their own signing keys. They connect to the quote service over a WebSocket feed described in [Developer access](developer-access.md), receive each request, and answer with a price signed by ed25519. Arcter forwards requests and ranks replies. A maker who disconnects drops out of the auction and nothing else changes.

## Monitoring

Monitoring is planned to rely on public state, so anyone can reproduce it. The watch list:

* oracle age, and divergence between Chainlink and Pyth,
* vault skew relative to limits,
* active halts and the current regime per market,
* mint configuration changes,
* cluster health, restarts and congestion,
* how close fills land to the oracle band,
* execution quality trends.

Alerts reach whichever guardian signer is on call. There is no private telemetry behind these signals, by design.

## Running your own

The intention is to publish the code for these services, so a third party can host a quote service, keeper and indexer against the RPC provider of their choice. Repository links will appear in [Web client](web-client.md) and [Developer access](developer-access.md) once they exist. Until then, treat self-hosting as a goal.
