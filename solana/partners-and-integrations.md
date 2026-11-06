---
description: Ecosystem projects Arcter depends on or plans to integrate with, what each would do, and the current status of that relationship.
---

# Partners and integrations

{% hint style="warning" %}
Arcter has no formal partners today. Despite this page's title, every project named here is an independent third party. Naming one means Arcter depends on its public product or plans to integrate with it. It does not imply an agreement, an endorsement or a commercial relationship.
{% endhint %}

Arcter is pre-launch, so even the planned integrations below are unbuilt or unproven. Anything marked as a dependency is already part of the planned stack. Anything marked as a target is a project Arcter hopes to reach once programs exist.

## At a glance

| Project | Function for Arcter | Status |
| --- | --- | --- |
| Chainlink | Primary price source | Planned; coverage and access to be confirmed |
| Pyth | Independent price cross-check | Planned |
| Circle (USDC) | Settlement asset | Dependency on the public token |
| xStocks | Listed equity tokens | Planned listing, subject to review |
| Ondo Global Markets | Listed equity tokens | Planned listing, subject to review |
| Jupiter and other aggregators | Order flow | Target |
| Raydium | Reference venue and arbitrage path | No integration needed |
| Kamino | Neighboring lending venue | No integration planned yet |
| Trading firms | Maker quotes | Target; none signed |
| Phantom, Solflare, Backpack | User wallets | Planned through Wallet Standard |
| Privy | Sign-in and embedded wallets | Dependency |
| Supabase | Read model storage | Dependency |
| Squads | Multisig and time lock | Planned |
| Jito | Transaction inclusion path | Under evaluation |
| Yellowstone gRPC, Helius | Chain data | Yellowstone planned; Helius optional |
| Solscan, Solana Explorer | Public verification | Used as public tools |

## Pricing

### Chainlink

Data Streams is the intended primary price source. A signed report travels inside the transaction and Chainlink's verifier program checks it. Chainlink also publicly offers xStocks Data Streams, with sub-second prices and corporate action verification, available since February 2026. Proof of Reserve and CCIP are also offered for xStocks.

Three points remain open: report schema, entitlement, and coverage per asset on Solana. Arcter does not claim Data Streams covers every candidate, and an asset that fails this check does not list. [Price data defenses](../safeguards/price-data-defenses.md) has the details.

### Pyth

Pull updates from Pyth provide the second opinion. When Pyth and Chainlink deviate, the market tightens to `HALTED`. Pyth also runs 24/7 equity indices, launched in 2026, and low-latency Lazer data. Neither is a committed input yet.

## Assets

### Circle and USDC

USDC is issued by Circle and native on Solana, where it is Arcter's settlement asset. Arcter has no agreement with Circle, and it uses the public token only. Circle can freeze USDC in any account, which the [risk model](../safeguards/risk-model.md) covers.

### Equity issuers

xStocks are issued by Backed Assets (JE) Limited, and Kraken distributes them. Ondo Global Markets is the other major catalog. Arcter consumes these tokens as public mints and has no issuer agreement. That matters, because clearance for vault accounts under default account state may need the issuer's cooperation. Whether issuers will grant it is an open question. See [Tokenized equities](../catalog/tokenized-equities.md) and [Issuer dependence](../safeguards/issuer-dependence.md).

## Order flow

### Jupiter and other aggregators

Aggregators reach users Arcter never would alone. Once programs are live, the plan is to register with Jupiter and others, so a competitive anchored quote can win routed orders on its merits.

### Other pools and proprietary AMMs

One report put Raydium's xStocks volume at $1.63B in Q2 2026, and proprietary AMMs make up a major part of Solana spot flow. They are competitors and also useful counterparts. Arbitrage between those venues and Arcter pulls pool prices toward the reference. An arbitrage fill moves a vault back toward target inventory, and skew pricing pays the arbitrageur for that work.

### Trading firms

Arcter intends to sign two or three makers already active in tokenized equities. No maker has been signed. [Market maker onboarding](../guides/market-maker-onboarding.md) describes the planned process.

### Kamino

Kamino held about $20.7M of xStocks collateral on 13 September 2026 and opened a PAXG market on 27 July 2026. Holders who move between borrowing against a position and trading it need a swap venue, which makes Kamino's users natural Arcter customers. No integration is scheduled.

## Wallets and sign-in

Phantom, Solflare and Backpack connect through Wallet Standard adapters. Inclusion in any wallet's own directory or swap screen is a later goal for the integration phase. Privy handles email and passkey onboarding and creates embedded Solana wallets. See [Connecting a wallet](connecting-a-wallet.md).

## Infrastructure

* **Squads.** Planned holder of program upgrade authority and settings control, through a multisig with a time lock.
* **Yellowstone gRPC and Geyser.** Planned stream into the read model, with Helius RPC as an option.
* **Supabase.** Storage behind that read model.
* **Jito.** Bundles and tips are being evaluated during testing, alongside stake-weighted paths.
* **Solscan and the Solana Explorer.** Public tools that let anyone open a fill or account. Arcter needs no arrangement with them.

Details are in [Supporting services](../system/supporting-services.md).

## Screening

Sanctions screening runs at the app layer. No vendor is named here, because none has been confirmed. Attestations under `arcter-access` are the on-chain record, and issuer registries stay the source of truth. See [Access controls](../system/access-controls.md).

## Ecosystem programs

Solana Foundation programs and Colosseum hackathons are natural early channels for a project at this stage. Arcter has not been accepted into, or funded by, any of them.

## How this page changes

This page will call a project a formal partner only once a signed agreement exists. Until then, read every entry as a plan or a technical dependency.
