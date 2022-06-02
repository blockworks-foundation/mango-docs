# 🦋 Bug Bounty

#### Program Overview <a href="#r4-program-overview" id="r4-program-overview"></a>

This bug bounty is specifically for Mango Markets' on-chain program code; UI only bugs are omitted.

| **Severity** | **Description**                                                                                                 | **Bug Bounty**                                               |
| ------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Critical     | Bugs that freeze user funds or drain the contract's holdings or involve theft of funds without user signatures. | 10% of the value of the hack up to $1,000,000.               |
| High         | Bugs that could _temporarily_ freeze user funds or incorrectly assign value to user funds.                      | $10,000 to $50,000 per bug, assessed on a case by case basis |
| Medium/Low   | Bugs that don't threaten user funds                                                                             | $1,000 to $5,000 per bug, assessed on a case by case basis   |

The severity guidelines are based on [Immunefi's classification system.](https://immunefi.com/severity-updated/)﻿

Note that these are simply guidelines for the severity of the bugs. Each bug bounty submission will be evaluated on a case-by-case basis.

#### Submission <a href="#iq-submission" id="iq-submission"></a>

Please email [hello@blockworks.foundation](mailto:hello@blockworks.foundation) with a detailed description of the attack vector. For critical and moderate bugs, we require a proof of concept done on a privately deployed mainnet contract. We will reach out in 1 business day with additional questions or next steps on the bug bounty.

#### Bug Bounty Payment <a href="#zj-bug-bounty-payment" id="zj-bug-bounty-payment"></a>

Bug bounties will be paid in USDC or locked MNGO, after a DAO vote. The Mango DAO has never refused a bug bounty so far.

#### Invalid Bug Bounties <a href="#i-invalid-bug-bounties" id="i-invalid-bug-bounties"></a>

The following are out of scope for the bug bounty:

* Attacks that the reporter has already exploited themselves, leading to damage.
* Attacks requiring access to leaked keys/credentials.
* Attacks requiring access to privileged addresses (governance, admin).
* Incorrect data supplied by third party oracles (this does not exclude oracle manipulation/flash loan attacks).
* Lack of liquidity.
* Third party, off-chain bot errors (for instance bugs with an arbitrage bot running on the smart contracts).
* Best practice critiques.
* Sybil attacks.

