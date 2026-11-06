---
description: The Solana mechanics that shape Arcter, from slots and fees to accounts and Token Extensions, with dated facts as of 7 November 2026.
---

# Solana network primer

Arcter prices assets against an outside reference, so the network underneath has to carry a signed price into a transaction and settle it within a moment. This page explains how Solana does that, where its limits sit, and which of those limits Arcter designs around. All network facts are stated as of 7 November 2026.

{% hint style="info" %}
Arcter is pre-launch. Its programs are in development, none is deployed, and no audit has been completed.
{% endhint %}

## Consensus and block production

Solana is a Proof of Stake layer 1 network, and its native asset, SOL, pays for every transaction. Time on the network is divided into slots of about 400 ms. Each slot belongs to a leader, a validator picked in advance from a published schedule. Leadership keeps rotating, so no central operator decides the order of transactions.

The consensus layer changed recently. Votor, a component of the Alpenglow upgrade, went live on 21 October 2026 at epoch 1064. It replaced TowerBFT and targets finality of about 150 ms.

| Alpenglow component | Status on 7 Nov 2026 | Effect |
| --- | --- | --- |
| Votor (consensus) | Live since 21 Oct 2026, epoch 1064 | Targets finality near 150 ms |
| Rotor (block propagation) | Not scheduled | Turbine continues to propagate blocks |
| Slot interval | Unchanged | Still about 400 ms |

Three validator clients run the network: Agave, Firedancer and Frankendancer.

## Fees

Fees are paid in SOL and have two parts.

* **Base fee.** Each signature costs 5,000 lamports, which is 0.000005 SOL.
* **Priority fee.** This optional bid is priced per compute unit and improves the odds of inclusion when the network is busy.

Stake-weighted quality of service and Jito bundles with tips are two further inclusion paths in common use. Arcter's handling of them is covered in [Supporting services](../system/supporting-services.md).

A third cost appears when an account is created. Accounts must hold a rent-exempt minimum balance, so opening a token account takes a small deposit. That deposit is refundable. [Connecting a wallet](connecting-a-wallet.md) shows when it applies.

## Transaction limits

Four limits decide what fits in one Arcter swap.

| Limit | Value | Design consequence |
| --- | --- | --- |
| Transaction size | 1,232 bytes | Oracle reports, signatures and hook accounts compete for the same space |
| Account list | Declared up front | Every account a route touches must be known before signing |
| Compute | A compute unit budget per transaction, with default limits | Budgets are measured route by route |
| Blockhash age | Expires after about 150 slots, roughly a minute or a little more | Each ticket gets a hard outer deadline |

Versioned transactions and address lookup tables exist for larger account lists, and Arcter's router depends on both. Whether the heaviest routes fit, with all hook accounts included, is still an open discovery question.

The declared account list also has a benefit. Sealevel, Solana's runtime, uses those lists to run transactions that touch different accounts in parallel.

## The account model

Solana keeps programs and data apart. A program is stateless executable code, and everything it remembers lives in separate accounts passed to it by the transaction.

| Concept | Meaning | Where Arcter uses it |
| --- | --- | --- |
| Account | A record with an owner program and a balance of lamports | Market state, attestations, settings |
| Program derived address (PDA) | An address derived from seeds, which only its program can sign for | One Market account per asset, plus the treasury |
| Cross-program invocation (CPI) | One program calling another within a transaction | Swap instructions reaching the vault, oracle and access programs |
| Signer and owner checks | Explicit tests that an account is who it claims to be | Required on every instruction, because accounts are supplied by the caller |
| Clock sysvar | The network's slot and unix timestamp | Expiry, data age, grace windows and the time lock |

Token permissions work differently from what many readers expect. There are no standing approvals in Arcter's design. Funds move only under the user's signature on that specific transaction, so an old allowance cannot be drained later.

The full account layout is in [Programs](../system/programs.md).

## Token programs and Token Extensions

Tokens on Solana come in two forms: mints under the classic SPL Token program, and mints under Token-2022, which adds optional extensions configured on the mint. A venue can read those settings directly, so an issuer's rules are visible on-chain.

Equity tokens lean heavily on them. xStocks are Token-2022 mints, and explorer inspection shows the extensions below configured. Some may be set up for future use and sit idle today.

| Extension | What it does | Why a venue cares |
| --- | --- | --- |
| Scaled UI Amount | Applies a multiplier for splits and dividends while raw balances stay put | Displayed and raw units differ, so mixing them misprices a fill |
| Transfer hook | Runs issuer logic on each transfer and may need extra accounts | Those accounts must fit in the transaction |
| Default account state | Can create new token accounts frozen | A vault or wallet account may need clearance first |
| Permanent delegate | Lets the issuer move tokens from any account | The reach extends to vault accounts |
| Pausable | Lets the issuer stop transfers | Quotes must stop too |
| Metadata pointer | Points to the token's metadata | Used for display and listing checks |
| Confidential transfer | Supports encrypted balances and amounts | Must be reviewed per mint before listing |

Arcter's programs support both token programs, since USDC uses the classic one and equity tokens use Token-2022.

## Reliability

The last cluster-wide halt happened on 6 February 2024 and came from a validator bug. Earlier halts occurred between 2021 and 2023. Solana reported 1,000 days without a network-wide outage on about 2 November 2026.

That record has limits as evidence. Uptime is measured at cluster level, so it says nothing about an individual user's RPC connection. A restart would be coordinated among validators and could arrive while a position is open. Day to day, the failures people actually meet are congestion and RPC errors. See [Network disruptions](../safeguards/network-disruptions.md) for the planned response.

## Ordering and MEV

No single party orders Solana's transactions, but block producers do see them. Sandwich attacks exist, and Jito's block engine is widely used. Users get the best protection from a tight minimum output and a short quote window, and Arcter's ticket is built around both.

## What Solana leaves to Arcter

The network supplies speed, low fees, open deployment and readable mints. It does not solve these four problems, so Arcter has to.

1. **Eligibility.** Rules differ per issuer, and a hook may or may not enforce them. Arcter's approach is in [Access controls](../system/access-controls.md).
2. **Closed markets.** Nothing on the network prices the gap between Friday's close and Monday's open. [Session regimes](../execution/session-regimes.md) does that with explicit multipliers.
3. **Restarts.** Programs resume with stale prices unless told otherwise, so markets hold `HALTED` for a one-hour grace window after a cluster restart.
4. **Asset-aware venues.** General-purpose pools treat a stock like any other token. The reasoning behind a dedicated venue is in [The Solana thesis](../start/solana-thesis.md).
