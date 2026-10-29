---
description: An A to Z of the Solana, token and market terms used throughout the Arcter documentation.
---

# Terminology

All terms appear in one alphabetical list. Program names begin with `arcter-` and sort under A. Definitions describe the intended design, since no Arcter program is deployed yet.

## A

**Address lookup table.** An on-chain list of addresses that a versioned transaction can reference by short index, so that more accounts fit within the size limit.

**Alpenglow.** A consensus upgrade for Solana made of two parts, Votor and Rotor. Votor is live, and Rotor is not yet scheduled.

**Anchor.** A Rust framework for Solana programs. All Arcter programs are written with it.

**`arcter-access`.** The program that records attestations and checks market access. See [Access controls](../system/access-controls.md).

**`arcter-config`.** The program that stores parameters and the treasury PDA. A Squads multisig with a time lock owns it.

**`arcter-oracle`.** The program that normalizes reports, applies age and deviation checks, and derives the regime.

**`arcter-rfq`.** The program that verifies a maker's signature, expiry and replay protection, then settles the trade atomically.

**`arcter-router`.** The single entry point for trades. It enforces minimum output and expiry, compares the vault and maker lanes, and composes multi-leg swaps.

**`arcter-vault`.** The program that holds Market accounts, reserves and share mints. `create_market` adds a listing without new code.

**Associated token account (ATA).** The token account at the standard derived address for a given wallet and mint. Creating one costs a small refundable deposit.

**Attestation.** A record that a wallet passed off-chain checks. It sits in a PDA and contains no personal information.

## B

**Band.** How far a fill may sit from the oracle mid. A fill beyond that limit fails, and so does a maker quote beyond it.

**Base fee.** The network charge of 5,000 lamports per signature, paid in SOL.

**Basis point (bp).** One hundredth of one percent. Spreads and fill differences are stated in basis points.

**Blockhash.** A recent hash included in each transaction. It expires after about 150 slots, roughly a minute or a little more, and a transaction carrying an expired one cannot land.

## C

**Clip.** The largest size one swap may take from a vault. Anything bigger goes to the maker lane.

**Clock sysvar.** A special account through which programs read the current slot and unix timestamp. Arcter uses it for expiry and session checks.

**Cluster restart.** A recovery of the whole network, coordinated among validators after a halt. Arcter markets stay `HALTED` for a one-hour grace window afterward.

**Compute unit.** The measure of work in a Solana transaction. Each transaction has a compute budget, and default limits apply.

**Confidential transfer.** A Token-2022 extension that hides transfer amounts and balances with encryption.

**CPI (cross-program invocation).** A call from one program into another within a transaction. The router reaches the vault this way, and token transfers run the same way.

## D

**Data Streams.** Chainlink's service that delivers signed reports, which travel in the transaction and are checked on-chain by Chainlink's verifier program. It is Arcter's intended primary price source. Report schema, access and per-asset coverage on Solana are to be confirmed.

**Default account state.** A Token-2022 extension that sets the state of new token accounts for a mint. Accounts may start frozen, so a vault account may need issuer clearance.

## E

**Ed25519 program.** A native Solana program that verifies ed25519 signatures. Maker quotes are checked through it.

**Equity token.** A token that tracks a listed share or ETF. Arcter also says tokenized stock.

**Execution quality.** How far a fill sat from the oracle mid, in signed basis points, measured in the slot where the transaction landed. See [Fill reports](../accountability/fill-reports.md).

## F

**Fee payer.** The account that pays a transaction's network fees. It is usually the user, and sponsorship through a relayer is an option under review.

**Finality.** The point at which a block can no longer be reversed. Votor targets about 150 ms.

## G

**Grace window.** The one hour after a cluster restart during which markets stay `HALTED`.

**Guardian.** The role that can pause a market by hand. A pause can stop trading and cannot move funds.

## H

**Half-spread.** The distance from the mid to one side of a quote. Arcter sets spread parameters in this form, in basis points.

## I

**Instruction introspection.** A program reading the other instructions in its own transaction. `arcter-rfq` uses it to confirm the Ed25519 program checked a maker's signature.

**Inventory skew.** A shift in the quote against the side that moves a vault further from its target mix. The helpful direction gets cheaper, which pays arbitrage to rebalance.

## J

**Jito.** A widely used block engine on Solana that accepts bundles and tips. Arcter is evaluating it as an inclusion path.

## K

**Keeper.** An automated caller of public instructions, such as regime transitions. Anyone can make those calls, so a keeper failure degrades freshness and does not put funds at risk.

## L

**Lamport.** The smallest unit of SOL. One SOL is 1,000,000,000 lamports.

**Leader.** The validator producing blocks at a given moment. Leaders rotate on a published schedule.

**LiteSVM and Mollusk.** Lightweight test harnesses for running Solana programs in unit tests.

## M

**Maker.** An approved firm that signs quotes off-chain for settlement inside the taker's transaction.

**Market account.** The PDA for one listing in `arcter-vault`. It records the asset, its reserves and its settings.

**Metadata pointer.** A Token-2022 extension that tells readers which account holds a mint's name and symbol.

