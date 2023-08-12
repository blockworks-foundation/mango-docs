# Lending

**How can I earn interest?**\
To earn interest, simply deposit into your Mango Markets Account; all assets automatically earn interest. Current and average rates are displayed in stats page.\
\
**Can I track how much interest I've earned?**\
Yes, in the "Account" section there is an "Interest" tab that displays total and day-by-day interest earned or paid for all assets.\
\
**Are there lending fees?**\
No.\
\
**Is there a lockup period?**\
\*\*\*\*No, you can withdraw anytime.

#### Lenders, Borrowers and Liquidators&#x20;

The lending pools work similar to the lending pools on Aave. With the big difference that users will earn interest on both their deposits as well as their positions (so you may be earning net interest on your margin position). The interest rate is a function of the utilisation ratio: total borrowed by all users divided by total deposits of all users. The interest rate will increase slowly approaching a utilisation ratio of 70% but will increase rapidly above that to encourage the market to deposit more funds.

For the calculation, have a look at [https://github.com/blockworks-foundation/mango-v3/blob/main/program/src/utils.rs#L138](https://github.com/blockworks-foundation/mango-v3/blob/main/program/src/utils.rs#L138)
