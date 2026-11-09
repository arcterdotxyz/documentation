---
description: Who may trade, deposit or quote on Arcter, which layer enforces each rule, and what stays off Solana.
---

# Access controls

{% hint style="info" %}
This page describes planned behavior. No Arcter program is deployed, so no attestation exists yet and no check below is running yet.
{% endhint %}

Arcter does not decide who may hold a tokenized stock. The issuer does, and those rules differ by product and by channel. Arcter's job is to read them correctly, apply the strictest reading for each market, and tell a user why a trade would fail before that user signs anything.

## Three layers

| Layer | Enforced by | Binds whom |
| --- | --- | --- |
| Mint rules | The issuer, through Token-2022 extensions such as a transfer hook or default account state | Every transfer of that token, on any venue |
| Market rules | `arcter-access`, through an Attestation PDA, when a market requires one | Anyone reaching Arcter's programs, through any client |
| Client rules | The Arcter web client, through location checks and sanctions screening | Users of the hosted app only |

The bottom layer is the weakest. Somebody calling a program directly never passes through the web client, so app checks are a courtesy to the user and not a boundary. Issuer registries remain the source of truth. A market that must be permissioned has to use the top two layers.

## The mint comes first

Token-2022 lets an issuer put rules on the token itself. A transfer hook can run the issuer's program on each transfer, and default account state can leave a new token account frozen until the issuer clears it. Reports say Ondo plans to use hooks for eligibility, so each mint must be inspected rather than assumed.

The consequences for Arcter:

* Before quoting, the router simulates the whole trade, hook included. A user who would be refused gets a readable reason and no failed transaction.
* Vault token accounts may need issuer clearance before a market can open. Whether hooks accept program-owned vault accounts is an open question for each mint, and a market cannot list until the answer is known.
* Both sets of rules apply together. Passing `arcter-access` never overrides a refusal from the mint.

See [Issuer dependence](../safeguards/issuer-dependence.md) for what this control means for holders.

## Roles

An attestation grants one or more of these roles:

| Role | Meant for |
| --- | --- |
| `TRADER` | People and companies that passed screening and sit outside restricted jurisdictions |
| `LP` | Vault depositors. Vaults carry inventory risk, so this role is expected to go to professional clients first. |
| `MAKER` | Professional firms approved to answer requests for quotes |
| `RELAYER` | Reserved for software acting for others. No planned instruction checks it. |

A sponsored fee payer does not need `RELAYER`. Paying the network fee gives a relayer no power over the user's funds, because the user still signs the transaction that moves them.

## What each instruction checks

| Action | Check |
| --- | --- |
| Swap on a permissioned market | `TRADER` for the signing wallet, and for a separate recipient if there is one |
| RFQ settlement | `TRADER` for the taker where the market requires it, and `MAKER` for the wallet whose ed25519 key signed the quote |
| Vault deposit | `LP` for the depositing wallet |
| Vault withdrawal | None. Redeeming your shares must never depend on a renewed credential. |
| Share transfer | None at program level. See the note below. |

{% hint style="warning" %}
Vault shares are a plain SPL mint, which has no hook that could stop a transfer. Shares can therefore move to a wallet without `LP`. Arcter gates entry at deposit, and a recipient can still redeem. If a future vault needs gated shares, it would need a different share mint design.
{% endhint %}

Withdrawing a token from a vault still passes through that token's own rules. An issuer freeze can block the equity side of an exit, while the unaffected USDC side continues.

## How a program reads an attestation

A gated instruction lists the wallet's Attestation PDA among its accounts. The program then confirms four things:

1. The address derives from the expected wallet under `arcter-access`, and that program owns the account.
2. The attestation issuer recorded on the account is on the approved list in `arcter-config`.
3. The attestation has not been revoked.
4. Its expiry is later than the unix timestamp in the Clock sysvar.

A failure aborts the transaction, and no state changes.

Reading is free. Any client, explorer or aggregator can derive the PDA and fetch the account over RPC, without sending a transaction or holding a key. The web client does this before it asks for a signature.

## The attestation record

Screening itself happens off-chain. Once a provider finishes identity, sanctions and residency checks, an approved attestation issuer writes the result to one PDA per wallet. This role is separate from the token issuer.

| Field | Contents |
| --- | --- |
| Wallet | The Solana address the record covers |
| Roles | The roles granted |
| Jurisdiction class | A class and not a country, unless the user opts to disclose the country |
| Investor class | Present where a market needs it, for example professional or qualified |
| Attestation issuer | The approved key that wrote the record |
| Expiry | A unix timestamp. Periodic re-screening extends it. |

Names, documents and identity numbers never reach the chain, and Arcter does not collect identity documents itself. The account is still public, so anyone can see that a given wallet holds a given class. Users who want more privacy should understand that before onboarding.

Revocation works from the first transaction after the attestation issuer's revoke lands. The funding of each account's rent-exempt deposit is still to be confirmed.

A route using private proofs, which would show eligibility without publishing the class, is a question for discovery. No such provider has been selected for Solana.

## Who can change the policy

The approved attestation issuers and accepted schemas are values in `arcter-config`. Editing them passes through the Squads multisig and its time lock, so a stricter policy or a new provider becomes a recorded change and needs no new program. Replacing the policy logic is different. That needs a program upgrade, which follows the path in [Releases and addresses](releases-and-addresses.md).

## Restricted jurisdictions

Token issuers publish the lists, and Arcter adopts the strictest applicable one for each market. As examples, Kraken's xStocks pages exclude the US, Canada, the UK and Australia, plus prohibited regions, and an EEA client answers a questionnaire. Ondo bars US persons and limits UK, Swiss and some EEA access to qualified or professional clients. These lists vary by channel and can change.

Exclusion relies on separate measures:

* attestation issuers refuse any wallet in a restricted place
* each user states their location during onboarding
* the web client blocks restricted locations by IP address
* the web client screens wallets against sanctions lists

Arcter does not serve US persons. Client checks can be bypassed by a determined user, which is why they sit alongside mint and market rules and never replace them. For the wider picture, see [Regulatory position](../legal/regulatory-position.md). Later asset types may need other rules, covered in [Expansion](../catalog/expansion.md).
