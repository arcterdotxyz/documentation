---
description: A map of Arcter's planned programs and supporting services, showing what each party is trusted with and what they can never do.
---

# System blueprint

Arcter is planned as six Solana programs surrounded by a few helper services. Rules live in the programs. Services read state, prepare transactions and send routine calls, and everything they produce is checked on-chain before it counts.

{% hint style="info" %}
This page describes a design. No Arcter program is deployed, no audit has been completed, and details can change before launch.
{% endhint %}

## The map

```
   Traders    Depositors    Makers    Integrators
       |           |           |           |
       v           v           v           v
  +---------+  +---------------+  +---------------+
  | Web app |  | Quote service |  | Maker systems |
  +----+----+  +-------+-------+  +-------+-------+
       |               |                  |
       +---------------+------------------+
                       |  user-signed transactions
  +--------------------v---------------------------+
  | Solana                                         |
  |                                                |
  |              arcter-router                     |
  |              /           \                     |
  |     arcter-vault      arcter-rfq               |
  |       /      \                                 |
  | arcter-oracle  arcter-access                   |
  |                                                |
  | arcter-config: settings, treasury PDA          |
  | (Squads multisig with a time lock)             |
  +--------------------+---------------------------+
                       |  read only
          +------------v-------------+
          | Yellowstone gRPC stream  |
          +------------+-------------+
                       |
       Supabase read model, ledger, fill reports, keepers
```

Two directions matter here. Anything entering the network arrives as a transaction signed by a wallet. Anything leaving it is a copy of committed data. Keepers sit on both sides, since they read state and also send open calls that anyone could send.

[Programs](programs.md) documents each program and its accounts. [Supporting services](supporting-services.md) does the same for the off-chain side.

## Who you are trusting

Self-custody narrows trust without removing it. These are the parties a user relies on, and where each one's power stops.

| Party | What you rely on | Limit of their power |
| --- | --- | --- |
| Chainlink and Pyth | Timely, correct price data | Cannot touch funds. Age limits, a 25% jump guard and cross-checking stop bad data from pricing a fill |
| Solana validators | Including transactions and agreeing on state | Can see, order or delay transactions. Cannot sign for a user. Minimum output and short quote windows limit the damage |
| Token issuers | Backing the token and running corporate actions | **Can pause, freeze or, with a permanent delegate, move tokens out of any account, including a vault.** This is a real exposure, and [Issuer dependence](../safeguards/issuer-dependence.md) covers it |
| Attestation issuers | Accurate eligibility decisions | Cannot price, settle or hold anything |
| Arcter, as service operator | Running the app and helper services | Cannot change what a fill pays or sign for a user |
| Squads multisig | Honest settings changes and upgrades | **Holds upgrade authority, so it could change program code.** The time lock delays any change and verified builds expose it, which gives users time to leave |
| Guardian | Pausing quickly in an emergency | Can stop trading. Cannot move funds or lift the pause |

The issuer and multisig rows deserve the most attention. Neither is reduced to zero by good code, and Arcter says so rather than claiming otherwise.

## Design rules

Four rules are planned to hold across all six programs.

1. **One door for prices.** Each price used anywhere passes through `arcter-oracle` and its checks.
2. **One door for permission.** `arcter-access` alone answers whether a wallet may act. Other programs ask it and keep no view of their own.
3. **Change is slow and public.** Parameters live in `arcter-config`, and both those and program upgrades pass through the multisig and its time lock, each with a published reason. Verified builds let anyone match deployed code to source.
4. **A pause only stops.** The guardian's pause halts quoting and settlement. It has no route to user funds. Deposits and withdrawals continue where safe, and a frozen asset limits this to the unaffected USDC side.

Solana's model adds a fifth habit. Accounts arrive as caller-supplied inputs, so every instruction must prove each one's signer status, owner and PDA derivation. Arcter's fuzzing and invariant tests are planned to target that surface.

## Services at a glance

| Service | Purpose | If it misbehaves |
| --- | --- | --- |
| Quote service | Simulates routes and gathers maker quotes | A wrong preview meets the signed minimum output and fails |
| Keepers | Send open calls for regime changes and watch for mint changes | Freshness suffers, and funds stay safe, because swaps derive the regime in-line |
| Read model | Serves ledger pages and fill reports | Displayed figures may be wrong, yet settlement is unaffected, and each one can be checked on Solscan |

## When parts fail

Each failure has a planned response.

| Failure | Response |
| --- | --- |
| Stale, jumping or disagreeing price data | Market moves to `HALTED` |
| Issuer or feed pause during a corporate action | Market moves to `HALTED` |
| Mint configuration changes | Market pauses until reviewed |
| Cluster restart | `HALTED` for a one-hour grace window |
| Congestion or RPC trouble | Quotes expire, and the user retries with a fresh one |
| Service outage | Vaults stay reachable through transactions built directly, with the data limits noted in [Supporting services](supporting-services.md) |

The whole catalog is in [Risk model](../safeguards/risk-model.md).
