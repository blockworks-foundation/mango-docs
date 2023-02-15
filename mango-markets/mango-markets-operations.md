# ⚙ Mango Markets Operations

#### Program Upgrade Authority

The upgrade authority for [the Mango Markets v4 on-chain program](https://explorer.solana.com/address/4MangoMjqJ2firMokCjjGgoK8d4MXcrgL7XJaL3w6fVg) is the Mango DAO. That means the program can only be upgraded through [a Mango DAO vote](https://dao.mango.markets/dao/MNGO).

#### **Group Admin**

The parameters and operation of the [primary Mango Markets group](https://explorer.solana.com/address/78b8f4cGCwmZ9ysPFMWLaLTkkaYnUjwMJYStWe5RTSSX/anchor-account?cluster=mainnet-beta) is controlled by the Mango DAO. They can be changed by [a Mango DAO vote](https://dao.mango.markets/dao/MNGO).

#### **Security Admin**

The group admin can assign a security admin. The security admin has the power to disable any mango instruction and to bring tokens or markets to reduce-only mode.&#x20;

The security admin is supposed to use its limited powers to quickly restrict group operation in emergencies, where waiting for a full DAO vote would take too long. Examples are an exploitable bug being found in the program, or exceptional situations relating to specific tokens or markets.

In practice, the security admin on the primary mango group will likely be a realms [multi sig wallet](https://app.realms.today/realms/new/multisig). The multi sig should be formed by trusted individuals who are geographically well distributed. In the end, the DAO decides what address shall hold the security admin authority and can revise that decision via vote at any time.
