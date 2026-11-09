---
description: The Solana-specific threats Arcter designs against, how the programs will be tested, and how to report a vulnerability.
---

# Security practices

{% hint style="danger" %}
No audit has finished. No Arcter program is deployed, and everything on this page is a plan or a design rule for code still in development. Treat it as a statement of intent and not as evidence of safety.
{% endhint %}

Arcter's programs will hold other people's assets on Solana. There, a program receives a list of accounts chosen by the caller and must prove that each one is the right account. Most serious losses in this environment start from that fact, so the security plan begins there too.

## Threat model

| Threat | How it goes wrong | Planned defense |
| --- | --- | --- |
| Missing signer check | An instruction trusts an authority account that never actually signed | Anchor `Signer` types on every privileged account, plus negative tests that omit each signature |
| Missing owner check | A program reads data from an account forged by another program | Typed Anchor accounts that verify owner and discriminator. Any raw account read needs a written reason. |
| Account substitution | A caller passes a valid account of the right type but the wrong identity, such as another market's vault or a lookalike mint | Seeds, `has_one` and mint constraints tying every reserve, share mint and oracle account to its Market PDA |
| PDA seed collision | Two logical records derive the same address, often from variable-length seeds joined without a separator | A distinct prefix per account type, fixed-length keys as seeds, and canonical bumps stored at creation |
| Unsafe CPI | A program invokes a caller-supplied program, or lends a PDA signature to one | Fixed program IDs for every CPI, an allowlist of exactly the two token programs, and PDA signing only on known targets |
| Stale state after CPI | Cached account data goes out of date once a called program has changed it | Reload after each CPI, and compute fills from measured balance changes |
| Arithmetic overflow | Rust wraps silently unless told otherwise, and an `as` cast can truncate | `overflow-checks = true` in release builds, checked math, wide intermediates, and no unchecked casts |
| Rounding drift | Repeated small errors move value out of a vault | One rounding direction, always toward the vault, tested over long random sequences |
| Hook account injection | A client adds accounts to a transfer so that a hook runs with values the caller picked | Extra accounts derived from the mint's own validation account, with the hook program read from the mint and never from client input |
| Forged maker signature | An introspection check reads a signature that does not cover the quote in use | A checked instructions sysvar address, and offsets that must point at the Ed25519 instruction's own data, with key and message compared in full |
| Quote replay | A signed quote is used twice | A per-maker nonce or quote-id PDA, plus expiry checked against the Clock sysvar |
| Raw and scaled confusion | Scaled UI Amount makes a displayed figure differ from the raw balance | Vault math on raw units only, with the multiplier read fresh for each quote |

Two further points on transfer hooks. Token-2022 strips signing and write access from the transfer's own accounts before it calls the hook, which limits what a hook can inherit. The runtime also refuses re-entry into a program already on the call stack, apart from direct self-calls. Neither replaces the checks above. A hook program can also be swapped by its authority, so a keeper is planned to watch each mint and pause the market on any change.

## Properties to prove

Each guarantee becomes a property that automated runs try to break. The planned set:

1. **Band safety.** From any reachable state, regime or parameter set, a fill outside the oracle band cannot happen.
2. **Share accounting.** Share value does not depend on the path taken to reach a state, and a swap cannot reduce value per share at the mid it fills against. Mint and burn rounding favors the vault. Market moves can still reduce a depositor's value.
3. **Exit availability.** No regime transition strands funds. Withdrawals continue through `HALTED` where safe. An issuer freeze can block the equity side, and the USDC side stays available.
4. **Inventory limits.** Repeated quoting cannot push inventory past its band.
5. **Conservation.** The trader's payment equals what the vault or maker and the treasury PDA receive.
6. **Unit handling.** Differing mint decimals and multiplier changes stay correct at the smallest and largest permitted sizes.

## Test plan

* Unit tests in LiteSVM or Mollusk, run against real Token-2022 mint configurations and not just simple mocks
* Trident fuzzing of instruction sequences, with the properties above as invariants
* Account substitution tests that replay every instruction with swapped, missing and foreign accounts
* Size and compute measurements for the heaviest routes, including all hook accounts
* Failure drills for stale reports, disagreeing oracles, a price jump above 25% in one update, a mid-session multiplier change, an issuer pause and a cluster restart with its one-hour grace window

Each drill must end in the correct halt. Tests run in continuous integration on every commit, and fuzzing is meant to run continuously.

## External review

We intend to commission two independent reviews before launch, and the first release is planned to contain audited programs only. One review is meant to cover economics: the spread formula, share accounting and band enforcement. The other is meant to cover all program code. No firm has been named here, and neither review has concluded.

Our intent is to publish both reports exactly as delivered. A later change to settlement logic would get its own review. A parameter change would not, which is one reason behavior lives in parameters where possible.

## Bug bounty

A public bounty is planned to open by launch, on a platform still to be chosen. The highest rewards are meant for a break of any listed property. Scope will extend past the programs to the quote service and SDK, because client software that misleads a user into approving a damaging transaction counts as an attack even when every program behaves.

## Keys and privileges

| Holder | What it can do | Limit |
| --- | --- | --- |
| Squads multisig | Change settings and upgrade programs, through a time lock | The largest trust assumption in the system. Code replaced through an upgrade could change how funds move, so [Releases and addresses](releases-and-addresses.md) explains how to watch it. |
| Guardian | Pause trading | Cannot lift a pause and cannot move funds. A stolen key can cause refused quotes, and no direct loss. |
| Multisig, for unpausing | Lift a pause without waiting out the time lock | Requires the multisig threshold |
| Keepers and services | Fire permissionless calls and simulate quotes | Hold no key that can reach funds. A failure costs freshness. |
| Makers | Sign quotes with their own hot keys | Makers carry that risk, and the oracle band caps their worst fill. |

Multisig and guardian keys are planned to live on hardware devices, with signers and thresholds made public.

## Monitoring and response

The on-call signer is to be paged for fills approaching the band edge, disagreement between oracles, skew at its limit, unusual fill patterns and any change in a listed mint's configuration. Every runbook starts by pausing quoting, since a pause stops trading without touching funds.

## Reporting a vulnerability

Write to security@arcter.xyz. A PGP key is planned for that address and should be published before launch. Please do not test against live programs or other people's funds.

We commit to a public post-mortem within 14 days of any incident. Any resulting fix goes through the time lock, citing that post-mortem as its reason. [Commitments](../accountability/commitments.md) lists the promises this work is meant to back.
