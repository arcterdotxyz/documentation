---
description: What an issuer can do to a tokenized stock on Solana, what that means for vaults and holders, and how Arcter limits it.
---

# Issuer dependence

An equity token is a promise from a company. xStocks are tracker certificates issued by Backed Assets (JE) Limited of Jersey, backed 1:1 under a base prospectus. Ondo Global Markets tokens come from a separate issuer on separate terms. Neither gives you shareholder rights. On Solana, the issuer also has a technical presence: controls written into the mint itself can reach into any token account, a vault included.

Arcter cannot remove that dependence. It can read it accurately, price it into listing decisions and state it plainly.

{% hint style="warning" %}
Arcter's programs are not deployed. A vault deposit in an equity token will carry the issuer's credit risk and the issuer's control over the mint. Both can cause losses that no Arcter parameter prevents.
{% endhint %}

## Controls at the mint

Explorer inspection of xStocks mints shows Token-2022 with metadata pointer, confidential transfer, Scaled UI Amount, default account state, transfer hook and permanent delegate. Some of these may be configured for future use and not active today. Arcter treats a configured power as a usable power.

| Control | What the issuer can do | Effect on holders | Effect on a vault |
| --- | --- | --- | --- |
| Permanent delegate | Move or burn tokens from any account for that mint, with no owner signature | Balance can fall without consent | Reserves can fall with no trade, and every depositor shares the shortfall |
| Freeze | Stop a specific token account | That account cannot send or receive | The asset side is stuck until thawed |
| Default account state | Make new token accounts start frozen | A new holder may not receive tokens until cleared | The vault account needs issuer clearance before a market opens |
| Transfer hook | Run issuer logic on each transfer | A transfer may be refused | Every swap and in-kind exit depends on the hook allowing it |
| Pausable, where present | Stop transfers for the whole mint | No movement at all | The market goes to `HALTED` |
| Scaled UI Amount | Change the multiplier for splits and dividends | Displayed balance moves, raw balance does not | Vault math stays in raw units, and the multiplier is read fresh in each quote |

### Permanent delegate

This is the broadest power on the list. If it is exercised against a vault token account, tokens leave and no Arcter program takes part. Shares redeem pro rata against what remains, so the loss spreads across depositors in proportion. How the program reconciles reserves after such a move is to be specified and tested with mock mints. The listing page will disclose this risk for every mint where the delegate exists.

### Freezes and pauses

A freeze on a vault's token account stops the asset leg of every swap. The market moves to `HALTED`. USDC remains withdrawable. The frozen asset does not. A depositor's in-kind exit therefore covers only what can actually move, and the token portion waits until the freeze ends. Arcter cannot promise otherwise. If your own token account is frozen, Arcter cannot sell for you, because the transfer would fail at the Token-2022 level.

### Default account state

If new accounts start frozen, a Market cannot open until the issuer clears its vault account. Whether issuers will do this, and on what terms, is an open discovery question. A mint whose issuer will not clear a program-owned account cannot list.

The same setting can affect buyers. A token account created during a purchase may start frozen, and the transfer into it would then fail. The router simulates this first, so the user sees a reason, not a failed transaction.

### Transfer hooks

A hook runs issuer-chosen code on each transfer, and it needs extra accounts in the transaction. Three risks follow:

* **Fit.** Hook accounts add to a 1,232-byte transaction and to its compute use. A route that does not fit cannot execute.
* **Refusal.** A hook may reject a program-owned vault account or a particular wallet. Whether hooks accept vault accounts must be confirmed for each mint.
* **Change.** The hook program can be altered or replaced by whoever controls it, which can change behavior after listing.

The router resolves required accounts ahead of time and simulates each trade. The rules that transfer hooks enforce sit alongside the checks in [Access controls](../system/access-controls.md).

## Beyond the code

The legal side of the dependence matters as much as the mint.

