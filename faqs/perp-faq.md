# PERP FAQ

### **How much leverage can I take?**

Perpetuals allow initial leverage of up to 10x, and maintenance leverage of 20x. If the value of your position (or collateral) falls and your leverage exceeds 20x, your account will be eligible for liquidation.

### What fees will I pay?

Mango currently chargers ‘takers’ on the platform a 5bps fee. There is no fee for providing a limit order into the book that is later executed as a ‘maker’. **** Mango does not take a fee for borrowing or lending.

### What is funding, and how does it work?

In a perpetual future, funding is the mechanism that ensures the future stays in line with current spot prices. The midpoint of the bids & offers are compared to the oracle price to determine the funding rate. When the future market is trading above the oracle price, long’s will have to pay shorts. When below the oracle price, shorts will pay longs. This rate is continually calculated and paid, and is shown above the market as an hourly rate.

For example, assume SOL-PERP’s current market is trading 100.45x100.55, also assume that the oracle says SOL’s ‘true price’ is $100.00. Because the current perp market is trading above oracle, holders who are long will pay holders who are short. This provides an incentive for market makers to sell the perpetual future, and for holders that are long to close their position and bring the market back into line with oracle.

How much do you pay? A simple rule of thumb is that each 24hours, the amount paid is the difference between the trading market and the oracle. Taking our previous example, if the difference averaged $0.50 over an entire day, holders who are long will pay shorts $0.50 per contract they held. Expressed as a 1 hour rate, this would show as ($0.50/$100=$0.005/24=0.00020) “%0.020 per hour”

### What is the PnL on my position?

The PnL is the profit or loss on your position. It will not include funding & fees. For instance, if you enter 1 SOL-PERP Long @ 150.00, and it is currently trading @ 200.00, your PnL will be $50.00 USDC.

### What is my unsettled PnL?

In perpetual futures markets Unsettled PnL accrues as the oracle changes. At any time any participant can “settle” their PnL which will find a person with the opposite PnL and settle the balances to USDC. This will not affect your position, but simply credit/debit your account with the balance that you are owed/owe.

Note that while you may settle any-time you wish, you do not have control to stop a settlement from happening.

For example, assume you buy 1 SOL-PERP long at $100.00, using 1 BTC as collateral. Now assume the oracle drops to $80.00. If no settlements have happened yet on your account, you would show an unsettled balance of -$20.00. If you settle your balance, you will still have your 1 BTC, and your sol-perp position, but it will now borrow $20.00 USDC. Because borrowing costs you money (long-term APR), you should prefer not to settle a negative balance. However remember, anyone with the other side of the position can settle at any time.

Funding (described above), also accrues in the unsettled PnL. This is why your balance may be different than the PnL of your position, even if no settlement has occurred.
