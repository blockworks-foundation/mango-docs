# ⚡ Flash Loans

Flash loans can be performed essentially by building a Solana transaction, where starting instruction is the `FlashLoanBegin` and last instructions is the `FlashLoanEnd` from the mango V4 program. The intermediate instructions can call any solana program of choice. `FlashLoanBegin` transfers tokens from the mango V4 vaults to the token accounts of choice, and `FlashLoanEnd` moves the excess (in comparison with amount before the transaction) amount per token back to the vaults.

The swap functionality in the mango V4 Web UI, is basically a margin trade using [jupiter](https://jup.ag/).

**Constraints**

* Borrowing for a particular token can only be done once
* Cannot be called from cross-program invocation
* The only mango V4 instructions allowed are the `FlashLoanBegin` and `FlashLoanEnd`
* The initial health of the involved mango account at the end of the transaction must either be above 0 or better than the initial health before the transaction
