---
description: How to prepare a Solana wallet for Arcter, including wallet choice, SOL for fees, token accounts, and RPC endpoints.
---

# Connecting a wallet

Any Arcter session starts with a Solana wallet, either one you already own or an embedded wallet created at sign-in. This page covers both, then explains the two things new Solana users trip over: holding SOL for fees, and token accounts.

{% hint style="warning" %}
Arcter has no public program addresses or mint addresses yet, because no program is deployed. Anyone presenting an Arcter address today is not presenting a real one. Official addresses will appear only on [Releases and addresses](../system/releases-and-addresses.md).
{% endhint %}

## Pick a wallet

| Wallet | Type | How it connects | Best suited to |
| --- | --- | --- | --- |
| Phantom | Self-managed | Wallet Standard adapter | People with an existing Solana wallet |
| Solflare | Self-managed | Wallet Standard adapter | People with an existing Solana wallet |
| Backpack | Self-managed | Wallet Standard adapter | People with an existing Solana wallet |
| Privy embedded wallet | Embedded | Email or passkey sign-in | First-time users without a wallet |

With a self-managed wallet, you hold the recovery phrase and approve each transaction in the wallet's own window. Privy creates an embedded Solana wallet during sign-in, so a newcomer needs no extension. Sponsoring the fee on a first trade through a relayer is an option under review, not a commitment.

Arcter never asks for a recovery phrase or private key. Treat any page that does as hostile.

## Fund the wallet with SOL

Every Solana transaction needs SOL, whatever token is being traded. Holding only USDC or an equity token is not enough.

* A signature costs 5,000 lamports (0.000005 SOL).
* Priority fees are added on top and rise when the network is busy.
* Creating a token account needs a small refundable deposit, covered below.

A small SOL balance covers many transactions. If a wallet runs out, transactions fail before anything is traded.

## Token accounts

On Solana, a wallet does not hold tokens directly. Each mint a wallet owns sits in its own token account, normally the associated token account (ATA) derived from the wallet and the mint.

An ATA must exist before it can receive anything. When a swap pays out an asset you have never held, the transaction creates the account first and takes the rent-exempt deposit from your SOL balance. Closing an empty token account later returns that deposit.

Equity tokens add two wrinkles:

* **Frozen by default.** Under the default account state extension, a new token account can start frozen. It then cannot receive tokens until the issuer clears it. Arcter's pre-trade simulation is designed to report this as a plain reason rather than a failed transaction.
* **Scaled balances.** With Scaled UI Amount, the figure on screen is the raw balance times a multiplier. Not every wallet renders this correctly, and at least one bug report describes an xStock shown at the wrong scale. If your wallet and Arcter disagree, compare against the raw amount on an explorer.

What each signature authorizes also differs from what some users expect. Arcter requests no standing approval. A signature covers one transaction, which fails or settles as a whole. Your wallet's preview shows the accounts and balance changes, so read it before approving.

## RPC endpoints

An RPC endpoint is the server your wallet uses to read the network and submit transactions. Wallets ship with their own defaults, so most users never change this. Developers and maker integrations will want to know the public option.

Solana runs a free public endpoint, listed in its own documentation. It is rate limited, and suits occasional reads and small experiments, and anything steady needs a private RPC provider. RPC failures and congestion are the common everyday causes of a stuck transaction, so a dropped request usually says more about the connection than about the network.

Solscan and the Solana Explorer both show transactions and accounts, and each fill will link to them.

## If a connection misbehaves

| Symptom | Likely cause | What to do |
| --- | --- | --- |
| Transaction expired | Its blockhash aged out after about 150 slots | Ask for a fresh quote and sign again |
| Insufficient funds with a token balance showing | Too little SOL for fees or an account deposit | Add SOL |
| Requests failing or timing out | Rate limits on a public endpoint | Switch to another RPC provider |
| Token account frozen | The mint's default account state | Check the issuer's clearance process |
| Balance looks wrong after a split or dividend | Scaled UI Amount display | Compare the raw amount on an explorer |

## Technical reference

| Standard or feature | Role at Arcter |
| --- | --- |
| SPL Token and Token-2022 | Both supported, since mints may use either program |
| Associated token accounts | Default location for user balances |
| Plain SPL share mint | One per vault, redeemable pro rata |
| Ed25519 program | Verifies maker quote signatures |
| Versioned transactions and address lookup tables | Fit long account lists within 1,232 bytes |
| Wallet Standard | Connects Phantom, Solflare and Backpack |
| Attestation PDA | Records eligibility for each wallet |

Decimals differ across mints, so Arcter reads them from each mint and never assumes a value.

Program timing relies on the Clock sysvar. Quote expiry, data age limits, grace windows and the time lock are all planned to read its unix timestamp. Slot numbers serve as ordering references and are not treated as a precise measure of time, because the 400 ms slot length is approximate.

The Chainlink products Arcter intends to use are described in [Partners and integrations](partners-and-integrations.md).
