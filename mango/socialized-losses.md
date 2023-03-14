# 👪 Socialised Losses

* If the insurance fund runs out, socialised losses will occur in pools with negative equity accounts.

## Socialised Losses

If an account still has liabilities but no more assets for liquidators to take, the account is bankrupt and the insurance fund starts paying. Socialised loss only kicks in after the insurance fund owned by the program is depleted. We hope this is a rare event.

For losses among token borrowers (i.e. margin traders), the amount of the token borrow for the bankrupt account is subtracted proportionally from all depositors of that token.

A loss among perp market participants means having unsettled negatve pnl on a bankrupt account. Socializing this loss means spreading it proportionally among users with an active perp position in the market.

Users who are not involved with the asset that the socialized loss event occurs in (have no deposits of that token or have no position in that perp market) are not directly affected by the socialized loss.
