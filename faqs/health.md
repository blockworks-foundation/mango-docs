# Health

In v3, the health of an account is used to determine if an account can open a new position or be liquidated. \
\
There are two types of health:  **initial health**, used for opening new positions **maintenance health,** used for liquidations   \
\
They are both calculated as a weighted sum of the assets minus the liabilities, but Maintenance health uses slightly larger weights for assets and slightly smaller weights for the liabilities.\
\
Zero is the threshold for both healths. If your Initial health is below falls below zero, you cannot open new positions and if your Maintenance health falls below zero you will be liquidated.

Health Calculation:&#x20;

$$
health = \sum\limits_{i} a_i \cdot p_i \cdot w^a_i - l_i \cdot p_i \cdot w^l_i \newline
where \newline a_i-quantity\space asset \space i \newline p_i - price \space of \space i
\newline w^a_i - asset \space weight
\newline w^l_i - liability \space weight
\newline l_i - quantity \space liability \space i
$$

The Health Ratio which is shown in the trading page, is health rearranged: The sum of all Assets weighted divided by sum of all Liabs weighted - 1\
\
The asset\_weight applies a haircut to the value of the collateral in health calculations. The lower the asset weight, the less the asset counts towards collateral. Initial Leverage and Maintenance Leverage can be converted to the corresponding asset\_weights with these calculations:

```
init_asset_weight = 0.9
init_liab_weight = 1.1
maint_asset_weight = 0.95
maint_liab_weight = 1.05
```

These values by asset are:\


| Token                                        | Maint. Asset Weight | Maint. Liab. Weight | Init. Asset Weight | Init. Liab. Weight | Init. Leverage | Maint. Leverage |
| -------------------------------------------- | ------------------- | ------------------- | ------------------ | ------------------ | -------------- | --------------- |
| <mark style="color:orange;">**SPOT**</mark>  |                     |                     |                    |                    |                |                 |
| USDC                                         | 1                   |                     |                    |                    |                |                 |
| USDT                                         | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| BTC                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| ETH                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| SOL                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| mSOL                                         | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| SRM                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| RAY                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| FTT                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| AVAX                                         | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| BNB                                          | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| LUNA                                         | 0.9                 | 1.1                 | 0.8                | 1.2                | 5              | 10              |
| COPE                                         | 0.75                | 1.25                | 0.5                | 1.5                | 2              | 4               |
| MNGO                                         | 0.6                 | 1.4                 | 0.2                | 1.8                | 1.25           | 2.5             |
| <mark style="color:orange;">**PERP**</mark>  |                     |                     |                    |                    |                |                 |
| BTC                                          | 0.975               | 1.025               | 0.95               | 1.05               | 20             | 40              |
| SOL                                          | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| FTT                                          | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| ETH                                          | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| ADA                                          | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| AVAX                                         | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| BNB                                          | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| LUNA                                         | 0.95                | 1.05                | 0.9                | 1.1                | 10             | 20              |
| SRM                                          | 0.9375              | 1.0625              | 0.875              | 1.125              | 8              | 16              |
| RAY                                          | 0.9375              | 1.0625              | 0.875              | 1.125              | 8              | 16              |
| MNGO                                         | 0.875               | 1.125               | 0.75               | 1.25               | 4              | 8               |

\
For instance, the Maint. Asset Weight is used when calculating health to determine if the position is eligible for liquidation (edited)

\




