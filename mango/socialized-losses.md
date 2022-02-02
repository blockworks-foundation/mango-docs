# 👪 Socialized Losses

* If the insurance fund runs out, socialized losses will occur in pools with negative equity accounts

## Socialized Losses

If an account still has liabilities but no more assets for liquidators to take, the account is bankrupt and insurance fund starts paying. Socialized loss only kicks in after the insurance fund owned by Mango v3 is depleted. We hope this is a rare event.&#x20;

For losses among token borrowers (i.e. margin traders), socialized loss works the same way it works in V2. That is, the amount of the token borrow is disappears and that amount is subtracted proportionally from all depositors.

For losses among perp market participants, the losses are spread equally among all other participants in that perp market.