* **Credit.** If an issuer fails, what a holder recovers depends on the issuer's terms and the custody arrangements behind the token. Recovery may be partial and slow.
* **Redemption.** Redeeming with the issuer is limited to eligible parties through its own channels. Arcter's pricing does not require it. However, primary access for makers affects how tightly arbitrage can hold a token near fair value, and that access is still a discovery question.
* **Regulatory action.** A supervisor could impose revised terms, push an issuer out of certain jurisdictions or stop new issuance.
* **Discontinuation.** If an issuer retires a token, its market stops quoting permanently and depositors can only withdraw.

## Issuer-specific points

Figures below are dated snapshots.

| Point | xStocks | Ondo Global Markets |
| --- | --- | --- |
| Catalog on Solana | 100+ US-listed stocks and ETFs | 250+ US stocks and ETFs, launched 21 Jan 2026 |
| Value on Solana | About $430M, early Nov 2026 reports | About $300M, RWA.xyz snapshot, Nov 2026 |
| Mint status | Extensions seen by explorer inspection, some possibly dormant | Reported plan to use transfer hooks for eligibility; to be checked on the mint |
| Access limits | Kraken lists the US, Canada, the UK, Australia and prohibited regions as excluded; varies by channel | US persons barred; UK, Switzerland and some EEA access limited to qualified or professional clients |
| Holder economics | Splits and dividends reach holders through the multiplier | 30% US withholding on dividends, per the issuer's FAQ |
| Supporting data | Chainlink offers xStocks Data Streams, Proof of Reserve and CCIP | Oracle coverage to be confirmed per asset |

Issuer terms remain the authoritative source. Arcter takes the strictest applicable reading for each market.

## How Arcter limits the exposure

### Reading the mint before listing

Before a market opens, a reviewer records the mint's token program, its extensions and their authorities. Findings go on the listing page and feed the market's tier and caps. See [Admission criteria](../catalog/admission-criteria.md).

### Watching for changes

Mint configuration is stored at listing. A keeper is planned to compare live state with that record, and any change pauses the market until it has been reviewed. A keeper can only detect a change after it has landed, so the first transaction under a new setting is not protected by this monitor.

### Plain disclosure

Before a first swap or deposit, users are to see a short summary of what the token is and what its issuer can do. No venue can engineer this risk away. A venue can make sure nobody carries it unknowingly.

### Reserve evidence

Chainlink offers Proof of Reserve for xStocks. Whether `arcter-oracle` will consume it on Solana is to be confirmed. A listing page will state whether a market has reserve evidence, including when it has none.

### Caps per market and per issuer

Launch caps bound each market. The ledger is planned to show one further number for each issuer: total vault inventory that depends on it, across all markets. See [Reading the explorer](../guides/reading-the-explorer.md).

### Several issuers, separate markets

A catalog spanning xStocks and Ondo means no single issuer stands behind every position. Each Market account holds one mint with its own feed and its own depositors, so a freeze or a failure at one issuer stops that issuer's markets and leaves others running. Gold and dollar markets sit outside both.

This softens the risk without removing it:

* The same stock from two issuers forms two markets. The tokens are not interchangeable, and a vault does not fail over from one to the other.
* A depositor in an affected market keeps the full exposure. Separation protects other markets, not that one.
* Issuers can share dependencies, such as a US market closure, an oracle vendor or a regulator.

More issuers may follow. See [Expansion](../catalog/expansion.md).

## What this means for you

An equity token in your wallet and an equity token behind a vault share lead to the same place: a claim that depends on an issuer. A vault adds pooled exposure to mint controls, because one action against one vault account touches every depositor in it. If an issuer fails, Arcter's programs would keep running as designed, and the value of what you withdraw would become a legal question. Caps and disclosure limit this risk. Taking less exposure than the caps allow is a reasonable decision.

## The settlement asset

Circle holds the same kind of power over USDC. It can freeze any account, including a vault's USDC reserve. That would stop the market and hold back the USDC side of withdrawals, while the asset side stayed available. Every market depends on this one issuer, and a catalog of many assets does not reduce that exposure.
