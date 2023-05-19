# 〽 Perp Markets

Mango offers perp markets, which are markets for perpetual futures contracts that allow users to go long or short on the future of an underlying index price.

* Perp positions pay or receive **funding payments** continuously based on current perp order book and perp index price. Check funding rates before entering positions.
* Gains and losses from perp positions materialize in the settle token (USDC) though **settlement**.
* Perp positions have **unsettled pnl** depending on the difference between the current index price and the index price at the time of the last settlement.

### Usage

Users can enter perp positions by placing bids and asks on the perp order books. When the orderbook matches a buyer and seller, two opposite-sign positions are created: the buyer receives an amount of LONG contracts and the seller receives the same amount of SHORT contracts and their entry price is recorded. There is no payment in either direction at this point.&#x20;

If they had existing contracts, then LONG and SHORT contracts cancel each other out: Buying 3 LONG when the account had 5 SHORT will leave it with 2 SHORT.

Holding a LONG contract means that the user is obligated to receive 1 USDC (settle token) from a SHORT contract holder when the index price increases by $1. If the index price decreases by $1, the LONG will have to pay 1 USDC to a SHORT holder. This process of paying USDC for changes in index price is called **settlement**.

To ensure the perp contracts are traded close to the index price, contract holders also pay or receive **funding payments** depending on how far the orderbook midpoint is from the index price (see below for details).

### Settlement

Settlement is the act of exchanging settle token (USDC) due to changes in index price since position creation or the last settlement.

In Mango, settlement is permissionless. Anyone is allowed to match up two accounts that hold perp contracts of opposite sign and cause a settlement. During settlement, USDC will be transfered from the account with negative unsettled pnl to the account with positive unsettled pnl.

Settlement is the only mechanism to convert perp market pnl into USDC token pnl. For example, if you LONG at $100 and sell it at $120 (and no one settles you along the way), your USDC balance will still be unchanged. You have 20 realized unsettled pnl that you can settle to receive 20 USDC though.

### Funding

Funding is the mechanism that keeps a perp market trading close to its index price. It does this by comparing the orderbook state to the index price and setting a funding rate based on the difference. If the funding rate is positive LONG contract holders will pay SHORTs and if it's negative SHOTs pay LONGs.

Funding payments change the unsettled pnl on accounts with perp positions.

Funding rates are expressed in percent per day and can be large when the market and index price disagree.

### USD vs USDC

Even though the perp markets use a price quoted in USD as index, pnl accrues and is settled in USDC. That means, for example, that a SOL-PERP short only hedges a positive SOL spot position as long as 1 USDC = $1.
