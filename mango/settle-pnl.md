# 🤝 PnL Settlement

* Settling Pnl moves a profit or loss from a perp market into the USDC token balance.
* This has no impact on your open positions or health.
* Warning: this can take up to 60 seconds to complete when using the feature on the UI.

#### Types of PnL

Accounts that interact with perp markets normally have a certain amount of _unsettled pnl_. Unsettled pnl represents a valuation change in the perp position that hasn't been settled into a token position yet.

Separately from the question of settlement, pnl can also be categorized as _realized_ or _unrealized_. Pnl is realized when a position is reduced at a price that's different from its average entry price. Realized pnl can be settled or unsettled.

Example:

* you buy 2 BTC-PERP at $20k: initially there's zero unsettled pnl
* the BTC price increases to $25k: now there is $10k of unsettled pnl (which is also _unrealized_, because the position isn't closed)
* you sell 1 BTC-PERP at $24k, price is still at $25k: you now have $9k of unsettled pnl ($4k realized from the sale and $5k unrealized from the remaining position)
* you settle $7k of the pnl: now you have $2k of unsettled pnl left (and your USDC token balance has increased by $7k)

#### PnL Settlement Action

The act of pnl settlement takes two perp market participants with unsettled pnl of opposite sign and reduces it equally on both sides by transfering a tokens between the accounts.

Example: Account A has +$10k unsettled pnl and account B has -$4k unsettled pnl. Settling between the two would transfer $4k token from B to A and leave the unsettled pnl at A: $6k and B: $0.

In this way pnl settlement converts potential profit or loss from the perp market into actual token profit or loss.

#### Automatic PnL Settlement

Pnl settlement is a permissionless action: anyone could settle two account's pnls at any time.&#x20;

Mango's risk engine considers positive unsettled pnl as more risky than USDC token balances, meaning that settling the pnl will improve the accounts health.&#x20;

To avoid long-lived large unsettled positions, Mango currently incentivizes the caller of a settle transaction in some scenarios:

* if at least $100 are settled, the caller receives $0.0001
* if an account is close to liquidation, the caller receives up to 1% of the settled value

These fees are paid by the account that had positive unsettled pnl.
