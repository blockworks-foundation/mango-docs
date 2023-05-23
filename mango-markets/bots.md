# 🤖 Bots

There are several off-chain bots that call permissionless Mango V4 instructions to keep the exchange operating normally.

### Keeper

The Keeper performs three main functions:

#### Updating Token Interest

This function updates the deposit/borrow indices on each bank in order to pay out interest to lenders based on time since the last update, and update the rates shown in the UI. Additionally, this function updates the stable price for each token.

#### Updating Perp Funding

This function update the instantaneous funding rate based on the difference between mark and oracle prices. Additionally, this function updates the stable price for each perp market.

#### Consuming Perp Events

This function performs updates to the state of a Mango account after its order is removed from the book, either by being filled, cancelled or evicted. This process is asynchronous since the taker may not know the account of the maker at the time the taker order is sent.



Execution of these instructions is not directly incentivised, although exchange users have some incentive to run a Keeper to ensure normal operations. Core contributors are running multiple redundant Keeper bots, with gas fees provided by the DAO.

Keeper Source Code: [https://github.com/blockworks-foundation/mango-v4/tree/dev/bin/keeper](https://github.com/blockworks-foundation/mango-v4/tree/dev/bin/keeper)

### Openbook Crank

Although it doesn't operate directly on the Mango V4 contract, it is essential that events on the spot markets are consumed for the same reasons as events on the perp market.

This role is not directly incentivised, although market participants have some incentive to call these functions so they can continue to trade the market and settle their funds from maker orders.

There are several Openbook Crank implementations (code not reviewed or endorsed by Mango):

* [https://github.com/SpaceMonkeyForever/openbook-cranker](https://github.com/SpaceMonkeyForever/openbook-cranker)
* [https://github.com/openbook-dex/program/tree/master/dex/crank](https://github.com/openbook-dex/program/tree/master/dex/crank)
* [https://github.com/sol-farm/serum-crank](https://github.com/sol-farm/serum-crank)

### Settler

The Settler will automatically settle perp PnL above a certain threshold in return for a small fee. The threshold and fee set by the DAO can vary per-market, and can be viewed on the [dashboard](https://app.mango.markets/dashboard).

Anyone can run a Settler bot, although the DAO has funded one which is run by core contributors. Any profits are used to purchase SOL to fund future transactions.

Settler Source Code: [https://github.com/blockworks-foundation/mango-v4/tree/dev/bin/settler/src](https://github.com/blockworks-foundation/mango-v4/tree/dev/bin/settler/src)

### Liquidator

The Liquidator acquires the position of accounts that are in danger of entering a negative equity state at a discount, and closes it. The discount received by the Liquidator is used as an incentive to take on risk temporarily, and is set by the DAO relative to the expected liquidity and volatility of a token or perp market.&#x20;

Liquidation parameters for each asset/market can be viewed on the [dashboard](https://app.mango.markets/dashboard). You can find more information about the liquidation process [here](../mango/liquidations.md).

Liquidator Source Code: [https://github.com/blockworks-foundation/mango-v4/tree/dev/bin/liquidator](https://github.com/blockworks-foundation/mango-v4/tree/dev/bin/liquidator)
