# Perpetual Contract Trading

### **How much leverage can I take?**

Perpetuals allow initial leverage of up to 10x, and maintenance leverage of 20x. If the value of your position (or collateral) falls and your leverage exceeds 20x, your account will be eligible for liquidation.

### What fees will I pay?

Mango Markets has a tiered taker fee scheme, which goes as follows:

| Tier      | Fee      | MNGO requirements |
| --------- | -------- | ----------------- |
| Base tier | 3.25 bps | 0                 |
| Tier 1    | 3.1 bps  | 10,000            |
| Tier 2    | 2.25 bps | 250,000           |

There is a 1.25 bps rebate for maker orders. Mango Markets does not charge fees for borrowing or lending.

### What is the PnL on my position?

The PnL is the profit or loss on your position. It will not include funding & fees. For instance, if you enter 1 SOL-PERP Long @ 150.00, and it is currently trading @ 200.00, your PnL will be $50.00 USDC.

### What is my unsettled PnL?

In perpetual futures markets _Unsettled PnL_ accrues as the oracle changes. At any time any participant can “settle” their PnL which will find a person with the opposite PnL and settle the balances to USDC. This will not affect your position, but simply credit/debit your account with the balance that you are owed/owe.

Note that while you may settle any time you wish, you do not have control to stop a settlement from happening.

For example, assume you buy 1 SOL-PERP long at $100.00, using 1 BTC as collateral. Now assume the oracle drops to $80.00. If no settlements have happened yet on your account, you would show an unsettled balance of -$20.00. If you settle your balance, you will still have your 1 BTC, and your sol-perp position, but it will now borrow $20.00 USDC. Because borrowing costs you money (long-term APR), you could prefer not to settle a negative balance. However remember, anyone with the other side of the position can settle at any time.

Funding (described above), also accrues in the unsettled PnL. This is why your balance may be different than the PnL of your position, even if no settlement has occurred.
