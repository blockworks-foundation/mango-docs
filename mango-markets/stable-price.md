# ⚖ Stable Price

The **stable price** is used in a safety mechanism that limits a user's ability to enter risky positions when the oracle price is changing rapidly.

#### Usage

When determining if a user can open a new position, Mango computes the initial health (see [health-overview.md](health-overview.md "mention")). For this, it adds together the value of all assets and subtracts the value of all liabilities. To derive the value of assets and liabilities, prices are needed. The price of an asset is set to be the lesser of the oracle and stable price, while the value of liabilities is set to the larger of the oracle and stable price.

#### Effects

The idea is that in normal operation, the stable price stays very close to the oracle price. For large jumps in price, it'll temporarily reduce user leverage until it has caught up. And for extreme jumps - likely due to attacks - it'll be very slow in granting leverage based on the new values. That means attackers need to keep up their attack for longer and users and operators have more time to react.

For example, in its current default configuration (stable\_growth\_limit=290% per hour, delay\_growth\_limit=6% per hour) for an upwards price jump of

* 5% it catches up in in three minutes
* 20% it catches up in 13 minutes
* 100% it catches up to 178% of the starting price in one hour
* 900% it catches up to 178% in one hour and 727% in one day

#### Definition

The stable price is updated based on the oracle price. It's updated by a permissionless keeper instruction (TokenUpdateIndexAndRate) in arbitrary intervals, but not more often than once every ten seconds.

The stable price is updated in a way that makes it attempt to track the oracle price, while limiting the allowed rate of change. This allowed rate of change depends on the difference between the current stable price and the 24h-delayed oracle price. It's largest if they are the same, and decreased otherwise.

For the full detail, check `stable_price.rs` in the source code. The size of the update is roughly

```
stable_price * stable_growth_limit * dt * (delay_price / stable_price)^2
```

into the direction of the oracle price (if stable\_price > delay\_price, otherwise the quotient is inverted).

