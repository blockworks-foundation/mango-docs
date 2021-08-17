---
description: >-
  This is a work in progress. More to be added as information about tokens is
  added
---

# Token Specs

Note: The asset\_weight applies a haircut to the value of the collateral in health calculations.  The lower the asset weight, the less the asset counts towards collateral. Initial Leverage and Maintenance Leverage can be converted to the corresponding asset\_weights with these calculations:

```text
init_asset_weight = 1 - 1 / init_leverage
init_liab_weight = 1 + 1 / init_leverage
maint_asset_Weight = 1 - 1 / maint_leverage
maint_liab_weight = 1 + 1 / maint_leverage
```

| Token | Initial Leverage | Maintenance Leverage | Liquidation Fee \(%\) |
| :--- | :--- | :--- | :--- |
| BTC | 5 | 10 | 5.00 |
| ETH | 5 | 10 | 5.00 |
| SOL | 5 | 10 | 5.00 |
| SRM | 5 | 10 | 5.00 |
| MNGO | 1.25 | 2.5 | 20.0 |
| RAY | 3 | 6 | 8.33 |

