# 🪙 Fees

Fees are charged for various actions within Mango. All fees are set by the DAO, with the exception of integrated programs such as Jupiter swaps and Openbook spot trading.

## Perp Market Fees
Takers are charged and makers are rebated. See the 'Perp Markets' tab on the [Stats page](https://app.mango.markets/stats) or the 'Market Details' button on the [Trade page](https://app.mango.markets/trade) for the fees of each market.

## Spot Market Fees
Takers are charged 4bps, makers are rebated 2bps, and 2bps is claimed by the DAO. Margin trading may incur additional borrowing fees (see below).

## Borrow Fees
Each asset has:
  - A **loan origination fee**, which is a one-time fee paid when the loan is initiated. 
  - **continuous interest payments** to the lender on the loan, which are ongoing payments made over the life of the loan based on the agreed-upon interest rate.
  - **continuous upkeep fees** paid to the DAO, contributing to the governance and operational expenses of the platform. 
  - **continuous collateral fees**, on select tokens, paid to the DAO are calculated based on the user's weighted liabilities in relation to their weighted assets, then multiplied by a constant collateral Annual Percentage Rate (APR). This fee is designed to reflect the cost of maintaining collateralized positions over time. The calculation formula for asset $j$ is as follows:

    $$
    collateral\_fees\_APR_{j} = \left( \frac{\sum\limits_{i} l_i \cdot p_i \cdot w^l_i}{\sum\limits_{i} a_i \cdot p_i \cdot w^a_i} \right) \cdot const\_collateral\_fees\_APR_{j}
    $$

    Where:

    - $$l_i$$ = quantity of liability \(i\)
    - $$p_i$$ = price of \(i\)
    - $$w^l_i$$ = maintenance liability weight of asset \(i\)
    - $$a_i$$ = quantity of asset \(i\)
    - $$w^a_i$$ = maintenance asset weight of asset \(i\)
    - $$const\_collateral\_fees\_APR_{j}$$ = constant collateral Annual Percentage Rate on asset j determined by the DAO,

    This formula calculates the ratio of the total weighted liabilities to the total weighted assets, then applies the collateral APR to determine the ongoing collateral fees. Collateral fees are levied on a daily basis and are applicable to only a specific subset of tokens. The gathered fees are meant to either reimburse depositors or finance proactive risk management endeavors by the DAO (although the DAO retains the discretion to allocate the fees to wherever it sees fit).


To see the fee and interest rates for each asset, select the asset on the 'Tokens' tab on the [Stats page](https://app.mango.markets/stats).

## Flash Loan Fees
Flash loans pay the loan origination fee as described above.

## Network Fees
A small network fee is paid in SOL for each transaction. This can vary depending on the priority fee set in Settings > Network and is shown by your wallet when signing the transaction.
Additionally, there is a small SOL deposit when creating a Mango account and for each spot market used. This deposit is fully refundable upon account closure.
