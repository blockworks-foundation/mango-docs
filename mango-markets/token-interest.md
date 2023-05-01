# 💱 Token Interest

Token interest rates adjust dynamically in response to borrow utilization (token borrows / token deposits).

Each token is governed by it's own set of interest parameters that determine it's responsiveness to different ranges of utilization. This is necessary as tokens differ in risk, inflation, etc. 

The interest rate calculation is as follows:
```
if utilization <= util0 {
    let slope = rate0 / util0;
    let interest_rate = slope * utilization
} else if utilization <= util1 {
    let extra_util = utilization - util0;
    let slope = (rate1 - rate0) / (util1 - util0);
    let interest_rate = rate0 + slope * extra_util
} else {
    let extra_util = utilization - util1;
    let slope = (max_rate - rate1) / (1 - util1);
    let interest_rate = rate1 + slope * extra_util
}
```

This curve is segmented into three linear sections according to utilization levels (util0 and util1). 
The piecewise linear curve can be visualized, using the current (as of the time of this writing) SOL token parameters. This curve depicts the relationship between utilization and interest rates.

![](<../.gitbook/assets/sol rate curve.png>)

Furthermore, an hourly rate adjustment factor is applied, which depends on the gap between the actual and optimal utilization (util0). The rate parameters (rate0, rate1, max_rate) are multiplied by this adjustment factor. Another curve (again using current SOL parameters), can illustrate this rate adjustment process.

![](<../.gitbook/assets/sol rate adjustment curve.png>)