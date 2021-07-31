---
description: >-
  This is intended to describe some of the key components of Mango V3. This is a
  work in progress. Any and all aspects of this overview may change at any
  point.
---

# Overview

## What?

Mango v3 is designed to make life easier for market participants. Towards that end, Mango v3 will have these features:

* Up to 16 different tokens to borrow, lend or margin trade with on Serum DEX
* Up to 16 different perp contracts with leverage up to 10x initially
* Plans to expand to 50+ markets and tokens after some anticipated Solana upgrades
* All token deposits and perp positions are used as collateral by default
* All the features of Solana, Serum and Mango v2

## Health

The health of an account is used to determine if one can open a new position or if one can be liquidated. There are two types of health, initial health used for opening new positions and maintenance health used for liquidations. They are both calculated as a weighted sum of the assets minus the liabilities but the maint. health uses slightly larger weights for assets and slightly smaller weights for the liabilities. Zero is used as the bright line for both i.e. if your init health falls below zero, you cannot open new positions and if your maint. health falls below zero you will be liquidated. They are calculated as follows:

$$
health = \sum\limits_{i} a_i \cdot p_i \cdot w^a_i - l_i \cdot p_i \cdot w^l_i \newline
where \newline a_i-quantity\space asset \space i \newline p_i - price \space of \space i
\newline w^a_i - asset \space weight
\newline w^l_i - liability \space weight
\newline l_i - quantity \space liability \space i
$$

If the health calculation is for determining liquidations, liability weight and asset weight will use the maintenance version. The quote currency, in the typical case USD, will always have all weights and prices equal to 1. 

#### Example - BTC-PERP

As an example, one asset might be BTC-PERP with 

```text
init_asset_weight = 0.9
init_liab_weight = 1.1
maint_asset_weight = 0.95
maint_liab_weight = 1.05
```

These correspond to a 10x initial leverage and 20x maintenance leverage. 

Suppose a user deposits 10k USDC and goes long 10 BTC-PERP at 10k each. Then the user has 10 BTC-PERP in assets, and a net 90k in USD liabilities. The init health would be exactly 0 and the maint health would be:   `maint_health = 10 * 10000 * 0.95 - 90000 = 5000`

Suppose the BTC-PERP mark price moves to $9400, then:

`maint_health = 10 * 9400 * 0.95 - 90000 = -700` 

Since the maint\_health is below zero, this account can be liquidated until init health is above zero.

## Liquidations

Every `MangoAccount` must have a `maint_health` above zero. If it slips below zero, liquidators can start liquidating the account until the `init_health` is above zero. The liquidator may forcibly cancel all open orders and start paying off liabilities in the account and withdrawing the assets \(defined as positive perp balances or token deposits\) in the account. The liquidator earns the `liquidation_fee` which 

If, at the end of a Liquidate instruction the account has no more assets , but still has liabilities, then the account is placed under bankruptcy. During bankruptcy, the liquidator continues offsetting liabilities, but receives USDC from the insurance fund. Finally, if there is no more USDC in the insurance fund, there will be a socialize loss event \(described below\). There are seven liquidation related instructions that can be called on Mango V3:

* ForceCancelSpotOrders
* ForceCancelPerpOrders
* LiquidateTokenAndToken
* LiquidateTokenAndPerp
* LiquidatePerpMarket
* ResolveTokenBankruptcy
* ResolvePerpBankruptcy

## Insurance Fund

The Mango program will have some portion of the Mango DAO treasury as the insurance fund. When there are bankrupt accounts, the insurance fund will pay off losses incurred by token lenders or perps contract participants. 

## Socialized Losses

If an account still has liabilities but no more assets for liquidators to take, the account is bankrupt and insurance fund starts paying. Socialized loss only kicks in after the insurance fund owned by Mango v3 is depleted. We hope this is a rare event. 

For losses among token borrowers \(i.e. margin traders\), socialized loss works the same way it works in V2. That is, the amount of the token borrow is disappears and that amount is subtracted proportionally from all depositors.

For losses among perp market participants, the losses are spread equally among all other participants in that perp market.

## Settle Pnl

In order to get USDC realized profits from unrealized profits on a perp contract, the trader must settle his profits by finding account\(s\) that have equivalent losses on the same contract and call the `SettlePnl` instruction on chain.

## Funding

Funding is computed continuously as a daily difference in the index price and current book price \(current mid\). The `UpdateFunding` instruction is called by the Keeper roughly every 5 seconds and so funding is paid out roughly every 5 seconds. The calculation is as follows:

```text
book_price = (bid + ask) / 2
diff = clamp(book_price / index_price - 1, -0.05, 0.05)
time_factor = (now - last_updated) / TIME_IN_DAY
funding = diff * time_factor * contract_size * index_price

```

Note: we plan to transition to an hourly funding rate similar to FTX before launching on mainnet. 

