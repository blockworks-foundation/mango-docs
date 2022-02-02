# Overview

**Mango offers an approachable and powerful platform for trading, lending and portfolio management.**&#x20;

Mango not only offers spot margined and leveraged perpetual futures markets but also allows collateralized loans against existing assets all under one unified experience.&#x20;

Trade spot margined markets with up to 5x leverage and perpetual futures markets with up to 10x leverage. Long and short everything, with limit or market orders, more advanced order types coming soon.&#x20;

Current on-chain lending protocols (e.g. Aave, Compound) provide up to 2x leverage for margin traders, but this comes with severe downsides. For one, Ethereum gas prices are prohibitive for the majority of traders. Secondly, the margin trading can only be done with market orders or swaps — not limit orders. This makes them completely useless for market makers.

Centralized trading platforms are able to provide leverage similar to Mango Markets, but they have opaque rules and ultimately take custody of traders funds. This creates extra security risks and the trader is ultimately at the discretion of the exchange on availability of his funds. We believe finance needs a fully decentralized, on-chain and open source alternative and Mango is our attempt to create one.

#### Collateral Requirements

The collateral ratio is the value of the user's deposits and positions divided by the value of their loans. For the initial set of markets, Mango will require an initial collateral ratio of 120%, and a maintenance collateral ratio of 110%. If the user's collateral ratio drops below 110%, the account will be liquidated and the user will lose the entire account. The value of the account will be calculated using a moving average of centralized exchange price feeds, provided by a decentralized oracle.

![](../.gitbook/assets/collat.png)

#### Liquidation Process

Accounts must maintain a minimum 110% collateral ratio. If an account falls below the 110% threshold, a liquidator absorbs your position and becomes the new account owner.  Anyone can run a liquidator with the opportunity to make a profit; we will open source our liquidator bot soon, but we encourage others to build their own versions. You can find more details [here](../development-resources/liquidator.md).

#### Lenders, Borrowers and Liquidators (oh my!)

The lending pools work similar to the lending pools on Aave. With the big difference that users will earn interest on both their deposits as well as their positions (so you may be earning net interest on your margin position!). The interest rate is a function of the utilization ratio: total borrowed by all users divided by total deposits of all users. The interest rate will increase slowly approaching a utilization ratio of 70% but will increase rapidly above that to encourage the market to deposit more funds.

![](../.gitbook/assets/chart\_1.png)

