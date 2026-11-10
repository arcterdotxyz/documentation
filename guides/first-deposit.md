---
description: How a depositor picks an Arcter vault, signs a deposit, watches the position and redeems shares in kind.
---

# Make your first deposit

{% hint style="info" %}
This walkthrough describes the intended flow for the planned release. Arcter's programs are not live, and vault deposits are planned for release 1.1, after the first release.
{% endhint %}

A deposit finances a market. Your capital joins one vault, that vault quotes traders around the oracle mid, and you earn a share of the spread on what it fills. This is inventory risk paid for by real order flow, and it should not be confused with farming a token reward. Token emissions are not planned.

The theory sits in [earning as an LP](../execution/earning-as-an-lp.md), and it is worth reading first. This page covers the mechanics.

## Risks to accept before you start

* **Inventory.** The vault may hold more of the token than you would like just as its price falls.
* **Gap risk.** Weekend flow pays a wider spread because Monday's open can move sharply against the vault.
* **Issuer power.** A permanent delegate can move tokens out of any account, vault accounts included. Issuers can also pause or freeze a token.
* **Program risk.** No audit has been completed yet, and two independent reviews are intended before launch.

[Issuer dependence](../safeguards/issuer-dependence.md) explains the issuer items in full, and you are asked to read it as part of onboarding, since part of every vault is held in equity tokens.

## Clear access

Deposits need an `LP` attestation, held in the Attestation PDA for your wallet. At launch it is reserved for professional clients. You obtain it through the same KYC provider flow used for trading. Once it is active, **Access** on your **Account** page lists it.

You also need SOL for network fees, and either USDC, the market's own token, or both.

## Compare vaults one at a time

Every market has its own vault, and there is deliberately no pooled product spreading your capital across tiers. Start on the **Vaults** page, which shows each vault's tier, current token share of inventory, tier half-spread and depositor share, with a tier filter. Then open the market in the [Ledger](reading-the-explorer.md) and ask these questions:

| Question | Where the answer is |
| --- | --- |
| What has this vault truly earned, and in which sessions? | Spread revenue and volume by regime since listing |
| Is it lopsided now? | Inventory mix, and how far skew sits from its band |
| What rules bind its quotes? | Tier parameters and caps |
| How often has it stopped, and why? | The halt log with reasons |
| What can the issuer do to it? | The mint's extensions, recorded at listing |

Tier A markets turn over more at tighter spreads. Tier C pays wider spreads on thinner flow, with gaps arriving more often.

## Sign the deposit

1. Open the vault and enter USDC, the token, or a mix of both. Fees to enter: none. Lockup: none.
2. Review the ticket. Your contribution is valued at the guarded mid, and you receive newly minted shares of matching value.
3. Approve one transaction in your wallet. No standing permission is granted.

A share is a plain SPL token with one mint per vault. If you have never held that vault's shares, the transaction also creates a token account for them, which takes a small refundable deposit of SOL. The base fee is 5,000 lamports per signature, plus any priority fee.

The wallet preview lists estimated balance changes. It may show the share mint as an unfamiliar token, so compare its address with the one on the vault page. Expect a small difference from the ticket as well, since the value is struck when the transaction executes and not when the preview ran.

A slow signature fails harmlessly. The blockhash lapses after about 150 slots, roughly a minute or a little more. Your funds stay where they were, and you can reload and sign again.

### When a deposit is refused

During the guarded launch period each vault has a published deposit ceiling. A full vault turns deposits away until the limit is raised, and that change must pass through the Squads multisig and its time lock.

Deposits also need a trustworthy mid. If a halt has removed that reference, the vault cannot value new capital and the deposit waits for quoting to resume.

## Watch the position

Share value rises a little with each fill. The position view pairs that value with your slice of the vault's present inventory. Three signals deserve regular attention:

| Signal | Reading it |
| --- | --- |
| Skew at its limit | The vault has gone one-sided, so revenue drops. If this persists, the market's flow is running steadily in one direction. |
| Regime mix | Fills during `CLOSED` are what pay you for gap risk. If few land then, that pay is thin. |
| Halts | A halted vault earns nothing while it is stopped. |

Keep an eye on the mint too. Any change to a listed mint's configuration pauses the market until it has been reviewed.

## Redeem your shares

Burning shares returns your pro rata part of what the vault holds at that moment. Payment is in kind, as USDC and tokens, and the outcome follows from that:

* The mix you receive reflects the vault on the day you leave. It can be heavier in the token or heavier in USDC than what you put in. Selling the token afterwards is an ordinary swap with ordinary costs.
* Redeeming during `CLOSED` leaves you holding a token that is expensive to sell until the home exchange reopens. Consider the timing before you exit.
* Solana token accounts apply here too. You need an account for each asset you receive, and it is created in your redemption transaction if it does not exist.

Arcter places no permission check on redemption, and withdrawals continue through a halt where that is safe. Issuer controls can still limit the token leg. A frozen or paused token cannot leave the vault, and under default account state a new token account of yours may start frozen until the issuer clears it. In those cases the unaffected USDC side continues while the token side waits. See [issuer dependence](../safeguards/issuer-dependence.md).

## Transferring shares

Shares are plain SPL tokens, so a wallet can send them like any other token. Treat that with care. A plain mint has no hook, so no program can stop a transfer, and a recipient without `LP` can still redeem. Arcter gates entry at deposit only. Whether that is acceptable for every market is a question for counsel, so read [access controls](../system/access-controls.md) before moving shares between wallets.

## Keeping records

Share value blends earned spread with price movement, and most jurisdictions treat an in-kind withdrawal as a disposal. The Ledger exports a full CSV history for each position. Take the treatment to your own adviser, because nothing in this documentation is tax advice.