**Mid.** The verified oracle reference price for an asset. With no valid mid, a vault does not quote.

**Mint.** The account that defines a token, including its supply, decimals, authorities and extensions.

## O

**Ondo Global Markets.** An issuer of tokenized US stocks and ETFs, live on Solana since 21 January 2026. See [Tokenized equities](../catalog/tokenized-equities.md).

## P

**PAXG.** A gold token from Paxos, on Solana since June 2026.

**Pausable.** A Token-2022 extension that lets an authority stop all transfers of a mint. Arcter moves the market to `HALTED` when this applies.

**PDA (program derived address).** An address derived from seeds and a program ID, with no private key. Only that program can sign for it.

**Permanent delegate.** A Token-2022 extension giving one authority the power to move or burn tokens from any account of a mint, a vault included.

**Priority fee.** An optional bid, priced per compute unit, on top of the base fee to improve the odds of inclusion.

**Program.** Code deployed on Solana. Programs keep state in separate accounts.

**Pyth pull update.** A signed Pyth price placed on-chain by the party that needs it. It serves as Arcter's independent cross-check.

## Q

**Quote asset.** USDC. All pairs are priced against it, and displayed prices and fees use it as their unit.

## R

**Raw amount.** The integer balance stored in a token account. Vault math uses it, and Scaled UI Amount leaves it unchanged.

**Regime.** A market's current state. `OPEN` has spread x1.0 and a full clip, `EXTENDED` has spread x1.5 and clip x0.75, `CLOSED` has spread x3.0 and clip x0.5, and `HALTED` has no quotes. See [Session regimes](../execution/session-regimes.md).

**Rent-exempt minimum.** The refundable lamport balance that keeps an account open on Solana.

**Replay protection.** A per-maker nonce or quote-id PDA that stops a signed quote from being used twice.

**RFQ (request for quote).** The lane for larger orders, where a maker signs a short-lived ed25519 quote and the taker submits it. See [RFQ desk](../execution/rfq-desk.md).

**Rotor.** The block propagation half of Alpenglow. It has no schedule yet.

**RPC.** The server interface that apps use to read Solana and submit transactions.

## S

**Scaled UI Amount.** A Token-2022 extension that applies a multiplier to the displayed balance. Issuers can use it for splits and dividends, and the raw amount does not change. See [Splits and dividends](../execution/splits-and-dividends.md).

**Sealevel.** Solana's parallel runtime. Transactions declare their accounts up front, so those that do not overlap can run at the same time.

**Session.** The state of the underlying home market: regular hours, extended hours or shut. Regimes are derived from it, together with the safety signals.

**Share mint.** A plain SPL token issued by a vault to depositors, redeemable pro rata for its reserves and earned spread.

**Slot.** Solana's unit of block time, about 400 ms long.

**SOL.** Solana's native asset. It pays network fees.

**Solscan.** A public Solana block explorer. Arcter also links to the Solana Explorer.

**SPL token.** A token under Solana's classic Token program. Arcter handles both this program and Token-2022.

**Squads.** A multisig system on Solana. It holds Arcter's upgrade and settings authority, with a time lock.

**Stake-weighted quality of service.** A method that gives traffic through staked validators better access to leaders.

## T

**Tier.** The grade given to a listing, which is A, B or C. It sets base spread, clip, band and caps. See [Admission criteria](../catalog/admission-criteria.md).

**Time lock.** A required delay between approving a change and applying it.

**Token account.** The account that holds one wallet's balance of one mint.

**Token-2022.** Solana's newer token program, which adds optional extensions at the mint. Also called Token Extensions.

**Tracker certificate.** The legal form of xStocks. It gives price exposure and not shareholder rights.

**Transfer hook.** A Token-2022 extension that calls a chosen program on each transfer. A swap must carry the extra accounts it needs, or the transaction fails.

**Trident.** A fuzzing framework for Solana programs.

**Turbine.** The protocol that currently propagates blocks across Solana.

## U

**USDC.** Circle's dollar stablecoin, native on Solana, and Arcter's settlement asset. See [Quote currency](../catalog/quote-currency.md).

## V

**Vault.** The liquidity for one market, held under its Market account in the listed asset and USDC. See [Vault liquidity](../execution/vault-liquidity.md).

**Verified build.** A reproducible build that lets anyone match a deployed program to its public source.

**Versioned transaction.** A transaction format that supports address lookup tables.

**Votor.** The consensus layer of Alpenglow, live on Solana since 21 October 2026 (epoch 1064). It replaced TowerBFT and targets finality of about 150 ms.

## W

**Wallet Standard.** A common interface through which apps connect to Solana wallets such as Phantom, Solflare and Backpack.

## X

**xStocks.** Tokenized US-listed stocks and ETFs issued by Backed Assets (JE) Limited of Jersey, using Token-2022 mints.

**XAUt0.** A Tether Gold token, omnichain and present on Solana.

## Y

**Yellowstone gRPC.** A Geyser-based stream of Solana account and transaction data. It feeds Arcter's read model.

**Yield-bearing dollar.** A dollar token that passes on income, priced from its published net asset value. USDY is one candidate, subject to review.
