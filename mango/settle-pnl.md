# 🤝 Settle Pnl

* Settling Pnl moves a profit or loss from a perp market into the USDC token balance.
* This has no impact on your open positions or health.
* Warning: this can take up to 60 seconds to complete when using the feature on the UI.

Accounts that interact with perp markets normally have a certain amount of _unsettled pnl_. For example:

* you buy 1 BTC-PERP at $20k: initially there's zero unsettled pnl
* the BTC price increases to $30k: now there is $10k of unsettled pnl (which is also _unrealized_, because the position isn't closed)
* you sell 1 BTC-PERP at $28k, price is still at $30k: you still have $10k of unsettled pnl (but now it's $8k realized and $2k unrealized)
* you settle $6k of the pnl: now you have $4k of unsettled pnl left (and your USDC token balance has increased by $6k)

Unsettled pnl represents a valuation change in the perp position that hasn't been settled into a token position yet.

The act of pnl settlement takes two perp market participants with unsettled pnl of opposite sign and reduces it equally on both sides by transfering a tokens between the accounts.

Example: Account A has +$10k unsettled pnl and account B has -$4k unsettled pnl. Settling between the two would transfer $4k token from B to A and leave the unsettled pnl at A: $6k and B: $0.

In this way pnl settlement converts potential profit or loss from the perp market into actual token profit or loss.
