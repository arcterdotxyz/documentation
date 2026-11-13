---
description: Plain answers on Arcter's status, pricing, tokens, vaults and building on the venue.
---

# Common questions

## Status

**Can I trade on Arcter today?**
No. Arcter is pre-launch, with no programs deployed and no audits completed. Answers below describe the planned design. Treat any site or token claiming to be a live Arcter product with suspicion, and check [official channels](brand-kit.md).

**When does it launch?**
The plan targets launch in months 3-5, and passing its tests matters more than the date. See [Delivery phases](../plan/delivery-phases.md).

**Does Arcter have a token?**
No token is described in the plan, and token emissions are not planned.

**Who can use it?**
US persons will not be served. Other restrictions follow each issuer, and those lists differ by product and channel. Arcter applies the strictest reading for each market. See [Access controls](../system/access-controls.md).

## Prices and fills

**Why does the spread jump at night?**
The home exchange has shut, so whoever fills you is exposed to a price gap when it reopens. In `CLOSED` the spread is multiplied by 3.0 and the largest clip falls to x0.5. The ticket shows this before you sign. See [Session regimes](../execution/session-regimes.md).

**Where does a quote come from?**
One public formula. It starts at the verified reference price, adds the tier spread times the regime multiplier, adjusts for inventory skew, and shows the fee as its own line. The design leaves staff no way to adjust a quote or a fill. See [Quote math](../execution/quote-math.md).

**How would I know a fill was fair?**
Check it yourself. Each fill is designed to record its components on Solana, and [Fill reports](../accountability/fill-reports.md) will publish how far all fills sat from the reference price.

**What does a trade cost beyond the quote?**
You pay a Solana network fee in SOL: 5,000 lamports per signature, plus any priority fee. A new token account also needs a small refundable deposit. Sponsored fees on a first trade are under review. See [Costs](../execution/costs.md).

**Can I be sandwiched?**
Block producers see transactions, and sandwiching exists on Solana. Vault fills are tied to the reference price and limited by the oracle band, and each ticket signs a tight minimum output inside a short quote window. That lowers the exposure without eliminating it.

**Can I trade during a halt?**
No. A halted market gives no quotes, so your wallet is untouched. Halts come from stale oracle data, issuer or feed pauses, a price jump above 25% in one update, the one-hour window after a cluster restart, or a guardian pause. Trading resumes when the cause clears.

**Why was I stopped before signing?**
The router simulates each trade first, including any transfer hook. A stop usually means the wallet has no required attestation, or your jurisdiction is excluded for that asset. You will see which rule applied.

**Why did my transaction fail or expire?**
A failed trade usually hit its minimum output. Solana blockhashes also expire after about 150 slots, roughly a minute or a bit more, so a transaction not included by then is dropped. Busy periods and RPC failures make this more likely. Your funds stay in your wallet.

## The tokens

**Does holding a tokenized stock make me a shareholder?**
No. An xStocks token is a tracker certificate from Backed Assets (JE) Limited, not a share. Ondo Global Markets tokens have their own terms. Both give price exposure, without shareholder rights. Read [Tokenized equities](../catalog/tokenized-equities.md) before trading in size.

**How are dividends and splits handled?**
It depends on the issuer. On xStocks, the Scaled UI Amount multiplier moves while your raw balance stays the same, so your displayed balance changes and you claim nothing. Ondo's FAQ describes a 30% US withholding on dividends. See [Splits and dividends](../execution/splits-and-dividends.md).

**Can an issuer touch my tokens?**
Potentially, yes. Mints can carry freeze, pause, transfer hook and permanent delegate powers, and some may be dormant. These sit outside Arcter's control. See [Issuer dependence](../safeguards/issuer-dependence.md).

## Vaults

**How do depositors earn?**
Vaults keep the spread paid on fills. The protocol takes 10%, and the other 90% lifts the value of vault shares. Token rewards are not planned. Public deposits are planned for the Depositors and holdings phase. See [Earning as an LP](../execution/earning-as-an-lp.md).

**Can a halt trap my deposit?**
An Arcter halt does not block exits by itself, and withdrawals continue where safe. Shares redeem pro rata against what the vault holds. An issuer freeze or pause is different, because it can stop the token side while the USDC side stays available. Nothing can process during a cluster outage.

**Do vaults suffer impermanent loss?**
Not in the usual form. That loss comes from arbitrage walking a curve back to the true price, and these vaults quote around the oracle midpoint instead. You still carry inventory risk if the token falls, gap risk on fills taken while closed, and issuer risk. Spread is your payment for those.

## Continuity and building

**What if the company behind Arcter disappears?**
The design aims to outlast it. Prices derive from on-chain state, keeper calls are open to anyone, and vault shares redeem without Arcter's help. Upgrade authority would still sit with the multisig signers, and a keeper gap would degrade freshness. See [Entities](../legal/entities.md).

**Can I integrate Arcter into my app?**
That is the intent. The router is planned to be callable without permission, on the same terms for every integrator. Eligibility is checked per wallet, against the person using your product, and the app itself needs no approval. The quote API and SDK arrive in the Platform phase. See [Developer access](../system/developer-access.md).
