---
description: >-
  This is a work in progress. More to be added as information about tokens is
  added
---

# ☑ Asset Specs

### Spot Market Leverage

Note: The asset\_weight applies a haircut to the value of the collateral in health calculations. The lower the asset weight, the less the asset counts towards collateral. Initial Leverage and Maintenance Leverage can be converted to the corresponding asset\_weights with these calculations:

```
init_asset_weight = 1 - 1 / init_leverage
init_liab_weight = 1 + 1 / init_leverage
maint_asset_Weight = 1 - 1 / maint_leverage
maint_liab_weight = 1 + 1 / maint_leverage
```

| Token | Initial Leverage | Maintenance Leverage | Liquidation Fee (%) |
| ----- | ---------------- | -------------------- | ------------------- |
| MNGO  | 1.25             | 2.5                  | 20.0                |
| BTC   | 5                | 10                   | 5.00                |
| ETH   | 5                | 10                   | 5.00                |
| SOL   | 5                | 10                   | 5.00                |
| mSOL  | 5                | 10                   | 5.00                |
| USDT  | 10               | 20                   | 2.50                |
| SRM   | 5                | 10                   | 5.00                |
| RAY   | 5                | 10                   | 5.00                |
| COPE  | 2                | 4                    | 12.50               |
| FTT   | 5                | 10                   | 5.00                |

### Asset Weights

The init asset weight (aka haircut) is the discount applied to the value of the collateral when initializing a new position. For example, for BTC, $100 in BTC is treated as 80 USDC when initializing a new position. The maint asset weight is the discount applied when calculating the value of the collateral for purposes of liquidation. The maint asset weight is higher to give you more of a buffer before liquidation.

| Token | Init Asset Weight | Maint Asset Weight |
| ----- | ----------------- | ------------------ |
| MNGO  | 0.2               | 0.6                |
| BTC   | 0.8               | 0.9                |
| ETH   | 0.8               | 0.9                |
| SOL   | 0.8               | 0.9                |
| mSOL  | 0.8               | 0.9                |
| USDT  | 0.9               | 0.95               |
| SRM   | 0.8               | 0.9                |
| RAY   | 0.8               | 0.9                |
| COPE  | 0.5               | 0.75               |
| FTT   | 0.8               | 0.9                |
| USDC  | 1                 | 1                  |

### Asset Interest Rates

The lending pools work similar to the lending pools on Aave. With the big difference that users will earn interest on both their deposits as well as their positions (so you may be earning net interest on your margin position!). The interest rate is a function of the utilization ratio: total borrowed by all users divided by total deposits of all users. The interest rate will increase slowly approaching the utilization ratio but will increase rapidly above that to encourage the market to deposit more funds.

| Asset | Optimal Rate | Max Rate | Optimal Utilization |
| ----- | ------------ | -------- | ------------------- |
| USDC  | 8%           | 150%     | 70%                 |
| BTC   | 3.5%         | 87.5%    | 70%                 |
| ETH   | 2.5%         | 62.5%    | 70%                 |
| SOL   | 2.5%         | 62.5%    | 70%                 |
| mSOL  | 2.5%         | 62.5%    | 70%                 |
| SRM   | 10%          | 250%     | 70%                 |
| MNGO  | 6%           | 150%     | 70%                 |
| USDT  | 5%           | 125%     | 70%                 |
| RAY   | 10%          | 250%     | 70%                 |
| COPE  | 6%           | 150%     | 70%                 |

![Utilization rate can vary by asset.](../.gitbook/assets/untitled.png)
