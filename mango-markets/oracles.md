# 🔮 Oracles

Mango uses third-party oracles to price assets for use in the smart contract. The oracle price is used for PnL calculations and liquidations, among other things.
Oracles must be updated frequently to ensure accurate pricing. As a safety measure, Mango will restrict actions on tokens where the oracle has not updated for some time.

Mango currently uses oracles provided by [Pyth](https://pyth.network) and [Switchboard](https://switchboard.xyz).

You can check the oracle provider and price feed address for each asset on the 'Details' tab of the [Token Stats page](https://app.mango.markets/stats) or by hovering over the Oracle Price heading on the [Trade page](https://app.mango.markets/trade).
