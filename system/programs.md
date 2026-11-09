---
description: The Arcter program set, with account layouts, PDA seeds, instructions, events and privileged roles. The code is written and tested, and none of it is deployed yet.
---

# Programs

{% hint style="warning" %}
These programs are written in Rust with Anchor and tested in a Solana VM, but they are not deployed yet and no audit has been completed. There are no live addresses to publish. In the current build, a publisher wallet posts test prices in place of verified oracle reports, and mints with transfer hooks are refused at listing. Names and signatures may still change.
{% endhint %}

Six programs divide the work, and each has one narrow job. Trading logic stays in code. Values that may need tuning stay in accounts controlled by `arcter-config`.

## Program set

| Program | Responsibility | Adjustable through `arcter-config` |
| --- | --- | --- |
| `arcter-router` | Sole trading entry point. Checks access, picks the lane, composes two-leg swaps, and enforces the band, expiry and signed minimum output | Route limits |
| `arcter-vault` | Keeps one Market account per asset with its reserves and share mint. Quotes around the mid, and mints or burns shares. `create_market` adds a listing with no new code | Spread, skew, clip and cap values per market |
| `arcter-rfq` | Confirms a maker's ed25519 signature through instruction introspection, checks expiry and replay, then settles atomically | Maker approvals |
| `arcter-oracle` | Normalizes Chainlink reports, cross-checks Pyth, applies age and deviation limits, and derives the regime | Feed configuration per asset |
| `arcter-access` | Records one Attestation PDA per wallet and answers access questions | Accepted attestation issuers |
| `arcter-config` | Stores parameters, guardian key and the treasury PDA, under a Squads multisig with a time lock | Everything here, after the delay |

No separate guardian program exists. The guardian is a key recorded in the Config account.

## Accounts and PDA seeds

Every account below is a PDA owned by the listed program, except the mints and token accounts, which belong to a token program.

| Account | Program | Draft seeds | Holds |
| --- | --- | --- | --- |
| Config | `arcter-config` | `["config"]` | Parameters, guardian key, pause flags |
| Treasury | `arcter-config` | `["treasury"]` | Authority over protocol fee balances |
| Queued change | `arcter-config` | `["change", change_id]` | Proposed value, earliest execution time, reason hash |
| Market | `arcter-vault` | `["market", asset_mint]` | Mints, spread and skew settings, recorded mint configuration, last regime |
| Asset reserve | `arcter-vault` | `["reserve", market, asset_mint]` | Token account for the asset side |
| USDC reserve | `arcter-vault` | `["reserve", market, usdc_mint]` | Token account for the USDC side |
| Share mint | `arcter-vault` | `["shares", market]` | Plain SPL mint for depositor shares |
| Feed config | `arcter-oracle` | `["feed", asset_mint]` | Report identifiers, age and deviation limits |
| Attestation | `arcter-access` | `["attestation", wallet]` | Issuer, expiry and allowed markets |
| Maker | `arcter-rfq` | `["maker", maker_key]` | Approval status and quote signing key |
| Maker inventory | `arcter-rfq` | `["inventory", maker_key, mint]` | Prefunded maker token balance, withdrawable only by the maker and drawn on only by a valid quote (planned) |
| Quote record | `arcter-rfq` | `["quote", maker_key, quote_id]` | Marks a quote as used, which blocks replay |

Seed choices have consequences worth checking in review. Market seeds allow one market per asset mint. Reserves are program-controlled accounts, so a mint's hook and default account state must both accept them, and that remains an open question for each issuer. Quote records cost rent, so a reclaim path for expired ones is needed.

## Main instructions

Integrators would mostly call these. Solana programs have no free read calls, so previews come from simulating the transaction.

```rust
// arcter-router
pub fn swap_exact_in(ctx: Context<SwapExactIn>, args: SwapArgs) -> Result<()>;

pub struct SwapArgs {
    pub buy_asset: bool,         // true pays USDC for the asset
    pub amount_in: u64,          // raw units
    pub min_amount_out: u64,     // raw units
    pub expires_at: i64,         // unix seconds, read against the Clock sysvar
}

pub fn swap_with_quote(ctx: Context<SwapWithQuote>, quote: MakerQuote) -> Result<()>;

// arcter-vault
pub fn deposit(ctx: Context<Deposit>, usdc_amount: u64, asset_amount: u64, min_shares: u64) -> Result<()>;
pub fn withdraw(ctx: Context<Withdraw>, shares: u64, min_usdc: u64, min_asset: u64) -> Result<()>;
pub fn create_market(ctx: Context<CreateMarket>, args: MarketArgs) -> Result<()>;

// arcter-oracle
pub fn sync_regime(ctx: Context<SyncRegime>) -> Result<()>;   // open to any caller
```

The input and output mints, recipient token accounts and hook accounts are not arguments, because Solana passes them in the account list. A swap's accounts are therefore built by the client and checked by the program.

All amounts are raw units. A Scaled UI Amount multiplier is read fresh for pricing and display, and is never stored as a balance.

Each fill records a `QuoteBreakdown` with four numbers (mid, half spread, skew, fee) and the regime. The record matches the lines on the trader's ticket.

## Transaction fit

A two-leg swap carrying a signed report, signature checks and hook accounts is a heavy transaction.

* Versioned transactions and address lookup tables keep it inside 1,232 bytes.
* Compute budgets are measured per route, then requested explicitly.
* The router resolves hook accounts before signing, because a missing one fails the transaction.
* The native Ed25519 program checks maker signatures in a separate instruction, and `arcter-rfq` confirms that result by introspection.
* Cross-program calls are limited to the programs and accounts that each instruction validates.

Whether the heaviest routes fit is still being tested, and a route that fails to fit will not be offered.

## Events

| Event | Emitted by | Records |
| --- | --- | --- |
| `SwapSettled` | `arcter-router` | Trader, both mints, both amounts, chosen lane |
| `VaultFill` | `arcter-vault` | `QuoteBreakdown` and regime |
| `ChangeQueued` | `arcter-config` | Change id, earliest execution time, reason |
| `ChangeApplied` | `arcter-config` | Change id, new value |

The plan is for every ledger figure to derive from these records. See [Reading the explorer](../guides/reading-the-explorer.md).

## Upgrades

Solana programs are upgradable while an upgrade authority exists. Arcter's will sit with the Squads multisig, behind its time lock, so an upgrade is possible but slow and visible. Depositors get the delay as a window to read the change and leave.

Verified builds complete the picture, since anyone can match deployed code to public source. This is weaker than code that can never change, and the trade-off is deliberate. Removing the authority would also remove the means to patch a fault during the early life of a set of programs facing fast-changing token extensions. Whether and when to revoke it is a decision for a later stage.

Pluggable inputs follow the same queue. Feed configuration and the list of accepted attestation issuers change through `arcter-config`, as data products and identity providers turn over faster than settlement logic should.

## Privileged roles

| Role | Holder | Powers |
| --- | --- | --- |
| Authority | Squads multisig | Queues changes with a published reason, executes them after the delay, holds upgrade authority, and may lift a pause without waiting |
| Guardian | Squads multisig at a lower quorum | Engages the trading pause and nothing else |
| Keeper calls | Anyone | Move a market between regimes and fire a halt when a trigger is met |
| Anyone else | None | No other privileged instruction is planned |

The guardian cannot lift its own pause, and it cannot move funds. The pause is not meant to block deposits, withdrawals where safe, or the cancellation of RFQ quotes.

These roles do not bind issuers. A permanent delegate or freeze works at the token level, outside Arcter's programs, so no setting here can prevent it.
