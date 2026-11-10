---
description: Current deployment status, how anyone will verify Arcter's programs, and who controls upgrades.
---

# Releases and addresses

{% hint style="danger" %}
No Arcter program is deployed on any Solana cluster, and no Arcter address has been published. Any address presented as Arcter's today is hostile. That covers programs, mints, vaults, airdrops and support accounts. Do not connect, sign or send funds. Report it to security@arcter.xyz.
{% endhint %}

## Current status

| Network | Status, November 2026 |
| --- | --- |
| Solana | Not deployed |

Launch follows the build and test phase of the delivery plan, and only with audited programs. See [Delivery phases](../plan/delivery-phases.md).

## Programs to expect

Six programs will make up each release: `arcter-router`, `arcter-vault`, `arcter-rfq`, `arcter-oracle`, `arcter-access` and `arcter-config`. [Programs](programs.md) explains what each one does. These are names and not identifiers. Only a program ID published through the channels below will mean anything.

## Where addresses will appear

When the time comes, program IDs will be published in this page, on [arcter.xyz](https://arcter.xyz), and in the tagged release record of Arcter's public repository. All three must agree. An ID found anywhere else, including a search result, a social post, a direct message or an explorer label, should be treated as hostile until it matches them.

## Verified builds

Solana stores a compiled program and not its source, so a reader needs a way to link the two. A verified build does that. Source at a tagged commit is compiled inside a fixed container image, which makes the output reproducible, and anyone can then compare a hash of that output with a hash of what is deployed.

The community `solana-verify` tool is the expected way to do this, and Solscan and the Solana Explorer can show verification status. A check by hand would look like this, once an ID exists:

```bash
# See who controls the program and when it last changed
solana program show <PROGRAM_ID>

# Hash the deployed program
solana-verify get-program-hash <PROGRAM_ID>

# Build the tagged source and hash the result
git checkout <RELEASE_TAG>
solana-verify build --library-name <PROGRAM_NAME>
solana-verify get-executable-hash target/deploy/<PROGRAM_NAME>.so
```

The two hashes must match. If they do not, treat that release as unverified, and do not assume the explanation is harmless.

Each release record is planned to list, per program: the program ID, source commit, toolchain version, executable hash, deploying transaction and the audit report that covers it. A job in continuous integration is meant to repeat the comparison at every release.

## Upgrade authority

Programs on Solana are upgradeable by default. The program ID stays the same while one key, the upgrade authority, can replace the code behind it.

That has a direct consequence for users. **Whoever holds that authority could ship code that changes how funds move.** The emergency pause cannot move funds, but an upgrade is a broader power. Arcter treats it as the central trust assumption of the system and plans to constrain it as follows:

* Authority over every program sits with a Squads multisig, and no single person can act alone.
* Upgrades pass through a time lock, which gives users a window to review the change and leave.
* Each proposed upgrade will point to a program buffer on-chain. Anyone can hash that buffer during the window, using `solana-verify get-buffer-hash`, and compare it with the tagged source.
* Signers and thresholds are to be made public.

The time lock duration, the signer set and any eventual decision to remove upgrade authority are all still to be confirmed before launch. You can check the current holder at any time, using the first command above. If it is not the published multisig, stop using the program and write to security@arcter.xyz.

## From tag to release

1. **Freeze.** A version is tagged, reviewed or audited according to what changed, and its source is made public.
2. **Rehearse.** Invariants and a scripted end-to-end run execute against mock mints with real extensions.
3. **Propose.** A buffer holding the build is written on-chain, and the multisig proposes it. A first deployment makes the program itself.
4. **Wait.** The time lock runs, and reviewers can compare hashes.
5. **Configure.** Programs start with no live parameters. Values reach `arcter-config` through the same governed path, so the settings history is complete from the first one.
6. **Publish.** The release record, verification and this page are updated, and only then is the release announced.

## Changes that need no release

A new listing is a call to `create_market`. It adds a Market account and share mint but no code, and parameter changes are handled the same way, through `arcter-config`. [Parameter governance](../accountability/parameter-governance.md) describes those changes. A change to logic does require an upgrade, and takes the route above.

## How clients find their configuration

The SDK and web client are planned to pin program IDs per release, because something has to anchor the chain of trust. Everything else comes from the chain. Market accounts, tiers, oracle settings and token programs are read from `arcter-vault` and `arcter-config`, so no market list ships inside the client. PDAs are derived and not looked up, which means a tampered settings file cannot redirect a client to a fake market without a mismatch showing up.

That protection depends on the pinned ID. A doctored copy of a client could pin a different one, so wallets should still show which programs a transaction touches, and users should read that before signing.
