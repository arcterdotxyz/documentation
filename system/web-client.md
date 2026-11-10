---
description: The Arcter web app at arcter.xyz/platform, covering its screens, how it gets its data, and the Solana wallet behavior it is moving to.
---

# Web client

{% hint style="warning" %}
The app is being migrated to Solana. Its screens and pricing math exist today, but trading and deposits are switched off, figures may be sample data, and the Solana wallet flow below is the target state. No Arcter program is deployed.
{% endhint %}

[arcter.xyz/platform](https://arcter.xyz/platform) is a Next.js 16 app built with React 19, TypeScript and Tailwind 4. Its job is narrow: put the venue's math where a person can read it. It holds no keys and no funds, and it is meant to have no privileged route to any program. Once released, the [developer surface](developer-access.md) is what it should use, exactly like an outside integrator.

## Screens

A left rail holds six destinations in three groups, along with the current session and a UTC clock. Below the large breakpoint, the rail collapses into a drawer.

| Group | Screen | Route | Contents | Sign-in |
| --- | --- | --- | --- | --- |
| Venue | Overview | `/platform` | Vault value, volume, regime scaling, listings by tier, latest trades | No |
| Venue | Trade | `/platform/swap` | The ticket, which can open on one market through `?market=` | To trade |
| Venue | Vaults | `/platform/liquidity` | Vault list, inventory position and value by listing | To deposit |
| Personal | Holdings | `/platform/portfolio` | The wallet's trades, role status and positions | Yes |
| Personal | Account | `/platform/settings` | Display name, wallet, email and access roles | Yes |
| Open record | Ledger | `/platform/explorer` | Itemized trades, parameters in force, change history | No |

A public record should stay public, so the Ledger and venue figures open without a wallet. The [explorer guide](../guides/reading-the-explorer.md) explains how to read them.

## What works today

| Area | In the code now | Target |
| --- | --- | --- |
| Quotes | The ticket runs the vault formula in the browser | Quotes come from the service and settle on `arcter-router` |
| Session | Derived locally from a UTC clock, refreshed every 30 seconds | Taken from oracle market status, shown the same way |
| Market data | Read from Supabase, falling back to a built-in catalog | A read model fed by a Yellowstone gRPC stream |
| Trade button | Disabled after sign-in, reading "Trading is not live" | Signs and sends a versioned transaction |
| Deposits | Disabled, reading "Deposits not open" | Live with release 1.1 |
| Balances | Empty panel until the indexer tracks them | Scaled balances and cost basis |

The app labels its sources honestly. If the read model is empty or unreachable, the built-in catalog keeps pages working, and they then say "Built-in sample figures" or describe the data as a sample. Do not read those numbers as real market activity.

## Anatomy of the ticket

The ticket shows [the quote math](../execution/quote-math.md) as separate lines, and the header carries a regime badge such as `CLOSED x3.0`.

| Line | Meaning |
| --- | --- |
| Oracle mid | Reference price per whole token, in USDC, after the guards |
| Half-spread | Tier base, scaled by the regime, with inventory skew shown in bps |
| Venue fee | Listed apart from the spread, in USDC and bps |
| Least you will get | The minimum output after the chosen price tolerance |

Tolerance options are 0.10%, 0.15%, 0.30% and 0.50%, with 0.15% as the default.

Beside the ticket, panels show the tier, the half-spread in force, the largest trade allowed now, and how a fill would settle. On phones they stack beneath it. The ticket offers no quote if a request exceeds the clip or would land outside the oracle band. When US exchanges are shut, a plain notice explains the wider spread and smaller clip, and points out that waiting will cost less.

Planned additions are a countdown to expiry for maker quotes, one itemized line per leg on two-leg swaps, and a receipt linking to Solscan or the Solana Explorer.

## Target wallet behavior

None of this ships yet.

**Connecting.** Two routes are planned. People who prefer email or a passkey sign in through Privy and receive an embedded Solana wallet. People with a wallet connect Phantom, Solflare or Backpack through Wallet Standard. [Connecting a wallet](../solana/connecting-a-wallet.md) will walk through both.

**Before signing.** The app checks the wallet's access state and simulates the trade, including any transfer hook. An ineligible user reads the relevant rule and never meets a failed transaction. See [Access controls](access-controls.md).

**Signing.** A trade is one versioned transaction with one signature. No standing approval is created, and nothing can be drawn from the wallet later. The transaction carries the minimum output, so a worse result settles nothing.

**Costs.** The user pays a base network fee of 5,000 lamports per signature, plus a priority fee set from current conditions. A first receipt of a token may also require a token account, which takes a small refundable deposit. The wallet therefore needs some SOL, unless fee sponsorship applies. Sponsored fees on a first trade are an option under review.

**Expiry and failure.** A blockhash lasts about 150 slots, roughly a minute or a little more. If a transaction lapses or an RPC endpoint fails, the app should say so and offer a new quote. It should never leave a spinner running.

## Holdings target

Today's screen shows the wallet's trades and role status. The planned version adds:

* balances converted with the mint's Scaled UI Amount multiplier, read fresh, alongside raw units
* marks from the same oracles the vaults use
* cost basis and realized PnL, calculated from the wallet's on-chain history
* vault positions with value per share, spread earned and each vault's inventory mix

## Where data comes from

Public venue reads go straight from the browser to Supabase with an anonymous key. This covers markets, latest prices, vault snapshots, fills and the change log.

Account operations take another route. The browser posts to `/api/platform/db` with a Privy token, and the server verifies that token before acting as that user. Very little can be written there: a user's own profile. Attestation records can only be read. The name you enter is visible only to you, and the public Ledger shows addresses only.

Trades are not stored by the app. They happen on Solana, and the Ledger is a convenience. If the two ever disagree, trust the chain.

## Interface standards

The [principles](../start/principles.md) ask for calm, exact screens. In practice:

* Figures use tabular numerals, so columns line up.
* Trade times appear in relative form, and Holdings adds the exact time on hover.
* Empty and error states say what happened, in plain words, and most offer a next step.
* The app avoids manufactured urgency and hidden fees.
* Fallback is announced. If the quote service fails, the target behavior is vault-only quotes read over RPC, with a notice saying so.

## Running your own copy

Publication of the source is planned. Until the migration lands, a local copy shows the same screens with no trading. It needs Node 24 or later.

1. Set `NEXT_PUBLIC_PRIVY_APP_ID`, `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` in `.env.local`. The server route also needs `PRIVY_APP_SECRET` and `SUPABASE_SERVICE_ROLE_KEY`.
2. Load `supabase/schema.sql`, followed by `supabase/seed.sql`.
3. Run `npm run dev`.

Any missing public key produces a setup screen naming it. After the migration, a self-hosted copy will also need a Solana RPC endpoint of your choosing, and onboarding will need an attestation issuer approved in `arcter-config`.
