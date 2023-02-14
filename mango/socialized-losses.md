# 👪 Socialised Losses

* If the insurance fund runs out, socialised losses will occur in pools with negative equity accounts.

## Socialised Losses

If an account still has liabilities but no more assets for liquidators to take, the account is bankrupt and the insurance fund starts paying. Socialised loss only kicks in after the insurance fund owned by the program is depleted. We hope this is a rare event.

For losses among token borrowers (i.e. margin traders), socialised loss works the same way that it worked in V2 & V3. That is, the amount of the token borrow for the bankrupt account is subtracted proportionally from all depositors.

For losses among perp market participants, the losses are spread equally among all other participants in that perp market.
