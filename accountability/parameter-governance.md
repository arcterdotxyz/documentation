---
description: How Arcter's settings and programs are planned to change, who can act, how long each change waits, and what stays out of reach.
---

# Parameter governance

{% hint style="warning" %}
This is a design for launch. No programs are deployed, and no multisig, signer set or threshold has been published yet.
{% endhint %}

Governance at Arcter is meant to be small. It tunes a venue. It should have no route to anyone's balance.

## What is adjustable

Settings live in the `arcter-config` program, which also holds the treasury PDA. Authority over that program sits with a Squads multisig that has a time lock.

| Group | Examples |
| --- | --- |
| Pricing | Spreads, regime multipliers, fees |
| Size and risk | Clips, caps, oracle bands |
| Sources of truth | Oracle sources, attestation issuers trusted by `arcter-access` |
| Listings | The tier assigned to each market |

## Waiting periods

| Class | Applies to | Wait |
| --- | --- | --- |
| Standard | Parameters not listed below | 48 hours |
| Sensitive | Fee levels, oracle sources, attestation issuers | 7 days |
| Program upgrade | New code for any Arcter program | Time-locked; length to be confirmed before launch |
| Emergency pause | Halting quotes and settlement | None |

How the two parameter delays map onto Squads and `arcter-config` is an implementation detail to settle before launch. The durations themselves are fixed design inputs.

The wait exists for the people affected. A depositor who dislikes a pending change has time to read it and leave.

## One change, start to finish

1. **Written proposal.** A signer submits the exact edit with its motivation and evidence. The evidence is normally a listing review or data from [Fill reports](fill-reports.md).
2. **Multisig approval.** The signer threshold must be met in Squads, where the proposal can be read in public.
3. **Time lock.** The full wait runs before anything can execute.
4. **Execution.** `arcter-config` records the prior value, the new value and a reference to the proposal. The **Change history** on the Ledger keeps this for every parameter. See [Reading the explorer](../guides/reading-the-explorer.md).

Signers and their threshold are planned as public information, and [Entities](../legal/entities.md) covers who stands behind them. Holding a say here earns nothing. No token or yield is attached, and token emissions are not planned.

No token vote or similar process is scheduled. If control ever moves away from the multisig, the successor must inherit the limits below unchanged.

## During an incident

A 48 hour wait is useless in the middle of a live fault, so a guardian role can pause a market at once. That power is safe because of what it leaves out:

* A pause stops trading. It cannot move funds.
* Deposits and withdrawals continue where safe while a market is `HALTED`.
* The guardian cannot lift its own pause. Only the owner multisig can resume, and it may do so without the time lock.
* Each guardian action must come with an incident note. A pause without one breaks the [launch commitments](commitments.md).

Pauses are one of several routes to `HALTED`. The automatic triggers are described in [Session regimes](../execution/session-regimes.md).

## Limits on authority

| The multisig must not be able to | Reason |
| --- | --- |
| Skip the wait on a substantive change | Only the pause and its lifting are immediate |
| Move or freeze user funds through settings | `arcter-config` will contain no such instruction |
| Block withdrawals through settings | No parameter gates a withdrawal |
| Charge a fee after the fact | Each fill uses the fee in force at that moment |
| Switch off the oracle band | No parameter overrides the band check |

## The upgrade caveat

Those limits describe the programs as written, and Solana programs can be replaced. A malicious or faulty upgrade approved by the multisig could change any rule on this page. Arcter does not claim otherwise.

Three planned safeguards cover that gap:

* Upgrade authority sits behind the multisig and its time lock, so new code is visible before it runs.
* Verified builds let anyone match deployed code to public source.
* Two independent reviews are intended before launch.

Depositors should treat signer honesty and the upgrade wait as part of the risk they accept. See [Releases and addresses](../system/releases-and-addresses.md).
