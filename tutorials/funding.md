# 💸 Funding

Funding is the mechanism used to ensure that the price of a futures contract stays in line with the current spot price.&#x20;

The midprice of the bids and asks is compared to the oracle price to determine the funding rate.&#x20;

When the contract is trading above the oracle price, longs will have to pay shorts and when below the oracle price, shorts will pay longs.&#x20;

This rate is continually calculated and paid, and is shown above the market as an hourly rate.

Funding is paid continuously and is automatically added to the user's unsettled PNL.

For example, assume SOL-PERP’s current market is trading 100.45x100.55, also assume that the oracle's current ‘true price’ for SOL is $100.00. Because the current perp market is trading above oracle, holders who are long will pay holders who are short. This provides an incentive for market makers to sell the perpetual contract, and for holders that are long to close their position and bring the market back in line with the oracle price.

To see past funding earned, visit the Account page in the UI.

### How to calculate the funding rate

Every 24 hours, the amount paid or received is the difference between the trading market and the oracle. Taking our previous example, if the difference averaged $0.50 over an entire day, holders who are long will pay shorts $0.50 per contract they hold. Expressed as a 1-hour rate, this would show as ($0.50/$100=$0.005/24=0.0002), “%0.02 per hour.”

For an example of how to calculate the funding rate programmatically, have a look here.
