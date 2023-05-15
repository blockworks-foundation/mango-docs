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

The DAO decides what address shall hold the security admin authority and can revise that decision via vote at any time. Currently, [the DAO assigned](https://dao.mango.markets/dao/MNGO/proposal/tK49KBmG6NJLRF7Tef7mvsuXQQFALamgirmHSEGUMCw) the security admin role to [the security council](https://dao.mango.markets/dao/AQbsV8b3Yv3UHUmd62hw9vFmNTHgBTdiLfaWzwmVfXB2), a realms 3 of 10 multisig.

#### Past Actions of the Security Council

* 2023-4-14: After receiving a report of a potentially exploitable bug in flash loans, [the security council voted to disable the FlashLoan instructions](https://app.realms.today/dao/AQbsV8b3Yv3UHUmd62hw9vFmNTHgBTdiLfaWzwmVfXB2/proposal/AUJM8v3an9zAuYHNmBQYH3UWzSGVkFUd7ErDdQK3ER3n). More thorough investigation made clear that the issue had not been exploitable. The action caused the swap ui to be disabled until [the DAO vote to reenable flash loans passed](https://dao.mango.markets/dao/MNGO/proposal/BcRNiqEBZE195jVrtgr7r2iBjFhvLhXHuYaq1Vbc8JxD) on 2023-4-18.
* 2023-3-18: As a response to preliminary findings of the ongoing security audit by OtterSec, the [security council disabled the HealthRegionBegin instruction](https://app.realms.today/dao/AQbsV8b3Yv3UHUmd62hw9vFmNTHgBTdiLfaWzwmVfXB2/proposal/8wzYcKGcSrx2V5WJUMGB3982uTfhMxVXMxyq6qthWfWz). It was an optional instruction for reducing compute requirements, used by some bots and market makers. Disabling it had no impact on users of the Mango Markets UI.
* 2023-3-11: The [security council agreed](https://dao.mango.markets/dao/AQbsV8b3Yv3UHUmd62hw9vFmNTHgBTdiLfaWzwmVfXB2/proposal/BtjbTJnxdpz113Gaz66yZsWXMWgLQ1khzjnGoZuHjwP) to set the USDC init asset weight to zero because its price dropped significantly below $1. Mango was using a fixed 1 USDC = $1 valuation, meaning that further drops in USDC price could have allowed attackers to profit by borrowing other tokens against incorrectly valued USDC deposits.



### **Delisting Process**

Force closing a token

* Move token to reduce only and make make init asset weight 0 via a dao proposal (security admins can also bring tokens to reduce only), Note: use value of 2 and not 1, so that borrows are prevented but deposits are valid, this is required for liquidators who are forced to have token deposits before they can liquidate liabilities in same token.
* After some buffer period, make maint asset weight 0 via a dao proposal. This should force users to deposit alternative collateral if needed.
* Move token to force close via a dao proposal.
* Liquidation fee can be optionally considered for change via a dao proposal.
* Anybody can then run the dedicated script [`force-close-token-borrows.ts`](https://github.com/blockworks-foundation/mango-v4/blob/main/ts/client/scripts/force-close-token-borrows.ts) to force close any borrows and earn liquidation fee.

Force closing a market

* Move token to reduce only and make make init asset weight 0 via a dao proposal (security admins can also bring tokens to reduce only).
* After some buffer period, make both init and maint assets weights 0 via a dao proposal. This should force users to deposit alternative collateral if needed.
* Move market to force close via a dao proposal.
* Liquidation fee can be optionally considered for change via. dao proposal.
* Anybody can then run the dedicated scripts [`force-close-serum3-market.ts`](https://github.com/blockworks-foundation/mango-v4/blob/main/ts/client/scripts/force-close-serum3-market.ts) or [`force-close-perp-positions.ts`](https://github.com/blockworks-foundation/mango-v4/blob/main/ts/client/scripts/force-close-perp-positions.ts) to cancel all existing spot and/or perp orders and liquidate open perp positions.
