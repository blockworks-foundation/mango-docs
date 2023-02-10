# ❤ Health Overview

* Collateral, borrows and perp position values are summed up to give the Health metric
* Each asset is weighted differently (Read: [Asset Specs](https://docs.mango.markets/mango/token-specs#asset-weights))
* Health is used to determine liquidations and new position eligibility
* Oracle price and stable price are used to determine the value of assets and liabilities

The health of an account is used to determine if it can open a new position or if it can be liquidated. There are three types of health: **initial health** is used to check if opening new positions may be openend, **maintenance health** controls when liquidations start and **liquidation end health** determines when they end.

All three variants are calculated as a weighted sum of the assets minus the liabilities, but the weights and prices used differ. Zero is used as the bright line for all three, i.e. if your init health falls below zero, you cannot open new positions, if your maint. health falls below zero you will be liquidated and once your liq. end health increases above zero, liquidations will stop.

They are calculated as follows:

$$
health = \sum\limits_{i} a_i \cdot p_i \cdot w^a_i - l_i \cdot p_i \cdot w^l_i \newline where \newline a_i-quantity\space asset \space i \newline p_i - price \space of \space i \newline w^a_i - asset \space weight \newline w^l_i - liability \space weight \newline l_i - quantity \space liability \space i
$$

The quote currency, in the typical case USDC, will have all weights and prices equal to 1.

#### Health Ratio

The Mango user interface often displays health ratio percentages. This ratio is:&#x20;

```
health_ratio = maint_weighted_assets / maint_weighted_liabilities - 1
```

It is a useful shorthand for judging liquidation risk: Imagine an account has a health ratio of 5%. That account will be liquidatable if the prices of its liabilities go up by 5%.

Of course there are other scenarios that would cause liquidation too: The prices of the assets could fall, maybe some liabilities increase in price and some drop, etc. The final rule is still: If the maintenance health (or equivalently health ratio) fall below zero, an account is liquidatable.

#### Prices

The prices used for initial health and maintenance health differ. Maintenance and liquidation end health use the oracle price directly and initial health uses a more conservative `min(oracle_price, stable_price)` for valuing assets and `max(oracle_price, stable_price)` for valuing liabilities.

For example, imagine the SOL oracle reports $50 and the stable price is at $40. The account liquidation check (maint. health) would then use $50 as the value of SOL assets and liabilities. But for checking if new positions may be openened (init. health), SOL assets would be valued at $40 and SOL liabilities at $50.

#### Scaled Weights

Safety features may dynamically restrict opening new positions further by affecting the initial health asset and liability weights.&#x20;

For example, imagine SOL has an init\_asset\_weight of 0.9 and is configured to allow only $100M of SOL to provide margin. If the value of total SOL deposited on the platform exceeds that number, the init asset weight would be reduced for everyone. (at $200M SOL, it'd be halved: 0.45)

#### Open Orders

The health computation gets more complex when a user has open orders. It is defined to be the min over all the scenarios where orders execute or don't execute.

That means that placing a limit order that would create a new liability once executed, will immediately reduce health as if the liability was already created. And a limit order that closes a borrow doesn't improve health until it actually executes.

#### Liquidation End

Account liquidation starts when the maintenance health becomes negative. Once started, it does not stop until an account's liquidation end health is positive again.

This overshooting helps prevent accounts from alternating between liquidatable and healthy too rapidly.

#### Example - BTC-PERP

As an example, one asset might be BTC-PERP with

```
init_asset_weight = 0.9
init_liab_weight = 1.1
maint_asset_weight = 0.95
maint_liab_weight = 1.05
```

These correspond to a 10x initial leverage and 20x maintenance leverage.

Suppose a user deposits 10k USDC and goes long 10 BTC-PERP at 10k each. Then the user has 10 BTC-PERP in assets, and a net 90k in USD liabilities. The health would be:&#x20;

```
init_health =
    10000 * 1.0 * 1.0      (asset value of 10k USDC spot deposit)
    + 10.0 * 10000 * 0.9   (asset value of 10 BTC-PERP long)
    - 100000 * 1.0 * 1.0   (liability value of 100k USD for BTC-PERP purchase)
  = 0
maint_health =
    10000 * 1.0 * 1.0      (asset value of 10k USDC spot deposit)
    + 10.0 * 10000 * 0.95  (asset value of 10 BTC-PERP long)
    - 100000 * 1.0 * 1.0   (liability value of 100k USD for BTC-PERP purchase)
  = 5000
```

Suppose the BTC-PERP mark price moves to $9400, then:

`maint_health = 10 * 9400 * maint_asset_weight - 90000 = -700`

Since the maint\_health is now below zero, this account can be liquidated until init health is above zero.

A short position would have negative contract sizes and use `maint_liab_weight` instead of `maint_asset_weight.`
