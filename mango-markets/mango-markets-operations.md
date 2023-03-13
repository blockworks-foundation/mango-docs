# ⚙ Mango Markets Operations

### Program Upgrade Authority

The upgrade authority for [the Mango Markets v4 on-chain program](https://explorer.solana.com/address/4MangoMjqJ2firMokCjjGgoK8d4MXcrgL7XJaL3w6fVg) is the Mango DAO. That means the program can only be upgraded through [a Mango DAO vote](https://dao.mango.markets/dao/MNGO).

### **Group Admin**

The parameters and operation of the [primary Mango Markets group](https://explorer.solana.com/address/78b8f4cGCwmZ9ysPFMWLaLTkkaYnUjwMJYStWe5RTSSX/anchor-account?cluster=mainnet-beta) is controlled by the Mango DAO. They can be changed by [a Mango DAO vote](https://dao.mango.markets/dao/MNGO).

### **Security Admin and Security Council**

The group admin can assign a security admin. The security admin has the following powers:

* Disable any mango instruction. For example, if an exploitable bug is discovered in a part of the code, the security admin can disable the affected instructions.
* Bring tokens or markets to reduce-only mode. In reduce-only mode, borrows can be closed and withdraws can be made but new deposits or new borrows are disabled.
* Set init\_asset\_weight to zero for a token or perp market. This action prevents users from making new borrows using the token or perp position as collateral.

The security admin is supposed to use its limited powers to quickly restrict group operation in emergencies, where waiting for a full DAO vote would take too long. Examples are an exploitable bug being found in the program, or exceptional situations relating to specific tokens or markets.

The DAO decides what address shall hold the security admin authority and can revise that decision via vote at any time. Currently, [the DAO assigned](https://dao.mango.markets/dao/MNGO/proposal/tK49KBmG6NJLRF7Tef7mvsuXQQFALamgirmHSEGUMCw) the security admin role to [the security council](https://dao.mango.markets/dao/AQbsV8b3Yv3UHUmd62hw9vFmNTHgBTdiLfaWzwmVfXB2), a realms 4 of 10 multisig.

#### Past Actions of the Security Council

* 2023-3-11: The [security council agreed](https://dao.mango.markets/dao/AQbsV8b3Yv3UHUmd62hw9vFmNTHgBTdiLfaWzwmVfXB2/proposal/BtjbTJnxdpz113Gaz66yZsWXMWgLQ1khzjnGoZuHjwP) to set the USDC init asset weight to zero because its price dropped significantly below $1. Mango was using a fixed 1 USDC = $1 valuation, meaning that further drops in USDC price could have allowed attackers to profit by borrowing other tokens against incorrectly valued USDC deposits.
